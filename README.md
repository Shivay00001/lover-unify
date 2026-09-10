# lover-unify

![lover-unify banner](https://image.pollinations.ai/prompt/sleek%20futuristic%20golang%20microservice%20banner%2C%20dark%20cyberpunk%20aesthetic%2C%20glowing%20teal%20and%20purple%20server%20nodes%20connected%20by%20light%20streams%2C%20minimalist%20high-tech%20design%2C%20wide%20cinematic%20composition)

> **A minimal, zero-dependency Go 1.20 HTTP microservice — containerized with Docker, reporting liveness and server time on port `8080`.**

---

## 🚀 Overview

**lover-unify** is a deliberately minimal Go microservice built entirely on the standard library. It exposes a single HTTP endpoint on port `8080` that responds with a plain-text operational status message containing the server's current timestamp. The repository consists of exactly one source file (`main.go`), a `Dockerfile` based on `golang:1.20-alpine`, and a `go.mod` with **zero external dependencies**.

This makes it an ideal:

- 🩺 **Liveness / health-check service**
- 🧱 **Skeleton foundation** for a larger Go microservice
- 📦 **Reference implementation** of a containerized Go deployment

## ✨ Features

| Feature | Detail |
|---|---|
| ⚡ Zero dependencies | Only `net/http`, `fmt`, `time`, `log` from the Go stdlib |
| 🐳 Docker-ready | Working `Dockerfile` (Go 1.20 Alpine base) |
| 🚀 Instant startup | Single static binary with sub-second boot |
| 🕐 Timestamped response | Confirms both service availability and clock state |
| 🔌 Single endpoint | Catch-all handler on `/` responding to every path |

## 🏗️ Architecture — How It Works

The entire application logic lives in `main.go` (~15 lines). Here is the exact execution flow derived from the code:

1. **Handler registration** — `http.HandleFunc("/", ...)` registers one catch-all handler on Go's default `http.ServeMux`. Because `/` is the root pattern, **every request to any path** receives the same response.
2. **Response generation** — the handler calls `fmt.Fprintf(w, "System Operational: %s", time.Now())`, writing the server's current local timestamp directly into the `http.ResponseWriter`.
3. **Startup logging** — `log.Println("Starting high-performance service on :8080")` announces boot to stdout.
4. **Blocking serve** — `http.ListenAndServe(":8080", nil)` binds to port `8080` and serves forever. On bind failure, `log.Fatal` prints the error and exits the process with a non-zero code.
5. **Containerization** — the `Dockerfile` copies the source into `golang:1.20-alpine`, compiles with `go build -o app`, and sets the binary as the container's `CMD`. Since `go.mod` has no `require` directives, the build performs **no network module fetches**.

### 🔄 Request Flow

```mermaid
sequenceDiagram
    participant C as Client (curl / browser)
    participant D as Docker Container
    participant M as Go http.ServeMux
    participant H as Handler func
    participant T as time.Now()

    C->>D: HTTP GET / (port 8080)
    D->>M: Route request (default mux)
    M->>H: Invoke handler for "/"
    H->>T: Fetch current server time
    T-->>H: time.Time
    H-->>C: 200 OK "System Operational: <timestamp>"
```

### 🧩 Component Structure

```mermaid
flowchart TB
    subgraph Repo["lover-unify repository"]
        MG["main.go<br/>(~15 lines, package main)"]
        GM["go.mod<br/>(module github.com/Shivay00001/lover-unify, go 1.20, no deps)"]
        DF["Dockerfile<br/>(golang:1.20-alpine)"]
        LC["LICENSE<br/>(VisionQuantech Custom Commercial)"]
    end

    subgraph Build["Build Pipeline"]
        CP["COPY . ."] --> GB["go build -o app"] --> BIN["Static binary: app"]
    end

    subgraph Runtime["Runtime (container :8080)"]
        LS["http.ListenAndServe(':8080', nil)"] --> MUX["Default ServeMux"]
        MUX --> HND["Handler: fmt.Fprintf(w, 'System Operational: %s', time.Now())"]
    end

    MG --> Build --> Runtime
    DF -.defines.-> Build
```

### 📊 Response Lifecycle

```mermaid
flowchart LR
    A["Incoming TCP<br/>connection :8080"] --> B["net/http parses<br/>HTTP request"]
    B --> C{"Default mux<br/>matches '/'"}
    C -->|always true| D["Handler executes"]
    D --> E["time.Now()"]
    E --> F["fmt.Fprintf to<br/>ResponseWriter"]
    F --> G["Plain-text response<br/>+ implicit 200 OK"]
```

## 🐳 Running with Docker

### Option 1 — Docker CLI (recommended)

```bash
# 1. Clone the repository
git clone https://github.com/Shivay00001/lover-unify.git
cd lover-unify

# 2. Build the image
docker build -t shivay00001/lover-unify .

# 3. Run the container (host:container port mapping)
docker run -d -p 8080:8080 --name lover-unify shivay00001/lover-unify

# 4. Verify
curl http://localhost:8080/
# → System Operational: 2026-01-01 12:00:00.000000000 +0000 UTC ...
```

### Option 2 — Docker Compose

No `docker-compose.yml` ships with the repo, but this minimal file works out of the box:

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "8080:8080"
    restart: unless-stopped
```

Then:

```bash
docker-compose up -d --build
```

### 🛑 Stopping & cleanup

```bash
docker stop lover-unify && docker rm lover-unify
# or, with Compose:
docker-compose down
```

## 🛠️ Running Locally (no Docker)

Requires **Go 1.20+**:

```bash
go build -o app .
./app
# or simply:
go run main.go
```

![Go microservice container illustration](https://image.pollinations.ai/prompt/isometric%203d%20illustration%20of%20a%20docker%20container%20running%20a%20golang%20gopher%20powered%20microservice%2C%20glowing%20network%20ports%2C%20clean%20dark%20background%2C%20technical%20diagram%20style)

## 📋 Workability Assessment

**Honest verdict: functional, but not production-ready.**

**✅ What works:**
- The code compiles and runs correctly as written.
- The Dockerfile is valid and produces a working container.
- The service responds as expected on port `8080` with zero external dependencies.

**⚠️ What needs work before production:**
- **Trivial functionality** — it is a skeleton, not a complete application.
- **No graceful shutdown** — no `os/signal` handling or `http.Server.Shutdown`; in-flight requests are dropped on termination.
- **No timeouts** — default `http.ListenAndServe` is vulnerable to Slowloris-style attacks; `ReadTimeout`/`WriteTimeout` should be set on an explicit `http.Server`.
- **Catch-all routing** — every path returns the same response; no `/healthz`, no routing, no 404 handling.
- **No tests / no CI** — zero test coverage and no linting pipeline.
- **Non-optimized Dockerfile** — single-stage build ships the full Go toolchain (~300MB+); a multi-stage build with a `scratch`/`alpine` final stage would shrink this to a few MB. The container also runs as root.
- **No configuration** — port `8080` is hardcoded; no environment variable support.

**Summary:** A clean, deployable Go microservice template — but it requires hardening (timeouts, graceful shutdown, non-root user, multi-stage build) and real business logic before production use.

## 📄 License

Distributed under the **VisionQuantech Custom Commercial License** — see [LICENSE](LICENSE) for full terms.

- 🆓 **Free** for personal, educational, and non-earning use.
- 💰 **Revenue share (15–30%)** required for individual/indie commercial earning use.
- 🏢 **Separate commercial license required** for business/enterprise use — contact: **visionquantech@proton.me**

---

*Copyright © 2026 Shivay00001 / VisionQuantech*