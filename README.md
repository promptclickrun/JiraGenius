# JiraGenius

An expert **Jira Cloud REST API** plugin for **GitHub Copilot CLI** and **Claude Code CLI**. JiraGenius provides specialized sub-agents and skills that work together to configure, manage, and support Jira Cloud instances for admin teams.

Built on the official [Jira Cloud REST API v3](https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/#about) documentation.

## Installation

### GitHub Copilot CLI

```
/plugin install promptclickrun/JiraGenius
```

### Claude Code CLI

```
/install-plugin https://github.com/promptclickrun/JiraGenius
```

After installation, the plugin's slash commands and agent capabilities become available in your terminal session.

## Configuration

Set up your Jira Cloud credentials as environment variables:

```bash
export JIRA_BASE_URL="https://your-domain.atlassian.net"
export JIRA_USER_EMAIL="your-email@example.com"
export JIRA_API_TOKEN="your-api-token"
```

Generate an API token at: https://id.atlassian.com/manage-profile/security/api-tokens

## Slash Commands

| Command | Description |
|---------|-------------|
| `/jira-configure` | Set up and validate your Jira Cloud connection |
| `/jira-issue` | Create, read, update, delete, and transition issues |
| `/jira-project` | Manage projects, components, and versions |
| `/jira-search` | Build and execute JQL queries |
| `/jira-workflow` | Design and manage workflows and statuses |
| `/jira-user` | Manage users, groups, roles, and permissions |
| `/jira-board` | Manage Scrum/Kanban boards and sprints |
| `/jira-field` | Configure custom fields, screens, and schemes |
| `/jira-webhook` | Register and manage webhooks |
| `/jira-bulk` | Perform bulk create, update, and transition operations |

## Sub-Agents

JiraGenius delegates tasks to eight specialist sub-agents, coordinated by a main orchestrator:

| Agent | Specialty |
|-------|-----------|
| **Coordinator** | Routes requests to the appropriate specialist and orchestrates multi-agent workflows |
| **Issue Manager** | Issue CRUD operations, transitions, comments, attachments, and linking |
| **Project Admin** | Project creation, configuration, components, versions, and schemes |
| **Workflow Designer** | Workflow statuses, transitions, conditions, validators, and post-functions |
| **Search Analyst** | JQL query construction, filters, dashboards, and reporting |
| **Permissions Manager** | Users, groups, roles, permission schemes, and security levels |
| **Board Manager** | Scrum and Kanban boards, sprints, backlogs, and velocity tracking |
| **Schema Expert** | Custom fields, field configurations, screens, issue types, and screen schemes |

## Skills

Cross-cutting skills available to all agents:

| Skill | Description |
|-------|-------------|
| **Authentication** | OAuth 2.0 (3LO), API tokens, and Basic auth patterns |
| **REST API v3** | Complete endpoint catalog with request/response formats |
| **Error Handling** | HTTP status codes, error responses, and troubleshooting |
| **Pagination** | Offset and cursor-based pagination for large result sets |
| **JQL Syntax** | Full query language reference with operators and functions |
| **Webhooks** | Event registration, payload formats, and security |
| **Bulk Operations** | Batch create, update, delete, and transition patterns |
| **Rate Limiting** | Throttling policies, backoff strategies, and concurrency limits |

## Project Structure

```
JiraGenius/
├── plugin.json                   # Plugin manifest (required for installation)
├── CLAUDE.md                     # Claude Code project instructions
├── copilot-extensions.yml        # GitHub Copilot Extensions configuration
├── .claude/commands/             # Claude Code slash commands
│   ├── jira-configure.md
│   ├── jira-issue.md
│   ├── jira-project.md
│   ├── jira-search.md
│   ├── jira-workflow.md
│   ├── jira-user.md
│   ├── jira-board.md
│   ├── jira-field.md
│   ├── jira-webhook.md
│   └── jira-bulk.md
├── agents/                       # Sub-agent definitions
│   ├── coordinator.md
│   ├── issue-manager.md
│   ├── project-admin.md
│   ├── workflow-designer.md
│   ├── search-analyst.md
│   ├── permissions-manager.md
│   ├── board-manager.md
│   └── schema-expert.md
├── skills/                       # Shared skill definitions
│   ├── authentication/SKILL.md
│   ├── rest-api-v3/SKILL.md
│   ├── error-handling/SKILL.md
│   ├── pagination/SKILL.md
│   ├── jql-syntax/SKILL.md
│   ├── webhooks/SKILL.md
│   ├── bulk-operations/SKILL.md
│   └── rate-limiting/SKILL.md
└── examples/                     # Usage examples
    ├── create-issue.md
    ├── bulk-update.md
    ├── jql-queries.md
    └── workflow-transitions.md
```

## Examples

### Create an Issue

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue" \
  -d '{
    "fields": {
      "project": { "key": "PROJ" },
      "issuetype": { "name": "Task" },
      "summary": "Implement user authentication",
      "description": {
        "type": "doc",
        "version": 1,
        "content": [
          {
            "type": "paragraph",
            "content": [
              { "type": "text", "text": "Add OAuth 2.0 authentication flow." }
            ]
          }
        ]
      }
    }
  }'
```

### Search with JQL

```bash
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search" \
  -d '{
    "jql": "project = PROJ AND status = \"In Progress\" ORDER BY priority DESC",
    "maxResults": 50,
    "fields": ["summary", "status", "assignee", "priority"]
  }'
```

### Transition an Issue

```bash
# Get available transitions
curl -s -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions"

# Execute transition
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  "$JIRA_BASE_URL/rest/api/3/issue/PROJ-123/transitions" \
  -d '{"transition": {"id": "31"}}'
```

See the [examples/](examples/) directory for more detailed usage examples.

## API Reference

This plugin covers the following Jira Cloud APIs:

- **Platform REST API v3**: `https://{domain}.atlassian.net/rest/api/3/`
- **Agile REST API**: `https://{domain}.atlassian.net/rest/agile/1.0/`

Full documentation: https://developer.atlassian.com/cloud/jira/platform/rest/v3/intro/#about

## License

MIT
