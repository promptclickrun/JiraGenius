---
name: pagination
description: "This skill covers pagination strategies for the Jira Cloud REST API, including offset-based and cursor-based pagination."
---

# Pagination Skill

This skill covers pagination strategies for the Jira Cloud REST API, including offset-based and cursor-based pagination.

## Offset-Based Pagination

Most Jira Cloud REST API endpoints use offset-based pagination with `startAt` and `maxResults` parameters.

### Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `startAt` | The index of the first item to return (0-based) | `0` |
| `maxResults` | The maximum number of items to return per page | Endpoint-specific (usually 50) |

### Response Fields

| Field | Description |
|-------|-------------|
| `startAt` | The index of the first item returned |
| `maxResults` | The maximum number of items that could be returned |
| `total` | The total number of items matching the query |
| `values` / `issues` | Array of items for the current page |

### Example: Paginate Through All Issues

```bash
# First page
curl -X POST \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/search" \
  -d '{
    "jql": "project = PROJ ORDER BY created DESC",
    "startAt": 0,
    "maxResults": 100,
    "fields": ["summary", "status", "assignee"]
  }'

# Response:
# { "startAt": 0, "maxResults": 100, "total": 350, "issues": [...] }

# Second page
curl -X POST \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/search" \
  -d '{
    "jql": "project = PROJ ORDER BY created DESC",
    "startAt": 100,
    "maxResults": 100,
    "fields": ["summary", "status", "assignee"]
  }'

# Continue until startAt + maxResults >= total
```

### Pagination Loop Pattern

```python
import requests
import math

base_url = "https://your-domain.atlassian.net"
auth = ("email@example.com", "api-token")

start_at = 0
max_results = 100
all_issues = []

while True:
    response = requests.post(
        f"{base_url}/rest/api/3/search",
        auth=auth,
        json={
            "jql": "project = PROJ ORDER BY created DESC",
            "startAt": start_at,
            "maxResults": max_results,
            "fields": ["summary", "status"]
        }
    )
    data = response.json()
    all_issues.extend(data["issues"])

    if start_at + max_results >= data["total"]:
        break
    start_at += max_results

print(f"Fetched {len(all_issues)} of {data['total']} issues")
```

### Bash Pagination

```bash
#!/bin/bash
START_AT=0
MAX_RESULTS=100
TOTAL=1  # Will be updated after first request

while [ $START_AT -lt $TOTAL ]; do
  RESPONSE=$(curl -s -X POST \
    -H "Authorization: Basic $(echo -n "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" | base64)" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/search" \
    -d "{
      \"jql\": \"project = PROJ\",
      \"startAt\": $START_AT,
      \"maxResults\": $MAX_RESULTS,
      \"fields\": [\"summary\", \"status\"]
    }")

  TOTAL=$(echo "$RESPONSE" | jq '.total')
  COUNT=$(echo "$RESPONSE" | jq '.issues | length')
  echo "Fetched $COUNT issues (startAt: $START_AT, total: $TOTAL)"

  # Process issues here
  echo "$RESPONSE" | jq '.issues[] | {key: .key, summary: .fields.summary}'

  START_AT=$((START_AT + MAX_RESULTS))
done
```

## Server-Enforced Limits

Different endpoints have different maximum page sizes:

| Endpoint | Max `maxResults` |
|----------|-----------------|
| `/rest/api/3/search` | 100 |
| `/rest/api/3/user/search` | 1000 |
| `/rest/api/3/group/member` | 50 |
| `/rest/api/3/project/search` | 100 |
| `/rest/agile/1.0/board` | 100 |
| `/rest/agile/1.0/sprint/{id}/issue` | 100 |

If you request more than the maximum, the server silently reduces `maxResults` to its limit.

## Cursor-Based Pagination

Some newer endpoints use cursor-based pagination with `cursor` or `after` parameters instead of `startAt`.

### Example: Paginate with Cursor

```bash
# First page
curl -X GET \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  "https://{domain}.atlassian.net/rest/api/3/field/search?maxResults=50"

# Response includes a 'nextPage' URL or cursor
# { "values": [...], "isLast": false, "nextPage": "https://...?cursor=abc123" }

# Follow the nextPage URL for subsequent pages
```

## Performance Best Practices

1. **Request only needed fields** — Use the `fields` parameter to limit returned data
2. **Use reasonable page sizes** — 50-100 is optimal; larger sizes increase response time
3. **Avoid deep pagination** — For large datasets, use JQL with date filters instead of paginating to page 1000+
4. **Cache total counts** — The `total` field may be expensive to compute; avoid re-requesting it if unchanged
5. **Parallelize cautiously** — Concurrent pagination requests may hit rate limits
6. **Use POST for search** — GET requests with long JQL may exceed URL length limits
7. **Add ORDER BY** — Without explicit ordering, results may shift between pages if data changes

## Handling Data Changes During Pagination

Data may change between paginated requests:
- New items may be added, shifting indices
- Items may be deleted, causing gaps
- Items may be updated, changing sort order

Mitigations:
- Use `ORDER BY key ASC` for stable ordering by immutable field
- Use `created >= "date"` filters for time-bounded pagination
- Accept that pagination is eventually consistent, not a point-in-time snapshot
