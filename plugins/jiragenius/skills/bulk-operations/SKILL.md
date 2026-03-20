---
name: bulk-operations
description: "This skill covers batch create, update, delete, and transition operations for the Jira Cloud REST API."
---

# Bulk Operations Skill

This skill covers batch create, update, delete, and transition operations for the Jira Cloud REST API.

## Bulk Create Issues

```
POST /rest/api/3/issue/bulk
```

Create up to 50 issues in a single request:

```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/issue/bulk" \
  -d '{
    "issueUpdates": [
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Task" },
          "summary": "Task 1",
          "description": {
            "type": "doc",
            "version": 1,
            "content": [
              {
                "type": "paragraph",
                "content": [
                  { "type": "text", "text": "Description for task 1" }
                ]
              }
            ]
          }
        }
      },
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Task" },
          "summary": "Task 2"
        }
      },
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Bug" },
          "summary": "Bug 1",
          "priority": { "name": "High" }
        }
      }
    ]
  }'
```

**Response:**
```json
{
  "issues": [
    { "id": "10001", "key": "PROJ-101", "self": "https://..." },
    { "id": "10002", "key": "PROJ-102", "self": "https://..." },
    { "id": "10003", "key": "PROJ-103", "self": "https://..." }
  ],
  "errors": []
}
```

Partial failures are possible — check the `errors` array.

## Bulk Update via JQL + Script

Jira Cloud does not have a single "bulk update" endpoint. Use a loop pattern:

### Bash Script for Bulk Update

```bash
#!/bin/bash
# Bulk update issues matching a JQL query

JQL="project = PROJ AND status = 'Open' AND labels = 'needs-triage'"
START_AT=0
MAX_RESULTS=50
TOTAL=1

while [ $START_AT -lt $TOTAL ]; do
  RESPONSE=$(curl -s -X POST \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/search/jql" \
    -d "{
      \"jql\": \"$JQL\",
      \"startAt\": $START_AT,
      \"maxResults\": $MAX_RESULTS,
      \"fields\": [\"key\"]
    }")

  TOTAL=$(echo "$RESPONSE" | jq '.total')
  KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')

  for KEY in $KEYS; do
    echo "Updating $KEY..."
    curl -s -X PUT \
      -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
      -H "Content-Type: application/json" \
      "$JIRA_BASE_URL/rest/api/3/issue/$KEY" \
      -d '{
        "fields": {
          "labels": ["triaged"],
          "priority": { "name": "Medium" }
        }
      }'
    # Rate limiting: wait between requests
    sleep 0.2
  done

  START_AT=$((START_AT + MAX_RESULTS))
done

echo "Bulk update complete."
```

### Python Script for Bulk Update

```python
import requests
import time

base_url = "https://your-domain.atlassian.net"
auth = ("email@example.com", "api-token")
jql = 'project = PROJ AND status = "Open"'

start_at = 0
max_results = 50
updated = 0

while True:
    response = requests.post(
        f"{base_url}/rest/api/3/search/jql",
        auth=auth,
        json={
            "jql": jql,
            "startAt": start_at,
            "maxResults": max_results,
            "fields": ["key"]
        }
    )
    data = response.json()

    for issue in data["issues"]:
        key = issue["key"]
        update_response = requests.put(
            f"{base_url}/rest/api/3/issue/{key}",
            auth=auth,
            json={
                "fields": {
                    "priority": {"name": "Medium"}
                }
            }
        )
        if update_response.status_code == 204:
            updated += 1
            print(f"Updated {key}")
        else:
            print(f"Failed to update {key}: {update_response.status_code}")
        time.sleep(0.2)  # Rate limiting

    if start_at + max_results >= data["total"]:
        break
    start_at += max_results

print(f"Updated {updated} issues")
```

## Bulk Transition

Transition multiple issues through a workflow step:

```bash
#!/bin/bash
# Bulk transition issues to "Done"

# First, find the transition ID
TRANSITION_ID=$(curl -s \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-1/transitions" \
  | jq -r '.transitions[] | select(.name == "Done") | .id')

echo "Using transition ID: $TRANSITION_ID"

# Search for issues to transition
RESPONSE=$(curl -s -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search/jql" \
  -d '{
    "jql": "project = PROJ AND status = \"In Review\" AND fixVersion = \"v1.0.0\"",
    "fields": ["key"]
  }')

KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')

for KEY in $KEYS; do
  echo "Transitioning $KEY to Done..."
  curl -s -X POST \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/issue/$KEY/transitions" \
    -d "{
      \"transition\": { \"id\": \"$TRANSITION_ID\" },
      \"fields\": {
        \"resolution\": { \"name\": \"Done\" }
      }
    }"
  sleep 0.2
done
```

## Bulk Delete

```bash
#!/bin/bash
# Bulk delete issues (use with caution!)

RESPONSE=$(curl -s -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search/jql" \
  -d '{
    "jql": "project = TEST AND issuetype = \"Test Issue\"",
    "fields": ["key"]
  }')

KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')

echo "About to delete $(echo "$KEYS" | wc -w) issues. Press Enter to continue or Ctrl+C to cancel."
read

for KEY in $KEYS; do
  echo "Deleting $KEY..."
  curl -s -X DELETE \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    "$JIRA_BASE_URL/rest/api/3/issue/$KEY?deleteSubtasks=true"
  sleep 0.2
done

echo "Bulk delete complete."
```

## Bulk Move Issues to Sprint

```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/agile/1.0/sprint/{sprintId}/issue" \
  -d '{
    "issues": ["PROJ-1", "PROJ-2", "PROJ-3", "PROJ-4", "PROJ-5"]
  }'
```

## Bulk Label Update

```bash
#!/bin/bash
# Add a label to all issues matching a query

RESPONSE=$(curl -s -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search/jql" \
  -d '{
    "jql": "project = PROJ AND component = \"Legacy API\"",
    "fields": ["key", "labels"]
  }')

echo "$RESPONSE" | jq -c '.issues[]' | while read -r ISSUE; do
  KEY=$(echo "$ISSUE" | jq -r '.key')

  echo "Adding label to $KEY..."
  curl -s -X PUT \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/issue/$KEY" \
    -d '{
      "update": {
        "labels": [
          { "add": "deprecated" }
        ]
      }
    }'
  sleep 0.2
done
```

## Best Practices

1. **Rate limiting** — Add delays between requests (200ms minimum recommended)
2. **Error handling** — Check each response status; log failures for retry
3. **Pagination** — Process results in pages of 50-100 items
4. **Dry run** — Log planned changes before executing, especially for deletes
5. **Idempotency** — Design operations to be safely retryable
6. **Backup** — Export affected issues before bulk modifications
7. **Off-peak hours** — Run bulk operations during low-usage periods
8. **Batch size** — Bulk create supports up to 50 issues per request
9. **Monitoring** — Track progress and errors throughout the operation
10. **Rollback plan** — Have a plan to revert changes if something goes wrong
