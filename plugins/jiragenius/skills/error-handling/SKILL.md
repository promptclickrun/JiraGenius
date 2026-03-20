---
name: error-handling
description: "This skill covers HTTP error codes, error response formats, and troubleshooting strategies for the Jira Cloud REST API."
---

# Error Handling Skill

This skill covers HTTP error codes, error response formats, and troubleshooting strategies for the Jira Cloud REST API.

## HTTP Status Codes

### Success Codes

| Code | Meaning | Typical Use |
|------|---------|-------------|
| `200` | OK | Successful GET, PUT |
| `201` | Created | Successful POST (resource created) |
| `204` | No Content | Successful DELETE or PUT with no response body |

### Client Error Codes

| Code | Meaning | Common Cause |
|------|---------|--------------|
| `400` | Bad Request | Malformed JSON, invalid field values, missing required fields |
| `401` | Unauthorized | Invalid or missing credentials |
| `403` | Forbidden | Valid credentials but insufficient permissions |
| `404` | Not Found | Resource does not exist or user lacks browse permission |
| `405` | Method Not Allowed | HTTP method not supported for this endpoint |
| `409` | Conflict | Resource already exists or version conflict |
| `413` | Content Too Large | Request body exceeds size limit |
| `422` | Unprocessable Entity | Semantically invalid request |
| `429` | Too Many Requests | Rate limit exceeded |

### Server Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| `500` | Internal Server Error | Retry after delay; report if persistent |
| `502` | Bad Gateway | Temporary; retry after delay |
| `503` | Service Unavailable | Jira is under maintenance or overloaded |
| `504` | Gateway Timeout | Request took too long; simplify query or retry |

## Error Response Format

Jira Cloud returns errors in this JSON format:

```json
{
  "errorMessages": [
    "Issue does not exist or you do not have permission to see it."
  ],
  "errors": {
    "summary": "You must specify a summary of the issue.",
    "customfield_10001": "This field is required."
  }
}
```

- `errorMessages` — Array of general error messages
- `errors` — Object mapping field names to field-specific errors

## Common Errors and Solutions

### 400 Bad Request

**Invalid ADF (Atlassian Document Format):**
```json
{
  "errorMessages": [],
  "errors": {
    "description": "Operation value must be a valid Atlassian Document"
  }
}
```
**Solution:** Ensure description/comment uses proper ADF structure with `type: "doc"`, `version: 1`, and valid content nodes.

**Missing required field:**
```json
{
  "errorMessages": [],
  "errors": {
    "project": "project is required",
    "issuetype": "issuetype is required"
  }
}
```
**Solution:** Include all required fields. At minimum: `project`, `issuetype`, and `summary`.

**Invalid field value:**
```json
{
  "errorMessages": [],
  "errors": {
    "priority": "Priority name 'Urgent' is not valid"
  }
}
```
**Solution:** Use `GET /rest/api/3/priority` to list valid priority names.

### 401 Unauthorized

```json
{
  "message": "Client must be authenticated to access this resource."
}
```

**Troubleshooting:**
1. Verify API token is correct and not expired
2. Ensure email address matches the Atlassian account
3. Check Basic auth header is properly base64-encoded
4. For OAuth, verify access token is not expired; refresh if needed
5. Confirm the Atlassian site URL is correct

### 403 Forbidden

```json
{
  "errorMessages": [
    "You do not have the permission to see the specified issue."
  ],
  "errors": {}
}
```

**Troubleshooting:**
1. Check user permissions with `GET /rest/api/3/mypermissions?projectKey=PROJ`
2. Verify user has the required project role
3. Check issue security level restrictions
4. Confirm permission scheme grants for the operation

### 404 Not Found

```json
{
  "errorMessages": [
    "Issue does not exist or you do not have permission to see it."
  ],
  "errors": {}
}
```

**Troubleshooting:**
1. Verify the issue key, project key, or resource ID is correct
2. Check that the user has `BROWSE_PROJECTS` permission
3. Confirm the resource has not been deleted
4. Note: Jira returns 404 instead of 403 to avoid leaking existence information

### 409 Conflict

```json
{
  "errorMessages": [
    "A project with that name already exists."
  ],
  "errors": {}
}
```

**Solution:** Choose a unique name/key, or update the existing resource instead.

### 429 Too Many Requests

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

**Solution:** See the Rate Limiting skill. Implement exponential backoff.

## Field Validation Errors

When creating or editing issues, Jira validates fields against:

1. **Field existence** — The field must exist in the project's field configuration
2. **Field type** — Values must match the field's expected format
3. **Required fields** — All required fields in the field configuration must be provided
4. **Allowed values** — Select fields must use valid option IDs or names
5. **Permission** — User must have permission to set certain fields

### Get Create Metadata

Use the create metadata endpoint to discover valid fields and values:

```bash
curl -X GET \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Accept: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/issue/createmeta?projectKeys=PROJ&issuetypeNames=Task&expand=projects.issuetypes.fields"
```

### Get Edit Metadata

```bash
curl -X GET \
  -H "Authorization: Basic $(echo -n '$JIRA_USER_EMAIL:$JIRA_API_TOKEN' | base64)" \
  -H "Accept: application/json" \
  "https://{domain}.atlassian.net/rest/api/3/issue/PROJ-123/editmeta"
```

## Retry Strategy

For transient errors (429, 500, 502, 503, 504):

1. Read the `Retry-After` header if present
2. Otherwise, use exponential backoff: wait `min(2^attempt * 1s, 60s)`
3. Add random jitter: `wait_time * (0.5 + random(0, 0.5))`
4. Maximum 5 retry attempts
5. Log failures for debugging

## Debugging Tips

- Enable verbose output in curl with `-v` to see request/response headers
- Check `X-RateLimit-Remaining` response header for rate limit status
- Use `GET /rest/api/3/serverInfo` to verify connectivity
- Validate JSON payloads with a linter before sending
- Test with minimal payloads first, then add optional fields
- Check Jira's status page for outages: https://status.atlassian.com
