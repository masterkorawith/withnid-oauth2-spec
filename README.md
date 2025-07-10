# 📘 OAuth 2.0 & OpenID Connect API Documentation

Base URL: https://id.withnlabs.com

OAuth2 Authorization URL e.g. https://id.withnlabs.com/oauth/authorize?response_type=code&client_id=6851244039c2be8c0bf80f34&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fcallback&scope=openid+profile+email&state=random

## Authorization Endpoint

**`GET /oauth/authorize`**

User interface to allow or reject an OAuth2/OpenID Connect request.

### ✅ Request

```http
GET /oauth/authorize
```

#### Query String Parameters

| Field                   | Required | Type   | Description                                             |
| ----------------------- | -------- | ------ | ------------------------------------------------------- |
| `response_type`         | ✅       | string | Must be `"code"`                                        |
| `client_id`             | ✅       | string | Application's client ID                                 |
| `scope`                 | ✅       | string | Space-separated scopes: openid, profile, email, isadmin |
| `redirect_uri`          | ✅       | string | Application's Redirect Uri                              |
| `state`                 | ❌       | string | State from user's application                           |
| `nonce`                 | ❌       | string | OpenID Connect nonce for ID token                       |
| `code_challenge`        | ❌       | string | PKCE code challenge                                     |
| `code_challenge_method` | ❌       | string | PKCE method: `"S256"` or `"plain"`                      |

### Supported Scopes

| Scope            | Description                                             |
| ---------------- | ------------------------------------------------------- |
| `openid`         | Required for OpenID Connect, enables ID token           |
| `profile`        | Access to profile information (name, username, picture) |
| `email`          | Access to email address                                 |
| `offline_access` | Enables refresh token issuance                          |
| `isadmin`        | Access to administrator status                          |

## 🔍 OpenID Connect Discovery

**`GET /.well-known/openid_configuration`**

Returns the OpenID Connect configuration document with all supported endpoints and capabilities.

### ✅ Response

```json
Status: 200 OK
{
  "issuer": "https://id.withnlabs.com",
  "authorization_endpoint": "https://id.withnlabs.com/oauth/authorize",
  "token_endpoint": "https://id.withnlabs.com/api/oauth/token",
  "userinfo_endpoint": "https://id.withnlabs.com/api/oauth/userinfo",
  "jwks_uri": "https://id.withnlabs.com/api/oauth/jwks",
  "response_types_supported": ["code", "id_token", "code id_token"],
  "scopes_supported": ["openid", "profile", "email", "offline_access", "isadmin"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "token_endpoint_auth_methods_supported": ["client_secret_basic", "client_secret_post"]
}
```

## 🔑 JSON Web Key Set (JWKS)

**`GET /api/oauth/jwks`**

Returns the JSON Web Key Set used for verifying JWT tokens.

### ✅ Response

```json
Status: 200 OK
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "alg": "RS256",
      "kid": "withn-identity-key-1",
      "n": "...",
      "e": "AQAB"
    }
  ]
}
```

---

## Authorization Endpoint

**`GET /oauth/authorize`**

User interface to allow or reject an OAuth2 request.

### ✅ Request

```http
GET /oauth/authorize
```

#### Query String Parameters

| Field           | Required | Type   | Description                                  |
| --------------- | -------- | ------ | -------------------------------------------- |
| `response_type` | ✅       | string | Must be `"code"`                             |
| `client_id`     | ✅       | string | Application’s client ID                      |
| `scope`         | ✅       | string | Application's scopes: profile, email, openid |
| `redirect_uri`  | ✅       | string | Application's Redirect Uri                   |
| `state`         | ❌       | string | State from user's application                |

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
  "scope": "profile email"
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
  "scope": "profile email"
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

#### OpenID Connect Response (includes additional claims based on scopes)

```json
Status: 200 OK
{
  "sub": "1234567890",
  "name": "Sky Walker",
  "preferred_username": "skydev",
  "nickname": "skydev",
  "picture": "https://cdn.withnlabs.com/id/avatars/avatar.png",
  "email": "user@example.com",
  "email_verified": true,
  "updated_at": 1641024000,
  "administrator": false
}
```

### ❌ Error Response

```json
Status: 400 Bad Request / 401 Unauthorized / 404 Not Found
{
  "error": "invalid_token",
  "error_description": "The access token is invalid or expired"
}
```

---

## 🔍 Token Introspection Endpoint

**`POST /api/oauth/introspect`**

Returns metadata about an access token (RFC 7662).

### ✅ Request

`Content-Type: application/x-www-form-urlencoded`

```http
POST /api/oauth/introspect
```

#### Body Parameters

| Field           | Required | Type   | Description                 |
| --------------- | -------- | ------ | --------------------------- |
| `token`         | ✅       | string | The token to introspect     |
| `client_id`     | ✅       | string | Application's client ID     |
| `client_secret` | ✅       | string | Application's client secret |

### ✅ Success Response

#### Active Token

```json
Status: 200 OK
{
  "active": true,
  "sub": "1234567890",
  "client_id": "your-client-id",
  "username": "skydev",
  "scope": "openid profile email",
  "exp": 1641027600,
  "iat": 1641024000,
  "token_type": "Bearer",
  "aud": "your-client-id",
  "iss": "https://id.withnlabs.com"
}
```

#### Inactive Token

```json
Status: 200 OK
{
  "active": false
}
```

---

## 🚪 End Session Endpoint (OpenID Connect)

**`GET /oauth/logout`**

Initiates logout from the OpenID Connect provider.

### ✅ Request

```http
GET /oauth/logout
```

#### Query Parameters

| Field                      | Required | Type   | Description                         |
| -------------------------- | -------- | ------ | ----------------------------------- |
| `post_logout_redirect_uri` | ❌       | string | URI to redirect to after logout     |
| `state`                    | ❌       | string | State parameter to maintain         |
| `id_token_hint`            | ❌       | string | ID token hint for logout validation |

### ✅ Response

Redirects to the specified `post_logout_redirect_uri` or login page.

---

## 📊 Claims Reference

### Standard OpenID Connect Claims

| Claim                | Scope     | Description                    |
| -------------------- | --------- | ------------------------------ |
| `sub`                | `openid`  | Subject identifier (user ID)   |
| `name`               | `profile` | Full name                      |
| `preferred_username` | `profile` | Preferred username             |
| `nickname`           | `profile` | Nickname                       |
| `picture`            | `profile` | Profile picture URL            |
| `email`              | `email`   | Email address                  |
| `email_verified`     | `email`   | Email verification status      |
| `updated_at`         | `profile` | Profile last updated timestamp |

### Custom Claims

| Claim           | Scope     | Description          |
| --------------- | --------- | -------------------- |
| `administrator` | `isadmin` | Administrator status |

---

## 🔐 Security Features

### JWT Tokens

- **Algorithm**: RS256 (RSA with SHA-256)
- **Key Size**: 2048-bit RSA keys
- **Key Rotation**: Supported via JWKS endpoint
- **Validation**: Use JWKS endpoint for public key verification

### PKCE Support

- **Methods**: `S256` (recommended), `plain`
- **Parameters**: `code_challenge`, `code_challenge_method`, `code_verifier`
- **Usage**: Recommended for all public clients

### Token Security

- **Access Token Lifetime**: 1 hour
- **Refresh Token Lifetime**: 7 days
- **ID Token Lifetime**: 1 hour
- **Nonce Support**: Prevents replay attacks in ID tokens

---

## 🧠 Notes

### OAuth 2.0 & OpenID Connect Compliance

- Follows OAuth 2.0 (RFC 6749) and OpenID Connect Core 1.0 specifications
- Supports PKCE (RFC 7636) for enhanced security
- JWT tokens use RS256 algorithm for signing
- Full OpenID Connect Discovery support

### Security Best Practices

- `client_id` and `client_secret` are required for confidential clients
- Use HTTPS for all communication. Never expose tokens in URLs
- Access tokens are short-lived (1 hour). Refresh tokens should be stored securely
- Revoke tokens on logout or session expiration
- Use PKCE for public clients (native apps, SPAs)
- Validate ID tokens using the JWKS endpoint

### Token Lifetimes

- **Access Token**: 1 hour (JWT format)
- **ID Token**: 1 hour (JWT format, OpenID Connect only)
- **Refresh Token**: 7 days (opaque format)
- **Authorization Code**: 10 minutes

### Integration Notes

- Use the OpenID Connect Discovery endpoint (`/.well-known/openid_configuration`) for automatic configuration
- Validate JWT tokens using the JWKS endpoint (`/api/oauth/jwks`)
- Support both OAuth 2.0 and OpenID Connect flows
- Refresh tokens can be used multiple times until expiration
