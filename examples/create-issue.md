# Example: Creating Issues

## Create a Simple Task

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "issuetype": { "name": "Task" },
      "summary": "Set up CI/CD pipeline",
      "description": {
        "type": "doc",
        "version": 1,
        "content": [
          {
            "type": "paragraph",
            "content": [
              { "type": "text", "text": "Configure GitHub Actions for automated builds and deployments." }
            ]
          }
        ]
      },
      "assignee": { "accountId": "5b10ac8d82e05b22cc7d4ef5" },
      "priority": { "name": "High" },
      "labels": ["devops", "infrastructure"]
    }
  }'
```

**Response:**
```json
{
  "id": "10001",
  "key": "PROJ-101",
  "self": "https://your-domain.atlassian.net/rest/api/3/issue/10001"
}
```

## Create a Bug with Rich Description

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "issuetype": { "name": "Bug" },
      "summary": "Login page shows 500 error on mobile Safari",
      "priority": { "name": "Critical" },
      "description": {
        "type": "doc",
        "version": 1,
        "content": [
          {
            "type": "heading",
            "attrs": { "level": 3 },
            "content": [
              { "type": "text", "text": "Steps to Reproduce" }
            ]
          },
          {
            "type": "orderedList",
            "content": [
              {
                "type": "listItem",
                "content": [
                  { "type": "paragraph", "content": [{ "type": "text", "text": "Open Safari on iOS 17" }] }
                ]
              },
              {
                "type": "listItem",
                "content": [
                  { "type": "paragraph", "content": [{ "type": "text", "text": "Navigate to /login" }] }
                ]
              },
              {
                "type": "listItem",
                "content": [
                  { "type": "paragraph", "content": [{ "type": "text", "text": "Enter valid credentials and submit" }] }
                ]
              }
            ]
          },
          {
            "type": "heading",
            "attrs": { "level": 3 },
            "content": [
              { "type": "text", "text": "Expected Result" }
            ]
          },
          {
            "type": "paragraph",
            "content": [
              { "type": "text", "text": "User is redirected to the dashboard." }
            ]
          },
          {
            "type": "heading",
            "attrs": { "level": 3 },
            "content": [
              { "type": "text", "text": "Actual Result" }
            ]
          },
          {
            "type": "paragraph",
            "content": [
              { "type": "text", "text": "500 Internal Server Error is displayed. Error log shows: " },
              { "type": "text", "text": "NullPointerException at AuthService.java:42", "marks": [{ "type": "code" }] }
            ]
          }
        ]
      },
      "components": [{ "name": "Authentication" }],
      "affectsVersions": [{ "name": "v2.3.0" }]
    }
  }'
```

## Create a Subtask

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "parent": { "key": "PROJ-101" },
      "issuetype": { "name": "Subtask" },
      "summary": "Write unit tests for auth module",
      "assignee": { "accountId": "5b10ac8d82e05b22cc7d4ef5" }
    }
  }'
```

## Create an Epic

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "issuetype": { "name": "Epic" },
      "summary": "User Authentication Overhaul",
      "customfield_10011": "Auth Overhaul",
      "description": {
        "type": "doc",
        "version": 1,
        "content": [
          {
            "type": "paragraph",
            "content": [
              { "type": "text", "text": "Modernize the authentication system with OAuth 2.0, MFA, and SSO support." }
            ]
          }
        ]
      }
    }
  }'
```

Note: The Epic Name field ID (`customfield_10011`) varies by instance. Use `GET /rest/api/3/field` to find the correct ID.
