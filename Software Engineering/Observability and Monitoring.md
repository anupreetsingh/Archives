# Observability and Monitoring

Observability and monitoring are related, but they are not the same thing.

**Monitoring** is the practice of watching known system health indicators and alerting when they cross expected boundaries.

**Observability** is the ability to understand what is happening inside a system by looking at the data it emits from the outside.

Monitoring usually answers:

- Is the system up?
- Is latency too high?
- Is the error rate above the alert threshold?
- Is CPU or memory usage abnormal?

Observability goes further and helps answer:

- Why is the system slow for only some users?
- Which dependency caused a request to fail?
- Which model call, retrieval step, or database query made an AI response slow?
- Is the issue in application code, infrastructure, a third-party API, or the model provider?

Relationship:

```text
Monitoring -> watches known symptoms
Observability -> investigates system behavior, including unknown problems
```

Monitoring depends on observability data. A system cannot be monitored well unless it emits useful telemetry.

## 1. Telemetry

**Telemetry** is operational data emitted by software and infrastructure.

In observability, the main telemetry signals are:

- **Metrics:** Numeric measurements over time
- **Logs:** Event records
- **Traces:** Request journeys across services

These signals work best together. Metrics tell you that something changed, logs show specific events, and traces show where time was spent across a request.

Relationship:

```text
Application behavior -> telemetry -> observability backend -> dashboards, queries, alerts
```

## 2. Metrics

A **metric** is a numeric measurement recorded over time.

Examples:

- Request count
- Error rate
- Response latency
- CPU usage
- Memory usage
- Queue depth
- Tokens generated per request
- LLM provider error count
- Vector database query latency

Metrics are usually aggregated and stored as time series.

A **time series** is a stream of timestamped values for a particular measurement.

Example:

```text
rag_api_request_duration_seconds{route="/chat", status="200"} = 1.42 at 12:00:00
rag_api_request_duration_seconds{route="/chat", status="200"} = 1.58 at 12:00:15
rag_api_request_duration_seconds{route="/chat", status="200"} = 2.04 at 12:00:30
```

The metric name describes what is being measured. Labels add dimensions that let you filter or group the data.

Useful metric labels:

- `service`
- `route`
- `status`
- `model`
- `tenant`
- `region`
- `dependency`

Labels are powerful, but too many unique label values create high cardinality.

**Cardinality** means the number of unique time series produced by a metric.

Bad label choice:

```text
llm_request_duration_seconds{user_id="123", prompt="full user prompt text"}
```

This creates a separate series for many users and prompts. That becomes expensive and hard to query.

Better label choice:

```text
llm_request_duration_seconds{model="general-llm", operation="answer_generation"}
```

Metrics are best for trends, alerts, and dashboards.

Use metrics when you need to know:

- How often something happens
- How long something takes
- How many resources are being used
- Whether the system is breaching an SLO

## 3. Logs

A **log** is a timestamped record of an event.

Examples:

```text
2026-08-16T13:00:15Z INFO request_started route=/chat request_id=req_123
2026-08-16T13:00:16Z WARN vector_search_slow duration_ms=850 collection=docs
2026-08-16T13:00:17Z ERROR llm_call_failed provider=openai status=429 request_id=req_123
```

Logs are good for concrete details:

- Error messages
- Stack traces
- Input metadata
- Retry attempts
- Business events
- Security events
- Model provider responses, without sensitive prompt contents

Logs are weaker when used alone because they usually show individual events, not the full request path.

Useful logs are structured.

Structured log:

```json
{
  "level": "error",
  "message": "llm_call_failed",
  "request_id": "req_123",
  "trace_id": "trace_abc",
  "provider": "openai",
  "model": "general-llm",
  "status_code": 429,
  "duration_ms": 1200
}
```

Structured logs are easier to search, filter, aggregate, and correlate with traces.

For AI systems, avoid logging raw prompts, raw completions, API keys, private documents, or user secrets unless there is a deliberate privacy and retention policy.

## 4. Traces

A **trace** shows the path of one request as it moves through a system.

A trace is made of **spans**.

A **span** is one unit of work inside a trace.

Example trace for a RAG request:

```text
Trace: POST /chat

Root span: api.request POST /chat
  Child span: auth.validate_session
  Child span: retrieval.embed_query
  Child span: vector_db.search
  Child span: prompt.build
  Child span: llm.generate_answer
  Child span: response.stream_to_client
```

Each span usually includes:

- Operation name
- Start time
- End time
- Duration
- Status
- Attributes
- Parent span ID
- Trace ID

Traces are best for distributed systems because one user request may cross many services.

Use traces when you need to know:

- Which service handled the request
- Which dependency was slow
- Whether time was spent in the app, database, vector search, queue, or model provider
- Which retries happened
- Which part of a multi-step AI workflow failed

Relationship between signals:

```text
Metric: p95 chat latency is high
Trace: vector search is taking 900ms inside slow requests
Log: vector database returned timeout warnings for those trace IDs
```

## 5. Monitoring

Monitoring is built on top of telemetry.

In practice, monitoring includes:

- Dashboards
- Alerts
- SLO tracking
- Uptime checks
- Incident notifications
- Runbooks

### SLIs and SLOs

A **Service Level Indicator (SLI)** is a measurement of service behavior.

Examples:

- Percentage of `/chat` requests that return successfully
- p95 response latency for chat requests
- Percentage of LLM calls that complete without provider errors
- Percentage of answers generated with retrieval context

A **Service Level Objective (SLO)** is a target for an SLI.

Example:

```text
99.5% of chat requests should complete successfully over 30 days.
p95 chat response time should stay under 4 seconds over 7 days.
```

SLIs measure reliability. SLOs define how reliable the system is expected to be.

## 6. OpenTelemetry

**OpenTelemetry (OTel)** is an open source observability framework for generating, collecting, and exporting telemetry data.

It is not mainly a dashboard, database, or alerting product. Its job is instrumentation and telemetry transport.

OpenTelemetry helps applications emit:

- Metrics
- Logs
- Traces

Main parts:

- **API:** Interfaces application code uses to create spans, metrics, and logs
- **SDK:** Implementation that processes and exports telemetry
- **Instrumentation libraries:** Automatic or manual instrumentation for frameworks, clients, databases, and runtimes
- **Collector:** A standalone service that receives, processes, and exports telemetry
- **Semantic conventions:** Standard names for common operations and attributes

Relationship:

```text
Application code -> OpenTelemetry SDK -> OpenTelemetry Collector -> backend
```

The backend can be Prometheus, Grafana stack components, Datadog, Jaeger, Tempo, Loki, or another observability system.

OpenTelemetry is useful because it reduces vendor lock-in. The application emits telemetry in a standard way, and the collector decides where to send it.

### OpenTelemetry in an AI System

In a RAG assistant, OpenTelemetry would instrument the services that handle user requests.

Example services:

- API service
- Retrieval service
- Embedding worker
- Vector database client
- LLM gateway
- Background evaluation worker

OpenTelemetry spans could represent:

- HTTP request handling
- Query embedding
- Vector search
- Document reranking
- Prompt construction
- LLM call
- Tool call
- Response streaming

OpenTelemetry metrics could represent:

- Request count
- Request latency
- Model call latency
- Token usage
- Provider error count
- Retrieval hit count
- Queue depth

OpenTelemetry logs could include:

- Failed dependency calls
- Retry decisions
- Rate limit events
- Model gateway errors
- Evaluation job failures

## 7. Prometheus

**Prometheus** is an open source monitoring system and time series database.

It is especially strong for metrics.

Prometheus commonly works by scraping HTTP endpoints exposed by applications or exporters.

Relationship:

```text
Application / exporter exposes /metrics -> Prometheus scrapes metrics -> PromQL queries metrics
```

Prometheus stores metrics as time series with labels.

Example metric:

```text
http_requests_total{service="rag-api", route="/chat", status="200"} 18293
```

Prometheus is used for:

- Metric storage
- Metric querying with PromQL
- Alert rule evaluation
- Infrastructure and application monitoring

Prometheus is not primarily a log store or trace store.

### Prometheus in an AI System

For a RAG assistant, Prometheus could collect:

- API request rate
- API error rate
- p50, p95, and p99 latency
- LLM request count by model
- LLM error count by provider
- Token usage counters
- Vector database query latency
- Worker queue depth
- GPU utilization if self-hosting models

Example questions Prometheus can answer:

- Did `/chat` latency increase after the latest deploy?
- Is the vector database slower than usual?
- Are LLM provider errors increasing?
- Is the embedding queue backing up?

Prometheus often feeds alerting rules.

Example alert idea:

```text
If p95 /chat latency is above 4 seconds for 10 minutes, page the on-call engineer.
```

## 8. Grafana

**Grafana** is a visualization and dashboarding platform for observability data.

Grafana connects to data sources, queries them, and displays dashboards.

Common data sources:

- Prometheus for metrics
- Loki for logs
- Tempo or Jaeger for traces
- PostgreSQL or other databases for application data
- Cloud provider monitoring systems

Relationship:

```text
Prometheus / Loki / Tempo / Datadog / other sources -> Grafana dashboards
```

Grafana is not usually the first place telemetry is generated. It is where telemetry is visualized and explored.

Use Grafana when you need:

- Dashboards
- Time series charts
- Service health views
- Alert visualizations
- Correlation between metrics, logs, and traces

### Grafana in an AI System

For a RAG assistant, Grafana dashboards could show:

- Chat request volume
- Chat latency by route
- Error rate by service
- LLM latency by model
- Token usage by model
- Cost estimate by provider
- Retrieval latency
- Vector database health
- Queue depth for background jobs
- Evaluation pass rate for answer quality checks

Example dashboard sections:

```text
User Experience
  request rate
  success rate
  p95 latency

Model Layer
  LLM calls by model
  token usage
  provider errors
  generation latency

Retrieval Layer
  vector search latency
  retrieval hit rate
  reranker latency

Infrastructure
  CPU
  memory
  container restarts
  queue depth
```

Grafana gives operators a shared visual surface during debugging and incident response.

## 9. Datadog

**Datadog** is a commercial observability platform.

It can collect, store, visualize, correlate, and alert on telemetry across infrastructure, applications, logs, traces, metrics, real user monitoring, synthetic checks, and many integrations.

Datadog often replaces several separate open source components with one managed platform.

Relationship:

```text
Application / host / cloud integrations -> Datadog Agent or OpenTelemetry Collector -> Datadog platform
```

Datadog is used for:

- Infrastructure monitoring
- Application Performance Monitoring (APM)
- Log management
- Distributed tracing
- Dashboards
- Alerts and incident workflows
- Cloud and SaaS integrations

Datadog can ingest OpenTelemetry data. That means applications can be instrumented with OpenTelemetry while Datadog acts as the backend.

### Datadog in an AI System

For a RAG assistant, Datadog could provide:

- Service maps showing API, retrieval, vector database, and LLM gateway dependencies
- APM traces for slow chat requests
- Log search for failed model calls
- Infrastructure monitoring for hosts and containers
- Alerts for latency, errors, saturation, and provider failures
- Cost and usage dashboards if custom metrics are emitted

Datadog is attractive when a team wants an integrated managed observability platform and is willing to pay for it.

Prometheus and Grafana are attractive when a team wants more open source control and is willing to operate more of the stack.

## 10. Cohesive AI Observability System

Assume the system is a RAG assistant for internal company documents.

Application flow:

```text
User -> Web app -> API service -> Retrieval service -> Vector database
                              -> LLM gateway -> Model provider
                              -> Streaming response
```

Background flow:

```text
Documents -> Ingestion worker -> Chunking -> Embeddings -> Vector database
Generated answers -> Evaluation worker -> Quality metrics
```

Observability flow:

```text
Services emit telemetry with OpenTelemetry
  -> OpenTelemetry Collector receives telemetry
  -> Metrics go to Prometheus
  -> Logs go to a log backend
  -> Traces go to a trace backend
  -> Grafana visualizes the data
  -> Alerts notify engineers
```

If using Datadog:

```text
Services emit telemetry with OpenTelemetry or Datadog libraries
  -> Datadog Agent or OpenTelemetry Collector forwards telemetry
  -> Datadog stores metrics, logs, and traces
  -> Datadog dashboards, monitors, and APM support debugging
```

### Tool Responsibilities

| Tool | Main role | What it does in the system |
| --- | --- | --- |
| OpenTelemetry | Instrumentation standard | Makes services emit metrics, logs, and traces in a vendor-neutral way |
| Prometheus | Metrics backend | Scrapes, stores, queries, and alerts on time series metrics |
| Grafana | Visualization layer | Builds dashboards from Prometheus and other data sources |
| Datadog | Managed observability platform | Provides hosted metrics, logs, traces, dashboards, APM, alerts, and integrations |

These tools are not all direct substitutes.

OpenTelemetry answers:

```text
How does the application produce telemetry?
```

Prometheus answers:

```text
Where do metrics live and how are they queried?
```

Grafana answers:

```text
How do people visualize and explore telemetry?
```

Datadog answers:

```text
Do we want a managed platform that combines telemetry storage, dashboards, tracing, logs, alerts, and integrations?
```

## 11. Debugging Example

Problem:

```text
Users report that the AI assistant is slow.
```

Step 1: Use metrics.

Prometheus shows:

```text
p95 /chat latency increased from 2.5s to 7.8s after 14:00.
Error rate is normal.
Request volume is normal.
```

This confirms the problem is latency, not total outage.

Step 2: Use traces.

A trace for a slow request shows:

```text
POST /chat total: 7.9s
  auth.validate_session: 20ms
  retrieval.embed_query: 180ms
  vector_db.search: 4.8s
  prompt.build: 30ms
  llm.generate_answer: 2.7s
  response.stream_to_client: 120ms
```

The trace shows that vector search is the largest unexpected delay.

Step 3: Use logs.

Logs correlated by `trace_id` show:

```text
WARN vector_search_slow collection=docs index=company_handbook duration_ms=4800
WARN vector_db_retry attempt=1 reason=timeout
```

The logs explain what happened inside the slow span.

Step 4: Check dashboard and alert context.

Grafana shows:

```text
Vector DB CPU increased.
Vector DB query latency increased.
Embedding queue depth is normal.
LLM provider latency is normal.
```

Now the investigation is focused on the vector database instead of the whole AI stack.

Step 5: Fix and verify.

Possible fixes:

- Add or rebuild vector indexes
- Reduce top-k retrieval size
- Split a large collection
- Scale the vector database
- Add a retrieval timeout and fallback

After the fix, metrics should show latency returning to normal, traces should show shorter vector search spans, and logs should stop showing timeout retries.

## 12. Practical Design Rules

Start with user-facing SLIs.

Good AI system SLIs:

- Chat success rate
- Chat p95 latency
- LLM provider success rate
- Retrieval success rate
- Answer quality evaluation pass rate

Instrument important boundaries:

- Incoming HTTP requests
- Database calls
- Vector searches
- Queue jobs
- External API calls
- LLM calls
- Tool calls

Correlate telemetry with IDs:

- `trace_id`
- `span_id`
- `request_id`
- `user_id` only if privacy policy allows it
- `tenant_id`
- `conversation_id` only if privacy policy allows it

Keep labels low-cardinality.

Good metric labels:

- `service`
- `route`
- `status`
- `model`
- `provider`
- `operation`

Risky metric labels:

- Raw prompt text
- Full generated answer
- User email
- Request ID
- Conversation ID
- Document text

Use logs for details, traces for request shape, and metrics for aggregate health.

Do not try to answer every question with one telemetry type.

## 13. Common Stack Choices

### Open Source Style

```text
OpenTelemetry -> Prometheus for metrics
OpenTelemetry -> Loki for logs
OpenTelemetry -> Tempo or Jaeger for traces
Grafana -> dashboards and exploration
Alertmanager -> alert routing
```

This gives strong control and avoids depending on one commercial observability vendor.

Tradeoff:

```text
More control, but more systems to operate.
```

### Managed Platform Style

```text
OpenTelemetry or Datadog Agent -> Datadog
Datadog -> metrics, logs, traces, dashboards, monitors, APM
```

This gives an integrated workflow with less self-hosting.

Tradeoff:

```text
Less infrastructure to manage, but higher vendor dependence and cost management concerns.
```

### Hybrid Style

```text
Applications emit OpenTelemetry
OpenTelemetry Collector exports to Prometheus/Grafana for local metrics
OpenTelemetry Collector also exports selected telemetry to Datadog
```

This keeps application instrumentation vendor-neutral while letting teams choose different backends for different needs.

## 14. Summary

Observability is the ability to understand a system from its emitted telemetry.

Monitoring is the operational practice of watching known signals and alerting when they move outside acceptable ranges.

Metrics show numeric behavior over time.

Logs show specific events.

Traces show how one request moved through a distributed system.

OpenTelemetry standardizes how applications generate and export telemetry.

Prometheus stores and queries metrics.

Grafana visualizes telemetry from Prometheus and other data sources.

Datadog provides a managed observability platform that can combine metrics, logs, traces, dashboards, alerts, and APM.

In an AI system, these tools help answer whether the assistant is reliable, where latency comes from, which model or retrieval step failed, and whether fixes actually improved user-facing behavior.
