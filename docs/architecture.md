# HTTP Server Architecture

## Overview

This document describes the architecture, design decisions, and component interactions for the C++ HTTP Server project.

## Design Principles

### 1. Single Responsibility Principle (SRP)
Each class and function has **one clear purpose**. Examples:
- `SocketHandler` only manages socket operations
- `HTTPParser` only parses HTTP requests
- `FileHandler` only reads and serves files

### 2. Top-Down Design
Code flows from high-level orchestration to low-level implementation:

```
main() → Server → Worker Threads → Specific Handlers → Utilities
```

### 3. Dependency Inversion
High-level modules don't depend on low-level details. Use interfaces where appropriate.

### 4. Open/Closed Principle
Designed for extension without modification. New HTTP methods or routes can be added without changing core logic.

## System Architecture

### Layered Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Application Layer                   │
│  ┌────────────────────────────────────────────────┐  │
│  │  main.cpp: Entry point, lifecycle management   │  │
│  │  Server: Orchestrates entire server operation  │  │
│  └────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────┤
│                Business Logic Layer                  │
│  ┌───────────────┬──────────────┬─────────────────┐  │
│  │  HTTPParser   │  HTTPResponse│     Router      │  │
│  │ Parse requests│ Build replies│  URL → Handler  │  │
│  └───────────────┴──────────────┴─────────────────┘  │
├──────────────────────────────────────────────────────┤
│               Infrastructure Layer                   │
│  ┌──────────────┬──────────────┬──────────────────┐  │
│  │SocketHandler │ FileHandler  │     Logger       │  │
│  │  TCP/IP ops  │ File I/O     │  Event logging   │  │
│  └──────────────┴──────────────┴──────────────────┘  │
├──────────────────────────────────────────────────────┤
│                   Utility Layer                      │
│  ┌──────────────┬──────────────┬──────────────────┐  │
│  │   Config     │ StringUtils  │   FileUtils      │  │
│  │ Parse config │ String ops   │ File operations  │  │
│  └──────────────┴──────────────┴──────────────────┘  │
└──────────────────────────────────────────────────────┘
```

## Component Design

### 1. Server (CEO Level)

**Responsibility**: Orchestrate the entire server lifecycle

**Key Methods**:
```cpp
class Server {
public:
    Server(const Config& config);
    void start();           // Initialize and begin listening
    void stop();            // Graceful shutdown
    
private:
    void acceptConnections();     // Main accept loop
    void handleClient(int socket); // Delegate to worker
};
```

**Behavior**:
- Reads configuration
- Initializes socket listener
- Creates thread pool
- Accepts incoming connections
- Dispatches connections to worker threads
- Handles shutdown signals (CTRL+C)

---

### 2. SocketHandler (Manager Level)

**Responsibility**: Manage all low-level socket operations

**Key Methods**:
```cpp
class SocketHandler {
public:
    SocketHandler();
    ~SocketHandler();
    
    bool initialize(int port);
    bool listen(int backlog);
    int acceptConnection();
    std::string receive(int clientSocket);
    bool send(int clientSocket, const std::string& data);
    void closeSocket(int socket);
    
private:
    int serverSocket_;
    // Windows-specific: WSADATA for Winsock initialization
};
```

**Platform Notes**:
- Windows uses Winsock2 (requires `WSAStartup` and `WSACleanup`)
- Handles socket creation, binding, listening, accepting
- Manages socket lifecycle and cleanup

---

### 3. HTTPParser (Worker Level)

**Responsibility**: Parse raw HTTP request into structured data

**Key Methods**:
```cpp
struct HTTPRequest {
    std::string method;        // GET, POST, etc.
    std::string path;          // /index.html
    std::string version;       // HTTP/1.1
    std::map<std::string, std::string> headers;
    std::string body;
};

class HTTPParser {
public:
    static HTTPRequest parse(const std::string& rawRequest);
    
private:
    static std::string parseRequestLine(const std::string& line, HTTPRequest& req);
    static void parseHeaders(const std::vector<std::string>& lines, HTTPRequest& req);
    static std::string parseBody(const std::string& raw, size_t headerEnd);
};
```

**Parsing Strategy**:
1. Split request by `\r\n`
2. Parse first line: `METHOD /path HTTP/1.1`
3. Parse headers: `Key: Value`
4. Extract body (if present)

---

### 4. HTTPResponse (Worker Level)

**Responsibility**: Build valid HTTP responses

**Key Methods**:
```cpp
class HTTPResponse {
public:
    HTTPResponse();
    
    void setStatusCode(int code);
    void setHeader(const std::string& key, const std::string& value);
    void setBody(const std::string& body);
    std::string build();
    
    // Convenience methods
    static HTTPResponse ok(const std::string& body, const std::string& contentType);
    static HTTPResponse notFound();
    static HTTPResponse serverError(const std::string& message);
    
private:
    int statusCode_;
    std::map<std::string, std::string> headers_;
    std::string body_;
    
    std::string getStatusMessage(int code);
};
```

**Response Format**:
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Connection: close

<html>...</html>
```

---

### 5. FileHandler (Worker Level)

**Responsibility**: Read files from disk and determine MIME types

**Key Methods**:
```cpp
class FileHandler {
public:
    FileHandler(const std::string& rootDirectory);
    
    bool fileExists(const std::string& path);
    std::string readFile(const std::string& path);
    std::string getMimeType(const std::string& path);
    
private:
    std::string rootDirectory_;
    std::map<std::string, std::string> mimeTypes_;
    
    void initializeMimeTypes();
    std::string sanitizePath(const std::string& path);
};
```

**Security Considerations**:
- Prevent directory traversal attacks (`../../../etc/passwd`)
- Validate paths stay within root directory
- Return 404 for files outside root

---

### 6. Logger (Infrastructure Level)

**Responsibility**: Record server events for debugging and monitoring

**Key Methods**:
```cpp
enum class LogLevel {
    DEBUG,
    INFO,
    WARNING,
    ERROR
};

class Logger {
public:
    static Logger& getInstance();
    
    void log(LogLevel level, const std::string& message);
    void setLogLevel(LogLevel level);
    void setLogFile(const std::string& filename);
    
private:
    Logger();
    LogLevel currentLevel_;
    std::ofstream logFile_;
    std::mutex logMutex_;  // Thread-safe logging
    
    std::string getCurrentTimestamp();
    std::string levelToString(LogLevel level);
};
```

**Usage Pattern** (Singleton):
```cpp
Logger::getInstance().log(LogLevel::INFO, "Server started on port 8080");
```

---

### 7. Config (Utility Level)

**Responsibility**: Parse and provide configuration values

**Key Methods**:
```cpp
class Config {
public:
    Config();
    bool loadFromFile(const std::string& filename);
    
    int getPort() const;
    std::string getRootDirectory() const;
    int getMaxConnections() const;
    std::string getLogLevel() const;
    
private:
    std::map<std::string, std::string> values_;
    
    void setDefaults();
    std::string trim(const std::string& str);
};
```

**Configuration File Format** (`server.conf`):
```ini
port=8080
root_directory=./public
max_connections=100
log_level=INFO
```

---

## Request Flow (Sequence Diagram)

```
Client                Server          SocketHandler    HTTPParser    FileHandler    HTTPResponse
  |                     |                    |              |              |              |
  |--HTTP Request------>|                    |              |              |              |
  |                     |---accept()-------->|              |              |              |
  |                     |<--clientSocket-----|              |              |              |
  |                     |                    |              |              |              |
  |                     |---receive()------->|              |              |              |
  |                     |<--rawRequest-------|              |              |              |
  |                     |                    |              |              |              |
  |                     |---parse()------------------------>|              |              |
  |                     |<--HTTPRequest---------------------|              |              |
  |                     |                    |              |              |              |
  |                     |---readFile()------------------------------------>|              |
  |                     |<--fileContent------------------------------------|              |
  |                     |                    |              |              |              |
  |                     |---build()------------------------------------------------------>|
  |                     |<--rawResponse---------------------------------------------------|
  |                     |                    |              |              |              |
  |                     |---send()---------->|              |              |              |
  |<--HTTP Response-----|                    |              |              |              |
```

## Threading Model

### Thread Pool Design

```
Main Thread                Worker Threads
     |                          |
     |-- Accept Loop            |
     |     |                    |
     |     |--Connection 1----->| Thread 1: handleClient()
     |     |                    |
     |     |--Connection 2----->| Thread 2: handleClient()
     |     |                    |
     |     |--Connection 3----->| Thread 3: handleClient()
     |                          |
```

**Implementation**:
- Fixed-size thread pool created at startup
- Queue of pending connections
- Worker threads pull from queue
- Thread-safe logger for concurrent access

**Why Thread Pool?**
- Avoids overhead of creating threads per request
- Limits resource usage (memory, CPU)
- Better performance under load

## Error Handling Strategy

### Defensive Programming
- Check return values for all system calls
- Validate all user input (paths, headers)
- Use exceptions for exceptional conditions only

### Error Response Codes
- `400 Bad Request`: Malformed HTTP
- `404 Not Found`: File doesn't exist
- `500 Internal Server Error`: Server-side failures
- `501 Not Implemented`: Unsupported methods (PUT, DELETE)

### Logging Errors
- Log all errors with context
- Include timestamp, client info, request details
- Differentiate between client errors (4xx) and server errors (5xx)

## Testing Strategy

### Unit Tests
- `test_http_parser.cpp`: Test request parsing with various inputs
- `test_http_response.cpp`: Test response building
- `test_file_handler.cpp`: Test file operations and security
- `test_config.cpp`: Test configuration parsing

### Integration Tests
- End-to-end request/response cycle
- Multi-threaded load testing
- Edge cases (large files, slow clients)

### Test Framework
- Google Test (gtest) for unit tests
- Manual testing with `curl` and browsers

## Security Considerations

### Path Traversal Prevention
```cpp
// Bad: /../../etc/passwd
// Good: Normalize and validate paths
std::string FileHandler::sanitizePath(const std::string& path) {
    // Remove .., validate within root
}
```

### Input Validation
- Limit request size (prevent memory exhaustion)
- Validate HTTP method (only allow GET, POST)
- Sanitize headers (prevent injection attacks)

### Resource Limits
- Maximum concurrent connections
- Request timeout (prevent slowloris attacks)
- Maximum file size to serve

## Future Enhancements

### Phase 2 Features
- Router for URL pattern matching
- POST data parsing (form data, JSON)
- CGI script execution

### Phase 3 Features
- HTTPS with OpenSSL/TLS
- HTTP/2 support
- WebSocket support

### Phase 4 Features
- Caching layer (in-memory, file)
- Load balancing (multi-process)
- Admin dashboard

## Performance Considerations

### Optimization Targets
- Minimize memory allocations
- Use string views where possible
- Avoid copying large data
- Efficient file I/O (mmap for large files)

### Benchmarking
- Requests per second (RPS)
- Latency (p50, p95, p99)
- Memory usage under load
- CPU utilization

## Build System (CMake)

### CMake Structure
```cmake
cmake_minimum_required(VERSION 3.15)
project(HTTPServer)

# Organize into libraries
add_library(http_core 
    src/HTTPParser.cpp 
    src/HTTPResponse.cpp
)

add_library(infrastructure
    src/SocketHandler.cpp
    src/FileHandler.cpp
    src/Logger.cpp
)

# Main executable
add_executable(http_server src/main.cpp)
target_link_libraries(http_server http_core infrastructure)

# Link Windows socket library
if(WIN32)
    target_link_libraries(http_server ws2_32)
endif()
```

## References

- [RFC 7230](https://tools.ietf.org/html/rfc7230): HTTP/1.1 Message Syntax and Routing
- [RFC 7231](https://tools.ietf.org/html/rfc7231): HTTP/1.1 Semantics and Content
- [Winsock Programming](https://docs.microsoft.com/en-us/windows/win32/winsock/): Windows socket API
- [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html): Robert C. Martin

---

**Document Version**: 1.0  
**Last Updated**: 30 January 2026  
**Author**: Tshepo Manyisa
