# Rate Limiting Skill

This skill covers Jira Cloud REST API rate limiting policies, response headers, backoff strategies, and best practices for staying within limits.

## Rate Limiting Overview

Jira Cloud enforces rate limits to ensure platform stability. Limits are applied per user, per app, and per tenant (Jira instance).

### Rate Limit Types

1. **Concurrent requests** — Maximum simultaneous requests per user
2. **Request rate** — Maximum requests per time window
3. **Request size** — Maximum payload and response sizes

### Current Known Limits

Atlassian does not publish exact rate limit numbers, but observed limits include:

- **Standard API calls:** ~100-200 requests per minute per user
- **Concurrent requests:** ~10 simultaneous requests per user
- **Bulk operations:** Lower limits apply to resource-intensive endpoints
- **Search queries:** Lower limits for complex JQL queries
- **Attachments:** Strict limits due to bandwidth usage

## Rate Limit Response Headers

When approaching or exceeding limits, Jira includes these headers:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed in the window |
| `X-RateLimit-Remaining` | Remaining requests in the current window |
| `X-RateLimit-Reset` | Unix timestamp when the rate limit resets |
| `Retry-After` | Seconds to wait before retrying (on 429 responses) |

### HTTP 429 Response

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1711843260
Content-Type: application/json

{
  "message": "Rate limit exceeded. Try again in 30 seconds.",
  "error": "rate_limit_exceeded"
}
```

## Backoff Strategies

### Exponential Backoff with Jitter

The recommended strategy for handling rate limits:

```python
import time
import random
import requests

def make_request_with_backoff(url, auth, method="GET", json=None, max_retries=5):
    for attempt in range(max_retries):
        if method == "GET":
            response = requests.get(url, auth=auth, headers={"Accept": "application/json"})
        elif method == "POST":
            response = requests.post(url, auth=auth, json=json,
                                     headers={"Content-Type": "application/json"})
        elif method == "PUT":
            response = requests.put(url, auth=auth, json=json,
                                    headers={"Content-Type": "application/json"})

        if response.status_code == 429:
            retry_after = int(response.headers.get("Retry-After", 2 ** attempt))
            jitter = random.uniform(0, retry_after * 0.5)
            wait_time = retry_after + jitter
            print(f"Rate limited. Waiting {wait_time:.1f}s (attempt {attempt + 1}/{max_retries})")
            time.sleep(wait_time)
            continue

        if response.status_code >= 500:
            wait_time = min(2 ** attempt + random.uniform(0, 1), 60)
            print(f"Server error {response.status_code}. Waiting {wait_time:.1f}s")
            time.sleep(wait_time)
            continue

        return response

    raise Exception(f"Max retries ({max_retries}) exceeded")
```

### Bash Backoff

```bash
#!/bin/bash
make_request() {
  local url="$1"
  local max_retries=5
  local attempt=0

  while [ $attempt -lt $max_retries ]; do
    HTTP_CODE=$(curl -s -o /tmp/response.json -w "%{http_code}" \
      -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
      -H "Accept: application/json" \
      "$url")

    if [ "$HTTP_CODE" = "429" ]; then
      RETRY_AFTER=$(curl -s -I -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" "$url" \
        | grep -i "retry-after" | awk '{print $2}' | tr -d '\r')
      WAIT=${RETRY_AFTER:-$((2 ** attempt))}
      echo "Rate limited. Waiting ${WAIT}s..."
      sleep "$WAIT"
      attempt=$((attempt + 1))
      continue
    fi

    if [ "$HTTP_CODE" -ge 500 ]; then
      WAIT=$((2 ** attempt))
      echo "Server error $HTTP_CODE. Waiting ${WAIT}s..."
      sleep "$WAIT"
      attempt=$((attempt + 1))
      continue
    fi

    cat /tmp/response.json
    return 0
  done

  echo "Max retries exceeded" >&2
  return 1
}
```

## Proactive Rate Limit Management

### Monitor Remaining Quota

```bash
# Check rate limit headers from any API call
curl -s -D - \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/myself" \
  -o /dev/null 2>&1 \
  | grep -i "x-ratelimit"
```

### Request Throttling

Spread requests evenly to avoid bursts:

```python
import time

class RateLimiter:
    def __init__(self, requests_per_second=5):
        self.min_interval = 1.0 / requests_per_second
        self.last_request = 0

    def wait(self):
        elapsed = time.time() - self.last_request
        if elapsed < self.min_interval:
            time.sleep(self.min_interval - elapsed)
        self.last_request = time.time()

# Usage
limiter = RateLimiter(requests_per_second=5)

for issue_key in issue_keys:
    limiter.wait()
    response = requests.get(f"{base_url}/rest/api/3/issue/{issue_key}", auth=auth)
```

## Best Practices

### Do

1. **Implement exponential backoff** — Always handle 429 responses with backoff
2. **Add jitter** — Randomize wait times to prevent thundering herd
3. **Use bulk endpoints** — `POST /rest/api/3/issue/bulk` instead of individual creates
4. **Request only needed fields** — Use `fields` parameter to minimize response size
5. **Cache responses** — Cache static data like issue types, priorities, and statuses
6. **Paginate efficiently** — Use optimal page sizes (50-100)
7. **Use webhooks** — Subscribe to events instead of polling for changes
8. **Throttle proactively** — Don't wait for 429; limit your request rate preemptively
9. **Monitor headers** — Check `X-RateLimit-Remaining` to adjust throttling dynamically
10. **Run off-peak** — Schedule bulk operations during low-usage hours

### Don't

1. **Don't retry immediately** — Always wait before retrying a rate-limited request
2. **Don't ignore 429** — Continued requests while rate-limited may result in longer blocks
3. **Don't poll frequently** — Avoid polling intervals shorter than 1 minute
4. **Don't parallelize excessively** — Limit concurrent requests to 5 or fewer
5. **Don't fetch full objects** — Avoid requesting all fields when you only need a few
6. **Don't ignore server errors** — 500/502/503 errors may indicate rate-related load issues
7. **Don't hardcode limits** — Atlassian may change rate limits; always read response headers

## Estimating Request Budget

For planning bulk operations:

```
Total requests = (number of items) × (requests per item) + (pagination requests)

Example: Update 1000 issues
- Search pages: ceil(1000 / 100) = 10 requests
- Updates: 1000 requests
- Total: ~1010 requests
- At 5 req/s: ~3.4 minutes
- At 2 req/s (conservative): ~8.4 minutes
```
