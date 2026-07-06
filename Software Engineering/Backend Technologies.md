# Terminology

## Languages and Frameworks

### Backend Programming Language

Backend Programming Language enable computers to be turned into servers. They let you write all sorts of logic in the backend.

Popular Languages:

1. Python
2. Typescript/Javascscript: Enabled by NodeJS to run outside of a browser
3. Java

### Package/Library

A code distribution written by someone else that can be installed and used in your project. Carry out tasks like doing calculations,talking to a database and setting up user authentication and login.

### Package Manager

Program for installing packages and frameworks from the internet. They could be System level and Language Level.

Packages can be installed individually, but most projects need a saved list of packages so the project can be installed again without dependency issues. Dependency issues happen when packages need different versions of the same package, when a package is missing, or when two developers are using different versions of a dependency.

Different package managers need to know **where** to install a package and **how** to record that package for the project.

**Python:** The "current environment" is usually a **virtual environment** or a **conda environment**. Installing a package adds it to that environment and may also update a dependency file for the project.

**TypeScript/Javascript:** With Node.js, the closest idea is the current **Node project**. Packages are usually installed into the project's `node_modules` folder and recorded in `package.json` and a lock file.

**Java:** Packages are usually called **dependencies**. Maven and Gradle do not normally install dependencies directly into a local project folder the same way npm does. Instead, dependencies are declared in a build file, downloaded into a local cache, and added to the project's build/classpath when the application is compiled or run.

| Ecosystem | Package Manager | Project Dependency File |
|---|---|---|
| Python | pip | `requirements.txt` |
| Python | uv | `pyproject.toml` / `uv.lock` |
| Python | Conda | `environment.yml` |
| TypeScript/Javascript | npm | `package.json` / `package-lock.json` |
| Java | Maven | `pom.xml` |
| Java | Gradle | `build.gradle` / `build.gradle.kts` |

### Backend Framework

A software framework built on top of a backend programming language that provides structure, conventions, and prebuilt functionality for developing server-side applications, reducing the amount of boilerplate code developers need to write.

### Examples Map

| Backend Programming Language | Runtime | Package Manager | Package/Library Examples | Frameworks |
|---|---|---|---|---|
| Python | Python interpreter | pip, uv, Poetry, conda | Pydantic, SQLAlchemy, psycopg, requests, bcrypt, PyJWT, python-dotenv | FastAPI, Django, pytest |
| Typescript/Javascscript | NodeJS | npm, yarn, pnpm | Prisma, Zod, bcrypt, jsonwebtoken, dotenv, pg, mongoose, axios | Express, NestJS, Fastify, Next.js |
| Java | JVM | Maven, Gradle | Hibernate, Jackson, Lombok, JUnit, Mockito, PostgreSQL JDBC Driver | Spring Boot, Quarkus, Micronaut, Jakarta EE |

## Requests

### Request-Response Cycle

Client: Machine/ App/ Program sending a request.<br>
Server: Machine/ App/ Program listening to and responding to requests.

The communication process between a client and a server is called a request response cycle.

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Database

    Client->>Server: Send HTTP request
    Server->>Server: Run backend logic
    Server->>Database: Query or update data
    Database-->>Server: Return data
    Server-->>Client: Send HTTP response
    Client->>Client: Render or use response
```

### HTTP Request Elements

Example request:

```http
POST https://amazon.com/orders?status=pending
Authorization: Bearer token
Content-Type: application/json

{
  "productId": 123,
  "quantity": 2
}
```

```mermaid
flowchart LR
    Request["HTTP Request"]

    Request --> Method["Method / Type<br/>POST"]
    Request --> URL["URL<br/>https://amazon.com/orders?status=pending"]
    Request --> Headers["Headers<br/>Authorization, Content-Type"]
    Request --> Body["Body<br/>Data sent with request"]

    URL --> Protocol["Protocol<br/>https://"]
    URL --> Domain["Domain name<br/>amazon.com"]
    URL --> Path["URL path<br/>/orders"]
    URL --> Query["Query params<br/>?status=pending"]
```

### APIs

We use our programming language and backend framework to define what *types* of requests are allowed and *how* we should handle these requests

Types of requests can vary by combination of HTTP Method + URL Path
Example:
GET /users
POST /users
GET /users/123

API(Application Programming Interface): The collection of requests that a backend exposes for clients to use, along with the rules for how those requests and responses are structured.
> Named after the fact that it allows applications to the interact with other programmatically

API's could be formed in accordance with different conventions. Some common ones are:

1. REST(Representational State Transfer): The most common way for apps to talk to each other over HTTP. It exposes resources through URL paths like `/users` or `/orders`, and uses HTTP methods like `POST`, `GET`, `PUT`, and `DELETE` (CRUD) to perform actions on those resources.

> The name comes from the fact that the client requests a representation of a resource's current state, commonly in JSON format.

2. GraphQL: Lets the client ask for exactly the data it needs, usually through one endpoint. Instead of exposing many resource-based URL paths like REST, GraphQL exposes a schema that describes the available data and relationships.

3. gRPC(Google Remote Procedure Call): A framework for calling functions on another service as if they were local functions. It uses strongly defined service contracts and is often used for fast communication between backend services or microservices.

### Data Validation

Data validation is the process of checking that data has the expected structure, types, required fields, and allowed values before the backend uses it.

It is important because backend applications receive data from many sources: API requests, forms, databases, environment variables, external APIs, files, and LLM responses.

All of these technologies define the expected shape of data, validate incoming data against that shape, and reject or handle invalid data before it causes problems deeper in the backend.

| Language | Backend Framework | Common Data Validation Technology |
|---|---|---|
| Python | **FastAPI** | **Pydantic** |
| Python | **Django** | Django Forms, Django Models, Django REST Framework Serializers |
| TypeScript/Javascript | **Express** | **Zod**, **Joi**, or **Yup** |
| TypeScript/Javascript | **NestJS** | **class-validator** |
| TypeScript/Javascript | **Next.js** | **Zod** |
| Java | **Spring Boot** | **Jakarta Bean Validation** with **Hibernate Validator** |

**Python / Pydantic** Example:

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

### Servers

After you have used the backend framework to set up API routes and logic, you still need a server to actually listen and respond to requests.

High-level pattern:

```text
Client -> server/runtime -> backend framework/app -> route handler
```

#### Python

**ASGI(Asynchronous Server Gateway Interface)** is a Python protocol that defines how an async-capable web server communicates with a Python backend application.

The **ASGI server** is the process that listens for network traffic and speaks HTTP/WebSocket with clients.<br>
Example: **Uvicorn**, **Hypercorn**, and **Daphne**.

Frameworks: **FastAPI**, **Django 3.0(With Async)**, and **Flask**<br>
Runtime/Server layer: **Python** provides the runtime, ASGI servers like **Uvicorn**, **Hypercorn** and **Daphne** run the app and handle HTTP/network traffic.

**FastAPI** Example:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"id": user_id, "name": "Manpreet"}
```

Run with:

```bash
uvicorn main:app --reload
```

Here, Uvicorn runs the FastAPI `app` object defined inside `main.py`.<br>
The `app` object is your FastAPI application. It contains the registered routes, middleware, validation rules, and other backend behavior defined across source files like `main.py`, `routes.py`, `models.py`, `database.py`, etc.

#### TypeScript/Javascript

In TypeScript/Javascript backends, the server is usually created by the runtime and framework together.

Frameworks: **Express**, **NestJS**, and **Next.js**<br>
Runtime/Server layer: **Node.js** provides the runtime and low-level HTTP/network capabilities. In **Express**, you usually start the server directly with `app.listen(...)`; in **Next.js**, the framework starts and manages the server when you run the app with `npm run dev` or deploy it.

#### Java

In Java backends, the application commonly runs inside or on top of a web server/container.

Frameworks: **Spring Boot**<br>
Runtime/Server layer: **JVM** provides the runtime. In modern **Spring Boot** applications, you usually do not start a separate server manually. The application is packaged with an embedded server such as **Tomcat**, **Jetty**, or **Undertow**, so when you run the app, Spring Boot starts the server automatically.

## Cloud Computing

Infrastructure is any kind of underlying software, hardware, network or cloud services that help the actual app to run.

### IaaS(Infrastructure as a Service)

Nowadays, Instead of buying your own computers, you often rent computers from Cloud Computing Companies like AWS(Amazon Web Services), GCP(Google Cloud Platform) and Microsoft Azure

Some infrastructure-focused services that they provide are:

1. Virtual Machines: Virtual computers in the cloud that run as software on physical servers.

    They can be vertically scaled by allocating more CPU, memory, or storage, and horizontally scaled by creating more VM instances.

    You can install an operating system, backend runtime, database, or other server software on them.

    Example include:
    - AWS EC2(Elastic Compute Cloud) instances
    - Azure Virtual Machines
    - Google Compute Engine VM instance

2. Load Balancers: A special VM that distributes incoming traffic across multiple servers, hosted on multiple VM instances, so no single server has to handle everything by itself. These servers can also be located in different geographic regions to reduce latency for users.

    Examples Include:
    - AWS Elastic Load Balancing
    - Azure Load Balancer
    - Google Cloud Load Balancing
    - Cloudflare Load Balancing

3. Object Storage(Blob Storage): Cloud storage for files like images, videos, backups, logs, and documents.

    Each file is stored as an independent object, making it easy to store large amounts of data and access it over the internet.

    It is commonly used for static assets, user uploads, backups, and application logs.

    Examples Include:
    - AWS S3(Simple Storage Service)
    - Google Cloud Storage
    - Azure Blob Storage

4. DNS Services: After you have rented a domain name(like `example.com`) from a domain registrar, a DNS service hosts the DNS records for that domain.

    DNS records map hostnames to targets like server IP addresses, load balancers, CDNs, mail servers, or other cloud services.

    The key detail is that each hostname(under the same domain name) can point to a different target. For example:

    ```text
    example.com        -> CDN
    www.example.com    -> CDN
    api.example.com    -> load balancer
    admin.example.com  -> VM IP address
    mail.example.com   -> mail server
    ```

    Examples Include:
    - Cloudflare DNS
    - AWS Route 53
    - Google Cloud DNS
    - Azure DNS

5. Networking and Security Components: Control how cloud resources communicate with each other and with the public internet.

    Virtual networks, private subnets, public IPs, and routing rules define how traffic moves between servers, databases, and external users.

    Firewalls and security groups define which traffic is allowed or blocked for specific resources.

    Examples Include:
    - AWS VPC(Virtual Private Cloud)
    - Azure Virtual Network
    - Google Virtual Private Cloud
    - Cloudflare Firewall Rules

6. CDN(Content Delivery Network): Caches frequently requested static files in an edge server that is geographically close to users around the world so websites and apps load faster.

    Static files can include images, videos, CSS, JavaScript, downloads, and other content that does not change for every user.

    Examples Include:
    - AWS CloudFront
    - Google Cloud CDN
    - Azure Front Door
    - Cloudflare CDN

### PaaS(Platform as a Service)

Cloud platforms that let you deploy and run applications by uploading your backend code, without directly managing the underlying VMs, load balancers, deployment setup, health checks, etc.

Examples Include:

- Heroku(owned by Salesforce)
- AWS Elastic Beanstalk
- Google App Engine
- Azure App Service

### SaaS(Software as a Service)

When a company provides a backend and an API that outside applications can use.

For Example: Twilio could provide the entire backend and API setup for the Email Backend service in the microservices example discussed in the next section.

Pretty much everything in the backend that is complicated is offered to be handled by a SaaS company out there.

## Microservices

A backend architecture where one large backend is split into multiple smaller backends, each responsible for a specific business capability or major feature of the application.

For example, an e-commerce app might have:

![Microservices in Ecommerce App](<Media/Screenshot 2026-06-09 at 5.08.28 AM.png>)

Each backend is its own service. It can have its own load balancer, multiple server instances, backend code, and database.

Each service can also use different technologies if needed. For example, the Orders Backend might use JavaScript with MongoDB, the Payments Backend might use Python with MySQL, and so on.

This means each service can be developed, deployed, scaled, and maintained separately. For example, if payment traffic increases, you can scale only the Payments Backend without scaling the Orders or Email services.

The tradeoff is that the system becomes more complex because these services now need to communicate with each other over the network, handle failures between services, and keep data consistent across separate databases.

## Data

An app needs to store and retrieve lots of types of data.

### Primary Database

Helps us store and manage data like user data, order history, product information(Description, Rating and the reviews).

It usually runs on a different computer(or VM) as a database server and talks to our main backend server.

When a user places an order, the backend usually writes that order to the primary database first because losing that data would be a serious problem.

Primary databases can be broadly divided into:

#### SQL

SQL databases organize data into tables with rows and columns.

They are a good fit when the data has a clear structure and relationships between different things.

Examples: PostgreSQL, MySQL, SQLite, Microsoft SQL Server

Example:

| users table | orders table |
|---|---|
| id, name, email | id, user_id, total_amount |

The `user_id` in the orders table can point back to the user who placed the order. This is called a relationship.

Commonly used in cases of **ACID** transactions like Order, payments, inventory, Banking and financial data.

#### NoSQL

NoSQL means "not only SQL". It is a broad category of databases that do not use the traditional table-based relational model as their main design.

Different NoSQL databases are designed for different kinds of data and scale problems.

| Type | Common Examples | Good For |
|---|---|---|
| Document Database | MongoDB, CouchDB | JSON-like documents, flexible data shapes |
| Key-Value Database | Redis, DynamoDB | Fast lookup by key, cache, sessions |
| Wide-Column Database | Cassandra, HBase, ScyllaDB | Very large-scale writes, distributed data |
| Graph Database | Neo4j, Amazon Neptune | Highly connected data like social networks and recommendations |

The tradeoff is that NoSQL databases are not all the same. MongoDB, Redis, Cassandra, and Neo4j solve different problems, so choosing "NoSQL" is not specific enough by itself.

### Blob Storage and CDN

Files like images, videos, PDFs, or backups are stored in **blob storage**, and the primary database stores metadata about that file.

Example:

| product_id | image_url | title |
|---|---|---|
| 123 | `https://cdn.example.com/products/123.png` | Running Shoes |

**CDN** caches that file closer to users around the world so it loads faster.

Common pattern:

```text
User uploads image -> Backend receives image -> Backend stores image in S3 -> Database stores image URL -> CDN serves image quickly
```

### Search Database

Search databases are built for features like:

1. Searching product names and descriptions
2. Handling spelling mistakes
3. Ranking more relevant results higher
4. Filtering by category, price, rating, or location
5. Autocomplete suggestions

Examples: **Elasticsearch**, **OpenSearch**, **Apache Solr**.

The primary database still stores the real product data. The search database usually stores a copy that is optimized for search.

### Cache

A cache stores data that is expensive to fetch or calculate, so the backend can return it faster next time.

Example: **Redis**. It keeps data in memory, which is much faster than reading from disk.

Workflow Example:

```text
First request:
Backend -> Primary Database -> Save result in Redis -> Return response

Later request:
Backend -> Redis -> Return response
```

Cache is useful for:

1. Frequently viewed products
2. User sessions
3. Rate limiting
4. Expensive calculations
5. API responses that do not change every second

The important tradeoff is that cached data can become stale. If the product price changes in the primary database, the old price might still exist in cache unless the backend updates or deletes it.

### Queue

A queue is used when work should happen later, in the background, or outside the normal request-response cycle. It serves as temporary storage for work instructions.

Example:

```text
User places order -> Backend saves order -> Backend puts "send confirmation email" job in queue -> Email worker sends email
```

The user does not need to wait for the email to be sent before seeing the order confirmation page.

Queues are useful for:

1. Sending emails
2. Processing uploaded videos
3. Generating reports
4. Running scheduled jobs
5. Retrying failed tasks
6. Communicating between microservices

Example: **RabbitMQ**, **Kafka**, **Amazon SQS**, and **Google Pub/Sub**.

RabbitMQ is often used for task queues and message routing.

Kafka is often used for high-volume event streaming, logs, and analytics pipelines.

### Analytical Database

An analytical database is optimized for asking large business questions across lots of data.

Examples:

1. How much revenue did we make last month?
2. Which products are trending?
3. What percentage of users return after 7 days?
4. Which marketing campaign brought the most orders?

Example: **Snowflake**, **BigQuery**, **Redshift**, and **Databricks**

### ORM(Object Relational Mapper)

ORMs are a convenience and abstraction layer over SQL. It helps the backend turn object oriented code into SQL queries. Without an ORM, backend code often writes SQL directly.

ORMs are useful because they:

1. They use classes/models to act as blueprints for the database tables.
2. Help create, read, update, and delete records
3. Reduce repetitive SQL code
4. Make relationships between tables easier to work with
5. Help with database migrations in many frameworks

**Tradeoff:** ORMs can hide what SQL is actually being run. For simple apps this is convenient, but for performance-heavy apps developers still need to understand SQL.

Common combinations:

| Language / Framework | ORM |
|---|---|
| Python + FastAPI | SQLAlchemy as ORM + Alembic for database migrations |
| Python + Django | Django ORM |
| TypeScript/Javascript + Express | Prisma |
| TypeScript/Javascript + Next.js | Prisma|
| Java + Spring Boot | Hibernate / JPA |

In a general ORM, object-oriented code maps to relational database structures like this:

| Object-oriented code | Relational database |
|---|---|
| Class / model | Table |
| Attribute / field | Column |
| Object / instance | Row |
| Object ID / primary key field | Primary key |
| Reference to another object | Foreign key |

Example mapping:

```python
class User:
    id: int
    name: str
    email: str
```

This `User` class maps to a `users` table:

```sql
users
-----
id      INTEGER PRIMARY KEY
name    TEXT
email   TEXT
```

One `User` object maps to one row in the `users` table:

```python
user = User(id=123, name="Manpreet", email="manpreet@example.com")
```

```text
id  | name     | email
----|----------|---------------------
123 | Manpreet | manpreet@example.com
```

Query example:

Without an ORM

```sql
SELECT id, name, email FROM users WHERE id = 123;
```

With an ORM:

```python
user = session.get(User, 123)
```

Here, the ORM knows that `User` maps to the `users` table, `id`, `name`, and `email` map to columns, and `123` is the primary key of the row to load. The database returns a row, and the ORM turns that row into a `User` object.

### Embedded Database vs Server Database

**SQLite** is an embedded database. The database runs inside the same application process and stores data in a local file.<br>
SQLite is excellent when simplicity matters.

**PostgreSQL** is a server database. The database runs as a separate server process, often on a different machine or container, and the backend connects to it over the network.<br>
PostgreSQL is usually better when many users are using the app at the same time and the backend needs a more powerful database server.

### Data Storage Map

Different kinds of data and workload usually go to different tools.

```mermaid
flowchart LR
    Client["Client App"] --> Backend["Backend Server"]

    Backend --> Cache["Cache<br/>Fast copy of frequently requested data"]
    Cache -. "cache miss" .-> Backend

    Backend --> Primary["Primary Database<br/>Source of truth"]
    Backend --> Blob["Blob Storage<br/>Images, videos, files"]
    Blob --> CDN["CDN<br/>Fast global delivery"]

    Backend --> Search["Search Database<br/>Fast text search"]
    Backend --> Queue["Queue<br/>Background or future work"]

    Primary -. "copy/index" .-> Cache
    Primary --> Analytics["Analytical Database<br/>Reports and data science"]
```

The backend server is the main place where business logic runs. It decides which storage system to use for a particular task.

The primary database is usually the source of truth for the application. If another system like cache, search database, or analytics database has a copy of the data, that copy usually came from the primary database.

The backend still talks to the cache directly because the backend is the one checking whether the fast copy is already available. If the data is not in cache, the backend reads it from the primary database and can save a copy in cache for future requests.

For images and videos, the CDN is usually the cache layer. The file lives in blob storage, and the CDN keeps frequently requested files close to users.
