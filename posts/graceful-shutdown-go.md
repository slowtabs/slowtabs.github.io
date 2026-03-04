# Graceful Shutdown Patterns in Go HTTP Servers

How to handle SIGINT and SIGTERM properly so in-flight requests finish before your process exits.

## The problem

You deploy a new version of your Go HTTP server. The process receives SIGTERM. What happens to the requests currently being handled?

By default: they get killed mid-response. The client sees a connection reset. If the request was writing to a database, you might have a partial write.

## The naive server

```go
func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

`ListenAndServe` blocks until the process is killed. There's no opportunity to clean up.

## The graceful version

Go 1.8 added `http.Server.Shutdown()`, which stops accepting new connections and waits for in-flight requests to complete:

```go
func main() {
    srv := &http.Server{Addr: ":8080"}
    http.HandleFunc("/", handler)

    // Start server in a goroutine
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatalf("Server error: %v", err)
        }
    }()

    // Wait for interrupt signal
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit

    log.Println("Shutting down server...")

    // Give outstanding requests 30 seconds to complete
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()

    if err := srv.Shutdown(ctx); err != nil {
        log.Fatalf("Forced shutdown: %v", err)
    }

    log.Println("Server stopped gracefully")
}
```

## Key details

### The shutdown timeout

The `context.WithTimeout` is critical. Without it, a slow or stuck request could prevent your server from ever shutting down. 30 seconds is a common default, but tune it for your workload.

### What `Shutdown` actually does

1. Closes the listener (stops accepting new connections)
2. Closes idle connections immediately  
3. Waits for active connections to become idle (i.e., in-flight requests to finish)
4. Returns when all connections are closed, or when the context expires

### Health check endpoints

If you're behind a load balancer, you need to coordinate the shutdown:

```go
var isShuttingDown atomic.Bool

http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    if isShuttingDown.Load() {
        w.WriteHeader(http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
})
```

Set `isShuttingDown` to `true` as soon as you receive the signal, *before* calling `Shutdown()`. This gives the load balancer time to stop routing traffic to this instance.

### Database connections

Don't forget to close database connections and other resources:

```go
<-quit
isShuttingDown.Store(true)

// Shutdown HTTP server
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()
srv.Shutdown(ctx)

// Close database
db.Close()

// Flush logs, metrics, etc.
logger.Sync()
```

## Testing graceful shutdown

You can test this with a slow handler and `curl`:

```bash
# Terminal 1: Start server
go run main.go

# Terminal 2: Send a slow request
curl localhost:8080/slow  # handler that sleeps 10 seconds

# Terminal 3: Send SIGTERM while request is in-flight
kill -TERM $(pgrep myserver)
```

The slow request should complete successfully, and the server should exit after it finishes.

## Production checklist

- [ ] Server uses `Shutdown()` with a timeout
- [ ] Health endpoint returns 503 during shutdown
- [ ] Database and external connections are closed after HTTP shutdown
- [ ] Shutdown timeout is longer than your slowest expected request
- [ ] Container orchestrator's `terminationGracePeriodSeconds` is longer than your shutdown timeout
