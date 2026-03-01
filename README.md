# goflow

[![Go Reference](https://pkg.go.dev/badge/github.com/1mb-dev/goflow.svg)](https://pkg.go.dev/github.com/1mb-dev/goflow)
[![Go Report Card](https://goreportcard.com/badge/github.com/1mb-dev/goflow)](https://goreportcard.com/report/github.com/1mb-dev/goflow)
[![CI](https://github.com/1mb-dev/goflow/actions/workflows/ci.yml/badge.svg)](https://github.com/1mb-dev/goflow/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/1mb-dev/goflow/branch/main/graph/badge.svg)](https://codecov.io/gh/1mb-dev/goflow)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A Go library for concurrent applications with rate limiting, task scheduling, and streaming.

## Features

**Rate Limiting** (`pkg/ratelimit`)
- Token bucket and leaky bucket algorithms
- Concurrency limiting
- Prometheus metrics

**Task Scheduling** (`pkg/scheduling`)
- Worker pools with graceful shutdown
- Cron-based scheduling
- Multi-stage pipelines
- Context-aware timeouts

**Streaming** (`pkg/streaming`)
- Functional stream operations
- Background buffering
- Backpressure control
- Channel utilities

## When to Use This

Use goflow when you need rate limiting, worker pools, or task scheduling in Go and want them from a single, composable library.

**Choose goflow over individual libraries ([uber-go/ratelimit](https://github.com/uber-go/ratelimit), [gammazero/workerpool](https://github.com/gammazero/workerpool)) when:**
- You need rate limiting + worker pools + scheduling together and want consistent APIs across all three
- You want built-in Prometheus metrics without extra wiring
- You want multiple rate limiting algorithms (token bucket, leaky bucket, concurrency) in one package

**Choose individual libraries instead when:**
- You only need one primitive (e.g., just a rate limiter) and want the smallest dependency footprint
- You need a specific algorithm not covered here

**Use stdlib alone when:**
- `sync.WaitGroup` and channels are enough for your concurrency pattern
- You don't need rate limiting or scheduling

## Installation

```bash
go get github.com/1mb-dev/goflow
```

## Usage

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"
    
    "github.com/1mb-dev/goflow/pkg/ratelimit/bucket"
    "github.com/1mb-dev/goflow/pkg/scheduling/workerpool"
    "github.com/1mb-dev/goflow/pkg/scheduling/scheduler"
)

func main() {
    limiter, err := bucket.NewSafe(10, 20) // 10 RPS, burst 20
    if err != nil {
        log.Fatal(err)
    }

    pool, err := workerpool.NewWithConfigSafe(workerpool.Config{
        WorkerCount: 5,
        QueueSize:   100,
        TaskTimeout: 30 * time.Second,
    })
    if err != nil {
        log.Fatal(err)
    }
    defer func() { <-pool.Shutdown() }()

    if limiter.Allow() {
        task := workerpool.TaskFunc(func(ctx context.Context) error {
            fmt.Println("Processing request...")
            return nil
        })

        if err := pool.Submit(task); err != nil {
            log.Printf("Failed to submit task: %v", err)
        }
    }
}
```

## Components

**Rate Limiters**
- `bucket.NewSafe(rate, burst)` - Token bucket with burst capacity
- `leakybucket.New(rate)` - Smooth rate limiting
- `concurrency.NewSafe(limit)` - Concurrent operations control

**Scheduling**
- `workerpool.NewSafe(workers, queueSize)` - Background task processing
- `scheduler.New()` - Cron and interval scheduling

**Streaming**
- `stream.FromSlice(data)` - Functional data processing
- `writer.New(config)` - Async buffered writing

## Documentation

- [Documentation Site](https://1mb-dev.github.io/goflow/)
- [API Reference](https://pkg.go.dev/github.com/1mb-dev/goflow)
- [Examples](./examples/)

## Development

```bash
make install-hooks  # Install pre-commit hook
make test           # Run tests with race detection
make lint           # Run linter
make benchmark      # Run performance benchmarks
```

The pre-commit hook automatically:
- Checks for potential secrets
- Formats Go code with `goimports` and `gofmt`
- Runs `golangci-lint` on changed files
- Verifies the build succeeds

## Contributing

See [Contributing](https://1mb-dev.github.io/goflow/contributing/) for contribution guidelines.

## License

MIT License - see [LICENSE](LICENSE) for details.