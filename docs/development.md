# Development Guide

## Project Setup Checklist

### Prerequisites Installation

#### 1. Git
- Download: [git-scm.com](https://git-scm.com/)
- Verify: `git --version`

#### 2. C++ Compiler (Choose one)

**Option A: MinGW-w64 (Recommended for pure C++)**
```bash
# Download from: https://www.mingw-w64.org/downloads/
# Or use MSYS2: https://www.msys2.org/
# Install and add to PATH: C:\msys64\mingw64\bin

# Verify:
g++ --version
```

**Option B: Visual Studio Community (Full IDE)**
```
# Download: https://visualstudio.microsoft.com/
# During install, select:
# - Desktop development with C++
# - Windows 10 SDK
# - C++ CMake tools
```

#### 3. CMake
```bash
# Download: https://cmake.org/download/
# Install and check "Add CMake to PATH"

# Verify:
cmake --version  # Should be 3.15+
```

#### 4. VS Code (Recommended editor)
```bash
# Download: https://code.visualstudio.com/

# Install extensions:
# - C/C++ (Microsoft)
# - CMake Tools (Microsoft)
# - CMake (twxs)
# - GitLens (optional but helpful)
```

### Initial Repository Setup

#### 1. Create GitHub Repository
```bash
# On GitHub.com:
# - Click "New Repository"
# - Name: cpp-http-server
# - Visibility: Public
# - Initialize: Add README, .gitignore (C++), License (MIT)
```

#### 2. Clone and Setup Local Directory
```bash
# Clone repository
git clone https://github.com/yourusername/cpp-http-server.git
cd cpp-http-server

# Create directory structure
mkdir include src tests docs public logs
mkdir docs/adr
mkdir .github
mkdir .github/workflows

# Initialize basic files (we've already created these)
# - .gitignore
# - README.md
# - server.conf
# - public/index.html
```

#### 3. Configure Git
```bash
# Set your identity
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Recommended settings
git config --global core.autocrlf true  # Windows line endings
git config --global init.defaultBranch main
```

## Development Workflow

### Branch Strategy

```
main              (stable, deployable)
  ├── develop     (integration branch)
      ├── feature/logger
      ├── feature/socket-handler
      ├── feature/http-parser
      └── feature/file-handler
```

#### Creating a Feature Branch
```bash
# Switch to develop
git checkout develop

# Create and switch to feature branch
git checkout -b feature/logger

# Work on your feature...

# When done, commit and push
git add .
git commit -m "feat: implement thread-safe logger"
git push origin feature/logger

# Create Pull Request on GitHub
# After approval, merge to develop
```

### Commit Message Convention

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code formatting (no logic change)
- `refactor`: Code restructure (no behavior change)
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples**:
```bash
git commit -m "feat: add HTTPParser class"
git commit -m "fix: handle empty request headers"
git commit -m "docs: update README with build instructions"
git commit -m "test: add unit tests for HTTPResponse"
git commit -m "refactor: extract MIME type logic to separate function"
```

## Build Process

### Step-by-Step Build

#### 1. Configure Build (First time only)
```bash
# From project root
mkdir build
cd build

# Generate build files
cmake ..

# For specific build type:
cmake .. -DCMAKE_BUILD_TYPE=Debug
cmake .. -DCMAKE_BUILD_TYPE=Release
```

#### 2. Compile Code
```bash
# Build everything
cmake --build .

# Build specific target
cmake --build . --target http_server
cmake --build . --target http_core

# Parallel build (faster)
cmake --build . --parallel 4
```

#### 3. Run Server
```bash
# From build directory
./http_server              # Linux/Mac
http_server.exe            # Windows

# Or with config file path
./http_server ../server.conf
```

### Clean Build
```bash
# Option 1: Delete build directory
cd ..
rm -rf build
mkdir build
cd build
cmake ..

# Option 2: CMake clean
cd build
cmake --build . --target clean
```

## Development Phases

### Phase 1: Foundation (Week 1)

**Goal**: Set up project infrastructure

**Tasks**:
- [x] Create repository and directory structure
- [x] Write design documents (architecture.md, ADRs)
- [ ] Create initial CMakeLists.txt
- [ ] Implement Config class (parse server.conf)
- [ ] Implement Logger class (thread-safe logging)
- [ ] Write unit tests for Config and Logger
- [ ] Set up GitHub Actions for CI

**Deliverable**: Working build system, configuration, and logging

**Test**: Can parse config and log messages

### Phase 2: Socket Foundation (Week 1-2)

**Goal**: Basic networking capability

**Tasks**:
- [ ] Implement SocketHandler class (Winsock on Windows)
- [ ] Create socket, bind to port, listen
- [ ] Accept single connection
- [ ] Receive and send raw data
- [ ] Write unit tests
- [ ] Create echo server demo

**Deliverable**: Echo server that accepts one connection

**Test**: `telnet localhost 8080` → type message → see echo

### Phase 3: HTTP Protocol (Week 2-3)

**Goal**: Parse and respond to HTTP requests

**Tasks**:
- [ ] Implement HTTPParser
  - Parse request line (method, path, version)
  - Parse headers
  - Extract body
- [ ] Implement HTTPResponse
  - Build response line
  - Add headers
  - Set body
- [ ] Handle GET requests
- [ ] Return "Hello World" HTTP response
- [ ] Write comprehensive tests

**Deliverable**: Server responds with valid HTTP

**Test**: `curl http://localhost:8080` → see "Hello World"

### Phase 4: File Serving (Week 3)

**Goal**: Serve static files

**Tasks**:
- [ ] Implement FileHandler
  - Read files from disk
  - MIME type detection
  - Path sanitization (security)
- [ ] Integrate with HTTPResponse
- [ ] Handle 404 Not Found
- [ ] Serve index.html by default
- [ ] Directory listing (optional)

**Deliverable**: Serves static HTML, CSS, JS, images

**Test**: Open browser → http://localhost:8080 → see index.html

### Phase 5: Multi-threading (Week 4)

**Goal**: Handle concurrent connections

**Tasks**:
- [ ] Implement thread pool
- [ ] Connection queue
- [ ] Worker thread logic
- [ ] Thread-safe resource access
- [ ] Graceful shutdown
- [ ] Load testing

**Deliverable**: Handle multiple clients simultaneously

**Test**: Apache Bench → `ab -n 1000 -c 10 http://localhost:8080/`

### Phase 6: Advanced Features (Week 4-5)

**Goal**: POST requests and routing

**Tasks**:
- [ ] Implement Router class
- [ ] URL pattern matching
- [ ] POST request parsing
- [ ] Form data handling
- [ ] Query parameter parsing
- [ ] Custom error pages

**Deliverable**: Full HTTP/1.1 GET and POST support

**Test**: Submit HTML form → server processes data

### Phase 7: Polish and Documentation (Week 5)

**Goal**: Production-ready quality

**Tasks**:
- [ ] Comprehensive error handling
- [ ] Performance optimization
- [ ] Memory leak testing (Valgrind/AddressSanitizer)
- [ ] Generate Doxygen documentation
- [ ] Create Docker container
- [ ] Write detailed README
- [ ] Record demo video

**Deliverable**: Portfolio-ready project

**Test**: Code review, performance benchmarks

## Testing Strategy

### Unit Tests with Google Test

#### Installing Google Test
```bash
# Using vcpkg (recommended)
vcpkg install gtest

# Or download manually from GitHub
# https://github.com/google/googletest
```

#### Writing Tests

**Example: test_http_parser.cpp**
```cpp
#include <gtest/gtest.h>
#include "HTTPParser.h"

TEST(HTTPParserTest, ParseGETRequest) {
    std::string rawRequest = 
        "GET /index.html HTTP/1.1\r\n"
        "Host: localhost:8080\r\n"
        "User-Agent: Test\r\n"
        "\r\n";
    
    HTTPRequest req = HTTPParser::parse(rawRequest);
    
    EXPECT_EQ(req.method, "GET");
    EXPECT_EQ(req.path, "/index.html");
    EXPECT_EQ(req.version, "HTTP/1.1");
    EXPECT_EQ(req.headers["Host"], "localhost:8080");
}

TEST(HTTPParserTest, HandleMalformedRequest) {
    std::string badRequest = "INVALID REQUEST";
    
    EXPECT_THROW(HTTPParser::parse(badRequest), std::runtime_error);
}
```

#### Running Tests
```bash
cd build
ctest --output-on-failure

# Or run test executable directly
./tests/test_http_parser
```

### Integration Testing

#### Manual Testing with curl
```bash
# GET request
curl http://localhost:8080/

# GET with headers
curl -H "Custom-Header: value" http://localhost:8080/

# POST request
curl -X POST -d "key=value" http://localhost:8080/api

# Save response
curl -o output.html http://localhost:8080/index.html
```

#### Load Testing with Apache Bench
```bash
# Install Apache Bench
# Windows: Download from Apache website
# Linux: apt-get install apache2-utils

# Run load test
ab -n 1000 -c 10 http://localhost:8080/
# -n 1000: 1000 total requests
# -c 10: 10 concurrent connections
```

### Memory Testing

#### AddressSanitizer (Compile-time)
```bash
# Add to CMakeLists.txt
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")

# Build and run
cmake --build .
./http_server
# Will detect memory leaks, buffer overflows, etc.
```

## Debugging

### VS Code Debugging

#### launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug HTTP Server",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/http_server.exe",
            "args": [],
            "cwd": "${workspaceFolder}",
            "environment": [],
            "MIMode": "gdb",
            "miDebuggerPath": "C:/msys64/mingw64/bin/gdb.exe"
        }
    ]
}
```

### Common Issues and Solutions

#### Issue: "Winsock initialization failed"
```cpp
// Solution: Call WSAStartup before socket operations
WSADATA wsaData;
if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
    // Handle error
}
```

#### Issue: "Address already in use"
```cpp
// Solution: Enable SO_REUSEADDR
int opt = 1;
setsockopt(serverSocket, SOL_SOCKET, SO_REUSEADDR, 
           (char*)&opt, sizeof(opt));
```

#### Issue: "Cannot find header files"
```cmake
# Solution: Add include directory to CMakeLists.txt
include_directories(${PROJECT_SOURCE_DIR}/include)
```

## Code Review Checklist

Before committing code, verify:

### Functionality
- [ ] Code compiles without warnings
- [ ] All tests pass
- [ ] Feature works as intended
- [ ] Edge cases handled

### Code Quality
- [ ] Follows Single Responsibility Principle
- [ ] Functions are small (<50 lines ideally)
- [ ] Meaningful variable names
- [ ] Comments explain "why", not "what"
- [ ] No magic numbers (use constants)

### Performance
- [ ] No memory leaks
- [ ] Efficient algorithms
- [ ] Avoid unnecessary copies
- [ ] Resource cleanup (RAII)

### Security
- [ ] Input validation
- [ ] Path sanitization
- [ ] No buffer overflows
- [ ] Error messages don't leak info

### Documentation
- [ ] Public functions have comments
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] Architecture docs updated

## Resources

### Learning C++
- [LearnCpp.com](https://www.learncpp.com/)
- [CPP Reference](https://en.cppreference.com/)
- [Modern C++ Features](https://github.com/AnthonyCalandra/modern-cpp-features)

### Network Programming
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)
- [Winsock Programming](https://docs.microsoft.com/en-us/windows/win32/winsock/)

### HTTP Protocol
- [HTTP/1.1 RFC 7230](https://tools.ietf.org/html/rfc7230)
- [MDN HTTP Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP)

### Tools
- [CMake Documentation](https://cmake.org/documentation/)
- [Google Test Primer](https://google.github.io/googletest/primer.html)
- [Valgrind Manual](https://valgrind.org/docs/manual/)

## Getting Help

### Self-Help Steps
1. Read error messages carefully
2. Check documentation (code comments, this guide)
3. Search for error online (Stack Overflow)
4. Use debugger to inspect values
5. Add logging to trace execution

### Asking for Help
When stuck, provide:
- What you're trying to do
- What you expected
- What actually happened
- Error messages (full text)
- Code snippet (minimal example)
- What you've already tried

---

**Remember**: Professional development is iterative. Build incrementally, test continuously, document thoroughly. You've got this! 🚀