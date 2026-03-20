# Example: Bulk Update Operations

## Bulk Create Issues

Create multiple issues in a single API call (max 50 per request):

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/bulk" \
  -d '{
    "issueUpdates": [
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Task" },
          "summary": "Set up monitoring dashboards",
          "priority": { "name": "High" },
          "labels": ["infrastructure"]
        }
      },
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Task" },
          "summary": "Configure alerting rules",
          "priority": { "name": "High" },
          "labels": ["infrastructure"]
        }
      },
      {
        "fields": {
          "project": { "key": "PROJ" },
          "issuetype": { "name": "Task" },
          "summary": "Set up log aggregation",
          "priority": { "name": "Medium" },
          "labels": ["infrastructure"]
        }
      }
    ]
  }'
```

**Response:**
```json
{
  "issues": [
    { "id": "10050", "key": "PROJ-150", "self": "https://..." },
    { "id": "10051", "key": "PROJ-151", "self": "https://..." },
    { "id": "10052", "key": "PROJ-152", "self": "https://..." }
  ],
  "errors": []
}
```

## Bulk Re-assign Issues

Re-assign all issues from one user to another:

```bash
#!/bin/bash
OLD_USER="old-account-id"
NEW_USER="new-account-id"

START_AT=0
MAX_RESULTS=50
TOTAL=1

while [ $START_AT -lt $TOTAL ]; do
  RESPONSE=$(curl -s -X POST \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/search" \
    -d "{
      \"jql\": \"assignee = '$OLD_USER' AND resolution = Unresolved\",
      \"startAt\": $START_AT,
      \"maxResults\": $MAX_RESULTS,
      \"fields\": [\"key\"]
    }")

  TOTAL=$(echo "$RESPONSE" | jq '.total')
  KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')

  for KEY in $KEYS; do
    echo "Re-assigning $KEY..."
    curl -s -X PUT \
      -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
      -H "Content-Type: application/json" \
      "$JIRA_BASE_URL/rest/api/3/issue/$KEY/assignee" \
      -d "{\"accountId\": \"$NEW_USER\"}"
    sleep 0.2
  done

  START_AT=$((START_AT + MAX_RESULTS))
done

echo "Re-assignment complete."
```

## Bulk Close Resolved Issues

Close all issues that are in "Resolved" status:

```bash
#!/bin/bash
# First, find the "Close" transition ID from a sample issue
TRANSITIONS=$(curl -s \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-1/transitions")

echo "Available transitions:"
echo "$TRANSITIONS" | jq '.transitions[] | {id, name}'

# Use the transition ID for "Close" (e.g., 41)
TRANSITION_ID="41"

RESPONSE=$(curl -s -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search" \
  -d '{
    "jql": "project = PROJ AND status = Resolved",
    "fields": ["key"],
    "maxResults": 100
  }')

KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')
TOTAL=$(echo "$RESPONSE" | jq '.total')
echo "Found $TOTAL issues to close."

for KEY in $KEYS; do
  echo "Closing $KEY..."
  curl -s -X POST \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/issue/$KEY/transitions" \
    -d "{
      \"transition\": { \"id\": \"$TRANSITION_ID\" }
    }"
  sleep 0.2
done
```

## Bulk Add Component to Issues

```bash
#!/bin/bash
COMPONENT="Backend API"
JQL="project = PROJ AND labels = backend AND component IS EMPTY"

RESPONSE=$(curl -s -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search" \
  -d "{
    \"jql\": \"$JQL\",
    \"fields\": [\"key\"],
    \"maxResults\": 100
  }")

KEYS=$(echo "$RESPONSE" | jq -r '.issues[].key')

for KEY in $KEYS; do
  echo "Adding component '$COMPONENT' to $KEY..."
  curl -s -X PUT \
    -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
    -H "Content-Type: application/json" \
    "$JIRA_BASE_URL/rest/api/3/issue/$KEY" \
    -d "{
      \"update\": {
        \"components\": [{ \"add\": { \"name\": \"$COMPONENT\" } }]
      }
    }"
  sleep 0.2
done
```
