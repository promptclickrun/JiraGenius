# Example: Workflow Transitions

## Get Available Transitions for an Issue

```bash
curl -s -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" | jq '.transitions[] | {id, name, to: .to.name}'
```

**Response:**
```json
[
  { "id": "21", "name": "Start Progress", "to": "In Progress" },
  { "id": "31", "name": "Done", "to": "Done" },
  { "id": "41", "name": "Won't Do", "to": "Won't Do" }
]
```

## Transition an Issue to "In Progress"

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" \
  -d '{
    "transition": { "id": "21" }
  }'
```

## Transition with Resolution

When moving to "Done", you typically need to set a resolution:

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" \
  -d '{
    "transition": { "id": "31" },
    "fields": {
      "resolution": { "name": "Done" }
    }
  }'
```

## Transition with Comment

Add a comment during the transition:

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" \
  -d '{
    "transition": { "id": "31" },
    "fields": {
      "resolution": { "name": "Done" }
    },
    "update": {
      "comment": [
        {
          "add": {
            "body": {
              "type": "doc",
              "version": 1,
              "content": [
                {
                  "type": "paragraph",
                  "content": [
                    { "type": "text", "text": "Completed and verified in staging environment." }
                  ]
                }
              ]
            }
          }
        }
      ]
    }
  }'
```

## Transition with Field Updates

Update fields during the transition:

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" \
  -d '{
    "transition": { "id": "21" },
    "fields": {
      "assignee": { "accountId": "YOUR_ACCOUNT_ID" }
    },
    "update": {
      "labels": [
        { "add": "in-review" }
      ]
    }
  }'
```

## Create a Custom Workflow

### Step 1: Create Statuses

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/statuses" \
  -d '{
    "statuses": [
      { "name": "Code Review", "statusCategory": "IN_PROGRESS", "description": "Code is under review" },
      { "name": "QA Testing", "statusCategory": "IN_PROGRESS", "description": "Being tested by QA" },
      { "name": "Ready to Deploy", "statusCategory": "IN_PROGRESS", "description": "Approved for deployment" }
    ],
    "scope": {
      "type": "PROJECT",
      "project": { "id": "10001" }
    }
  }'
```

### Step 2: Create the Workflow

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/workflow" \
  -d '{
    "name": "Development Pipeline",
    "description": "Full development lifecycle workflow",
    "statuses": [
      { "id": "1" },
      { "id": "3" },
      { "id": "10100" },
      { "id": "10101" },
      { "id": "10102" },
      { "id": "10001" }
    ],
    "transitions": [
      { "name": "Start Work", "from": ["1"], "to": "3", "type": "directed" },
      { "name": "Submit for Review", "from": ["3"], "to": "10100", "type": "directed" },
      { "name": "Request Changes", "from": ["10100"], "to": "3", "type": "directed" },
      { "name": "Approve for QA", "from": ["10100"], "to": "10101", "type": "directed" },
      { "name": "QA Passed", "from": ["10101"], "to": "10102", "type": "directed" },
      { "name": "QA Failed", "from": ["10101"], "to": "3", "type": "directed" },
      { "name": "Deploy & Close", "from": ["10102"], "to": "10001", "type": "directed" },
      { "name": "Reopen", "from": ["10001"], "to": "1", "type": "directed" }
    ]
  }'
```

This creates the workflow:
```
Open → In Progress → Code Review → QA Testing → Ready to Deploy → Done
                   ↖ Request Changes ↙         ↖ QA Failed ↙      ↗ Reopen ↘ Open
```

### Step 3: Create a Workflow Scheme

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/workflowscheme" \
  -d '{
    "name": "Development Workflow Scheme",
    "description": "Maps issue types to the development pipeline workflow",
    "defaultWorkflow": "jira",
    "issueTypeMappings": {
      "10001": "Development Pipeline",
      "10002": "Development Pipeline",
      "10003": "Development Pipeline"
    }
  }'
```

### Step 4: Assign to Project

Assign the workflow scheme to a project via the Jira admin UI, or use:

```bash
curl -X PUT \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/workflowscheme/project" \
  -d '{
    "workflowSchemeId": "SCHEME_ID",
    "projectId": "PROJECT_ID"
  }'
```
