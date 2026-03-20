# REST API v3 Reference Skill

Complete endpoint reference for the Jira Cloud REST API v3.

**Base URL:** `https://{your-domain}.atlassian.net/rest/api/3/`

**Documentation:** https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/#about

## Endpoint Categories

### Issues

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rest/api/3/issue` | Create issue |
| `POST` | `/rest/api/3/issue/bulk` | Create issues in bulk |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}` | Get issue |
| `PUT` | `/rest/api/3/issue/{issueIdOrKey}` | Edit issue |
| `DELETE` | `/rest/api/3/issue/{issueIdOrKey}` | Delete issue |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/transitions` | Get transitions |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/transitions` | Transition issue |
| `PUT` | `/rest/api/3/issue/{issueIdOrKey}/assignee` | Assign issue |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/changelog` | Get changelog |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/comment` | Get comments |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/comment` | Add comment |
| `PUT` | `/rest/api/3/issue/{issueIdOrKey}/comment/{commentId}` | Update comment |
| `DELETE` | `/rest/api/3/issue/{issueIdOrKey}/comment/{commentId}` | Delete comment |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/attachments` | Add attachment |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/watchers` | Get watchers |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/watchers` | Add watcher |
| `DELETE` | `/rest/api/3/issue/{issueIdOrKey}/watchers` | Remove watcher |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/votes` | Get votes |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/votes` | Add vote |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/worklog` | Get worklogs |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/worklog` | Add worklog |
| `GET` | `/rest/api/3/issue/{issueIdOrKey}/remotelink` | Get remote links |
| `POST` | `/rest/api/3/issue/{issueIdOrKey}/remotelink` | Create remote link |
| `POST` | `/rest/api/3/issueLink` | Link issues |
| `DELETE` | `/rest/api/3/issueLink/{linkId}` | Delete issue link |
| `GET` | `/rest/api/3/issueLinkType` | Get link types |

### Search

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/search` | Search with JQL (GET) |
| `POST` | `/rest/api/3/search` | Search with JQL (POST) |
| `GET` | `/rest/api/3/search/approximate-count` | Get approximate issue count |

### Projects

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/project/search` | Search projects |
| `POST` | `/rest/api/3/project` | Create project |
| `GET` | `/rest/api/3/project/{projectIdOrKey}` | Get project |
| `PUT` | `/rest/api/3/project/{projectIdOrKey}` | Update project |
| `DELETE` | `/rest/api/3/project/{projectIdOrKey}` | Delete project |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/components` | Get components |
| `POST` | `/rest/api/3/component` | Create component |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/versions` | Get versions |
| `POST` | `/rest/api/3/version` | Create version |
| `PUT` | `/rest/api/3/version/{id}` | Update version |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/role` | Get project roles |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/role/{roleId}` | Get role actors |
| `POST` | `/rest/api/3/project/{projectIdOrKey}/role/{roleId}` | Add role actors |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/features` | Get project features |
| `PUT` | `/rest/api/3/project/{projectIdOrKey}/features/{featureKey}` | Toggle feature |
| `GET` | `/rest/api/3/project/{projectIdOrKey}/statuses` | Get project statuses |

### Users and Groups

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/myself` | Get current user |
| `GET` | `/rest/api/3/user?accountId={id}` | Get user |
| `GET` | `/rest/api/3/user/search` | Find users |
| `GET` | `/rest/api/3/user/bulk` | Bulk get users |
| `GET` | `/rest/api/3/user/assignable/search` | Find assignable users |
| `POST` | `/rest/api/3/group` | Create group |
| `DELETE` | `/rest/api/3/group` | Delete group |
| `GET` | `/rest/api/3/group/member` | Get group members |
| `POST` | `/rest/api/3/group/user` | Add user to group |
| `DELETE` | `/rest/api/3/group/user` | Remove user from group |
| `GET` | `/rest/api/3/groups/picker` | Find groups |

### Workflows

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/workflow/search` | Search workflows |
| `POST` | `/rest/api/3/workflow` | Create workflow |
| `GET` | `/rest/api/3/status` | Get all statuses |
| `POST` | `/rest/api/3/statuses` | Create statuses |
| `PUT` | `/rest/api/3/statuses` | Update statuses |
| `DELETE` | `/rest/api/3/statuses` | Delete statuses |
| `GET` | `/rest/api/3/workflowscheme/{id}` | Get workflow scheme |
| `POST` | `/rest/api/3/workflowscheme` | Create workflow scheme |
| `PUT` | `/rest/api/3/workflowscheme/{id}` | Update workflow scheme |
| `DELETE` | `/rest/api/3/workflowscheme/{id}` | Delete workflow scheme |
| `GET` | `/rest/api/3/workflow/transitions/{transitionId}/properties` | Get transition properties |
| `PUT` | `/rest/api/3/workflow/transitions/{transitionId}/properties` | Set transition property |

### Fields and Screens

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/field` | Get all fields |
| `POST` | `/rest/api/3/field` | Create custom field |
| `PUT` | `/rest/api/3/field/{fieldId}` | Update custom field |
| `DELETE` | `/rest/api/3/field/{fieldId}` | Delete custom field |
| `GET` | `/rest/api/3/field/{fieldId}/context` | Get field contexts |
| `POST` | `/rest/api/3/field/{fieldId}/context` | Create context |
| `GET` | `/rest/api/3/field/{fieldId}/context/{contextId}/option` | Get options |
| `POST` | `/rest/api/3/field/{fieldId}/context/{contextId}/option` | Create options |
| `GET` | `/rest/api/3/screens` | Get screens |
| `POST` | `/rest/api/3/screens` | Create screen |
| `GET` | `/rest/api/3/screens/{screenId}/tabs` | Get screen tabs |
| `POST` | `/rest/api/3/screens/{screenId}/tabs/{tabId}/fields` | Add field to tab |
| `GET` | `/rest/api/3/screenscheme` | Get screen schemes |
| `POST` | `/rest/api/3/screenscheme` | Create screen scheme |
| `GET` | `/rest/api/3/fieldconfiguration` | Get field configurations |
| `POST` | `/rest/api/3/fieldconfiguration` | Create field configuration |
| `GET` | `/rest/api/3/fieldconfigurationscheme` | Get field config schemes |

### Issue Types

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/issuetype` | Get all issue types |
| `POST` | `/rest/api/3/issuetype` | Create issue type |
| `PUT` | `/rest/api/3/issuetype/{id}` | Update issue type |
| `DELETE` | `/rest/api/3/issuetype/{id}` | Delete issue type |
| `GET` | `/rest/api/3/issuetypescheme` | Get issue type schemes |
| `POST` | `/rest/api/3/issuetypescheme` | Create issue type scheme |
| `GET` | `/rest/api/3/issuetypescreenscheme` | Get issue type screen schemes |
| `POST` | `/rest/api/3/issuetypescreenscheme` | Create issue type screen scheme |

### Permissions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/mypermissions` | Get my permissions |
| `POST` | `/rest/api/3/permissions/check` | Check permissions in bulk |
| `GET` | `/rest/api/3/permissionscheme` | Get all permission schemes |
| `GET` | `/rest/api/3/permissionscheme/{schemeId}` | Get permission scheme |
| `POST` | `/rest/api/3/permissionscheme` | Create permission scheme |
| `PUT` | `/rest/api/3/permissionscheme/{schemeId}` | Update permission scheme |
| `POST` | `/rest/api/3/permissionscheme/{schemeId}/permission` | Add grant |
| `DELETE` | `/rest/api/3/permissionscheme/{schemeId}/permission/{permissionId}` | Remove grant |
| `GET` | `/rest/api/3/role` | Get all project roles |
| `POST` | `/rest/api/3/role` | Create project role |
| `GET` | `/rest/api/3/issuesecurityschemes` | Get security schemes |

### Filters and Dashboards

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rest/api/3/filter` | Create filter |
| `GET` | `/rest/api/3/filter/{filterId}` | Get filter |
| `PUT` | `/rest/api/3/filter/{filterId}` | Update filter |
| `DELETE` | `/rest/api/3/filter/{filterId}` | Delete filter |
| `GET` | `/rest/api/3/filter/favourite` | Get favourite filters |
| `POST` | `/rest/api/3/filter/{filterId}/permission` | Add share permission |
| `GET` | `/rest/api/3/dashboard/search` | Search dashboards |
| `POST` | `/rest/api/3/dashboard` | Create dashboard |
| `GET` | `/rest/api/3/dashboard/{dashboardId}` | Get dashboard |
| `POST` | `/rest/api/3/dashboard/{dashboardId}/gadget` | Add gadget |

### Webhooks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rest/api/3/webhook` | Register webhooks |
| `GET` | `/rest/api/3/webhook` | Get registered webhooks |
| `DELETE` | `/rest/api/3/webhook` | Delete webhooks |
| `PUT` | `/rest/api/3/webhook/refresh` | Extend webhook life |
| `GET` | `/rest/api/3/webhook/failed` | Get failed webhooks |

### Server Info

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/api/3/serverInfo` | Get server info |
| `GET` | `/rest/api/3/configuration` | Get global configuration |

### Agile REST API (Jira Software)

Base path: `/rest/agile/1.0/`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/rest/agile/1.0/board` | List boards |
| `POST` | `/rest/agile/1.0/board` | Create board |
| `GET` | `/rest/agile/1.0/board/{boardId}` | Get board |
| `DELETE` | `/rest/agile/1.0/board/{boardId}` | Delete board |
| `GET` | `/rest/agile/1.0/board/{boardId}/configuration` | Get board config |
| `GET` | `/rest/agile/1.0/board/{boardId}/sprint` | Get sprints |
| `POST` | `/rest/agile/1.0/sprint` | Create sprint |
| `PUT` | `/rest/agile/1.0/sprint/{sprintId}` | Update sprint |
| `POST` | `/rest/agile/1.0/sprint/{sprintId}/issue` | Move issues to sprint |
| `GET` | `/rest/agile/1.0/sprint/{sprintId}/issue` | Get sprint issues |
| `GET` | `/rest/agile/1.0/board/{boardId}/backlog` | Get backlog |
| `POST` | `/rest/agile/1.0/backlog/issue` | Move to backlog |
| `GET` | `/rest/agile/1.0/board/{boardId}/epic` | Get epics |
| `POST` | `/rest/agile/1.0/epic/{epicIdOrKey}/issue` | Move issues to epic |
| `PUT` | `/rest/agile/1.0/issue/rank` | Rank issues |

## HTTP Methods

- `GET` — Retrieve resources (idempotent, safe)
- `POST` — Create resources or execute actions
- `PUT` — Update resources (full replacement or specific updates)
- `DELETE` — Remove resources
- `PATCH` — Partial update (limited use in Jira API)

## Common Query Parameters

| Parameter | Description |
|-----------|-------------|
| `expand` | Comma-separated list of entities to expand inline |
| `fields` | Comma-separated list of fields to include |
| `startAt` | Pagination offset (0-based) |
| `maxResults` | Maximum results per page (server may enforce lower limit) |
| `orderBy` | Sort field and direction |

## Response Format

Successful responses return JSON with the resource data.
Paginated responses include:

```json
{
  "startAt": 0,
  "maxResults": 50,
  "total": 125,
  "values": [...]
}
```

Or for search:
```json
{
  "startAt": 0,
  "maxResults": 50,
  "total": 125,
  "issues": [...]
}
```
