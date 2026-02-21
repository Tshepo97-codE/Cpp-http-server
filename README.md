# C++ HTTP Server

A lightweight, educational HTTP/1.1 server implementation in modern C++ demonstrating clean architecture and professional development practices.

## 🎯 Project Goals

- Learn network programming fundamentals (TCP/IP, sockets)
- Implement HTTP/1.1 protocol from scratch
- Apply SOLID principles and clean code practices
- Practice professional development workflow (Git, testing, CI/CD)
- Build portfolio-worthy project with comprehensive documentation

## ✨ Features

### Current Version (v0.1.0 - In Development)
- [ ] TCP socket server (multi-threaded)
- [ ] HTTP/1.1 GET request handling
- [ ] Static file serving (HTML, CSS, JS, images)
- [ ] MIME type detection
- [ ] Configurable via `server.conf`
- [ ] Comprehensive logging system
- [ ] Error handling (404, 500, etc.)

### Future Enhancements
- [ ] POST request handling
- [ ] URL routing system
- [ ] Query parameter parsing
- [ ] HTTP/2 support
- [ ] HTTPS/TLS encryption
- [ ] CGI/FastCGI support

## 🏗️ Architecture

This server follows a **layered architecture** with clear separation of concerns:

```
┌─────────────────────────────────────┐
│         Application Layer           │
│         (main.cpp, Server)          │
├─────────────────────────────────────┤
│        Business Logic Layer         │
│    (Router, HTTPParser, Response)   │
├─────────────────────────────────────┤
│       Infrastructure Layer          │
│  (SocketHandler, FileHandler, Log)  │
├─────────────────────────────────────┤
│          Utility Layer              │
│  (Config, StringUtils, FileUtils)   │
└─────────────────────────────────────┘
```

See [docs/architecture.md](docs/architecture.md) for detailed design decisions.

## 🚀 Quick Start

### Prerequisites

- **Compiler**: MinGW-w64 (GCC 9.0+) or MSVC (Visual Studio 2019+)
- **CMake**: Version 3.15 or higher
- **Git**: For version control
- **vcpkg**: For dependency management (optional but recommended)

### Building from Source

```bash
# Clone the repository
git clone https://github.com/Tshepo97-codE/cpp-http-server.git
cd cpp-http-server

# Create build directory
mkdir build
cd build

# Generate build files with CMake
cmake ..

# Build the project
cmake --build .

# Run the server
./http_server
```

### Configuration

Edit `server.conf` to customize server behavior:

```ini
port=8080
root_directory=./public
max_connections=100
log_level=INFO
```

### Usage

1. Start the server:
   ```bash
   ./http_server
   ```

2. Open your browser and navigate to:
   ```
   http://localhost:8080
   ```

3. Place static files in the `public/` directory to serve them.

## 🧪 Testing

```bash
# Run all tests
cd build
ctest --output-on-failure

# Run specific test suite
./tests/test_http_parser
```

## 📖 Documentation

- [Architecture Overview](docs/architecture.md)
- [API Documentation](docs/api.md) (Generated with Doxygen)
- [Architecture Decision Records](docs/adr/)
- [Development Guide](docs/development.md)

## 🛠️ Development

### Project Structure

```
cpp-http-server/
├── include/          # Header files (.h)
├── src/              # Implementation files (.cpp)
├── tests/            # Unit tests
├── docs/             # Documentation
├── public/           # Static files to serve
└── .github/          # GitHub Actions workflows
```

### Code Style

- Follow [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
- Use meaningful names that describe intent
- Keep functions small and focused (Single Responsibility Principle)
- Comment complex logic and design decisions

### Git Workflow

1. Create feature branch: `git checkout -b feature/your-feature`
2. Make changes with descriptive commits: `git commit -m "feat: add HTTP parser"`
3. Push and create Pull Request
4. Merge after review

### Commit Message Format

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add new feature
fix: fix bug
docs: update documentation
test: add tests
refactor: refactor code
style: formatting changes
chore: maintenance tasks
```

## 🤝 Contributing

This is an educational project, but suggestions and improvements are welcome!

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

## 📝 License

MIT License - See [LICENSE](LICENSE) file for details

## 🙏 Acknowledgments

- Inspired by various HTTP server implementations
- Built as a learning project for understanding network programming
- Thanks to the C++ community for excellent resources

## 📧 Contact

Your Name - Tshepo Manyisa

Project Link: [https://github.com/Tshepo97-codE/cpp-http-server](https://github.com/Tshepo97-codE/cpp-http-server)

---

**Note**: This is an educational project. For production use, consider mature solutions like nginx, Apache, or Node.js.