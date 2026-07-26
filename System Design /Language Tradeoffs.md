# Language Tradeoffs

Programming languages are often chosen based on the tradeoffs their runtime model, memory model, tooling, and ecosystem create for a specific domain.

## Go

- Go is ideal for software where you want to handle huge number of requests so essentially backend, networking, cloud and infrastructure systems.
- It has simple syntax, fast compilation, built-in concurrency through goroutines, and easy deployment as a single compiled binary.
- Go's automatic garbage collection makes memory management easier for many server-side programs, where developer productivity and operational simplicity often matter more than exact control over every allocation.
- Go is usually not preferred for game engines or performance-critical game loops because garbage collection can introduce unpredictable pauses. Games need consistent frame times, so C++ is often preferred when deterministic memory control is more important than the convenience of automatic garbage collection.
- Go naturally favours composition(has-a) over inheritance(is-a).
