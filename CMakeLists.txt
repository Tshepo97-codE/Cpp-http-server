# CMake minimum version
cmake_minimum_required(VERSION 3.15)

# Project definition
project(HTTPServer 
    VERSION 0.1.0
    DESCRIPTION "Educational HTTP/1.1 Server in C++"
    LANGUAGES CXX
)

# C++ Standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Compiler warnings (good practice for catching bugs)
if(MSVC)
    # Visual Studio warnings
    add_compile_options(/W4)  # Warning level 4
    add_compile_options(/WX-) # Don't treat warnings as errors (for now)
else()
    # GCC/Clang warnings
    add_compile_options(-Wall -Wextra -Wpedantic)
    # add_compile_options(-Werror)  # Uncomment to treat warnings as errors
endif()

# Build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING "Build type" FORCE)
endif()

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ Compiler: ${CMAKE_CXX_COMPILER}")
message(STATUS "C++ Standard: C++${CMAKE_CXX_STANDARD}")

# Include directories
include_directories(${PROJECT_SOURCE_DIR}/include)

# ============================================================================
# LIBRARIES
# ============================================================================

# Utility Library (lowest level - no dependencies)
add_library(utils STATIC
    src/Config.cpp
    src/Logger.cpp
)
target_include_directories(utils PUBLIC ${PROJECT_SOURCE_DIR}/include)

# Infrastructure Library (depends on utils)
add_library(infrastructure STATIC
    src/SocketHandler.cpp
    src/FileHandler.cpp
)
target_include_directories(infrastructure PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_link_libraries(infrastructure utils)

# HTTP Core Library (business logic)
add_library(http_core STATIC
    src/HTTPParser.cpp
    src/HTTPResponse.cpp
    src/Router.cpp
)
target_include_directories(http_core PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_link_libraries(http_core utils)

# Server Library (orchestration)
add_library(server STATIC
    src/Server.cpp
)
target_include_directories(server PUBLIC ${PROJECT_SOURCE_DIR}/include)
target_link_libraries(server http_core infrastructure)

# ============================================================================
# EXECUTABLE
# ============================================================================

add_executable(http_server
    src/main.cpp
)

# Link all libraries to main executable
target_link_libraries(http_server 
    server
    http_core
    infrastructure
    utils
)

# Windows-specific: Link Winsock library
if(WIN32)
    target_link_libraries(http_server ws2_32)
    message(STATUS "Linking Winsock2 library (ws2_32)")
endif()

# ============================================================================
# TESTING
# ============================================================================

# Enable testing
enable_testing()

# Google Test (optional - uncomment when ready to add tests)
# find_package(GTest REQUIRED)
# add_subdirectory(tests)

# ============================================================================
# INSTALLATION (optional)
# ============================================================================

# Install executable
install(TARGETS http_server
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

# Install configuration file
install(FILES ${PROJECT_SOURCE_DIR}/server.conf
    DESTINATION etc
)

# Install public directory
install(DIRECTORY ${PROJECT_SOURCE_DIR}/public/
    DESTINATION share/http_server/public
)

# ============================================================================
# PRINT CONFIGURATION SUMMARY
# ============================================================================

message(STATUS "")
message(STATUS "=== HTTP Server Configuration ===")
message(STATUS "Project: ${PROJECT_NAME} v${PROJECT_VERSION}")
message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "C++ Standard: C++${CMAKE_CXX_STANDARD}")
message(STATUS "Compiler: ${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "Platform: ${CMAKE_SYSTEM_NAME}")
message(STATUS "")
message(STATUS "Source Directory: ${PROJECT_SOURCE_DIR}")
message(STATUS "Build Directory: ${PROJECT_BINARY_DIR}")
message(STATUS "Install Prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "")
message(STATUS "Targets:")
message(STATUS "  - utils (static library)")
message(STATUS "  - infrastructure (static library)")
message(STATUS "  - http_core (static library)")
message(STATUS "  - server (static library)")
message(STATUS "  - http_server (executable)")
message(STATUS "")
message(STATUS "To build:")
message(STATUS "  cmake --build .")
message(STATUS "")
message(STATUS "To run:")
if(WIN32)
    message(STATUS "  .\\bin\\http_server.exe")
else()
    message(STATUS "  ./bin/http_server")
endif()
message(STATUS "=================================")
message(STATUS "")