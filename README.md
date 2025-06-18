# 📘 OAuth 2.0 API Documentation

Base URL: https://id.withnlabs.com

All exchange endpoints are prefixed with `/api/oauth`

Example TypeScript SDK Snippet: https://cdn.withnlabs.com/assets/sdk/withn-id-oauth-sdk-snippet.ts.txt

OAuth2 Authorization URL e.g. https://id.withnlabs.com/oauth/authorize?response_type=code&client_id=6851244039c2be8c0bf80f34&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcallback&scope=email+basic&state=random

---

## Authorization Endpoint

**`GET /oauth/authorize`**

User interface to allow or reject an OAuth2 request.

### ✅ Request

```http
GET /oauth/authorize
```

#### Query String Parameters

| Field           | Required | Type   | Description                          |
| --------------- | -------- | ------ | ------------------------------------ |
| `response_type` | ✅       | string | Must be `"code"`                     |
| `client_id`     | ✅       | string | Application’s client ID              |
| `client_secret` | ✅       | string | Application’s client secret          |
| `scope`         | ✅       | string | Application's scopes: basic, email   |
| `redirect_uri`  | ✅       | string | Application's Redirect Uri           |
| `state`         | ❌       | string | Application’s client secret          |

## 🔐 Token Endpoint

**`POST /api/oauth/token`**

Exchanges an authorization code for an access token (and optionally a refresh token).

### ✅ Request

`Content-Type: application/x-www-form-urlencoded`

```http
POST /api/oauth/token
```

#### Body Parameters

| Field           | Required | Type   | Description                          |
| --------------- | -------- | ------ | ------------------------------------ |
| `grant_type`    | ✅       | string | Must be `"authorization_code"`       |
| `code`          | ✅       | string | The authorization code received      |
| `redirect_uri`  | ✅       | string | Must match the URI used in auth step |
| `client_id`     | ✅       | string | Application’s client ID              |
| `client_secret` | ✅       | string | Application’s client secret          |

### ✅ Success Response

```json
Status: 200 OK
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rfh...",
  "scope": "basic email"
}
```

### ❌ Error Responses

```json
Status: 400 Bad Request / 401 Unauthorized / 404 Not Found
{
  "error": "...",
  "error_description": "..."
}
```

## 🔄 Token Refresh Endpoint

**`POST /api/oauth/token/refresh`**

Exchanges an refresh token for an access token (max 2 times).

### ✅ Request

`Content-Type: application/x-www-form-urlencoded`

```http
POST /api/oauth/token/refresh
```

#### Body Parameters

| Field           | Required | Type   | Description                          |
| --------------- | -------- | ------ | ------------------------------------ |
| `grant_type`    | ✅       | string | Must be `"refresh_token"`            |
| `token`         | ✅       | string | The refresh token received           |
| `redirect_uri`  | ✅       | string | Must match the URI used in auth step |
| `client_id`     | ✅       | string | Application’s client ID              |
| `client_secret` | ✅       | string | Application’s client secret          |

### ✅ Success Response

```json
Status: 200 OK
{
  "access_token": "...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rfhl...",
  "scope": "basic email"
}
```

### ❌ Error Responses

```json
Status: 400 Bad Request / 401 Unauthorized / 403 Forbidden / 404 Not Found
{
  "error": "...",
  "error_description": "..."
}
```

---

## 🚫 Token Revocation Endpoint

**`POST /api/oauth/token/revoke`**

Revokes an access or refresh token to invalidate its use.

### ✅ Request

`Content-Type: application/x-www-form-urlencoded`

```http
POST /api/oauth/token/revoke
```

#### Body Parameters

| Field             | Required | Type   | Description                                         |
| ----------------- | -------- | ------ | --------------------------------------------------- |
| `token`           | ✅       | string | The access or refresh token to be revoked           |
| `token_type_hint` | ❌       | string | `"access_token"` or `"refresh_token"` (a hint only) |
| `client_id`       | ✅       | string | Application’s client ID                             |
| `client_secret`   | ✅       | string | Application’s client secret                         |

### ✅ Success Response

Status: 200 OK

### ❌ Error Responses

```json
Status: 400 Bad Request / 401 Unauthorized
{
  "error": "...",
  "error_description": "..."
}
```

---

## 👤 User Info Endpoint

**`GET /api/oauth/userinfo`**

Returns user profile information for a valid access token.

### 🔐 Auth Required

`Authorization: Bearer <access_token>`

### ✅ Response

```json
Status: 200 OK
{
  "sub": "1234567890",
  "name": "Sky Walker",
  "username": "skydev",
  "picture": "https://example.com/avatar.jpg",
  "email": "user@example.com"
}
```

### ❌ Error Response

```json
Status: 400 Bad Request / 401 Unauthorized / 404 Not Found
{
  "error": "...",
  "error_description": "..."
}
```

---

## 🧠 Notes

- All responses follow standard OAuth 2.0 semantics.
- `client_id` and `client_secret` are required for confidential clients.
- Use HTTPS for all communication. Never expose tokens in URLs.
- Access tokens are short-lived. Refresh tokens should be stored securely.
- Revoke tokens on logout or session expiration.
- Refresh token can only be used 2 times. (1. rfh-xxx, 2. rfhl-xxx, 3. undefined)
