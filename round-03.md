### Question 34 — Conceptual — Programming — Concurrency

What is the difference between **concurrency** and **parallelism**, and why does the distinction matter in system design?

> Concurrency is about structuring and coordinating multiple tasks that make progress over time, while parallelism is about executing tasks simultaneously, and the distinction matters because concurrency introduces coordination and correctness concerns independent of actual parallel execution.

### Question 35 — Hands-on — Programming — Process design / IPC

Write a short example (language of your choice) that demonstrates **inter-process communication (IPC)** using:

* a parent process,
* a child process,
* and a pipe or equivalent mechanism.

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
)

func main() {
	cmd := exec.Command("cat")

	stdin, _ := cmd.StdinPipe()
	cmd.Stdout = os.Stdout

	cmd.Start()
	stdin.Write([]byte("hello child\n"))
	stdin.Close()

	cmd.Wait()
	fmt.Println("done")
}
```

### Question 36 — Conceptual — Programming — Parsing and code generation

Why is it often preferable to use an **AST-based approach** rather than regular expressions for non-trivial parsing tasks?

> Regular expressions only perform pattern matching without structural or contextual understanding, while ASTs represent grammatical structure explicitly, enabling correct, maintainable, and extensible processing of complex languages.

### Question 37 — Hands-on — Programming — Testing / benchmarking

Write a short example (language of your choice) that demonstrates a **micro-benchmark** measuring the performance of a function.

```go
func BenchmarkMyFunc(b *testing.B) {
	for i := 0; i < b.N; i++ {
		_ = myFunc()
	}
}
```

Optionally:

```go
b.ResetTimer()
```

after any setup.

### Question 38 — Conceptual — Programming — Debugging

Why can adding logging or print statements sometimes make a bug “disappear”?

> Logging can change execution timing and memory ordering, masking race conditions or other timing-sensitive bugs and making them appear to disappear.

### Question 39 — Hands-on — Programming — Debugging / profiling

Write a short example (language of your choice) that demonstrates **capturing a stack trace** when an error occurs.

```go
import (
	"fmt"
	"runtime/debug"
)

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Printf("panic: %v\n%s", r, debug.Stack())
		}
	}()

	panic("something went wrong")
}
```

This captures and logs the stack trace at the point of failure.

### Question 40 — Conceptual — Business — Software project management

Why is **scope control** often more critical to project success than schedule optimization?

> Controlling scope reduces uncertainty and coordination overhead by keeping work aligned with clearly understood goals, whereas schedule optimization cannot compensate for poorly defined or continuously expanding requirements.

### Question 41 — Conceptual — Software service implementation — Encryption (disk / transport)

What is the practical difference between **encryption at rest** and **encryption in transit**, and why do most systems need both?

> Encryption at rest protects data from compromise via storage media or physical access, while encryption in transit protects data from interception or tampering over networks, addressing different threat models.

### Question 42 — Hands-on — Programming — Serialization / encoding

Write a short example (language of your choice) that shows **JSON serialization** of a struct or object with an **optional field** that is omitted when empty.

```go
type Payload struct {
	Name  string `json:"name"`
	Email string `json:"email,omitempty"`
}

p := Payload{Name: "Alice"}
b, _ := json.Marshal(p)
fmt.Println(string(b)) // {"name":"Alice"}
```

The `omitempty` tag ensures the optional field is excluded when empty.

### Question 43 — Conceptual — Programming language fluency — Go

Why is it generally discouraged to rely on **`init()` functions** for critical application logic?

> `init()` functions execute implicitly with runtime-defined ordering and hidden side effects, making behavior harder to reason about, test, and control.

### Question 44 — Hands-on — Software development / deployment — Git

Write a short sequence of Git commands that:

* creates a new branch,
* makes a commit,
* and rebases that commit onto `main`.

Assume a clean working tree and that `main` is up to date.

```bash
git checkout -b feature-branch
# make changes
git add .
git commit -m "Add feature"
git fetch origin
git rebase origin/main
```

This creates an isolated branch, commits work, and rebases cleanly onto the latest `main`.

### Question 45 — Conceptual — Programmable platform expertise — Containers / Docker

Why is it considered a best practice to run containers as **non-root** whenever possible?

> Running containers as non-root limits the blast radius of vulnerabilities by preventing container escapes or host compromise through elevated privileges.

### Question 46 — Hands-on — Programming — API design

Write a short example (language of your choice) of an API endpoint or function that demonstrates **idempotency** for a create operation.

```go
func CreateOrder(idempotencyKey string, payload Order) (Order, error) {
	if existing, ok := store.Get(idempotencyKey); ok {
		return existing, nil
	}

	order := processCreate(payload)
	store.Put(idempotencyKey, order) // durable + atomic in practice
	return order, nil
}
```

A durable idempotency key ensures repeated requests return the same result without duplicating side effects.

### Question 47 — Conceptual — Software service implementation — Distributed systems

Why is **clock synchronization** unreliable as a source of truth in distributed systems?

> Because clocks can skew or drift and synchronization protocols are themselves subject to network delays and failures, time becomes observer-dependent and unsafe as a global source of truth.

### Question 48 — Hands-on — Programming — Concurrency

Write a short example (language of your choice) that shows **safe concurrent access** to shared state.

```go
state := 0
updates := make(chan int)

go func() {
	for v := range updates {
		state += v
	}
}()

for i := 0; i < 1000; i++ {
	go func(i int) {
		updates <- i
	}(i)
}
```

A single goroutine owns the state, and all concurrent access is serialized through a channel, avoiding data races.

### Question 49 — Conceptual — Programming — Parsing / meta-programming

Why are **AST-based transformations** generally preferred over regular expressions for non-trivial code manipulation?

> ASTs encode the full syntactic and semantic structure of code, allowing transformations that preserve meaning and correctness, whereas regex operates on raw text without context.

### Question 50 — Hands-on — Software development / deployment — CI/CD

Write a short example of a **GitHub Actions workflow step** that:

* builds a Docker image
* and fails the pipeline if the build fails.

```yaml
- name: Build Docker image
  run: docker build -t my-app:latest .
```

If the build fails, the step exits non-zero and the workflow fails.

### Question 51 — Conceptual — Business — Software licensing

What is a key risk of incorporating **copyleft-licensed** code into a proprietary product?

> Copyleft licenses can impose reciprocal obligations that require disclosure or licensing of derivative source code, undermining proprietary distribution models.

### Question 52 — Hands-on — Programming — Testing

Write a short example (language of your choice) that demonstrates a **table-driven test**.

```go
func TestAdd(t *testing.T) {
	tests := []struct {
		name     string
		a, b int
		want int
	}{
		{"both positive", 1, 2, 3},
		{"with zero", 0, 5, 5},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			if got := Add(tt.a, tt.b); got != tt.want {
				t.Fatalf("got %d, want %d", got, tt.want)
			}
		})
	}
}
```

### Question 53 — Conceptual — Software service implementation — Observability

Why is **structured logging** preferred over plain text logs in distributed systems?

> Structured logging emits machine-parseable fields that enable efficient indexing, querying, and correlation across systems without brittle post-processing.
