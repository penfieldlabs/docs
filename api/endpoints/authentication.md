# Authentication

Penfield uses OAuth 2.1 for authentication, compliant with the [MCP 2025-11-25 Authorization Specification](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization).

---

## Quick Reference: Which Method?

| Use Case | Method | Complexity |
|----------|--------|------------|
| Getting started / testing | [API Key Exchange](#api-key-exchange) | Simple |
| Server-to-server automation | [API Key Exchange](#api-key-exchange) | Simple |
| CLI tool / headless device | [Device Code Flow](#device-code-flow-rfc-8628) | Medium |
| Desktop/web app with browser | [Authorization Code + PKCE](#authorization-code-pkce) | Standard |

All methods produce the same JWT tokens. Choose based on your use case.

---

## Discovery Endpoints (MCP Compliant)

Penfield implements OAuth 2.0 metadata discovery per RFC 8414 and RFC 9728.

### Authorization Server Metadata

```bash
curl https://api.penfield.app/.well-known/oauth-authorization-server
```

**Response:**
```json
{
  "issuer": "https://auth.penfield.app",
  "authorization_endpoint": "https://auth.penfield.app/oauth/authorize",
  "token_endpoint": "https://auth.penfield.app/oauth/token",
  "device_authorization_endpoint": "https://auth.penfield.app/oauth/device_authorization",
  "registration_endpoint": "https://auth.penfield.app/oauth/register",
  "token_endpoint_auth_methods_supported": ["none"],
  "response_types_supported": ["code"],
  "grant_types_supported": [
    "authorization_code",
    "client_credentials",
    "refresh_token",
    "urn:ietf:params:oauth:grant-type:device_code"
  ],
  "code_challenge_methods_supported": ["S256"],
  "scopes_supported": ["read", "write", "profile", "offline_access"],
  "response_modes_supported": ["query"],
  "authorization_response_iss_parameter_supported": true,
  "service_documentation": "https://www.penfield.app/docs",
  "dpop_signing_alg_values_supported": ["RS256", "ES256"]
}
```

> **Note on `token_endpoint_auth_methods_supported: ["none"]`:** This is standard OAuth 2.1 for public clients. The value `"none"` means no client secret is required at the token endpoint. Security is provided by PKCE (Proof Key for Code Exchange) - the `code_verifier` proves client identity without a shared secret. This is the recommended approach for native apps, SPAs, and CLI tools per OAuth 2.1 and RFC 7636.

### Protected Resource Metadata (RFC 9728)

```bash
curl https://api.penfield.app/.well-known/oauth-protected-resource
```

**Response:**
```json
{
  "resource": "https://auth.penfield.app",
  "authorization_servers": ["https://auth.penfield.app"],
  "resource_name": "Penfield Memory System",
  "scopes_supported": ["read", "write", "openid", "profile", "offline_access"],
  "bearer_methods_supported": ["header"],
  "resource_signing_alg_values_supported": ["HS256"],
  "resource_documentation": "https://github.com/penfieldlabs/docs",
  "resource_policy_uri": "https://portal.penfield.app/privacy",
  "resource_tos_uri": "https://portal.penfield.app/terms"
}
```

**Key Point:** Always discover endpoints dynamically. Never hardcode OAuth URLs.

---

## Scopes

Discover available scopes from the metadata endpoints:

```bash
# Get scopes from authorization server metadata
curl -s https://api.penfield.app/.well-known/oauth-authorization-server | \
  python3 -c "import sys,json; print(json.load(sys.stdin)['scopes_supported'])"
```

**Scopes:**

| Scope | Description |
|-------|-------------|
| `read` | Read memories, search |
| `write` | Create, update, delete memories |
| `profile` | Access user profile information |
| `offline_access` | Receive refresh tokens |

**Note:** Always check the discovery endpoint for the current list of supported scopes. The Protected Resource Metadata may list additional scopes like `openid`.

---

## API Key Exchange

The simplest authentication method. Exchange an API key for JWT tokens.

**When to use:**
- Getting started quickly
- Server-to-server integrations
- Scripts and automation

### Exchange API Key for JWT

```
POST https://api.penfield.app/api/v2/auth/token
```

```bash
curl -X POST https://api.penfield.app/api/v2/auth/token \
-H "Authorization: Bearer tm_your_tenant_ak_your_key" \
-H "Content-Type: application/json"
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 259200,
    "tenant_id": "pf_...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
  },
  "meta": {
    "timestamp": "2026-01-30T12:00:00Z",
    "version": "2.0.0",
    "request_id": "..."
  }
}
```

> **Note:** `tenant_id` is your account/workspace identifier. All memories and data are scoped to this tenant.

**Note:** The response includes a `refresh_token`. Use it to obtain new tokens without re-exchanging your API key. See [Refresh Tokens](#refresh-tokens).

### Verify Token

```
GET https://api.penfield.app/api/v2/auth/verify
```

```bash
curl -X GET https://api.penfield.app/api/v2/auth/verify \
-H "Authorization: Bearer $JWT_TOKEN"
```

---

## Device Code Flow (RFC 8628)

For CLI tools, headless applications, and devices without a browser.

**When to use:**
- CLI tools
- IoT devices
- Any headless environment

**Benefits:**
- Supports refresh tokens with `offline_access` scope
- User authorizes on a separate device
- Follows RFC 8628 standard

### Step 1: Discover Endpoints

```bash
DISCOVERY=$(curl -s https://api.penfield.app/.well-known/oauth-authorization-server)
DEVICE_AUTH_URL=$(echo $DISCOVERY | python3 -c "import sys,json; print(json.load(sys.stdin)['device_authorization_endpoint'])")
TOKEN_URL=$(echo $DISCOVERY | python3 -c "import sys,json; print(json.load(sys.stdin)['token_endpoint'])")
```

### Step 2: Register Client (Optional)

If you don't have a `client_id`, use Dynamic Client Registration:

```bash
REG_URL=$(echo $DISCOVERY | python3 -c "import sys,json; print(json.load(sys.stdin)['registration_endpoint'])")

curl -X POST "$REG_URL" \
-H "Content-Type: application/json" \
-d '{
  "client_name": "my-cli-tool",
  "redirect_uris": ["http://localhost:8080/callback"],
  "grant_types": ["urn:ietf:params:oauth:grant-type:device_code", "refresh_token"],
  "token_endpoint_auth_method": "none",
  "scope": "read write offline_access"
}'
```

**Response:**
```json
{
  "client_id": "dyn_abc123...",
  "client_id_issued_at": 1704542400
}
```

### Step 3: Request Device Code

```bash
curl -X POST "$DEVICE_AUTH_URL" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "client_id=YOUR_CLIENT_ID" \
-d "scope=read write offline_access"
```

**Response:**
```json
{
  "device_code": "GmRhmhcxhwAzkoRubiFzegLbSwE4SBAAqEOwmI2NODBxINzONPUBAQl7PUPSDIw",
  "user_code": "WDJB-MJHT",
  "verification_uri": "https://portal.penfield.app/device",
  "verification_uri_complete": "https://portal.penfield.app/device?user_code=WDJB-MJHT",
  "expires_in": 900,
  "interval": 5
}
```

### Step 4: User Authorization

Direct the user to: `https://portal.penfield.app/device?user_code=WDJB-MJHT`

### Step 5: Poll for Token

Poll until user authorizes:

```bash
curl -X POST "$TOKEN_URL" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=urn:ietf:params:oauth:grant-type:device_code" \
-d "device_code=YOUR_DEVICE_CODE" \
-d "client_id=YOUR_CLIENT_ID"
```

**Polling Responses:**

| Error | Action |
|-------|--------|
| `authorization_pending` | Continue polling at `interval` seconds |
| `slow_down` | Increase poll interval by 5 seconds |
| `access_denied` | User denied - stop polling |
| `expired_token` | Device code expired - restart flow |

**Success Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 259200,
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "scope": "read write offline_access"
}
```

**Note:** `refresh_token` is only included if `offline_access` scope was requested. Check `expires_in` for actual token lifetime.

---

## Authorization Code + PKCE

For desktop/web applications with browser redirect capability.

**When to use:**
- Desktop applications
- Web applications
- Any app that can open a browser and handle redirects

### Step 1: Discover Endpoints

```bash
DISCOVERY=$(curl -s https://api.penfield.app/.well-known/oauth-authorization-server)
AUTH_URL=$(echo $DISCOVERY | python3 -c "import sys,json; print(json.load(sys.stdin)['authorization_endpoint'])")
TOKEN_URL=$(echo $DISCOVERY | python3 -c "import sys,json; print(json.load(sys.stdin)['token_endpoint'])")
```

### Step 2: Generate PKCE Parameters

```bash
# Generate code_verifier (43-128 characters, URL-safe)
CODE_VERIFIER=$(openssl rand -hex 32)

# Generate code_challenge (S256)
CODE_CHALLENGE=$(printf '%s' "$CODE_VERIFIER" | openssl dgst -sha256 -binary | openssl base64 -A | tr '+/' '-_' | tr -d '=')

# Generate state
STATE=$(openssl rand -hex 16)
```

### Step 3: Authorization Request

Open browser to:

```
$AUTH_URL?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=http://localhost:8080/callback&scope=read+write+offline_access&code_challenge=$CODE_CHALLENGE&code_challenge_method=S256&state=$STATE
```

**MCP Compliance:** Include the `resource` parameter to bind tokens to the target:

```
&resource=https://api.penfield.app
```

### Step 4: Handle Callback

After user authorizes, browser redirects to:
```
http://localhost:8080/callback?code=AUTH_CODE&state=YOUR_STATE
```

**Always verify `state` matches your original value.**

### Step 5: Exchange Code for Tokens

```bash
curl -X POST "$TOKEN_URL" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=authorization_code" \
-d "code=AUTH_CODE" \
-d "client_id=YOUR_CLIENT_ID" \
-d "redirect_uri=http://localhost:8080/callback" \
-d "code_verifier=$CODE_VERIFIER"
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 259200,
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "scope": "read write offline_access"
}
```

---

## Refresh Tokens

Refresh tokens allow obtaining new access tokens without re-authentication.

**Requirements:**
- OAuth flows: Request `offline_access` scope during initial authorization
- API Key Exchange: Returns refresh tokens automatically

### Token Rotation (RFC 9700)

Penfield implements refresh token rotation. Each refresh returns a **new** refresh token and invalidates the old one.

**Always store and use the latest refresh token.**

### Refresh Request (API Key Tokens)

For tokens obtained via API Key Exchange, use the API refresh endpoint:

```bash
curl -X POST https://api.penfield.app/api/v2/auth/refresh \
-H "Content-Type: application/json" \
-d '{"refresh_token": "YOUR_REFRESH_TOKEN"}'
```

**Response:**
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 259200,
    "tenant_id": "pf_...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
  },
  "meta": {...}
}
```

### Refresh Request (OAuth Tokens)

For tokens obtained via Device Code or Authorization Code flows, use the OAuth token endpoint:

```bash
TOKEN_URL=$(curl -s https://api.penfield.app/.well-known/oauth-authorization-server | python3 -c "import sys,json; print(json.load(sys.stdin)['token_endpoint'])")

curl -X POST "$TOKEN_URL" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=refresh_token" \
-d "refresh_token=YOUR_REFRESH_TOKEN" \
-d "client_id=YOUR_CLIENT_ID"
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 259200,
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "scope": "read write offline_access"
}
```

---

## Token Lifetimes

Token lifetimes are configured by Penfield and may vary by environment.

**To check your actual token lifetime:**

1. **Decode the JWT** (access token):
   ```bash
   # Decode the payload (middle section between dots)
   echo "YOUR_ACCESS_TOKEN" | cut -d'.' -f2 | base64 -d | python3 -m json.tool
   ```

2. **Check the `exp` claim** - expiration timestamp (Unix epoch)
3. **Or use the `expires_in` value** from the token response (seconds until expiration)

**Typical values** (subject to change):

| Token | Typical Lifetime |
|-------|-----------------|
| Access token | 24-72 hours |
| Refresh token | 7-30 days |
| Device code | 15 minutes |

> **Important:** Do not hardcode these values. Always check `expires_in` in the response or decode the JWT to get the actual expiration.

---

## Using Tokens

Include the access token in the `Authorization` header:

```bash
curl -X GET https://api.penfield.app/api/v2/memories \
-H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Never include tokens in URLs or query parameters.**

---

## Error Handling

| Status | Error | Description |
|--------|-------|-------------|
| 401 | `invalid_token` | Token expired or invalid |
| 401 | `invalid_credentials` | Bad API key |
| 400 | `invalid_grant` | Invalid authorization code or refresh token |
| 400 | `invalid_request` | Missing required parameters |

### Token Refresh Example

```python
def make_request(url, token, refresh_token, client_id):
    response = requests.get(url, headers={"Authorization": f"Bearer {token}"})

    if response.status_code == 401:
        # Token expired, refresh it
        new_tokens = refresh_access_token(refresh_token, client_id)
        token = new_tokens["access_token"]
        refresh_token = new_tokens["refresh_token"]  # Save the new refresh token!

        # Retry request
        response = requests.get(url, headers={"Authorization": f"Bearer {token}"})

    return response, token, refresh_token
```

---

## Environments

| Environment | API URL | Auth Server | Portal |
|-------------|---------|-------------|--------|
| Production | `api.penfield.app` | `auth.penfield.app` | `portal.penfield.app` |
| Development | `api-dev.penfield.app` | `auth-dev.penfield.app` | `portal-dev.penfield.app` |

**Important:** Always use the same environment consistently. Discovery from `api.penfield.app` returns `auth.penfield.app` endpoints.

---

## Security Best Practices

1. **Never log tokens** - Truncate in logs: `token[:10]...`
2. **Store tokens securely** - Use encrypted storage, not plain files
3. **Refresh proactively** - Refresh tokens before they expire (e.g., 5 min buffer)
4. **Validate state** - Always verify the `state` parameter in PKCE flow
5. **Use HTTPS only** - All endpoints require HTTPS
6. **Rotate refresh tokens** - Always use the newest refresh token
