# HTTP

HTTP (Hypertext Transfer Protocol) was originally designed to transfer hypertext on the Web, with hypertext primarily referring to HTML documents containing hyperlinks.

Nowadays, HTTP is an **application-layer** protocol used by clients and servers to exchange requests and responses on the Web. The protocol dictactes how an HTTP message is structured and how its bytes are interpreted

Depending on the version of HTTP being used, the way the data is structured and transmitted may differ. For HTTP/1.1:

- Client send an HTTP request which contains a request line, headers, and optionally a body.
- The server returns an HTTP response containing a status code, headers, and optionally a body.

## HTTP Request Elements

An **HTTP request** is the message a client sends to a server asking to act on a resource. It is self-contained: HTTP keeps no memory of earlier messages, so every request carries everything the server needs to handle it on its own.

Each request answers four questions:

| Question | Element |
|---|---|
| What action? | The **method** — `GET`, `POST`, `DELETE`, and so on. |
| On which resource? | The **request target** — a path, optionally followed by a query string. |
| Under what conditions? | The **headers** — body format, host, credentials, caching rules. |
| With what data? | The **body** — optional, and only used by methods that send data. |

### How the Client Creates a Request

A request is never typed out by hand. Any language — Python, JavaScript, Java — builds one through an HTTP library, passing the pieces as ordinary values. A client ordering two units of product `123` might write:

```python
import requests

requests.post(
    "https://api.example.com/orders",
    params={"status": "pending"},
    json={"productId": 123, "quantity": 2},
    headers={"Authorization": "Bearer abc123"},
)
```

The library assembles the message from those arguments. It splits the URL into a host to connect to and a target to write inside the message, encodes `params` into the query string, serializes the `json` dictionary into bytes, and adds `Host`, `Content-Type`, and `Content-Length` without being asked — only `Authorization` is passed through as given.

What it puts on the connection is no longer a Python object. It is a block of text with every piece written in a fixed order.

### Request Structure

That order is the **structure** of an HTTP/1.1 request: the request line, headers, a blank line, and an optional body. The request line and headers are text, while the body may be text or binary. For HTTPS, TLS encrypts all of it before it travels over the network.

The call above becomes:

```http
POST /orders?status=pending HTTP/1.1
Host: api.example.com
Content-Type: application/json
Content-Length: 30
Authorization: Bearer abc123

{"productId":123,"quantity":2}
```

1. **Request line:** `POST` is the [method](#methods), `/orders?status=pending` is the request target, and `HTTP/1.1` is the protocol version. This single line states the action and the resource.
2. **Headers:** Describe the message. `Host` names the domain being addressed, `Content-Type` declares the body format, `Content-Length` gives the body's size in bytes, and `Authorization` carries the credentials used to authenticate the client.
3. **Body:** Contains the data being sent — here, the order to create. A blank line separates it from the headers.

This text layout is specifically HTTP/1.1 message syntax. HTTP/2 and HTTP/3 carry the same elements using different framing.

### Methods

The method is the first token of the request line. It states what the client wants done with the resource.

| Method | Common use | Safe | Idempotent |
|---|---|---|---|
| `GET` | Read or fetch a resource. | ✓ | ✓ |
| `POST` | Create a new resource or submit data. | ✗ | ✗ |
| `PUT` | Replace an entire resource. | ✗ | ✓ |
| `PATCH` | Update part of a resource. | ✗ | ✗ |
| `DELETE` | Remove a resource. | ✗ | ✓ |
| `HEAD` | Get response headers without the body. | ✓ | ✓ |
| `OPTIONS` | Ask which methods or options a route supports. | ✓ | ✓ |

The last two columns are what clients, proxies, and caches actually act on:

- **Safe** — the request only reads and does not change server state, so a browser may prefetch or retry it freely.
- **Idempotent** — sending the request many times leaves the same state as sending it once.

Every safe method is idempotent; the reverse does not hold. `DELETE /orders/42` changes state, yet repeating it still leaves order 42 deleted. `POST` is neither, which is why a retried `POST /orders` can create a second order — and why checkout pages warn against refreshing.

Example request lines:

```http
GET /users/123
POST /users
PATCH /users/123
DELETE /users/123
```

### URL to Request Target

The client starts from a full URL and splits it in two. The scheme and host decide *where the message is delivered*; what remains becomes the request target written *inside* the message.

```text
https://api.example.com/orders?status=pending
└─┬─┘   └──────┬──────┘└──┬──┘└──────┬──────┘
scheme       host        path  query string
                       └─────────┬──────────┘
                           request target
```

- **Scheme** (`https`) selects the protocol, its default port (`80` for `http`, `443` for `https`), and whether TLS protects the exchange.
- **Host** (`api.example.com`) is used twice and independently: DNS resolves it to an IP address to open the connection, and the same string is copied unchanged into the `Host` header by the HTTP library of our programming langugae. The IP never appears in the request; without the name, a server holding many domains on one address could not tell which site was wanted.
- **Path** (`/orders`) identifies the resource exposed by the server within that website or application.

    The leading `/` in a path such as `/events/42` starts at the root of the site's URL space. It does not necessarily correspond to folders or files on the server. The application defines what its paths mean: here, `/events` represents the events collection and `/events/42` identifies one event. The same path on a different host can refer to a completely different resource.

- **Query string** (`?status=pending`) is everything after the first `?` in the target. It carries `key=value` pairs joined by `&`:

    ```text
    /events?city=LA&sort=date&limit=20
            └──┬──┘ └───┬───┘ └──┬───┘
            pair     pair     pair
    ```

    It exists because two requests can act on the *same* resource in different ways. `/events` is always the events collection; the query string picks which slice of it to return — filtering (`city=LA`), searching (`q=backend`), sorting (`sort=date&order=desc`), pagination (`limit=20&offset=40`), and response shaping such as field selection or format (`fields=id,title`, `format=csv`). Anything that would name a *different* resource belongs in the path instead.

    **Security:** The query string sits in the request line, so it lands in server logs, browser history, and proxy caches even under HTTPS. Credentials and tokens belong in headers or the body, never here.

### Body

The body is the data itself. The method, target, and headers all *describe* the request; the body *is* the payload.

The client **serializes** a value in memory — a Python dictionary, a file on disk — into bytes and writes them after the blank line. HTTP never looks inside those bytes, so two headers describe them: `Content-Type` says **how to interpret** them, and `Content-Length` says **where they end**. The server reads exactly that many bytes, parses them with the format named by `Content-Type`, and hands the result to the route handler.

`POST`, `PUT`, and `PATCH` normally carry a body.

`GET`, `HEAD`, and `DELETE` normally do not. A body on a `GET` has no defined meaning and is often dropped or rejected, which is why filters and pagination travel in the query string instead.

#### Content Type

The **Content-Type** written as `type/subtype`:

- The type says who consumes the bytes, it could be `text` for something readable as characters, `image`/`audio`/`video` for media, `application` for bytes handed to a program to parse.
- The subtype names the exact format of the information. Eg: JSON, HTML, mp4, png, etc.

| Media type | Body contains |
|---|---|
| `application/json` | Application data intended to be interpreted according to the JSON data format. |
| `application/x-www-form-urlencoded` | Flat `key=value` pairs encoded like a query string. What a plain HTML `<form>` posts. |
| `multipart/form-data` | Several parts in one body, each with its own headers. Used when a form includes a file, since raw file bytes cannot fit the flat format above. |
| `text/html` | An HTML document. |
| `image/png`, `video/mp4`, `application/pdf` | Binary file contents. |

A media type can take parameters after `;` — `charset=utf-8` for textual types, or `boundary=X7MA4YWxk` naming the delimiter between multipart parts.

```http
POST /events HTTP/1.1
Content-Type: application/json
Content-Length: 84

{"title":"Backend Conference","location":{"city":"LA","venue":"Hall 3"},"seats":200}
```

#### Where the Body Ends?

A TCP connection provides an ordered byte stream without message boundaries, so the receiver needs application-layer rules to determine where each message ends. In HTTP/1.1, two common ways to indicate the body’s boundaries use a header in the same header block as `Content-Type`:

- **`Content-Length: 84`** — the body is exactly 84 bytes long, so the receiver treats the next 84 bytes after the blank line as the body.
- **`Transfer-Encoding: chunked`** — the body is sent in chunks, each prefixed by its size in hexadecimal. A zero-size chunk marks the end of the body data, followed by optional trailers and a final blank line. This allows the sender to transmit a body without knowing its total size beforehand.

The HTTP library or framework usually handles these headers. It can use `Content-Length` when the body’s byte length is known, including when streaming a file of known size. When the length is unknown, such as for content generated while it is being sent, it can use chunked encoding.

## HTTP Response Elements

An **HTTP response** is the message sent by the server to answer an HTTP request. It tells the client what happened and can contain returned data or error details. A response is not another request sent in the opposite direction.

Each response answers three questions:

| Question | Element |
|---|---|
| What happened? | The **status code** — `200`, `404`, `500`, and so on. |
| Under what conditions? | The **headers** — body format, length, caching rules, redirect location. |
| With what data? | The **body** — optional, and absent when there is nothing to return. |

The request asks *what to do, on which resource*; the response reports *what happened, with what result*. Headers and body work the same way in both.

### How the Backend Creates a Response

A response is not typed out by hand either. In the [FastAPI route handler example](API.md#resources-and-routes), the function returns a Python dictionary. FastAPI serializes it into a JSON response, with the default status `200 OK` and `Content-Type: application/json`, and Uvicorn sends it back to the client.

The backend can choose a different status, add headers, or return another body format. For the order above, it would select `201 Created` and add the `Location` header pointing at the new order. See [FastAPI responses](https://fastapi.tiangolo.com/advanced/response-directly/).

What Uvicorn puts back on the connection is a block of text again, laid out in almost the same fixed order as the request.

### Response Structure

Almost, because only the first line changes: the **structure** of an HTTP/1.1 response is the status line, headers, a blank line, and an optional body. Where the request opens by stating an action to perform, the response opens by reporting the result of one; everything after it follows the rules already described.

Answering the `POST /orders?status=pending` request above, that backend returns:

```http
HTTP/1.1 201 Created
Content-Type: application/json
Location: /orders/42
Content-Length: 57

{"id":42,"productId":123,"quantity":2,"status":"pending"}
```

1. **Status line:** `HTTP/1.1` is the version, `201` is the status code, and `Created` is a short description called the **reason phrase**. The client uses the numeric code to understand the result.
2. **Headers:** Describe the response. Here, `Content-Type` identifies the body format, `Location` points to the newly created order, and `Content-Length` gives the body's length in bytes.
3. **Body:** Contains the returned order data. A blank line separates it from the headers.

### Status Codes

The status code is the second token of the status line. It states the outcome of the request as a number the client acts on, with the reason phrase beside it for humans to read.

The first digit places a code in one of five classes. That digit alone is enough for a client to react sensibly, even to a specific code it has never seen:

| Range | Meaning |
|---|---|
| `1xx` | Informational. An interim response before the final result, or a protocol switch. |
| `2xx` | Success. The request worked. |
| `3xx` | Redirection. Further action is needed, such as following another URL or using a cached response. |
| `4xx` | Client error. The request was invalid or not allowed. |
| `5xx` | Server error. The server failed to fulfill the request. |

The split between the last two is what clients and proxies actually act on:

- **`4xx` blames the request.** Resending it unchanged will fail again, so the client must fix the request or the caller's credentials before retrying.
- **`5xx` blames the server.** The request may have been fine, so retrying later — normally with a growing delay — can succeed.

Common status codes:

| Code | Meaning |
|---|---|
| `200 OK` | Request succeeded. |
| `201 Created` | A new resource was created. |
| `204 No Content` | Request succeeded, but there is no response body. |
| `301 Moved Permanently` | Resource has a permanent new URL. Permanent redirect|
| `302 Found` | Resource is temporarily at another URL. Temporary redirect|
| `400 Bad Request` | Request is malformed or invalid. |
| `401 Unauthorized` | Authentication is missing or invalid. |
| `403 Forbidden` | Server understood the request but refuses to fulfill it. |
| `404 Not Found` | Resource does not exist at that URL. |
| `409 Conflict` | Request conflicts with current server state. |
| `422 Unprocessable Content` | Request format is valid, but the data fails validation. |
| `500 Internal Server Error` | Unexpected server-side failure. |
| `502 Bad Gateway` | A proxy or gateway got a bad response from an upstream server. |
| `503 Service Unavailable` | Server is temporarily unavailable or overloaded. |

Common REST-style method and status pairings:

| Request | Successful response |
|---|---|
| `GET /users/123` | `200 OK` with the user data. |
| `POST /users` | `201 Created` with the created user. |
| `PATCH /users/123` | `200 OK` with updated data or `204 No Content`. |
| `DELETE /users/123` | `204 No Content`. |

### Response Body

A response body is serialized, typed, and framed exactly like a [request body](#body). What differs is which formats show up: a response commonly returns `application/json` to an API client or `text/html` to a browser, and rarely uses the form-encoded or multipart layouts that exist for submitting forms. `Transfer-Encoding: chunked` also appears more often here, since a server frequently starts sending before it knows the total size — a large export, or a response streamed as it is generated.

Some responses have no body. For example, `204 No Content` reports success without returned content, and responses to `HEAD` contain headers without a body. An error response can include a body explaining what went wrong.
