# lover-unify

A minimal, containerized Go HTTP microservice that reports liveness and the current server time. Built with the Go standard library only — zero external dependencies.

---

## 🚀 Overview

**lover-unify** is a lightweight Go 1.20 service exposing a single HTTP endpoint on port `8080`. When hit, it responds with a plain-text operational status message containing the server's current timestamp. The repository is intentionally minimal: a single `main.go`, a multi-stage-ready `Dockerfile` based on Alpine, and no third-party modules (`go.mod` declares no dependencies).

This makes it a suitable foundation for:

- A health-check / liveness probe service
- A skeleton for building out a larger Go microservice
- A demonstration of containerized Go deployment

## ✨ Features

- **Zero-dependency**: uses only the Go standard library (`net/http`, `fmt`, `time`, `log`)
- **Containerized**: ships with a working `Dockerfile` (Go 1.20 Alpine base)
- **Instant startup**: single static binary, sub-second boot time
- **Timestamped liveness response**: useful for verifying clock sync and service availability

## 🏗️ Architecture / How It Works

The entire application lives in `main.go` (~15 lines):

```
Client ──HTTP GET /──▶ :8080 ──▶ handler ──▶ "System Operational: <timestamp>"
```

1. **Handler registration** — `http.HandleFunc("/", ...)` registers a single catch-all handler on Go's default `ServeMux`. Every request to any path receives the same response.
2. **Response generation** — the handler writes `System Operational: <time.Now()>` to the `http.ResponseWriter`, embedding the server's current local timestamp.
3. **Server startup** — `log.Println` announces startup, then `http.ListenAndServe(":8080", nil)` blocks and serves requests. If the server fails to bind, `log.Fatal` exits the process with the error.
4. **Containerization** — the `Dockerfile` copies the source into a `golang:1.20-alpine` image, compiles with `go build -o app`, and runs the resulting binary as the container's entrypoint.

Because `go.mod` has no `require` directives, the build requires no network fetches beyond the base image.

## 🐳 Running with Docker

### Option 1: Docker (recommended)

```bash
# Build the image
docker build -t shivay00001/lover-unify .

# Run the container, mapping host port 8080 to container port 8080
docker run -d -p 8080:8080 --name lover-unify shivay00001/lover-unify
```

### Option 2: Docker Compose

No `docker-compose.yml` is included, but this minimal one works:

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "8080:8080"
```

Then:

```bash
docker-compose up -d --build
```

### Verify it works

```bash
curl http://localhost:8080/
# → System Operational: 2026-01-01 12:00:00.000000000 +0000 UTC ...
```

## 🛠️ Running Locally (without Docker)

Requires Go 1.20+:

```bash
go build -o app .
./app
# or simply:
go run main.go
```

## 📋 Workability Assessment

**Honest verdict: functional but not production-ready.**

**What works:**
- ✅ The code compiles and runs correctly as written.
- ✅ The Dockerfile is valid and will produce a working container.
- ✅ The service responds as expected on port 8080.

**What is missing / needs work before production use:**
- ⚠️ **Trivial functionality** — the service does one thing (returns a timestamp). It is a skeleton, not a complete application.
- ⚠️ **No graceful shutdown** — no signal handling (`os/signal`, `http.Server.Shutdown`), so in-flight requests are dropped on termination.
- ⚠️ **No timeouts configured** — `http.ListenAndServe` with default settings is vulnerable to slow-client (Slowloris) attacks. `ReadTimeout`/`WriteTimeout` should be set via an explicit `http.Server`.
- ⚠️ **Catch-all routing** — every path returns the same response; there is no dedicated `/healthz` endpoint, routing, or 404 handling.
- ⚠️ **No tests, no CI** — zero test coverage and no linting/verification pipeline.
- ⚠️ **Non-optimized Dockerfile** — no multi-stage build, so the final image carries the full Go toolchain (~300MB+); a `scratch` or `alpine` final stage would reduce this to a few MB. The container also runs as root.
- ⚠️ **No configuration** — port is hardcoded; no environment variable support.

**Summary:** This is a solid starting template for a Go microservice and deploys cleanly via Docker, but it requires hardening (timeouts, graceful shutdown, non-root user, multi-stage build) and actual business logic before it can be considered production-grade.

## 📄 License

Distributed under the **VisionQuantech Custom Commercial License** — see [LICENSE](LICENSE) for full terms.

- **Free** for personal, educational, and non-earning use.
- **Revenue share (15–30%)** required for individual/indie commercial earning use.
- **Separate commercial license required** for business/enterprise use — contact: visionquantech@proton.me

---

*Copyright © 2026 Shivay00001 / VisionQuantech*