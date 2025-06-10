# HTTP Proxy Server with Caching

This project implements a multi-threaded HTTP proxy server in C, featuring an in-memory cache for enhanced performance. It acts as an intermediary between web clients and origin web servers, serving cached content when available and forwarding requests otherwise.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Project Theory](#project-theory)
  - [Basic Working Flow](#basic-working-flow)
  - [Multi-threading Implementation](#multi-threading-implementation)
  - [Motivation and Learning Objectives](#motivation-and-learning-objectives)
  - [Operating System Components Used](#operating-system-components-used)
  - [Current Limitations](#current-limitations)
  - [Potential Extensions](#potential-extensions)
- [How to Run](#how-to-run)
  - [Prerequisites](#prerequisites)
  - [Building and Running (with Docker - Recommended)](#building-and-running-with-docker---recommended)
  - [Building and Running (Native Linux)](#building-and-running-native-linux)
  - [Testing the Proxy](#testing-the-proxy)
- [Demo](#demo)

---

## Overview

This HTTP proxy server is developed in C to provide a fundamental understanding of network programming, concurrency, and caching mechanisms. It supports `GET` requests, utilizing a Least Recently Used (LRU) cache policy to store and retrieve frequently accessed web content, thereby reducing latency and bandwidth usage.

## Features

*   **HTTP Request Parsing:** Robust parsing of HTTP/1.0 and HTTP/1.1 `GET` requests, extracting method, URL components (protocol, host, port, path), and headers.
*   **Header Management:** Dynamic setting, getting, and removing of HTTP headers (e.g., ensuring `Connection: close` and `Host` headers are correctly set for origin server communication).
*   **Multi-threading:** Handles concurrent client connections using POSIX threads (`pthread`).
*   **Concurrency Control:** Employs a counting semaphore (`sem_t`) to limit the number of active client-handling threads, preventing server overload.
*   **In-Memory Caching (LRU):**
    *   Stores HTTP responses to minimize redundant requests to origin servers.
    *   Implements a Least Recently Used (LRU) eviction strategy using a thread-safe doubly linked list.
    *   Configurable cache size (`MAX_CACHE_SIZE`) and individual element size (`MAX_ELEMENT_SIZE`).
*   **Thread Safety:** Utilizes `pthread_mutex_t` to protect shared resources, specifically the cache data structure, from race conditions.
*   **HTTP Error Handling:** Sends appropriate HTTP error responses (e.g., 400, 404, 500, 501, 505) to clients for malformed requests or server issues.
*   **Docker Support:** Includes a `Dockerfile` for easy containerization, ensuring a consistent and isolated development/deployment environment.

## Project Theory

[[Back to top]](#table-of-contents)

#### Basic Working Flow of the Proxy Server:

![Proxy Server Diagram](https://github.com/Garvit240205/HTTP-Proxy-Server/MultiThreadedProxyServerClient-main/pics/image.png)
_Conceptual diagram illustrating the flow of requests and responses through the proxy server._

#### Multi-threading Implementation:

The proxy utilizes `pthread` for multi-threading. A key design choice was the use of a counting semaphore (`sem_t`) instead of traditional `pthread_join()` for managing thread concurrency.
*   `pthread_join()` requires explicitly waiting for specific thread IDs to complete, which can be cumbersome for a server continuously accepting new, ephemeral connections.
*   Semaphores (`sem_wait()` and `sem_post()`) provide a simpler mechanism for resource limiting. The proxy's main loop uses `sem_wait()` before creating a thread, effectively pausing new client acceptance if the maximum number of concurrent threads (`MAX_CONCURRENT_CLIENTS`) is already active. Once a thread completes its task, it calls `sem_post()` to release a slot, allowing another client to be served. `pthread_detach()` is used for threads to automatically reclaim their resources upon exit, avoiding "zombie" threads.

#### Motivation and Learning Objectives:

This project was developed to deepen understanding of:
*   The intricacies of HTTP request/response flow from client to server.
*   Techniques for handling multiple concurrent client requests efficiently.
*   Synchronization primitives (locks and semaphores) for managing shared resources in concurrent environments.
*   The principles of web caching, including cache hit/miss scenarios and eviction algorithms (LRU).
*   Practical application of low-level C socket programming.

Proxy servers offer several benefits, including:
*   **Performance Enhancement:** Caching frequently accessed content reduces latency for clients and offloads traffic from origin servers.
*   **Access Control:** Can be configured to filter or restrict access to specific websites.
*   **Anonymity/Security:** Can mask client IP addresses or be extended to encrypt traffic for enhanced privacy and security.

#### Operating System Components Used:

*   **Threading:** `pthread` library for concurrent execution.
*   **Synchronization:** `sem_t` (semaphores) for concurrency limits and `pthread_mutex_t` (mutexes) for critical section protection (e.g., cache access).
*   **Memory Management:** Dynamic allocation using `malloc`, `realloc`, `free`, and `strdup`.
*   **Networking:** Standard Sockets API (`socket`, `bind`, `listen`, `accept`, `connect`, `send`, `recv`).
*   **Caching:** Implemented LRU algorithm for cache eviction.

#### Current Limitations:

*   **Fixed Cache Element Size:** Individual cached responses are limited by `MAX_ELEMENT_SIZE`, meaning very large web pages or files may not be cached.
*   **Partial Response Caching:** If a complex web page loads multiple clients/resources itself (e.g., embedded images, scripts), the cache currently treats each of these as a separate HTTP response. A single URL might not fully open from cache if it relies on many sub-elements being cached individually and the cache is small.
*   **`GET` Method Only:** The proxy only supports HTTP `GET` requests. Other methods like `POST`, `PUT`, `DELETE` are not implemented.
*   **No HTTPS Support:** Does not handle encrypted HTTPS traffic (which typically requires a different proxying approach like CONNECT tunneling).

#### Potential Extensions:

*   **Multi-processing:** Explore using `fork()` for a multi-process architecture, leveraging parallelism across CPU cores.
*   **Advanced Filtering:** Implement more sophisticated content filtering or URL blocking rules.
*   **Support for Other HTTP Methods:** Extend functionality to handle `POST`, `PUT`, `DELETE`, etc.
*   **Persistent Cache:** Implement a disk-based cache to persist content across proxy restarts.
*   **HTTPS (CONNECT) Tunneling:** Add support for HTTPS traffic.
*   **Robust Error Handling:** Further refine error handling and logging for production-grade robustness.
*   **Configuration File:** Allow port, cache size, and other parameters to be configured via a file instead of command-line arguments.

---

## How to Run

The project can be run directly on a Linux machine or, preferably, using Docker for a consistent and isolated environment.

### Prerequisites

*   **Git:** For cloning the repository.
*   **Docker Desktop:** For building and running the server in a containerized environment.
*   **`make`:** Build tool.
*   **C/C++ Compiler (GCC/G++):** For native compilation.

### Building and Running (with Docker)

Docker provides a clean and portable way to run the proxy without worrying about local dependencies.

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Garvit240205/HTTP-Proxy-Server.git
    cd HTTP-Proxy-Server
    ```

2.  **Build the Docker image:**
    This command compiles the C code and packages it into a Docker image named `http-proxy`.
    ```bash
    docker build -t http-proxy .
    ```

3.  **Run the Docker container:**
    This starts the proxy server in the background, mapping port `8080` from your host machine to port `8080` inside the container.
    ```bash
    docker run -d -p 8080:8080 --name my-proxy http-proxy
    ```
    *   `-d`: Runs the container in detached (background) mode.
    *   `-p 8080:8080`: Maps host port `8080` to container port `8080`.
    *   `--name my-proxy`: Assigns a readable name to your container.

4.  **Verify container status:**
    ```bash
    docker ps
    ```
    You should see `my-proxy` listed with `Up` status.

5.  **View proxy logs (important for debugging):**
    ```bash
    docker logs -f my-proxy
    ```
    Keep this terminal open to observe the proxy's internal operations and debug messages.

### Building and Running (Native Linux)

If you prefer to run the server directly on a Linux system.

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/Garvit240205/HTTP-Proxy-Server.git
    cd HTTP-Proxy-Server
    ```

2.  **Build the executable:**
    The `Makefile` will compile the source files.
    ```bash
    make all
    ```

3.  **Run the proxy server:**
    Replace `<port_no.>` with your desired port (e.g., `8080`).
    ```bash
    ./proxy <port_no.>
    ```
    *Note: This code can only be run in Linux-like environments due to specific system calls used.*

### Testing the Proxy

Once the proxy server is running (either via Docker or natively):

1.  **Configure your browser:**
    Go to your browser's proxy settings and configure it to use an HTTP proxy at `localhost` and the port you specified (e.g., `8080`).

2.  **Use `curl` (command-line):**
    ```bash
    curl -x localhost:<port_no.> http://example.com
    ```
    Replace `<port_no.>` with the port your proxy is listening on.

## Demo

![Starting Container](https://raw.githubusercontent.com/Garvit240205/HTTP-Proxy-Server/main/MultiThreadedProxyServerClient-main/pics/starting%20container%20(not%20in%20cache).png)

![First Response](https://raw.githubusercontent.com/Garvit240205/HTTP-Proxy-Server/main/MultiThreadedProxyServerClient-main/pics/First%20Response%20(not%20in%20cache).png)

![Second Response](https://raw.githubusercontent.com/Garvit240205/HTTP-Proxy-Server/main/MultiThreadedProxyServerClient-main/pics/Second%20Response%20(not%20in%20cache).png)

_Illustrates cache hit and miss scenarios in the proxy logs._

*   **Cache Miss:** When a website/resource is accessed for the first time, the proxy logs will show "url not found" (or similar debug output), indicating a cache miss. The request is forwarded to the origin server.
*   **Cache Hit:** If you access the same website/resource again, the logs will show "Data retrieved from the Cache", confirming a cache hit and faster retrieval.

---

[[Back to top]](#table-of-contents)
