# API(Application Programming Interface)

An **API** is the definitive collection of requests that the backend exposes for clients to use, along with the rules for how those requests and responses are structured.

This note only focuses on API's exposed **over the web**. But APIs also exist in libraries and Operating Systems.

## API Contracts

Among other things, a backend’s **API documentation** describes its **API contract**, which defines how clients interact with the API, including available endpoints, accepted inputs, expected responses, and authentication requirements.

For a REST API, the contract specifies:

- **Endpoints:** the base URL, along with available HTTP method + URL path combinations.
- **Inputs:** accepted path parameters, query parameters, headers, and request bodies, including types and required fields.
- **Responses:** response formats, fields, and status codes for success and failure.
- **Authentication:** any required credentials or tokens.

The client developer uses this contract to write application logic that chooses the appropriate endpoint, supplies its inputs, and handles its response. This applies whether your client communicates with:

- **Your own backend:** you know the contract because you designed it and build your frontend accordingly.
- **Another server:** you learn its contract from its API documentation and build your client accordingly.

An API contract can be described in a custom format or according to an established standard.

### OpenAPI

The **OpenAPI Specification (OAS)** is a standard for describing an HTTP API's contract in a format that both humans and software can read.

An **OpenAPI document** expresses a particular API's contract using that standard, in **JSON or YAML**.

For example, this small `openapi.yaml` describes an endpoint for retrieving an event:

```yaml
openapi: 3.0.4          # OpenAPI format version
info:
  title: Events API
  version: 1.0.0        # This API's version
servers:
  - url: https://api.example.com
paths:
  /events/{event_id}:
    get:
      operationId: getEvent
      tags: [Events]
      parameters:
        - name: event_id
          in: path
          required: true
          schema:
            type: integer
      responses:
        "200":
          description: Event found
          content:
            application/json:
              schema:
                type: object
                required: [id, title]
                properties:
                  id: { type: integer }
                  title: { type: string }
        "404":
          description: Event not found
```

`operationId` gives the operation a unique name that tools can use, while `tags` group related operations. Here, `getEvent` accepts an integer path parameter and returns a JSON object with `id` and `title` on success. Other endpoints can be added under the same `paths` section.

#### Creating and Maintaining the Document

You can write the OpenAPI document before implementing the backend, or generate it from the backend's route definitions, type declarations, and metadata.

**FastAPI**, for example, generates the document at `/openapi.json` and serves Swagger UI documentation at `/docs` by default. See [FastAPI's OpenAPI documentation](https://fastapi.tiangolo.com/tutorial/first-steps/#openapi).

The description needs to stay consistent with the implementation. Generated descriptions depend on what the backend declares, including its response models and error responses. The backend remains responsible for [validating requests](#data-validation) and enforcing the contract; writing an OpenAPI document alone does not enforce those rules.

#### Tools and Benefits

Because the document follows a standard structure, tools can read it to:

- **Generate documentation:** tools such as **Swagger UI** display endpoints, parameters, and response schemas in an interactive page where developers can try requests.
- **Generate client libraries:** client generators write reusable request methods and data types from the contract, reducing repetitive code and manual transcription mistakes.
- **Support development and testing:** tools can generate mock responses for frontend development or check actual requests and responses against the declared schemas.

A **client generator** is a development tool that reads an OpenAPI document and automatically writes code for calling that API. The developer chooses the target language and library; the generator produces source files that the application can import.

For example, with the [OpenAPI Generator CLI](https://openapi-generator.tech/docs/usage/#generate) installed and the YAML above saved as `openapi.yaml`, this command generates a Python client using `urllib3`, a Python library for sending HTTP requests:

```sh
openapi-generator-cli generate \
  -i openapi.yaml \
  -g python \
  --library urllib3 \
  -o ./generated-client

python -m pip install ./generated-client
```

The first command writes the client code, and the second installs it into your Python environment. The `Events` tag groups the method into `EventsApi`, and `operationId: getEvent` becomes the Python method `get_event`. The path parameter keeps its name, `event_id`. An application can then use the generated client:

```python
from openapi_client import ApiClient, Configuration
from openapi_client.api.events_api import EventsApi

configuration = Configuration(host="https://api.example.com")

with ApiClient(configuration) as api_client:
    client = EventsApi(api_client)
    event = client.get_event(event_id=42)
    print(event.title)
```

The generated method constructs `https://api.example.com/events/42`, sends the `GET` request through `urllib3`, and parses the successful JSON response into a generated model with `id` and `title` attributes. Generated type hints also help editors and type checkers check how the application uses the request parameters and returned data. See the [Python generator documentation](https://openapi-generator.tech/docs/generators/python/).

**The generator runs during development; the application uses the generated code at runtime.** The developer still writes the logic that decides when to call `get_event`, how to display the result, and how to handle failures. When the contract changes, regenerating the client updates its request methods and types; application logic may also need updating.

## API Conventions

APIs could be formed in accordance with different conventions. Some common ones are REST, GraphQL, and RPC. REST is an architectural style, GraphQL is a query language and execution system for APIs, and gRPC is a framework that implements RPC.

### REST(Representational State Transfer)

An API following the REST convention exposes resources through URL paths like `/users` or `/orders`, and uses HTTP methods like `POST`, `GET`, `PUT`, and `DELETE` to perform actions on those resources. Such an API is called **RESTful API**.

>It is named so because the client exchanges representations of a resource's state, commonly in JSON format.

#### Resources and Routes

REST API **URL paths** model *resources*, meaning they represent resources in the structure of the API. These resources are generally expressed as plural nouns representing the core entities in a system design, such as `events`, `venues`, `tickets`, and `bookings`.

The **HTTP method** expresses the client's intended action on the resource.

A unique combination of an **HTTP method and URL path** identifies a **REST API endpoint** that clients can send requests to.

```http
GET  /events                    # Get all events
GET  /events/{id}               # Get a specific event
GET  /venues/{id}               # Get a specific venue
GET  /events/{id}/tickets       # Get the available tickets for an event
POST /events/{id}/bookings      # Create a new booking for an event
GET  /bookings/{id}             # Get a specific booking
```

On the backend, a **route** maps an HTTP method and URL path combination to a **route handler**, the function that runs when a request for that endpoint arrives. The same path can support different handlers for different methods, such as `GET /events` and `POST /events`

For example, this FastAPI decorator registers the route `GET /events/{event_id}` → `get_event`:

```python
from fastapi import FastAPI

app = FastAPI() # Instance of FastAPI that represents your backend application.

@app.get("/events/{event_id}") # Decorator registers get_event() as the route handler for GET requests on /events/{event_id}
# Path parameter names in the route must match the corresponding handler parameter names.
async def get_event(event_id: int):  # value in {event_id} is passed into the handler parameter with matching name i.e. event_id
    return {"id": event_id, "title": "Backend Conference"}
```

When `GET /events/42` arrives, FastAPI matches this route, calls `get_event` with `event_id=42`, and turns its returned dictionary into a JSON response. This example returns fixed data; a real handler could look up the event in a database.

#### Request Inputs

A REST API commonly uses HTTP and can accept input through:

1. **Path parameters** identify the specific resource being addressed. They are part of the URL path and are required when the route needs a particular resource, such as the `id` in `GET /events/{id}`.
2. **Query parameters** keep the resource the same and modify how it is read — filtering, searching, sorting, pagination, and response shaping such as field selection or format. They appear after `?` in the URL and are separated by `&`, such as `GET /events?city=LA&date=2025-01-01`.
3. **Request body** carries a structured payload, commonly JSON, containing the data needed to create or update a resource. It is the only input with a real format, so nested data goes here; a path segment or query value is always flat text.

```http
# Path parameter: identify event 42
GET /events/42

# Query parameters: filter the events collection
GET /events?city=LA&date=2025-01-01

# Request body: provide data for a new event
POST /events
Content-Type: application/json

{
  "title": "Backend Conference",
  "description": "A conference about backend engineering",
  "location": "Los Angeles",
  "date": "2025-01-01"
}
```

Headers can also provide API inputs, such as an authorization token. Query parameters can be required if the API contract requires them.

#### Pagination

An endpoint such as `GET /events` may have thousands or millions of matching records. Returning all of them in one response increases the data the backend must load and serialize, the amount transferred over the network, and the work the client must do to process it—even when the user only needs to see the first few events.

**Pagination** lets the client retrieve that collection in smaller portions, called **pages**. Each request returns one page, and the client can make further requests as the user moves through the results, such as by selecting “Next” or scrolling to load more.

For a database-backed endpoint, the backend applies pagination in the **database query** so that it retrieves the requested portion of the result set without loading the entire collection into application memory first. This limits the records returned to the backend and client, although the database may still need to scan or sort more records to produce that page.

The client specifies **how many records to return** and **where in the results to begin** through **query parameters in the request URL**. The route handler is implemented to read and validate these parameters, apply defaults for omitted values, and use them to construct the database query. The backend also enforces a maximum page size so a client cannot bypass the limit by requesting an arbitrarily large page.

##### Deterministic Ordering

Every paginated query needs a **deterministic order**, meaning unchanged data must appear in the same order across requests. Sorting only by `created_at` leaves events with identical timestamps in an unspecified order. If those events fall around a page boundary, separate queries may place them on different pages, causing the client to see a record twice or miss it entirely.

For example, suppose the `events` table contains:

| `id` | `created_at`        |
|------|---------------------|
| 43   | 2025-01-01 10:00:00 |
| 42   | 2025-01-01 10:00:00 |
| 41   | 2025-01-01 10:00:00 |
| 40    2025-01-01 09:00:00  |

The client requests two records per page. Each request runs a separate database query using only `ORDER BY created_at DESC`:

- **First request — page 1 (`LIMIT 2 OFFSET 0`):** the database may order the matching IDs as `[43, 42, 41, 40]`. Taking the first two returns **`[43, 42]`**.
- **Second request — page 2 (`LIMIT 2 OFFSET 2`):** the query runs again. Because events with identical timestamps have no defined order relative to each other, the order could now be `[41, 43, 42, 40]`. Skipping the first two and taking the next two returns **`[42, 40]`**.

The client therefore receives **event 42 twice** and **misses event 41**, even though the table's contents have not changed.

To resolve ties, include an **existing unique column**, such as `id`, in the sorting rules: `ORDER BY created_at DESC, id DESC`. This sorts by creation time first, then by ID within each group of matching timestamps, giving every record a definite position.

For this unchanged table, both queries now use the order `[43, 42, 41, 40]`: page 1 returns `[43, 42]`, and page 2 returns `[41, 40]`.

##### Types of Pagination

The main difference between pagination approaches is **how the client specifies the page's starting position**. Two common approaches are:

1. **Offset pagination** tells the server how many records to skip. A request such as `GET /events?limit=20&offset=40` asks for records 41–60 in the ordered result. Page-number pagination is the same idea expressed as `page` and `page_size`, where `offset = (page - 1) * page_size`.

   ```sql
   SELECT id, title, created_at
   FROM events
   ORDER BY created_at DESC, id DESC
   LIMIT 20 OFFSET 40;
   ```

   Offset pagination is simple and lets clients jump to a particular page. However, large offsets become slower because the database still has to pass over the skipped records. Inserts or deletions before the current offset can also cause records to be repeated or missed while a client moves through the pages.

2. **Cursor pagination**, commonly implemented using **keyset pagination**, asks for records after the last record previously returned. The cursor is normally an opaque string encoding the ordered values, such as `created_at` and `id`.

   ```http
   GET /events?limit=20&after=eyJjcmVhdGVkX2F0IjoiMjAyNS0wMS0wMVQxMDowMDowMFoiLCJpZCI6NDJ9
   ```

   ```sql
   SELECT id, title, created_at
   FROM events
   WHERE (created_at, id) < ('2025-01-01T10:00:00Z', 42)
   ORDER BY created_at DESC, id DESC
   LIMIT 21;
   ```

   The server requests one extra record to determine whether another page exists, returns only the first 20, and builds `next_cursor` from the last returned record:

   Example response (items shortened for readability):

   ```json
   {
     "items": [
       {"id": 41, "title": "Backend Conference", "created_at": "2025-01-01T09:30:00Z"}
     ],
     "next_cursor": "eyJjcmVhdGVkX2F0IjoiMjAyNS0wMS0wMVQwOTozMDowMFoiLCJpZCI6NDF9",
     "has_more": true
   }
   ```

   With a suitable index, keyset pagination avoids scanning large offsets on large, frequently changing datasets, but it does not naturally support jumping directly to an arbitrary page.

A `total_count` can be useful for page-based interfaces, but counting a very large or heavily filtered collection may be expensive, so cursor-based APIs often return only `has_more` and the next cursor.

#### Responses and Fetching Data

The backend chooses which fields to return for each resource. For HTTP status codes and successful method/status pairings, see [Status Codes](HTTP.md#status-codes).

Suppose the frontend only needs the user's name:

```http
GET /users/42
```

but the server returns:

```json
{
  "id": 42,
  "name": "Alice",
  "email": "alice@example.com",
  "address": "...",
  "created_at": "...",
  "preferences": {}
}
```

You might receive more data than you need. This is called **over-fetching**.

The opposite can also happen: you need a user and their orders, requiring:

```http
GET /users/42
GET /users/42/orders
```

That's potentially **under-fetching** — one request doesn't give you everything you need.

This is one of the problems GraphQL was designed to address. A REST API can also provide field selection or include related resources if its design supports those options.

### GraphQL

Lets the client ask for exactly the data it needs, usually through one endpoint such as `/graphql`. The API defines a **schema** that describes the available data and relationships.

#### Schema and Resolvers

The backend defines the types and operations available to the client. **Resolvers** are the functions that fetch or calculate the requested fields. They can read from a database or call another service. See the GraphQL documentation on [schemas](https://graphql.org/learn/schema/) and [execution](https://graphql.org/learn/execution/).

Using the same user example:

```graphql
type User {
  id: ID!
  name: String!
  email: String!
}

type Query {
  user(id: ID!): User
}
```

`!` means the value cannot be null. The backend connects `Query.user` to a resolver that looks up the user by ID. A GraphQL library validates and executes incoming operations against this schema.

#### Queries and Responses

The client sends a query describing exactly what it wants:

```graphql
query {
  user(id: "42") {
    name
    email
  }
}
```

The server might return:

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com"
    }
  }
}
```

This makes it especially useful for complex frontends where different screens need different combinations of data. Over HTTP, the query can be sent in a JSON request body with a `query` field.

**Queries** read data, **mutations** change data, and **subscriptions** receive ongoing updates. The schema defines which operations are available. A subscription also needs a way to deliver those updates, such as the mechanisms in [Real-Time Communication](<Real-Time Communication.md>).

#### N+1 Query Problem

GraphQL can cause an **N+1 query problem** when one query fetches a list of `N` records and a nested-field resolver performs another database query for each record. A request-scoped **DataLoader** can address this by batching those individual lookups and caching repeated lookups during the request.

For example, fetching users and their orders should not require a separate database query for each user's orders. The batch function can fetch orders for all requested user IDs together.

### RPC(Remote Procedure Call)

A general approach where one service calls a function or procedure on another service as if it were local. The call still crosses a network, so it can time out or fail independently of the caller.

Examples: gRPC, Apache Thrift.

#### gRPC

For example, imagine you have microservices:

```text
Order Service -> gRPC -> User Service
```

The Order Service might effectively call `userService.GetUser(...)`, even though `GetUser()` is actually executing on another machine/container/service.

gRPC typically uses Protocol Buffers (Protobuf) rather than JSON. The service methods and message structures are defined in a `.proto` file:

```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser (GetUserRequest) returns (User);
}

message GetUserRequest {
  int64 id = 1;
}

message User {
  int64 id = 1;
  string name = 2;
  string email = 3;
}
```

The numbers are field identifiers used in the encoded messages. Code generation produces client and server code from this contract. The backend implements `GetUser`; the client calls the generated method with a request containing `id: 42`.

This example is a **unary RPC**: one request and one response. gRPC also supports client streaming, server streaming, and streaming in both directions. See [gRPC core concepts](https://grpc.io/docs/what-is-grpc/core-concepts/).

## Handling API Requests

The API convention defines how the client interacts with the backend. The backend still needs to validate input, check access, run its logic, and return the result.

### Data Validation

For the concept and common libraries, see [Data Validation](<Backend Technologies.md#data-validation>).

**Python / Pydantic** Example (requires Pydantic and its email validation dependency):

```python
from pydantic import BaseModel, EmailStr

class CreateUserRequest(BaseModel):
    email: EmailStr
    age: int

# This works because the data matches the expected shape.
user = CreateUserRequest(
    email="manpreet@example.com",
    age=25
)

# This fails because the email is invalid and age is not an integer.
user = CreateUserRequest(
    email="not-an-email",
    age="twenty-five"
)
```

An API framework can use a model like this to validate the request body before calling the route handler.

### Authentication and Authorization

The backend checks who is calling and whether they are allowed to perform the requested action. See [Authentication](Authentication.md).

## Event Notifications

### Webhooks

A webhook lets one application notify another when an event happens by sending an HTTP request to a URL registered by the receiving application. The sender commonly uses `POST` with event details in the body. See [GitHub's explanation of webhooks](https://docs.github.com/en/webhooks/about-webhooks).

```text
Payment completes -> Payment service sends POST /webhooks/payments
                  -> Backend handles the event and responds
```

The receiving backend sets up a route for the webhook. This reverses who starts the request: the application where the event happens calls your backend, instead of your backend repeatedly checking for changes.

For live updates to a connected client, see [WebSockets and SSE](<Real-Time Communication.md>). For events and background work passed between services through a broker, see [Messaging Services](<Messaging Service.md>).

# Real-Time Communication

**WebSockets and SSE(Server-Sent Events)** are different communication mechanisms that an application can choose depending on the communication pattern it needs.

| WebSockets | SSE (Server-Sent Events) |
| --- | --- |
| Client ↔ Server | Server → Client after the client opens the stream |
| Both sides can send messages anytime while connected | Only server pushes messages on the stream |
| Used for interactive communication | Used for live updates |
| Examples: Chat, games, trading | Examples: Notifications, dashboards, news feeds |

## WebSockets

WebSockets establish a persistent connection over which both the client and server can send messages. The connection starts with a handshake; after that, they exchange WebSocket messages. See the [WebSocket protocol](https://www.rfc-editor.org/rfc/rfc6455.html).

For example, in a chat app, the client sends a new chat message and the server pushes incoming chat messages over the same connection.

## SSE(Server-Sent Events)

The client opens an HTTP request, and the server keeps the response open to send updates as events happen. The response uses `Content-Type: text/event-stream`. In a browser, `EventSource` receives these events and normally reconnects if the connection is interrupted. See the [SSE specification](https://html.spec.whatwg.org/dev/server-sent-events.html).

For example, a dashboard receives a new order status whenever it changes. The client can still send separate HTTP requests to perform actions; the event stream itself carries updates from server to client.

## Related Communication Patterns

[Webhooks](API.md#webhooks) send a new HTTP request to another application's endpoint when an event happens.

[Messaging Services](<Messaging Service.md>) cover queues and event streams between services. A backend can consume those events and then send relevant updates to connected clients through WebSockets or SSE.
