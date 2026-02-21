# ADR-001: Use Thread Pool for Connection Handling

**Date**: 2026-01-12  
**Status**: Accepted  
**Context**: Initial design phase

## Context

Our HTTP server needs to handle multiple client connections concurrently. We need to decide on a concurrency model that balances performance, complexity, and learning value for a student project.

## Decision

We will implement a **fixed-size thread pool** to handle client connections.

## Options Considered

### Option 1: Single-Threaded (Blocking I/O)
**Pros**:
- Simplest to implement
- No concurrency issues
- Easy to debug

**Cons**:
- Can only handle one client at a time
- Poor performance
- Not realistic for real-world servers

### Option 2: Thread-per-Connection
**Pros**:
- Simple mental model
- Natural isolation between requests
- Easy to implement

**Cons**:
- Thread creation overhead for each connection
- Unbounded resource usage (memory, CPU)
- Can exhaust system resources under load
- Not scalable

### Option 3: Thread Pool (CHOSEN)
**Pros**:
- ✅ Bounded resource usage
- ✅ Good performance (reuse threads)
- ✅ Scalable to reasonable loads
- ✅ Teaches important concurrency patterns
- ✅ Realistic approach used in production

**Cons**:
- More complex than single-threaded
- Requires thread synchronization (queue)
- Need to handle thread-safe logging

### Option 4: Async I/O (epoll/IOCP)
**Pros**:
- Best performance for high concurrency
- Single-threaded simplicity in some ways

**Cons**:
- Very complex to implement correctly
- Platform-specific (epoll on Linux, IOCP on Windows)
- Harder to understand for learning purposes
- Overkill for this project's scope

## Rationale

We chose **Option 3 (Thread Pool)** because:

1. **Learning Value**: Teaches thread management, synchronization, and concurrent programming without excessive complexity
2. **Realistic**: This approach is used in production servers (Apache worker MPM, Java servlet containers)
3. **Performance**: Good balance between simplicity and performance
4. **Resource Management**: Bounded threads prevent resource exhaustion
5. **Maintainability**: Clear separation between thread management and request handling

## Implementation Details

### Thread Pool Size
- Default: 10 worker threads
- Configurable via `server.conf`
- Rule of thumb: 2x number of CPU cores for I/O-bound workload

### Queue Management
- Blocking queue for pending connections
- If queue is full, reject new connections (send 503 Service Unavailable)
- Queue size: 2x thread pool size

### Thread Safety
- Logger must be thread-safe (use mutex)
- Avoid shared mutable state where possible
- Each thread gets its own SocketHandler instance for client communication

### Graceful Shutdown
- Signal handler for CTRL+C
- Set shutdown flag
- Wait for all threads to complete current requests
- Join all threads before exiting

## Consequences

### Positive
- Server can handle multiple clients simultaneously
- Predictable resource usage
- Good foundation for learning concurrency
- Can easily test with tools like Apache Bench (`ab`)

### Negative
- More complex than single-threaded
- Need to implement thread-safe logging
- Must handle synchronization correctly
- Potential for deadlocks if not careful (will mitigate with RAII and clear locking rules)

### Neutral
- Will need C++11 or later (for `<thread>`, `<mutex>`)
- Platform-specific thread APIs not needed (C++ standard library sufficient)

## Future Considerations

If we later need to handle thousands of concurrent connections:
- Could migrate to async I/O (libuv, Boost.Asio)
- Could implement hybrid model (thread pool + async I/O per thread)
- Could add connection pooling and keep-alive support

For now, thread pool is the right choice for our educational and performance goals.

## References

- [POSIX Threads Programming](https://computing.llnl.gov/tutorials/pthreads/)
- [C++ Threading](https://en.cppreference.com/w/cpp/thread)
- [Java Thread Pool Pattern](https://docs.oracle.com/javase/tutorial/essential/concurrency/pools.html)
- [Apache MPM Worker](https://httpd.apache.org/docs/2.4/mod/worker.html)

## Review Notes

- Approved by: Tshepo Manyisa
- Reviewed by: 
- Next review: After implementing basic server, evaluate if thread count should be tuned