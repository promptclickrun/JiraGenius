---
name: webhooks
description: "This skill covers Jira Cloud webhook registration, event types, payload formats, and security."
---

# Webhooks Skill

This skill covers Jira Cloud webhook registration, event types, payload formats, and security.

## Overview

Jira Cloud supports two webhook approaches:
1. **Dynamic Webhooks** — Registered via REST API (recommended for apps)
2. **System Webhooks** — Configured via Jira admin UI

## Dynamic Webhook Registration

### Register Webhooks
```
POST /rest/api/3/webhook
```

```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/webhook" \
  -d '{
    "webhooks": [
      {
        "url": "https://your-app.example.com/webhooks/jira",
        "events": [
          "jira:issue_created",
          "jira:issue_updated",
          "jira:issue_deleted"
        ],
        "jqlFilter": "project = PROJ",
        "fieldIdsFilter": ["summary", "status", "assignee"]
      }
    ]
  }'
```

### Get Registered Webhooks
```
GET /rest/api/3/webhook
```

### Delete Webhooks
```
DELETE /rest/api/3/webhook
```

```bash
curl -X DELETE \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/webhook" \
  -d '{
    "webhookIds": [10001, 10002]
  }'
```

### Extend Webhook Life
```
PUT /rest/api/3/webhook/refresh
```

Dynamic webhooks expire after 30 days. Refresh them before expiry:

```bash
curl -X PUT \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Content-Type: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/webhook/refresh" \
  -d '{
    "webhookIds": [10001, 10002]
  }'
```

### Get Failed Webhooks
```
GET /rest/api/3/webhook/failed
```

## Available Events

### Issue Events

| Event | Triggered When |
|-------|---------------|
| `jira:issue_created` | An issue is created |
| `jira:issue_updated` | An issue is updated (fields, status, etc.) |
| `jira:issue_deleted` | An issue is deleted |

### Comment Events

| Event | Triggered When |
|-------|---------------|
| `comment_created` | A comment is added |
| `comment_updated` | A comment is edited |
| `comment_deleted` | A comment is removed |

### Issue Link Events

| Event | Triggered When |
|-------|---------------|
| `issuelink_created` | Issues are linked |
| `issuelink_deleted` | An issue link is removed |

### Attachment Events

| Event | Triggered When |
|-------|---------------|
| `attachment_created` | A file is attached |
| `attachment_deleted` | An attachment is removed |

### Sprint Events

| Event | Triggered When |
|-------|---------------|
| `sprint_created` | A sprint is created |
| `sprint_updated` | A sprint is updated |
| `sprint_started` | A sprint is started |
| `sprint_closed` | A sprint is completed |
| `sprint_deleted` | A sprint is deleted |

### Board Events

| Event | Triggered When |
|-------|---------------|
| `board_created` | A board is created |
| `board_updated` | A board is updated |
| `board_deleted` | A board is deleted |
| `board_configuration_changed` | Board config changes |

### Project Events

| Event | Triggered When |
|-------|---------------|
| `project_created` | A project is created |
| `project_updated` | A project is updated |
| `project_deleted` | A project is deleted |
| `project_soft_deleted` | A project is moved to trash |
| `project_restored_deleted` | A project is restored from trash |

### User Events

| Event | Triggered When |
|-------|---------------|
| `user_created` | A user is added |
| `user_updated` | A user is updated |
| `user_deleted` | A user is removed |

### Worklog Events

| Event | Triggered When |
|-------|---------------|
| `worklog_created` | Work is logged |
| `worklog_updated` | A worklog is updated |
| `worklog_deleted` | A worklog is removed |

## Webhook Payload Format

### Issue Created/Updated Payload

```json
{
  "timestamp": 1711843200000,
  "webhookEvent": "jira:issue_created",
  "issue_event_type_name": "issue_created",
  "user": {
    "accountId": "user-account-id",
    "displayName": "John Doe",
    "emailAddress": "john@example.com"
  },
  "issue": {
    "id": "10001",
    "key": "PROJ-123",
    "fields": {
      "summary": "New issue summary",
      "status": {
        "name": "Open",
        "statusCategory": {
          "key": "new",
          "name": "To Do"
        }
      },
      "issuetype": {
        "name": "Task"
      },
      "project": {
        "key": "PROJ",
        "name": "My Project"
      },
      "assignee": {
        "accountId": "assignee-account-id",
        "displayName": "Jane Smith"
      },
      "priority": {
        "name": "High"
      },
      "created": "2026-03-19T00:00:00.000+0000",
      "updated": "2026-03-19T00:00:00.000+0000"
    }
  },
  "changelog": {
    "items": [
      {
        "field": "status",
        "fieldtype": "jira",
        "from": "1",
        "fromString": "Open",
        "to": "3",
        "toString": "In Progress"
      }
    ]
  }
}
```

## JQL Filtering

Webhooks can include a `jqlFilter` to receive events only for matching issues:

```json
{
  "jqlFilter": "project = PROJ AND issuetype = Bug AND priority = High"
}
```

Only `jira:issue_created`, `jira:issue_updated`, and `jira:issue_deleted` events support JQL filtering.

## Field Filtering

Use `fieldIdsFilter` to limit the fields included in webhook payloads:

```json
{
  "fieldIdsFilter": ["summary", "status", "assignee", "priority"]
}
```

This reduces payload size and improves processing efficiency.

## Security Best Practices

1. **Use HTTPS** — Webhook URLs must use HTTPS in production
2. **Validate payloads** — Verify the source IP is from Atlassian's IP ranges
3. **Idempotent processing** — Handle duplicate webhook deliveries gracefully
4. **Respond quickly** — Return HTTP 200/202 within 10 seconds; process asynchronously
5. **Handle failures** — Implement retry logic for failed webhook processing
6. **Monitor health** — Use `GET /rest/api/3/webhook/failed` to check for delivery failures
7. **Refresh regularly** — Dynamic webhooks expire after 30 days; set up automated refresh

## Key Considerations

- Dynamic webhooks require the app to be a Connect app or Forge app
- System webhooks (admin UI) do not expire but require admin access
- Webhook payloads have a maximum size; use field filtering to reduce size
- Failed webhooks are retried by Jira with exponential backoff
- Webhooks are delivered asynchronously; order is not guaranteed
- Test webhooks with tools like webhook.site or ngrok during development
