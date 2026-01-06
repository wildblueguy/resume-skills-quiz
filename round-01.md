### Question 1 — Conceptual — Programming — State management

In a distributed service, what is the most important distinction between *application state* and *system-of-record state*, and why does that distinction matter for correctness?

> Application state is temporary, local, and derivable, while system-of-record state is authoritative, durable, and shared; correctness depends on treating only the latter as the source of truth that must survive crashes, retries, and restarts.

### Question 2 — Hands-on — Programming — Asynchronous / parallel programming

Write a short **Go** example that demonstrates **safe state ownership** using a single goroutine and channels, where multiple goroutines request state updates without directly mutating shared memory.

```go
package main

import "fmt"

func main() {
	updateCh := make(chan int)
	done := make(chan struct{})

	// State owner
	go func() {
		state := 0
		for delta := range updateCh {
			state += delta
		}
		fmt.Println("final state:", state)
		close(done)
	}()

	// Non-owners
	for i := 0; i < 5; i++ {
		go func(v int) {
			updateCh <- v
		}(i)
	}

	// Close after all sends complete
	go func() {
		// in real code you'd use a WaitGroup
		for i := 0; i < 5; i++ {
		}
		close(updateCh)
	}()

	<-done
}
```

### Question 3 — Conceptual — Software service implementation — User authentication and authorization

What is the **core conceptual difference** between *authentication* and *authorization*, and why is it dangerous to conflate them?

> Authentication verifies the identity of a principal, while authorization determines what that identity is allowed to do; conflating them risks treating identity proof as permission, leading to privilege escalation.

### Question 4 — Hands-on — Software service implementation — Logging / structured logging

Write a short example (language of your choice) that demonstrates **structured logging** for an HTTP request, including:

* a request ID,
* an authenticated user or principal ID,
* and an error with context.

```go
logger := slog.With(
	"request_id", req.Header.Get("X-Request-ID"),
	"user_id", userID,
)

if err := processRequest(req); err != nil {
	logger.Error("request failed",
		"error", err,
		"path", req.URL.Path,
		"method", req.Method,
	)
}
```

This preserves machine-readable context and avoids embedding structure in log messages.

### Question 5 — Conceptual — Programmable platform expertise — Kubernetes

Why are **CPU requests** more critical than **CPU limits** for Kubernetes scheduling correctness?

> CPU requests are used by the scheduler to make placement decisions and guarantee capacity, while CPU limits are enforced only at runtime via throttling and do not influence scheduling.

### Question 6 — Hands-on — Software development / deployment — Git

Show a short sequence of Git commands that:

1. Squashes the last **three** commits into one,
2. Preserves the combined commit message,
3. Does **not** rewrite history beyond those three commits.

Assume the commits are local and not yet pushed.

```bash
git reset --soft HEAD~3
git commit
```

This keeps all changes staged and creates a single new commit containing the combined diff and message.

### Question 7 — Conceptual — Software service implementation — Distributed system design

Why is **idempotency** critical for distributed APIs, especially under retries?

> Retries are inevitable in distributed systems, and idempotency ensures that repeated requests do not cause repeated or unintended side effects.

---

### Question 8 — Hands-on — Programming — Serialization and binary encodings

Write a short example (language of your choice) that shows **version-tolerant serialization**, where:

* a new field is added,
* older readers can safely ignore it.

```proto
// v1
message User {
  int64 id = 1;
  string name = 2;
}

// v2
message User {
  int64 id = 1;
  string name = 2;
  string email = 3; // new field
}
```

Unknown fields are ignored by older protobuf readers, preserving forward compatibility.

### Question 9 — Conceptual — Business — Software licensing

What is the key practical difference between **permissive** and **copyleft** open-source licenses?

> Permissive licenses allow redistribution and modification without imposing the same license on derivatives, while copyleft licenses require derivative works to inherit the original licensing terms.

### Question 10 — Hands-on — Programmable platform expertise — Linux / systemd

Write a minimal **systemd service unit** that runs a web service binary on boot, restarts on failure, and writes logs to journald.

```ini
[Unit]
Description=My Web Service
After=network.target

[Service]
ExecStart=/usr/local/bin/my-service
Restart=on-failure
User=www-data

[Install]
WantedBy=multi-user.target
```

By default, stdout/stderr are captured by journald.

### Question 11 — Conceptual — Programming — API design

What makes an API **backward compatible**, and why is backward compatibility harder to maintain than forward compatibility?

> An API is backward compatible if existing clients continue to work unmodified against newer versions, and this is harder than forward compatibility because old clients cannot adapt to new behavior, forcing the server to preserve legacy semantics.

### Question 12 — Hands-on — Software service implementation — Metrics collection

Write a short example (language of your choice) that records:

* a request latency metric, and
* labels/tags it by endpoint.

Assume a Prometheus-style metrics system.

```go
var requestLatency = prometheus.NewHistogramVec(
	prometheus.HistogramOpts{
		Name: "http_request_latency_seconds",
		Help: "Request latency by endpoint",
	},
	[]string{"path"},
)

start := time.Now()
handleRequest()
requestLatency.WithLabelValues("/users").Observe(time.Since(start).Seconds())
```

### Question 13 — Conceptual — Programming — Debugging and profiling

Why is **profiling in production** often more valuable than profiling in a local or staging environment?

> Production profiling captures real workloads, data distributions, and hardware/software interactions that are difficult or impossible to reproduce in non-production environments.

### Question 14 — Hands-on — Programmable platform expertise — Kubernetes

Write a minimal **Kubernetes Deployment** snippet that:

* sets CPU and memory **requests**,
* sets higher **limits**,
* runs a single-container app.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: app-image
          resources:
            requests:
              cpu: "500m"
              memory: "100Mi"
            limits:
              cpu: "2"
              memory: "200Mi"
```

### Question 15 — Conceptual — Software development / deployment — CI/CD

Why should **build artifacts** be immutable across environments (dev, staging, prod)?

> Using the same immutable build artifact across environments isolates failures to configuration or environment differences and ensures what was tested is exactly what is deployed.

### Question 16 — Hands-on — Programming — API design

Write a short example (language of your choice) that demonstrates **pagination** in an API response using:

* a **stable cursor**,
* and a **limit** parameter.

```go
type Page struct {
	Items      []Item
	NextCursor string
}

func listItems(afterID string, limit int) Page {
	rows := db.Query(
		`SELECT id, data
		 FROM items
		 WHERE id > ?
		 ORDER BY id
		 LIMIT ?`,
		afterID, limit,
	)

	var items []Item
	var lastID string
	for rows.Next() {
		var it Item
		rows.Scan(&it.ID, &it.Data)
		items = append(items, it)
		lastID = it.ID
	}

	return Page{
		Items:      items,
		NextCursor: lastID,
	}
}
```

This cursor remains stable even if rows are inserted or deleted.
