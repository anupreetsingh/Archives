# HTTP

HTTP(HyperText Transfer Protocol) is the protocol clients and servers use to exchange requests and responses on the web. A client sends a request with a method, URL, headers, and sometimes a body. The server returns a response with headers, usually a body, and a status code.

## HTTP Actions

HTTP actions are called **methods**. They describe what the client wants to do with a resource.

| Method | Common use |
|---|---|
| `GET` | Read or fetch a resource. |
| `POST` | Create a new resource or submit data. |
| `PUT` | Replace an entire resource. |
| `PATCH` | Update part of a resource. |
| `DELETE` | Remove a resource. |
| `HEAD` | Get response headers without the body. |
| `OPTIONS` | Ask which methods or options a route supports. |

Example:

```http
GET /users/123
POST /users
PATCH /users/123
DELETE /users/123
```

## HTTP Status Codes

Status codes tell the client what happened after the server handled the request.

| Range | Meaning |
|---|---|
| `1xx` | Informational. The request is still being processed. |
| `2xx` | Success. The request worked. |
| `3xx` | Redirection. The client needs to go somewhere else. |
| `4xx` | Client error. The request was invalid or not allowed. |
| `5xx` | Server error. The server failed while handling a valid-looking request. |

Common status codes:

| Code | Meaning |
|---|---|
| `200 OK` | Request succeeded. |
| `201 Created` | A new resource was created. |
| `204 No Content` | Request succeeded, but there is no response body. |
| `301 Moved Permanently` | Resource has a permanent new URL. |
| `302 Found` | Resource is temporarily at another URL. |
| `400 Bad Request` | Request is malformed or invalid. |
| `401 Unauthorized` | Authentication is missing or invalid. |
| `403 Forbidden` | User is authenticated but not allowed. |
| `404 Not Found` | Resource does not exist at that URL. |
| `409 Conflict` | Request conflicts with current server state. |
| `422 Unprocessable Entity` | Request format is valid, but the data fails validation. |
| `500 Internal Server Error` | Unexpected server-side failure. |
| `502 Bad Gateway` | A proxy or gateway got a bad response from an upstream server. |
| `503 Service Unavailable` | Server is temporarily unavailable or overloaded. |

## Method And Status Pairing

Common REST-style pairings:

| Request | Successful response |
|---|---|
| `GET /users/123` | `200 OK` with the user data. |
| `POST /users` | `201 Created` with the created user. |
| `PATCH /users/123` | `200 OK` with updated data or `204 No Content`. |
| `DELETE /users/123` | `204 No Content`. |
