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
