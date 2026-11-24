# Webserv

![C++](https://img.shields.io/badge/C++-98-00599C?style=flat-square&logo=cplusplus&logoColor=white)
![42 Network](https://img.shields.io/badge/42-Network-black?style=flat-square&logo=42)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![HTTP](https://img.shields.io/badge/Protocol-HTTP-blue?style=flat-square)
![Networking](https://img.shields.io/badge/Networking-epoll-orange?style=flat-square)
![Make](https://img.shields.io/badge/Build-Makefile-green?style=flat-square)

## Overview

Webserv is a lightweight HTTP server built from scratch in C++98 as part of the **1337 Coding School — 42 Network** curriculum.

Inspired by NGINX, the server implements an event-driven architecture using non-blocking sockets and the Linux `epoll` API to manage multiple client connections.

The project supports HTTP request processing, static file serving, CGI execution, file uploads, configurable routing, and basic session management without relying on external web server libraries.

It provides practical experience with network programming, HTTP protocols, asynchronous I/O, system programming, and backend server architecture.

## Features

### Networking

- Non-blocking socket programming
- Event-driven I/O using Linux `epoll`
- Multiple simultaneous client connections
- Multiple server configurations and listening ports
- Configurable host addresses

### HTTP

- GET, POST, and DELETE methods
- HTTP request parsing
- HTTP response generation
- Regular and chunked POST body parsing
- MIME type detection
- Custom HTTP error pages
- HTTP redirects
- Configurable request-size limits

### File Management

- Static file serving
- Multipart form-data uploads
- File deletion
- Automatic directory listing
- Configurable document roots
- Default index pages

### CGI and Sessions

- CGI execution through configurable interpreters
- Python, Perl, and shell script support when installed
- CGI environment variable management
- Request body forwarding
- Basic cookie-backed sessions
- Login and profile routes

## Architecture

Webserv follows an event-driven architecture that allows the server to manage multiple connections without creating a dedicated thread for each client.

```text
                 HTTP Clients
                       │
                       ▼
              Listening Sockets
                       │
                       ▼
              epoll Event Loop
                       │
                       ▼
               Request Parser
                       │
                       ▼
             Server Configuration
                       │
                       ▼
                Route Selection
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        GET           POST        DELETE
          │            │            │
     Static Files   File Upload   File Removal
     Directory      CGI
     Listing        Sessions
          │            │            │
          └────────────┼────────────┘
                       ▼
              Response Builder
                       │
                       ▼
                   Client
```

## Request Lifecycle

Each incoming HTTP request passes through the following stages:

1. Accept the incoming client connection.
2. Monitor socket readiness through `epoll`.
3. Receive and parse the HTTP request.
4. Validate request headers and body.
5. Resolve the corresponding server and location configuration.
6. Execute the appropriate request handler.
7. Generate the HTTP response.
8. Send the response and close the connection.

The server uses non-blocking sockets to avoid blocking the main event loop while handling socket I/O.

## Getting Started

### Requirements

- Linux operating system
- C++ compiler supporting C++98
- GNU Make
- Optional CGI interpreters:
  - Python 3
  - Perl
  - POSIX shell

### Build

Compile the project:

```bash
make
```

The Makefile builds the `webserv` executable using:

```bash
-Wall -Wextra -Werror -std=c++98
```

### Run

Start the server using the default configuration:

```bash
./webserv
```

Alternatively, provide a custom configuration file:

```bash
./webserv path/to/server.conf
```

The default configuration is located at:

```text
config/Default.conf
```

The server listens on:

```text
http://127.0.0.1:8080
```

Stop the server using `Ctrl+C`.

### Document Root Setup

The default configuration expects a `www/` document root, which is not included in the repository.

Create the required directories:

```bash
mkdir -p www/html www/cgi-bin www/uploads
```

Create a basic HTML page:

```bash
printf '<h1>Webserv is running</h1>\n' > www/html/index.html
```

Start the server:

```bash
./webserv
```

Test the server:

```bash
curl -i http://127.0.0.1:8080/
```

## Configuration

Webserv uses an NGINX-inspired configuration syntax.

- Statements end with a semicolon.
- Configuration blocks are enclosed in braces.
- Comments begin with `#`.
- Multiple server blocks can be defined.

### Example Configuration

```nginx
server {
    listen 8080;
    host 127.0.0.1;
    server_name localhost;

    root ./www;
    index html/index.html;
    client_max_body_size 1000000;

    error_page 404 /html/errors/404.html;

    return /search https://www.google.com;

    location /files {
        root ./www;
        index index.html;
        autoindex on;
        allow_methods GET POST DELETE;
        upload_store ./www/uploads;
    }

    cgi_interpreter .py /usr/bin/python3;
    cgi_interpreter .pl /usr/bin/perl;
    cgi_interpreter .sh /usr/bin/sh;
}
```

### Supported Directives

| Context | Directive | Description |
|---|---|---|
| Top level | `include mime.types` | Load MIME type mappings |
| Server | `listen` | Configure listening ports |
| Server | `host` | Configure the bind address |
| Server | `server_name` | Define the server name |
| Server | `root` | Configure the document root |
| Server | `index` | Define the default index file |
| Server | `client_max_body_size` | Configure the maximum request buffer size |
| Server | `error_page` | Define custom HTTP error pages |
| Server | `return` | Redirect a request path to another URL |
| Server | `cgi_interpreter` | Configure CGI interpreters |
| Server | `location` | Define path-specific configuration |
| Location | `root` | Override the document root |
| Location | `index` | Override the default index file |
| Location | `autoindex` | Enable or disable directory listing |
| Location | `allow_methods` | Declare supported HTTP methods |
| Location | `upload_store` | Define the file upload destination |

The optional `include mime.types` directive requires a `mime.types` file in the working directory.

When omitted, the server uses built-in MIME mappings for common file extensions.

## HTTP Methods

| Method | Responsibility |
|---|---|
| GET | Retrieve resources, serve static files, and display directory listings |
| POST | Process request bodies, upload files, and execute CGI scripts |
| DELETE | Remove files from the permitted upload directory |

DELETE requests are restricted to paths below `<root>/uploads`.

## CGI Execution

CGI (Common Gateway Interface) allows the web server to execute external scripts and generate dynamic content.

Scripts are selected according to their file extensions and configured interpreters.

```text
HTTP Request
      │
      ▼
CGI Handler
      │
      ▼
External Interpreter
      │
      ▼
Script Execution
      │
      ▼
CGI Output
      │
      ▼
HTTP Response
```

The server provides common CGI environment variables:

- `REQUEST_METHOD`
- `SCRIPT_NAME`
- `QUERY_STRING`
- `CONTENT_LENGTH`
- `CONTENT_TYPE`
- `SERVER_PROTOCOL`
- `GATEWAY_INTERFACE`

For POST requests, the request body is forwarded to the CGI process through its standard input.

CGI scripts are expected inside the `www/cgi-bin/` directory.

## Testing

### GET Request

```bash
curl -i http://127.0.0.1:8080/
```

### POST Request

```bash
curl -i -X POST \
    -d 'message=hello' \
    http://127.0.0.1:8080/submit
```

### File Upload

```bash
curl -i -X POST \
    -F "file=@example.txt" \
    http://127.0.0.1:8080/files
```

### DELETE Request

```bash
curl -i -X DELETE \
    http://127.0.0.1:8080/uploads/example.txt
```

### HTTP Error Handling

```bash
curl -i http://127.0.0.1:8080/nonexistent
```

POST and upload tests require the corresponding routes and destination directories to be configured.

## Project Structure

```text
webserv/
├── config/
├── headers/
├── src/
│   ├── CGI/
│   ├── Helper/
│   ├── parsing/
│   └── server/
├── Makefile
└── README.md
```

| Directory | Responsibility |
|---|---|
| `config/` | Server configuration files |
| `headers/` | Classes, structures, and function declarations |
| `src/CGI/` | CGI process execution |
| `src/Helper/` | File management and upload utilities |
| `src/parsing/` | Configuration, request, and response parsing |
| `src/server/` | Socket initialization and epoll event management |

## Makefile Commands

| Command | Description |
|---|---|
| `make` | Build the executable |
| `make clean` | Remove object files |
| `make fclean` | Remove object files and the executable |
| `make re` | Rebuild the project from scratch |

## Technical Concepts

Throughout this project, we explored:

- TCP/IP networking
- Client-server architecture
- Socket programming
- Non-blocking I/O
- Event-driven architecture
- I/O multiplexing
- HTTP request and response processing
- HTTP headers and status codes
- Configuration file parsing
- Process management
- CGI execution
- File descriptor management
- Multipart form-data parsing
- Chunked transfer encoding
- Cookies and session management
- Object-oriented programming
- Resource management and error handling

### Key System Functions

The project relies on low-level networking and Unix system programming concepts, including:

`socket()` · `bind()` · `listen()` · `accept()` · `recv()` · `send()` · `epoll_create()` · `epoll_ctl()` · `epoll_wait()` · `fork()` · `execve()` · `pipe()` · `waitpid()`

## Technologies

| Technology | Purpose |
|---|---|
| C++98 | Core server implementation |
| TCP/IP | Network communication |
| HTTP | Application-layer protocol |
| epoll | I/O event notification and multiplexing |
| POSIX | Unix system programming |
| CGI | Dynamic content generation |
| Linux | Runtime environment |
| Makefile | Build automation |

## Project Scope

Webserv is an educational HTTP server designed to implement the requirements of the 42 curriculum.

Current implementation notes:

- HTTP/1.1 request lines are accepted using HTTP/1.0-style responses.
- Connections are closed after each response.
- The server depends on Linux-specific networking APIs.
- Relative configuration and asset paths require running the executable from the repository root.
- The server is not intended to replace production-ready web servers such as NGINX or Apache.

## Resources

- [RFC 9110 — HTTP Semantics](https://datatracker.ietf.org/doc/html/rfc9110)
- [RFC 9112 — HTTP/1.1](https://datatracker.ietf.org/doc/html/rfc9112)
- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [Linux epoll Documentation](https://man7.org/linux/man-pages/man7/epoll.7.html)
- [Linux Socket Documentation](https://man7.org/linux/man-pages/man2/socket.2.html)
- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/)
- [NGINX Documentation](https://nginx.org/en/docs/)
- [RFC 3875 — CGI](https://datatracker.ietf.org/doc/html/rfc3875)

## Author

**Hassan Benydir — habenydi**
**Yassir Ait Ammi — yaait-am**
**Bouhcine Amezouar — bamezoua**
---

Developed as part of the **1337 Coding School — 42 Network** curriculum.
