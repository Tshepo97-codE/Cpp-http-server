# CMake Tutorial - Understanding Build Systems

## What is CMake?

**CMake** is a **build system generator**. It doesn't compile your code directly - instead, it generates build files (like Makefiles or Visual Studio projects) that *then* compile your code.

### Why Use CMake?

**Without CMake** (manual compilation):
```bash
g++ -c src/main.cpp -o main.o
g++ -c src/Server.cpp -o Server.o
g++ -c src/SocketHandler.cpp -o SocketHandler.o
# ... repeat for every file
g++ main.o Server.o SocketHandler.o -o http_server -lws2_32
# Have to remember every flag, every file, every dependency
```

**With CMake**:
```bash
cmake ..
cmake --build .
# CMake figures out all dependencies automatically
```

### Key Concepts

1. **CMakeLists.txt**: Recipe file that tells CMake how to build your project
2. **Out-of-source builds**: Build files go in a separate `build/` directory (keeps source clean)
3. **Targets**: Things CMake builds (executables, libraries)
4. **Dependencies**: CMake tracks which files depend on which headers

## Installing CMake on Windows

### Option 1: Download from Website
1. Visit [cmake.org/download](https://cmake.org/download/)
2. Download "Windows x64 Installer"
3. Run installer, check "Add CMake to system PATH"
4. Verify: Open Command Prompt, type `cmake --version`

### Option 2: Chocolatey (if you have it)
```bash
choco install cmake
```

### Option 3: Visual Studio Installer
If you have Visual Studio, CMake might already be installed as part of "C++ CMake tools"

## CMake Workflow

```
Step 1: Write CMakeLists.txt  →  Step 2: Configure  →  Step 3: Build
   (describe project)            (generate build)      (compile code)
                                  
   CMakeLists.txt                cmake ..              cmake --build .
   (written once)                (run once)            (run every time)
```

## Basic CMakeLists.txt Structure

Here's a simple example:

```cmake
# Minimum CMake version required
cmake_minimum_required(VERSION 3.15)

# Project name and version
project(HTTPServer VERSION 0.1.0)

# Tell CMake to use C++17 standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Create an executable called "http_server" from these files
add_executable(http_server
    src/main.cpp
    src/Server.cpp
)
```

### Breaking It Down

**Line 1**: `cmake_minimum_required(VERSION 3.15)`
- Says "CMake 3.15 or newer required"
- Ensures features we use are available

**Line 4**: `project(HTTPServer VERSION 0.1.0)`
- Names your project "HTTPServer"
- Sets version number (useful for packaging)

**Lines 7-8**: C++ Standard
- We're using C++17 features (like `std::filesystem`)
- `REQUIRED` means CMake will error if compiler doesn't support it

**Lines 11-14**: `add_executable`
- Creates a build target called `http_server`
- Lists all `.cpp` files needed
- CMake automatically finds matching `.h` files

## Our Full CMakeLists.txt

For our HTTP server project, we'll use a more organized structure:

```cmake
cmake_minimum_required(VERSION 3.15)
project(HTTPServer VERSION 0.1.0 LANGUAGES CXX)

# C++ Standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Compiler warnings (good practice)
if(MSVC)
    add_compile_options(/W4)  # Warning level 4 for MSVC
else()
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Include directories (where .h files are)
include_directories(include)

# Core HTTP library (reusable components)
add_library(http_core
    src/HTTPParser.cpp
    src/HTTPResponse.cpp
    src/Router.cpp
)

# Infrastructure library (I/O, logging)
add_library(infrastructure
    src/SocketHandler.cpp
    src/FileHandler.cpp
    src/Logger.cpp
    src/Config.cpp
)

# Main server library
add_library(server
    src/Server.cpp
)

# Link dependencies: server needs http_core and infrastructure
target_link_libraries(server http_core infrastructure)

# Main executable
add_executable(http_server src/main.cpp)
target_link_libraries(http_server server)

# Windows-specific: Link Winsock library
if(WIN32)
    target_link_libraries(http_server ws2_32)
endif()

# Testing (optional, we'll add later)
enable_testing()
add_subdirectory(tests)

# Installation rules (optional)
install(TARGETS http_server DESTINATION bin)
```

### Advanced Concepts Explained

**`add_library` vs `add_executable`**:
- `add_library`: Creates a `.lib` file (collection of compiled code)
- `add_executable`: Creates a `.exe` file (runnable program)
- Libraries are reusable, executables are entry points

**`target_link_libraries`**:
- Tells CMake "this target needs that library"
- Example: `http_server` needs `server`, which needs `http_core`
- CMake handles linking order automatically

**`if(WIN32)`**:
- Conditional compilation
- `WIN32` is true on Windows
- Useful for platform-specific code (Winsock on Windows)

**`include_directories`**:
- Tells compiler where to find `.h` files
- Example: `#include "Server.h"` looks in `include/` directory

## Using CMake - Step by Step

### Step 1: Create build directory
```bash
cd cpp-http-server
mkdir build
cd build
```

Why separate directory?
- Keeps build artifacts separate from source
- Can delete `build/` to clean everything
- Can have multiple build directories (Debug, Release)

### Step 2: Configure (Generate build files)
```bash
cmake ..
```

What this does:
- Reads `CMakeLists.txt` from parent directory (`..`)
- Checks for compiler (g++, MSVC)
- Generates build files (Makefiles, Visual Studio solution)
- Creates `CMakeCache.txt` (remembers settings)

Options you can add:
```bash
cmake .. -DCMAKE_BUILD_TYPE=Debug         # Debug build
cmake .. -DCMAKE_BUILD_TYPE=Release       # Optimized build
cmake .. -G "Visual Studio 17 2022"       # Specific generator
```

### Step 3: Build (Compile code)
```bash
cmake --build .
```

What this does:
- Uses generated build files to compile
- Links libraries and executables
- Shows compilation progress
- Creates `http_server.exe` in `build/` directory

Options:
```bash
cmake --build . --config Debug            # Debug build (MSVC)
cmake --build . --config Release          # Release build (MSVC)
cmake --build . --target http_server      # Build specific target
cmake --build . --parallel 4              # Use 4 cores
```

### Step 4: Run your program
```bash
./http_server          # Linux/Mac
http_server.exe        # Windows
```

## Common CMake Commands

### Cleaning Build
```bash
# Option 1: Delete build directory
rm -rf build/
mkdir build

# Option 2: CMake clean target
cmake --build . --target clean
```

### Verbose Output (See actual compile commands)
```bash
cmake --build . --verbose
```

### List Available Targets
```bash
cmake --build . --target help
```

## Troubleshooting

### "CMake not found"
- Install CMake (see installation section)
- Make sure it's in PATH

### "Could not find compiler"
- Windows: Install MinGW-w64 or Visual Studio
- Make sure compiler is in PATH
- Specify generator: `cmake .. -G "MinGW Makefiles"`

### "Cannot find header file"
- Check `include_directories` in CMakeLists.txt
- Verify file paths are correct
- Use forward slashes `/` even on Windows

### Build fails with linker errors
- Check `target_link_libraries` order
- On Windows, make sure `ws2_32` is linked
- Verify all `.cpp` files are listed in `add_library`/`add_executable`

## CMake Variables (Useful)

```cmake
# Project info
${PROJECT_NAME}           # "HTTPServer"
${PROJECT_VERSION}        # "0.1.0"

# Directories
${CMAKE_SOURCE_DIR}       # Top-level source directory
${CMAKE_BINARY_DIR}       # Top-level build directory
${CMAKE_CURRENT_SOURCE_DIR}  # Current CMakeLists.txt location

# Build type
${CMAKE_BUILD_TYPE}       # Debug, Release, etc.

# Compiler
${CMAKE_CXX_COMPILER}     # Path to C++ compiler

# Platform
WIN32                     # True on Windows
UNIX                      # True on Linux/Mac
APPLE                     # True on Mac
```

## Best Practices

1. **Always use out-of-source builds** (separate `build/` directory)
2. **Organize code into libraries** (easier to test, reuse)
3. **Use `target_link_libraries`** instead of manual linking
4. **Set C++ standard** (`CMAKE_CXX_STANDARD`)
5. **Enable warnings** (`-Wall`, `/W4`)
6. **Use `if(WIN32)` for platform-specific code**
7. **Keep CMakeLists.txt clean** (use comments)

## Next Steps

Once you understand CMake:
1. We'll create the initial `CMakeLists.txt` for our project
2. Add components incrementally (one library at a time)
3. Test each component as we build
4. Add unit tests with `enable_testing()`

## Resources

- [Official CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/)
- [CMake Documentation](https://cmake.org/documentation/)
- [Modern CMake](https://cliutils.gitlab.io/modern-cmake/)

---

**Key Takeaway**: CMake automates the tedious parts of building C++ projects. You describe *what* to build, CMake figures out *how* to build it for your platform.