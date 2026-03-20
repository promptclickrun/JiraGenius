---
name: authentication
description: "This skill covers all authentication methods for the Jira Cloud REST API."
---

# Authentication Skill

This skill covers all authentication methods for the Jira Cloud REST API.

## API Token Authentication (Recommended for Scripts)

Generate a token at: https://id.atlassian.com/manage-profile/security/api-tokens

### Basic Auth with API Token

Jira Cloud uses your email and API token combined as Basic authentication:

```bash
# Using curl with inline encoding
curl -X GET \
  -H "Authorization: Basic $(echo -n 'your-email@example.com:your-api-token' | base64)" \
  -H "Accept: application/json" \
  "https://your-domain.atlassian.net/rest/api/3/myself"

# Using curl's built-in basic auth
curl -u "your-email@example.com:your-api-token" \
  -H "Accept: application/json" \
  "https://your-domain.atlassian.net/rest/api/3/myself"
```

### Environment Variables

```bash
export JIRA_BASE_URL="https://your-domain.atlassian.net"
export JIRA_USER_EMAIL="your-email@example.com"
export JIRA_API_TOKEN="your-api-token"

# Then use in commands
curl -u "$JIRA_USER_EMAIL:$JIRA_API_TOKEN" \
  -H "Accept: application/json" \
  "$JIRA_BASE_URL/rest/api/3/myself"
```

## OAuth 2.0 (3LO) — For Apps and Integrations

OAuth 2.0 three-legged authorization flow for Jira Cloud:

### Step 1: Register an OAuth 2.0 App

1. Go to https://developer.atlassian.com/console/myapps/
2. Create a new app
3. Add the **Jira API** under Permissions
4. Configure Authorization with callback URL
5. Note your `Client ID` and `Client Secret`

### Step 2: Authorization URL

Redirect users to:
```
https://auth.atlassian.com/authorize
  ?audience=api.atlassian.com
  &client_id={YOUR_CLIENT_ID}
  &scope=read:jira-work write:jira-work manage:jira-project manage:jira-configuration
  &redirect_uri={YOUR_CALLBACK_URL}
  &state={RANDOM_STATE}
  &response_type=code
  &prompt=consent
```

### Step 3: Exchange Code for Token

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  "https://auth.atlassian.com/oauth/token" \
  -d '{
    "grant_type": "authorization_code",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "code": "AUTHORIZATION_CODE",
    "redirect_uri": "YOUR_CALLBACK_URL"
  }'
```

Response:
```json
{
  "access_token": "eyJ...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "refresh_token": "..."
}
```

### Step 4: Get Accessible Resources

```bash
curl -X GET \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Accept: application/json" \
  "https://api.atlassian.com/oauth/token/accessible-resources"
```

Returns the cloud ID needed for API calls:
```json
[
  {
    "id": "cloud-id-here",
    "url": "https://your-domain.atlassian.net",
    "name": "Your Site",
    "scopes": [...]
  }
]
```

### Step 5: Make API Calls

```bash
curl -X GET \
  -H "Authorization: Bearer {ACCESS_TOKEN}" \
  -H "Accept: application/json" \
  "https://api.atlassian.com/ex/jira/{CLOUD_ID}/rest/api/3/myself"
```

### Step 6: Refresh Token

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  "https://auth.atlassian.com/oauth/token" \
  -d '{
    "grant_type": "refresh_token",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "refresh_token": "YOUR_REFRESH_TOKEN"
  }'
```

## OAuth 2.0 Scopes

| Scope | Description |
|-------|-------------|
| `read:jira-work` | Read Jira project and issue data |
| `write:jira-work` | Create and edit issues, comments, worklogs |
| `manage:jira-project` | Create and edit projects, manage versions and components |
| `manage:jira-configuration` | Manage Jira settings, workflows, custom fields |
| `manage:jira-data-provider` | Manage data providers |
| `read:jira-user` | Read user information |

## Personal Access Tokens (PAT) — Data Center Only

PATs are only available in Jira Data Center, not Jira Cloud. For Cloud, use API tokens or OAuth 2.0.

## Required Headers

Every API request should include:

```
Authorization: Basic {base64(email:token)}   # or Bearer {oauth_token}
Content-Type: application/json                # for POST/PUT requests
Accept: application/json                      # for all requests
X-Atlassian-Token: no-check                   # required for file uploads only
```

## Security Best Practices

- Never commit API tokens or secrets to source control
- Use environment variables or secret managers for credentials
- Rotate API tokens regularly
- Use the minimum required OAuth scopes
- Implement token refresh for OAuth 2.0 integrations
- Use HTTPS for all API calls (enforced by Atlassian Cloud)
- Monitor API token usage in Atlassian admin console
