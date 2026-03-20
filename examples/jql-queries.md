# Example: JQL Queries

A collection of practical JQL queries for common Jira use cases.

## Daily Standup Queries

### My work in progress
```jql
assignee = currentUser() AND status = "In Progress" ORDER BY priority DESC
```

### My open issues by priority
```jql
assignee = currentUser() AND resolution = Unresolved ORDER BY priority DESC, created ASC
```

### What I completed yesterday
```jql
assignee = currentUser() AND status changed to "Done" DURING (startOfDay(-1), endOfDay(-1))
```

## Sprint Management

### Current sprint items
```jql
sprint in openSprints() AND project = PROJ ORDER BY rank ASC
```

### Sprint burndown (remaining work)
```jql
sprint in openSprints() AND project = PROJ AND statusCategory != Done ORDER BY rank ASC
```

### Carry-over from last sprint
```jql
sprint in closedSprints() AND resolution = Unresolved AND project = PROJ
```

### Ready for sprint planning
```jql
sprint IS EMPTY AND status = "Ready" AND issuetype IN (Story, Task, Bug) AND project = PROJ ORDER BY priority DESC
```

## Bug Tracking

### Open bugs by severity
```jql
issuetype = Bug AND resolution = Unresolved ORDER BY priority DESC, created ASC
```

### Bugs created this week
```jql
issuetype = Bug AND created >= startOfWeek() ORDER BY created DESC
```

### Critical bugs without assignee
```jql
issuetype = Bug AND priority IN (Highest, High) AND assignee IS EMPTY AND resolution = Unresolved
```

### Bug aging (open for more than 30 days)
```jql
issuetype = Bug AND resolution = Unresolved AND created <= -30d ORDER BY created ASC
```

### Regression bugs
```jql
issuetype = Bug AND labels = regression AND resolution = Unresolved
```

## Release Management

### Issues in a specific version
```jql
fixVersion = "v2.0.0" ORDER BY issuetype ASC, priority DESC
```

### Unresolved issues blocking a release
```jql
fixVersion = "v2.0.0" AND resolution = Unresolved ORDER BY priority DESC
```

### Issues resolved for release
```jql
fixVersion = "v2.0.0" AND resolution IS NOT EMPTY ORDER BY resolved DESC
```

### Issues affecting a version
```jql
affectedVersion = "v1.9.0" AND resolution = Unresolved
```

## Team Management

### Team workload
```jql
assignee IN membersOf("dev-team") AND resolution = Unresolved ORDER BY assignee ASC, priority DESC
```

### Unassigned work
```jql
assignee IS EMPTY AND resolution = Unresolved AND project = PROJ ORDER BY priority DESC
```

### Work completed by team this week
```jql
assignee IN membersOf("dev-team") AND resolved >= startOfWeek() ORDER BY resolved DESC
```

## Date-Based Queries

### Overdue issues
```jql
due < now() AND resolution = Unresolved ORDER BY due ASC
```

### Due this week
```jql
due >= startOfWeek() AND due <= endOfWeek() AND resolution = Unresolved ORDER BY due ASC
```

### Updated in the last hour
```jql
updated >= -1h ORDER BY updated DESC
```

### Created in the last 7 days
```jql
created >= -7d ORDER BY created DESC
```

### Not updated in 90 days (stale issues)
```jql
updated <= -90d AND resolution = Unresolved ORDER BY updated ASC
```

## Cross-Project Queries

### All high-priority work across projects
```jql
priority IN (Highest, High) AND resolution = Unresolved ORDER BY priority DESC, project ASC
```

### Issues from multiple projects
```jql
project IN ("FRONTEND", "BACKEND", "INFRA") AND status = "In Progress" ORDER BY project ASC
```

### Global text search
```jql
text ~ "authentication" AND resolution = Unresolved ORDER BY updated DESC
```

## Advanced Queries

### Issues with no story points
```jql
issuetype = Story AND "Story Points" IS EMPTY AND resolution = Unresolved
```

### Issues transitioned to Done this month
```jql
status changed to "Done" DURING (startOfMonth(), now()) ORDER BY updated DESC
```

### Issues that were reopened
```jql
status WAS "Done" AND status != "Done" AND resolution = Unresolved
```

### Issues with more than 5 comments
```jql
issueFunction IN commented("comments >= 5") ORDER BY updated DESC
```

### Epics with incomplete children
```jql
issuetype = Epic AND issueFunction IN hasLinks("is parent of") AND statusCategory != Done
```

## Executing Queries via API

```bash
# Using POST (recommended for long queries)
curl -X POST \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search/jql" \
  -d '{
    "jql": "project = PROJ AND sprint in openSprints() ORDER BY rank ASC",
    "startAt": 0,
    "maxResults": 50,
    "fields": ["summary", "status", "assignee", "priority", "story_points"]
  }'

# Using GET (for short queries)
curl -G \
  -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/search/jql" \
  --data-urlencode "jql=assignee = currentUser() AND resolution = Unresolved" \
  --data-urlencode "maxResults=20"
```
