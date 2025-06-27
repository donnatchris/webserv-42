# PROJECT WEBSERV FOR 42  
By **chdonnat** (Christophe Donnat, 42 Perpignan – France)  
With teammates: **Olivier Thorel** and **Lucas Matkowski**, also from 42 Perpignan – France.

<p align="center">
  <img src="https://github.com/donnatchris/webserv/blob/main/screenshots/index.png" />
</p>
<p align="center">
  <img src="https://github.com/donnatchris/webserv/blob/main/screenshots/webserv.png" />
</p>



## AIM OF THE PROJECT:

**Webserv** is a team project (3 members) that consists in creating a fully functional HTTP server in C++, compliant with the HTTP/1.1 specification (RFC 2616).  
The server must be able to:
- Parse and apply multiple configuration files
- Handle the HTTP methods: GET, POST, and DELETE
- Serve static files and generate autoindex pages
- Execute CGI scripts
- Support multiple virtual servers
- Handle multiple simultaneous connections using `poll()`

The goal is to gain a deeper understanding of how web servers work under the hood, and to implement low-level socket programming and I/O multiplexing in C++98.


## BONUS PART:

The following bonus features have been implemented:
- Cookie-based session management
- Multiple CGI support (in our project: Python and PHP)

These bonuses demonstrate a deeper grasp of HTTP mechanisms and reflect how real-world web applications operate.


## WHAT WE'RE PROUD OF:

Since we really enjoyed working on the project, we went further and implemented:
- Full compatibility with **macOS** and **Linux**
- File upload support via `multipart/form-data` (limited to one file at a time), enabling uploads of any file type (PNG, PDF, MP3, etc.)
- Chunked communication (4096-byte blocks), even for uploads, to optimize memory usage
- A complete and fun **zombie-themed test website** to demonstrate and validate all features


## SOME COMMANDS YOU CAN USE:

Compile the program and suppress the .o files:

	make && make clean

 ---

Execute the program with default configuration file:

(lanch programm without argumets uses the config/default.conf config file)

	./webserv webserv.conf

 With that configuration file, you can now test a website we have created to test the whole project:

 In your favorite browser type:

	http://localhost:8000

 ---

 Execute the program with another configuration file

	./webserv config/

 With that configuration file, you can now test 3 virtual servers, each with its own website for testing.

  In your favorite browser type:

	http://localhost:8000

 	http://localhost:80808

  	http://localhost:8888


## ARCHITECTURE:

- `config/` — Contains example configuration files used to test the project.
- `documentation_fr/` — French-language documentation and technical notes.
- `include/` — All project header files.
- `src/` — Source code of the server, organized by responsibility:
  - `config/` — Classes for parsing and storing configuration data.
  - `http/` — Classes for parsing HTTP requests and building HTTP responses.
  - `process/` — Classes for processing client data and handling communication logic.
  - `server/` — Classes for managing sockets, connections, and the main server loop.
- `www/` — Contains static files and test websites (HTML, CSS, images, and CGI scripts).
- `Makefile` — Build system with the following rules: `make`, `bonus`, `clean`, `fclean`, `re`.
- `README.md` — Overview of the project and key usage instructions.


## DOCUMENTATION

Unlike most of my projects, you’ll find **French-language documentation and technical notes** inside the `documentation_fr/` directory.

This directory contains the following files, written during our research phase before diving into the implementation:

- `CGI.fr.md` — A quick explanation of what CGI is and how it works
- `fichiers_de_config.fr.md` — A breakdown of what must be handled in configuration files for a compliant server
- `fonctions_doc` — A directory containing files with lists and explanations of all allowed C functions in the project
- `HTTP1.1.fr.md` — A summarized and accessible version of RFC 2616, which defines the HTTP/1.1 protocol
- `webserv_doc.fr.md` — A comprehensive overview of all key concepts required for the project - You should definitely start here if you re about to do this project


If you do not speak French, below is an English summary of the main concepts you need to understand to work on the project:

---

### A Web Server in C++

#### Definition

A web server is a program that:

- Listens for HTTP requests from clients (like browsers)  
- Parses these requests  
- Looks up the requested resource (file, script, etc.)  
- And sends back a structured HTTP response, usually HTML or other content.

> In short, a web server is a translator between a client that asks a question, and a filesystem or an execution environment that provides the answer.

#### Configuration File

The Webserv program must be launched with a configuration file path as an argument. This is a **plain text file**, usually with the `.conf` extension, which describes **how the web server should behave**, and must be parsed at runtime.  
The program must **read this file** and **dynamically configure the server** according to its contents.

> If the program is launched without any argument, it must use a default configuration file located at a known hardcoded path (for example: `"./default.conf"` or `"/etc/webserv.conf"`).

There is no strictly enforced grammar for config files, but in Webserv we mostly follow the style of **NGINX config files**, as NGINX is a lightweight and highly configurable web server. These are structured with:

1. **Blocks** (e.g., `server`, `location`)
2. **Directives** (e.g., `listen`, `root`, `index`) ending with `;`
3. A **logical hierarchical structure**

***→ Configuration files are explained in detail in the file `fichiers_de_config.fr.md`***

#### 1. Listening

The server opens a **TCP socket** on a **port** (typically 80 or 8080) to wait for incoming connections.

#### 2. Accepting

The server accepts a connection and receives an HTTP request, for example:

	GET /index.html HTTP/1.1
	Host: localhost


#### 3. Parsing

The server analyzes the request:

- Method (`GET`, `POST`, etc.)
- Path (`/index.html`)
- Headers (`Host`, `Content-Type`, etc.)

#### 4. Responding

The server builds an HTTP response including:

- A **status line** (`HTTP/1.1 200 OK`)
- **Headers** (`Content-Type`, `Content-Length`, etc.)
- A **body** (often HTML, JSON, or an image)

#### 5. Handling Multiple Clients

It must be able to handle **multiple simultaneous connections** without blocking.

#### 6. Executing Scripts (CGI)

If the request targets a script (e.g., `.py`, `.php`), the server must:

- Create a child process with `fork()`
- Execute the script with `execve()`
- Capture its output
- Send that output as the response

#### Webserv Summary

In summary, the Webserv program must be able to:

- Read and apply a configuration file
- Listen on sockets
- Understand the HTTP protocol (requests/responses)
- Open and read files
- Execute CGI scripts
- Handle errors (404, 500, etc.)
- Support multiple configurable servers

---

### A Web Server in C++

#### Definition

A web server is a program that:

- Listens for HTTP requests from clients (like browsers)  
- Parses these requests  
- Looks up the requested resource (file, script, etc.)  
- And sends back a structured HTTP response, usually HTML or other content.

> In short, a web server is a translator between a client that asks a question, and a filesystem or an execution environment that provides the answer.

#### Configuration File

The Webserv program must be launched with a configuration file path as an argument. This is a **plain text file**, usually with the `.conf` extension, which describes **how the web server should behave**, and must be parsed at runtime.  
The program must **read this file** and **dynamically configure the server** according to its contents.

> If the program is launched without any argument, it must use a default configuration file located at a known hardcoded path (for example: `"./default.conf"` or `"/etc/webserv.conf"`).

There is no strictly enforced grammar for config files, but in Webserv we mostly follow the style of **NGINX config files**, as NGINX is a lightweight and highly configurable web server. These are structured with:

1. **Blocks** (e.g., `server`, `location`)
2. **Directives** (e.g., `listen`, `root`, `index`) ending with `;`
3. A **logical hierarchical structure**

***→ Configuration files are explained in detail in the file `fichiers_de_config.fr.md`***

#### 1. Listening

The server opens a **TCP socket** on a **port** (typically 80 or 8080) to wait for incoming connections.

#### 2. Accepting

The server accepts a connection and receives an HTTP request, for example:

	GET /index.html HTTP/1.1
	Host: localhost


#### 3. Parsing

The server analyzes the request:

- Method (`GET`, `POST`, etc.)
- Path (`/index.html`)
- Headers (`Host`, `Content-Type`, etc.)

#### 4. Responding

The server builds an HTTP response including:

- A **status line** (`HTTP/1.1 200 OK`)
- **Headers** (`Content-Type`, `Content-Length`, etc.)
- A **body** (often HTML, JSON, or an image)

#### 5. Handling Multiple Clients

It must be able to handle **multiple simultaneous connections** without blocking.

#### 6. Executing Scripts (CGI)

If the request targets a script (e.g., `.py`, `.php`), the server must:

- Create a child process with `fork()`
- Execute the script with `execve()`
- Capture its output
- Send that output as the response

#### Webserv Summary

In summary, the Webserv program must be able to:

- Read and apply a configuration file
- Listen on sockets
- Understand the HTTP protocol (requests/responses)
- Open and read files
- Execute CGI scripts
- Handle errors (404, 500, etc.)
- Support multiple configurable servers

---

### Some Basic Definitions

#### Client

> A **client** is a program (often a **web browser**) that sends an **HTTP request** to a web server to request a **resource** (HTML page, image, file, script, etc.).

In the context of Webserv:

- The **client** connects to the Webserv server through the network
- It sends a request
- It waits for an HTTP response

#### HTTP Request

> An **HTTP request** is a **text-based message** sent by a client to a server to request or send data, following a specific structure.

It begins with a **request line** like:

	GET /index.html HTTP/1.1


Then comes a series of **headers** with additional information (like the host name or language preferences: `Host`, `User-Agent`, etc.), followed by a blank line, and sometimes a **body** (especially in `POST` requests).

The server reads and interprets the request, then sends back a **structured HTTP response**.

***→ HTTP requests are explained in detail in the file `HTTP1.1.fr.md`***

#### HTML (HyperText Markup Language)

> **HTML** is a **markup language** used to structure and display content on web pages.

An HTML file is a **resource** that the server can deliver to the client.

Simple example:

```html
<html>
  <head><title>My page</title></head>
  <body><h1>Hello!</h1></body>
</html>
```

#### HTML (HyperText Markup Language)

The Webserv server does not parse HTML — **it simply sends it as-is**.  
The browser is responsible for interpreting it.


#### Socket

A **socket** is a **software interface** (a piece of code inside a program) that allows communication over a network using protocols like TCP or UDP.

In Webserv:

- The server creates a socket using `socket()`
- Binds it to an address and port with `bind()`
- Listens for incoming connections with `listen()`
- Accepts connections with `accept()`

In C/C++, a socket is represented by a **file descriptor** (an `int`), which allows the use of standard functions like `read()` and `write()` for network communication.


#### Port

A **port** is a **logical number** used to identify a **specific application** on a machine.

- **Port 80** is the **default port for HTTP**
- **Port 443** is used for **HTTPS**
- **Port 8080** is often used as an **alternative to port 80**, especially:
  - when you don’t have root privileges
  - for local development/testing servers

> The Webserv server listens on a port (e.g., `8080`) to accept HTTP connections.


#### TCP (Transmission Control Protocol)

**TCP** is a **reliable transport protocol** used to transmit data between two machines over a network, ensuring that data:

- arrives in the **correct order**
- is **not lost**
- is **not duplicated**

Unlike other protocols like UDP (which is fast but unreliable), TCP establishes a **stable connection** (a session) between two endpoints before transmitting any data.

HTTP is built on top of TCP, which means:

- An HTTP request is sent **through a TCP stream**
- Webserv uses a **TCP socket** to receive requests
- Clients can rely on **complete and ordered transmission** of messages

> Thanks to TCP, the server can read an HTTP request **safely**, without having to manually handle packet order or integrity.


#### UDP (User Datagram Protocol)

**UDP** is a **fast but unreliable transport protocol**, used to send **small packets of data** without establishing a prior connection.

Unlike TCP, UDP does **not guarantee**:

- That packets arrive **in the correct order**
- That packets arrive **at all** (packet loss is possible)
- That packets are **not duplicated**

UDP is therefore:

- **Lighter**
- **Faster**
- But also **less reliable**

Each message sent is called a **datagram**, independent from the others.  
UDP offers **no error checking or retransmission**.

It is often used for:

- Online games
- Real-time video streaming
- DNS queries
- Applications that tolerate some data loss

> In short, **UDP trades reliability for speed**, making it suitable for communication where **responsiveness matters more than perfection**.


#### CGI (Common Gateway Interface)

**CGI** is a **standard interface** that allows a web server to **execute an external program** (such as a PHP or Python script) and **return its output as an HTTP response**.

Webserv must:

- Detect that a file is a **CGI script**
- Call `fork()` to create a child process
- Use `execve()` to execute the script
- Read the script’s output (via a pipe)
- Send that output back as the HTTP response

***→ CGI is explained in detail in the file `CGI.fr.md`***

### Allowed Functions

The Webserv project allows the use of anything compatible with **C++98**, without Boost or other external libraries.  
The subject lists many system-level functions, which can be grouped into categories:

#### Process Management and Execution

> These functions allow the server to **launch external programs (like CGI scripts)**, **manage child processes**, and **redirect their input/output**.

| Function   | Main Purpose                                              |
| ---------- | --------------------------------------------------------- |
| `execve`   | Executes an external program (e.g., a CGI script)         |
| `fork`     | Creates a child process (used for CGI)                    |
| `waitpid`  | Waits for the termination of a child process              |
| `kill`     | Sends a signal to a process                               |
| `signal`   | Sets a custom handler for a signal (e.g., `SIGINT`)       |
| `errno`    | Global variable containing the latest system error code   |
| `strerror` | Returns a readable message for the current `errno` value  |


#### Redirection and File Descriptor Duplication

> Used to **redirect standard input/output**, especially for CGI execution and pipe handling.

| Function | Main Purpose                        |
|----------|-------------------------------------|
| `dup`    | Duplicates a file descriptor        |
| `dup2`   | Duplicates a descriptor to a specific target |
| `pipe`   | Creates a pair of connected descriptors for IPC |


#### Networking — **Sockets**

> These are core functions for building a **TCP web server**, accepting connections, and sending/receiving requests.

| Function      | Main Purpose                                                      |
|---------------|-------------------------------------------------------------------|
| `socket`      | Creates a TCP socket                                              |
| `bind`        | Binds a socket to an IP address and port                          |
| `listen`      | Puts the socket in listening mode                                 |
| `accept`      | Accepts an incoming connection                                    |
| `connect`     | Initiates a connection to a remote socket                         |
| `recv`        | Receives data from a socket                                       |
| `send`        | Sends data through a socket                                       |
| `setsockopt`  | Configures options on a socket                                    |
| `getsockname` | Retrieves the local address of a socket                           |
| `socketpair`  | Creates a pair of connected sockets (e.g., bidirectional CGI pipe)|


#### I/O Multiplexing — **Handling Multiple Connections Simultaneously**

> These interfaces allow the server to **monitor multiple sockets** for read/write readiness without using threads.

| Interface | Associated Functions                            |
|-----------|--------------------------------------------------|
| `select`  | Portable method, limited to 1024 file descriptors |
| `poll`    | More flexible than `select`, supports more fds   |
| `epoll`   | For Linux: `epoll_create`, `epoll_ctl`, `epoll_wait` |
| `kqueue`  | For macOS: `kqueue`, `kevent`                    |


#### Name Resolution and Network Configuration

> These functions help **translate hostnames to IPs**, configure protocols, and manage ports.

| Function         | Main Purpose                                        |
|------------------|-----------------------------------------------------|
| `getaddrinfo`    | Resolves a hostname (e.g., `localhost` → IP address)|
| `freeaddrinfo`   | Frees memory allocated by `getaddrinfo`             |
| `getprotobyname` | Retrieves the number for a protocol (e.g., `"tcp"`) |
| `htons`, `htonl` | Converts integers to network byte order             |
| `ntohs`, `ntohl` | Converts integers **from** network byte order       |


#### File and Filesystem Access

> Used to **serve HTML files, images, etc.**, check existence and permissions, and browse directories.

| Function                          | Main Purpose                                   |
|-----------------------------------|------------------------------------------------|
| `access`                          | Checks if a file exists or has the right permissions |
| `stat`                            | Retrieves file metadata                        |
| `open`, `read`, `write`, `close` | Open, read, write, and close a file            |
| `chdir`                           | Changes the current working directory (e.g., for CGI) |
| `opendir`, `readdir`, `closedir` | Open, read, and close directories              |

---

### HTTP Connection Lifecycle in Webserv

#### 1. **Server Initialization**

- `socket()` → creates a listening socket (`SOCK_STREAM`)
- `setsockopt()` → enables `SO_REUSEADDR` option
- `bind()` → binds the socket to a specific `host:port`
- `listen()` → puts the socket into listening mode
- `epoll_create()` / `poll()` → sets up the event loop mechanism


#### 2. **Waiting for Connections (`EPOLLIN` or `POLLIN` Event)**

- `accept()` → accepts an incoming connection
- `epoll_ctl(ADD)` → adds the new client socket to the poll/epoll set
- client socket is set to **non-blocking mode**


#### 3. **Receiving the HTTP Request**

- `recv()` → reads the request from the client socket
- `parse()` → analyzes:
  - HTTP method (GET, POST…)
  - URI / path
  - headers (Host, Content-Length…)
  - optional body (e.g., for POST)


#### 4. **Processing the Request**

- **Static file:**
  - `access()` → checks if the file exists and if access is allowed
  - `stat()` → retrieves file size/type
  - `open()` + `read()` → reads the requested file
  - constructs a **complete HTTP response**

- **CGI script:**
  - `fork()` → creates a child process
  - `pipe()` / `socketpair()` → sets up communication with the CGI
  - `dup2()` → redirects `stdin` and `stdout`
  - `execve()` → executes the script
  - `waitpid()` → waits for the CGI to finish
  - `read()` → reads the CGI output
  - parses and **injects the output into the HTTP response**


#### 5. **Sending the Response**

- `send()` → sends the HTTP response (headers + body)
- (optional: switch from `EPOLLIN` to `EPOLLOUT` in non-blocking mode)


#### 6. **Closing the Connection**

- `close()` → closes the client socket (or `epoll_ctl(DEL)`)
- or **keep-alive** is maintained based on the `Connection: keep-alive` header and timeout configuration


#### Request Handling Summary

```text
client connects → accept() → recv() → parse()
          ↓
  [static file]             [CGI script]
       ↓                          ↓
access/stat/open         fork() → execve()
       ↓                          ↓
     read()                     pipe/read
       ↓                          ↓
 HTTP response             HTTP response
          ↓
        send()
          ↓
       close()
```

---

### Multiplexing

#### Definition of Multiplexing (aka **I/O Multiplexing**)

> **Multiplexing** refers to the ability of a program to **monitor multiple input/output sources simultaneously** without blocking, and to react **as soon as any of them becomes active** (e.g., a socket ready for reading or writing).

Instead of doing:

```cpp
read(socket1);
read(socket2);
read(socket3);
```

### Multiplexing

#### ...and blocking on each socket?

Instead of blocking on each socket one by one, we ask the OS:

> “Let me know as soon as **any one of them** has something to say.”

---

### Multiplexing in Webserv

The server must:

- Listen to **multiple clients simultaneously**
- **Avoid blocking** on inactive sockets
- **React immediately** when a client becomes ready

Multiplexing allows all of this to be done with a **single thread/process**, in a **non-blocking** and **efficient** way.

> ⚠️ The subject explicitly forbids using `fork()` or threads for handling multiple connections:
>
> *"You may not use fork for anything other than CGI (like PHP or Python, etc)."*


### Common Multiplexing Mechanisms

The subject allows choosing **one and only one** multiplexing mechanism.  
You must **never call `read()` or `write()`** unless the chosen mechanism has confirmed that the file descriptor is ready.

| Mechanism   | Supported Systems    | Level        |
|-------------|----------------------|--------------|
| `select()`  | POSIX (Linux, macOS) | Basic        |
| `poll()`    | POSIX (Linux, macOS) | Intermediate |
| `epoll()`   | Linux only           | Advanced     |
| `kqueue()`  | BSD/macOS only       | Advanced     |


#### 1. `select()`

This function uses **static sets (`fd_set`)** and macros like `FD_SET`.

**Advantages:**
- Very simple to understand
- Universal (supported everywhere)

**Drawbacks:**
- Limited to **1024 file descriptors** (`FD_SETSIZE`)
- Poor performance with many sockets (linear scan)
- Verbose and rigid syntax


#### 2. `poll()`

This function uses a **dynamic array of `pollfd` structures** — a good portable choice.

**Advantages:**
- No 1024 FD limit
- Cleaner interface than `select()`
- Available on all POSIX systems

**Drawbacks:**
- Still uses **linear scanning**
- Requires rebuilding the array each iteration


#### 3. `epoll()` (Linux only)

An **event-driven** interface designed for high performance.

**Advantages:**
- Very fast: O(1) for `add`, `remove`, and `wait`
- Ideal for **thousands of concurrent connections**
- Only returns **ready FDs**

**Drawbacks:**
- Linux-specific
- Slightly more complex API (`epoll_create`, `epoll_ctl`, `epoll_wait`)
- Not portable


#### 4. `kqueue()` (macOS / BSD)

BSD/macOS equivalent of `epoll()`.

**Advantages:**
- High performance (similar to `epoll`)
- Can also monitor **files, signals, and timers**

**Drawbacks:**
- macOS/BSD-specific
- More verbose syntax (`kqueue`, `kevent`, etc.)


### Webserv Multiplexing Constraints

The **Webserv** server must:

- Work in **non-blocking** mode
- Use a **single multiplexing mechanism** (`poll()`, `select()`, `epoll()`, or `kqueue()`)
- Use it to handle **all I/O**, including:
  - The **listening socket**
  - **Client sockets**
  - **CGI communication pipes**


### Mandatory Rules

- Use **one main loop**
- Only one call to `poll()` (or equivalent) per iteration
- This call must monitor:
  - **Sockets ready to read** (`POLLIN`)
  - **Sockets ready to write** (`POLLOUT`)
- You must **not call `read()`/`recv()` or `write()`/`send()`** unless the polling mechanism confirms readiness
- You must **not rely on `errno` values** (e.g., `EAGAIN`) to manage readiness — this should be avoided by using `poll()` or equivalent


### Exception: Reading the Configuration File

Reading the `.conf` file **can be done in blocking mode**.  
These rules **only apply to I/O operations on sockets and pipes**.


### Notes for `select()`

If you choose to use `select()`, the following macros are **required** for managing monitored file descriptors:

- `FD_ZERO` — Clears a set of descriptors
- `FD_SET`  — Adds a descriptor to the set
- `FD_CLR`  — Removes a descriptor from the set
- `FD_ISSET` — Checks if a descriptor is ready (read/write)

> These macros are **essential with `select()`**,  
> but they are **not used with `poll()` or `epoll()`**.

---

### Telnet and NGINX

`telnet` and `nginx` are **command-line tools (CLI)** that can help you understand and test Webserv.  
They are useful to:

- Manually test **raw HTTP requests**, combining `telnet` and `nginx` to learn how an HTTP request/response is structured
- See how the server reacts to malformed or minimal requests
- Test your own Webserv server by sending HTTP requests through `telnet`

---

#### Telnet

> `telnet` is a **command-line network client** that allows you to **open a TCP connection** to a given host and port.  
> It's very useful to **manually send HTTP requests** and observe the **raw HTTP response** from the server.

Launch `telnet` like this:

```bash
telnet <host> <port>

# for example:
telnet localhost 8080    # default port on macOS
telnet localhost 80      # default port on Linux
```

#### Telnet — Typing a Raw HTTP Request

Once connected, `telnet` gives you a prompt where you can type a **raw HTTP request line-by-line**, ending with a **blank line** (press Enter twice) to signal the end of the request.

Example of a raw HTTP request in `telnet`:

	GET / HTTP/1.1
	Host: localhost


If `nginx` (or Webserv) is listening on the given port, it will receive the request and **send back an HTTP response**, which will appear directly in the terminal.


#### NGINX

`nginx` is a **high-performance and widely used HTTP server**, often deployed in production environments.  
It serves as a useful reference to understand **how a real-world HTTP server handles requests**.

In the context of **Webserv**, `nginx` is helpful for:

- Comparing the **behavior of your server** with NGINX (even during evaluation — NGINX is the reference)
- Observing **responses to error cases** (e.g., `404 Not Found`, `405 Method Not Allowed`)
- Studying the **structure of a configuration file** (`nginx.conf`)
- Using it as a **template** to write your own `.conf` file for Webserv


##### Starting `nginx`

```bash
brew services start nginx    # on macOS (default port: 8080)
sudo systemctl start nginx   # on Linux (default port: 80)
```

Verifying if nginx is running:

	lsof -i :8080    # on macOS
	lsof -i :80      # on Linux

---


