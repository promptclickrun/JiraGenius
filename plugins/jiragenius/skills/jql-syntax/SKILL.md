---
name: jql-syntax
description: "Complete JQL (Jira Query Language) syntax reference for searching issues in Jira Cloud."
---

# JQL Syntax Skill

Complete JQL (Jira Query Language) syntax reference for searching issues in Jira Cloud.

## Basic Syntax

```
field operator value [AND|OR field operator value] [ORDER BY field [ASC|DESC]]
```

## Operators

### Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equals | `status = "In Progress"` |
| `!=` | Not equals | `status != Done` |
| `>` | Greater than | `priority > Medium` |
| `>=` | Greater than or equal | `created >= "2026-01-01"` |
| `<` | Less than | `due < now()` |
| `<=` | Less than or equal | `updated <= -7d` |
| `~` | Contains (text search) | `summary ~ "login error"` |
| `!~` | Does not contain | `summary !~ "test"` |

### Membership Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `IN` | In list | `project IN ("PROJ", "DEV", "OPS")` |
| `NOT IN` | Not in list | `status NOT IN ("Done", "Closed")` |

### Null Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `IS EMPTY` | Field has no value | `assignee IS EMPTY` |
| `IS NOT EMPTY` | Field has a value | `due IS NOT EMPTY` |
| `IS NULL` | Alias for IS EMPTY | `fixVersion IS NULL` |
| `IS NOT NULL` | Alias for IS NOT EMPTY | `resolution IS NOT NULL` |

### Logical Operators

| Operator | Description |
|----------|-------------|
| `AND` | Both conditions must match |
| `OR` | Either condition must match |
| `NOT` | Negates a condition |

Precedence: `NOT` > `AND` > `OR`. Use parentheses to control grouping.

## Fields

### Standard Fields

| Field | Type | Notes |
|-------|------|-------|
| `project` | Project | Key or name |
| `issuetype` | Issue Type | Name (e.g., "Bug", "Story") |
| `status` | Status | Name (e.g., "In Progress") |
| `statusCategory` | Status Category | "To Do", "In Progress", "Done" |
| `resolution` | Resolution | Name or `Unresolved` |
| `priority` | Priority | Name (e.g., "High", "Medium") |
| `assignee` | User | `accountId`, `currentUser()`, or `EMPTY` |
| `reporter` | User | `accountId` or `currentUser()` |
| `creator` | User | `accountId` or `currentUser()` |
| `summary` | Text | Supports `~` operator |
| `description` | Text | Supports `~` operator |
| `comment` | Text | Searches comment bodies |
| `text` | Text | Searches summary, description, comments |
| `labels` | Label | Exact match |
| `component` | Component | Name or ID |
| `fixVersion` | Version | Name (e.g., "v1.0.0") |
| `affectedVersion` | Version | Name |
| `sprint` | Sprint | Name, ID, or function |
| `epic` | Epic | Key (e.g., "PROJ-50") |
| `parent` | Issue | Key (for subtasks and child issues) |
| `created` | Date | ISO date or relative |
| `updated` | Date | ISO date or relative |
| `due` | Date | ISO date or relative |
| `resolved` | Date | ISO date or relative |
| `lastViewed` | Date | ISO date or relative |
| `originalEstimate` | Duration | Minutes or time string |
| `remainingEstimate` | Duration | Minutes or time string |
| `timeSpent` | Duration | Minutes or time string |
| `workRatio` | Number | Percentage (0-100+) |
| `voter` | User | `currentUser()` |
| `watcher` | User | `currentUser()` |
| `level` | Security Level | Name |
| `key` | Issue Key | e.g., "PROJ-123" |
| `id` | Issue ID | Numeric ID |
| `type` | Issue Type | Alias for `issuetype` |

### Custom Fields

Reference by name or ID:
```jql
"Team Name" = "Backend"
cf[10001] = "Backend"
```

## Functions

### Date Functions

| Function | Description | Example |
|----------|-------------|---------|
| `now()` | Current date/time | `created > now()` |
| `currentLogin()` | When user last logged in | `updated > currentLogin()` |
| `startOfDay()` | Start of today | `created >= startOfDay()` |
| `endOfDay()` | End of today | `due <= endOfDay()` |
| `startOfWeek()` | Start of current week | `created >= startOfWeek()` |
| `endOfWeek()` | End of current week | `due <= endOfWeek()` |
| `startOfMonth()` | Start of current month | `created >= startOfMonth()` |
| `endOfMonth()` | End of current month | `due <= endOfMonth()` |
| `startOfYear()` | Start of current year | `created >= startOfYear()` |
| `endOfYear()` | End of current year | `due <= endOfYear()` |

Date functions accept offset parameters:
```jql
startOfDay(-1)     -- Start of yesterday
endOfWeek(1)       -- End of next week
startOfMonth(-2)   -- Start of two months ago
```

### Relative Dates

| Format | Meaning |
|--------|---------|
| `-30m` | 30 minutes ago |
| `-4h` | 4 hours ago |
| `-7d` | 7 days ago |
| `-2w` | 2 weeks ago |
| `-3M` | 3 months ago (uppercase M) |
| `-1y` | 1 year ago |

### User Functions

| Function | Description |
|----------|-------------|
| `currentUser()` | The logged-in user |
| `membersOf("group")` | All members of a group |

### Sprint Functions

| Function | Description |
|----------|-------------|
| `openSprints()` | Currently active sprints |
| `closedSprints()` | Completed sprints |
| `futureSprints()` | Planned sprints |

### Issue Link Functions

| Function | Description |
|----------|-------------|
| `linkedIssues(key)` | All issues linked to the given issue |
| `linkedIssues(key, "link type")` | Issues linked with specific type |

### List Functions

| Function | Description |
|----------|-------------|
| `issueHistory()` | Issues you have recently viewed |
| `votedIssues()` | Issues you have voted for |
| `watchedIssues()` | Issues you are watching |
| `updatedBy(user, dateRange)` | Issues updated by user in range |

## ORDER BY

```jql
ORDER BY created DESC
ORDER BY priority DESC, created ASC
ORDER BY rank ASC               -- Board rank order
ORDER BY lastViewed DESC        -- Recently viewed
ORDER BY key ASC                -- Stable ordering
```

Sortable fields: `key`, `summary`, `assignee`, `reporter`, `priority`, `status`, `resolution`, `created`, `updated`, `due`, `resolved`, `rank`, `lastViewed`, plus most custom fields.

## Escaping and Quoting

- Wrap multi-word values in double quotes: `status = "In Progress"`
- Escape special characters in values: `summary ~ "can\'t login"`
- Reserved words must be quoted when used as values
- Custom field names with spaces must be quoted: `"Story Points" > 5`

Reserved characters: ` + - & | ! ( ) { } [ ] ^ ~ * ? \ :`

## Common Query Patterns

### By Assignment
```jql
-- My open issues
assignee = currentUser() AND resolution = Unresolved

-- Unassigned issues
assignee IS EMPTY AND resolution = Unresolved

-- Issues assigned to my team
assignee IN membersOf("dev-team") AND resolution = Unresolved
```

### By Date
```jql
-- Created this week
created >= startOfWeek()

-- Updated in the last 24 hours
updated >= -24h

-- Overdue issues
due < now() AND resolution = Unresolved

-- Due this week
due >= startOfWeek() AND due <= endOfWeek()
```

### By Status
```jql
-- All in-progress work
statusCategory = "In Progress"

-- Blocked issues (using label convention)
labels = blocked AND resolution = Unresolved

-- Recently resolved
resolved >= -7d
```

### By Version
```jql
-- Issues fixed in a release
fixVersion = "v1.0.0"

-- Issues affecting a version
affectedVersion = "v0.9.0" AND resolution = Unresolved
```

### Sprint Planning
```jql
-- Current sprint
sprint in openSprints()

-- Backlog items ready for sprint
sprint IS EMPTY AND status = "Ready" AND type IN (Story, Task, Bug)

-- Sprint carryover
sprint in closedSprints() AND resolution = Unresolved
```

### Cross-Project
```jql
-- All bugs across projects
issuetype = Bug AND resolution = Unresolved ORDER BY priority DESC

-- Issues from multiple projects
project IN ("PROJ", "DEV") AND updated >= -7d
```
