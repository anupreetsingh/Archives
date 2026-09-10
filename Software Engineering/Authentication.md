# Authentication

**Authentication** establishes who a user or client is. Users can authenticate with a password, a one-time code, a passkey, or an external identity provider. API clients can present credentials such as API keys or access tokens. **Authorization** then determines which resources and actions that identity is allowed to access.

The backend chooses which [endpoints](API%20Design.md#resources-and-routes) require authentication and enforces the check through **middleware, dependencies, decorators, or authentication guards**. For example, a custom decorator can protect a specific route:

```python
@app.get("/profile")
@require_auth
def get_profile():
    ...  # Return the authenticated user's profile.
```

Here, `@app.get("/profile")` registers the endpoint, while `@require_auth` checks authentication before `get_profile()` runs. Missing or invalid credentials prevent the handler from running, typically producing `401 Unauthorized` for an API or a redirect to a login page for a browser application. The decorator is conceptual; see [Django's view protection](Django%20Guide.md#protecting-views) for a concrete implementation.

A **session** is the application context associated with a client across multiple requests, such as its signed-in identity. After login, an authenticated session lets the client continue making requests without repeating the original sign-in process. Because [HTTP is stateless](HTTP.md#http-request-elements), each protected request still carries a credential that the server validates to recognize that session.

## Session

### Stateful

The client stores only an opaque session ID, usually in a cookie, while the corresponding session data (the state) remains something rapidly accessible such as an in-memory data store like **Redis**.

```mermaid
flowchart LR
    C[Client] -->|Session ID cookie| A[Application server]
    A <-->|Read or update session| R[(Redis)]
```

#### Mechanism

1. After authentication, the server creates a session in Redis and returns its ID in a `Set-Cookie` response header.
2. The browser stores the cookie and sends the session ID in the `Cookie` header of subsequent requests that match the cookie's scope and rules.
3. The server looks up the session in Redis to identify the user and retrieve session data.
4. Logout or forced revocation deletes the session from Redis.

##### HTTP Cookie Exchange

For a browser application at `https://app.example.com`, a successful login can return this response with no body. The session ID below is illustrative; the server generates an unpredictable value for each new session.

```http
HTTP/1.1 204 No Content
Set-Cookie: session_id=7e9c14a6b3804d258f621a0c9d73e5b2; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
```

The browser processes `Set-Cookie` and saves the cookie directly; frontend JavaScript cannot read this response header. On a later same-origin request, it automatically attaches the stored name and value:

```http
GET /profile HTTP/1.1
Host: app.example.com
Cookie: session_id=7e9c14a6b3804d258f621a0c9d73e5b2
```

The server extracts `session_id` from `Cookie` and uses its value to look up the Redis session. Cookie attributes such as `HttpOnly` are browser instructions from the response; they are not repeated in the request.

The attributes in this example control storage and delivery:

- `Path=/`: Applies to all paths on the host. Omitting `Domain` restricts the cookie to `app.example.com`.
- `HttpOnly`: Prevents JavaScript from reading the cookie; the browser can still send it with requests initiated by `fetch()`.
- `Secure`: Sends the cookie only over HTTPS, with a localhost exception.
- `SameSite=Lax`: Allows same-site requests and cross-site top-level navigations using safe methods, such as following a `GET` link. It excludes cross-site `fetch()` requests.
- `Max-Age=3600`: Expires the browser's cookie after one hour. The server must enforce session expiration separately, for example with a Redis TTL (time to live).

On logout, the server can also clear this browser cookie with `Set-Cookie: session_id=; Path=/; HttpOnly; Secure; SameSite=Lax; Max-Age=0`. Deletion must match the original cookie's name, domain scope, and path. Clearing the cookie alone does not revoke the Redis session.

#### Tradeoffs

1. Sessions can be revoked immediately and updated centrally.
2. Only a small, meaningless identifier is exposed to the client.
3. Every authenticated request normally requires a Redis lookup.
4. Redis becomes infrastructure that must be secured, scaled, and kept available.
5. Storage usage increases with the number and size of active sessions.

### Stateless

The client stores a signed JWT containing user claims. The server verifies the token locally instead of retrieving a session from a central store.

```mermaid
flowchart LR
    C[Client] -->|Signed JWT| A[Application server]
    A -->|Verify signature and claims| K[Signing or public key]
```

#### Mechanism

1. After authentication, the server issues a signed JWT containing claims such as the user ID, roles, and expiration time.
2. The client sends the JWT with each authenticated request.
3. The server validates its signature, expiration, issuer, intended audience, and other required claims.
4. If the token is valid, the server trusts its claims without loading a session.

##### HTTP Bearer Token Exchange

A JWT is a token format. **Bearer** describes how a token is used: whoever possesses a valid token can present it to exercise its granted access without proving possession of a separate cryptographic key. A bearer token can be a JWT or an opaque string. Bearer tokens must be kept private and transmitted over HTTPS.

A token-issuing endpoint commonly returns the JWT in a JSON response body. Here, `header.payload.signature` is a placeholder for the complete signed JWT:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
  "access_token": "header.payload.signature",
  "token_type": "Bearer",
  "expires_in": 900
}
```

`access_token` contains the token, `token_type` identifies the authentication scheme, and `expires_in` gives its lifetime in seconds. These are conventional OAuth response fields, not requirements of the JWT format. For a JWT, the server enforces expiration using the signed `exp` claim; `expires_in` informs the client when renewal is needed.

`Cache-Control: no-store` tells caches not to store this credential-bearing response. `Pragma: no-cache` provides compatibility with older HTTP caches.

Client code reads the JSON, retains the token, and explicitly adds it to subsequent API requests:

```http
GET /profile HTTP/1.1
Host: app.example.com
Authorization: Bearer header.payload.signature
```

`Bearer` is the scheme name followed by a space and the token; it is not a separate token or part of the JWT. Returning JSON does not make the browser automatically attach this header. The server extracts and validates the JWT using the checks above.

##### JWTs in Cookies

A server can also return a JWT through `Set-Cookie`, with the JWT as the cookie value. The browser then follows the same [cookie exchange](#http-cookie-exchange), sending it in `Cookie`. Using a cookie does not itself require server-side session storage: the server can still validate the JWT locally. The token format and its transport are separate choices.

#### Tradeoffs

1. Authentication scales easily across application servers because no shared session lookup is required.
2. Local verification avoids a Redis request, but the larger token is transmitted on every request.
3. Immediate revocation is difficult; a stolen token can remain valid until it expires.
4. Role or permission changes may not take effect until a new token is issued.
5. Refresh tokens, rotation, or revocation lists can improve security but reintroduce server-side state.
6. JWT payloads are encoded rather than encrypted, so they must not contain secrets.
