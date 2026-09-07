# Demo App — Setup Guide

## Running with Podman or Docker on Windows

**Platform:** Windows 11 | Podman 5.x / Docker Desktop 24+

---

## 1. Application Overview

The demo app is a three-tier microservices platform for insurance claims management. It consists of three custom services plus supporting infrastructure.

| Tier | Technology | Host Port |
|---|---|---|
| UI | Next.js 16 + React 19 | 3001 |
| BFF Service | Spring Boot 4 + Java 21 | 8090 |
| Claims Service | Spring Boot 4 + Java 21 | 8081 (Podman) / 8080 (Docker) |
| Auth | Keycloak 26 (OAuth2/OIDC) | 8180 |
| Database | PostgreSQL 15 | 5432 |
| Cache | Redis 7 | 6379 |
| Event Streaming | Apache Kafka | 9092 |
| Kafka UI | provectuslabs/kafka-ui | 9093 |

---

## 2. Prerequisites

| Tool | Min Version | How to Check | Notes |
|---|---|---|---|
| Podman Desktop | 5.x+ | `podman --version` | Includes podman CLI + WSL2 backend |
| podman-compose | latest | `podman-compose --version` | `pip install podman-compose` |
| Java JDK 21 | 21.x | `java -version` | Eclipse Temurin recommended |
| Maven | 3.8+ | `mvn --version` | Must resolve to Java 21 (see Step 3) |
| Node.js | 20+ | `node --version` | Required for UI Dockerfile build |
| Python | 3.x | `python --version` | Required for podman-compose |

> **DOCKER:** Docker Desktop 24+ can be used as a drop-in alternative. All commands are identical except: remove `--in-pod false`, and port 8080 is available by default.

---

## 3. One-Time Setup

### 3.1 Install podman-compose

```bash
pip install podman-compose
```

### 3.2 Verify Maven uses Java 21

Maven may default to a different JDK via `JAVA_HOME`. Check which Java it resolves to:

```bash
mvn --version
# Expected: Java version: 21.x.x, vendor: Eclipse Adoptium
```

If it shows Java 17, prefix every `mvn` command in this guide with:

```bash
env -u JAVA_HOME mvn ...
```

> **TIP:** This unsets `JAVA_HOME` for that command so Maven falls back to the `java` on PATH, which should be Java 21 if Temurin 21 was installed last.

### 3.3 Windows — Check port 8080 availability

Port 8080 is sometimes reserved by Windows HTTP.sys (visible as PID 4 in Task Manager). Check:

```bash
netstat -ano | findstr ":8080"
```

If it shows `LISTENING` on PID 4, port 8080 is blocked by the Windows kernel. You must use port **8081** for the Claims Service host mapping (covered in Step 6).

---

## 4. Initialize and Start the Podman Machine (First Time Only)

Podman on Windows runs containers inside a WSL2-backed Linux VM called the Podman Machine.

### 4.1 Initialize the machine

```bash
podman machine init
```

This downloads the Podman Machine OS image and sets up the WSL2 distro. Takes 2–3 minutes on first run.

### 4.2 Start the machine

```bash
podman machine start
```

### 4.3 Verify

```bash
podman machine list
# Expected output:
# NAME                     VM TYPE   CREATED   LAST UP            CPUS   MEMORY   DISK SIZE
# podman-machine-default*  wsl       ...       Currently running  ...
```

> **NOTE:** On subsequent runs, only `podman machine start` is needed — skip machine init.

---

## 5. Build the Java JARs

The Dockerfiles for `claims-service` and `bff-service` are runtime-only images — they copy a pre-built JAR from the `target/` directory. Maven must run before the image build.

### 5.1 Navigate to the demo-app directory

```bash
cd /path/to/tech-assess-qa/demo-app
```

### 5.2 Build both services (parallel)

```bash
# Build claims-service
env -u JAVA_HOME mvn -f code/claims-service/pom.xml package -Dmaven.test.skip=true

# Build bff-service (run in a second terminal for parallel execution)
env -u JAVA_HOME mvn -f code/bff-service/pom.xml package -Dmaven.test.skip=true
```

> **IMPORTANT:** Use `-Dmaven.test.skip=true` (not `-DskipTests`). The flag `-DskipTests` skips test execution but still compiles tests — a known broken test in `UpdateClaimStatusUseCaseTest.java` will cause a compile failure. `-Dmaven.test.skip=true` skips both compilation and execution.

### 5.3 Verify JARs were created

```bash
ls code/claims-service/target/*.jar
ls code/bff-service/target/*.jar
# Each should show one JAR file, e.g. demo-0.0.1-SNAPSHOT.jar
```

---

## 6. Configure Port Mapping (Windows / Podman Only)

If port 8080 is reserved by Windows (see Step 3.3), edit `docker-compose.apps.yml` and change the `claims-service` host port from `8080` to `8081`.

**File:** `docker-compose.apps.yml` — claims-service ports section

Change this:

```yaml
    ports:
      - "8080:8080"
```

To this:

```yaml
    ports:
      - "8081:8080"
```

> **NOTE:** Container-to-container communication uses `CLAIMS_SERVICE_URL: http://claims-service:8080` (the container port), so service communication is unaffected. Only the host-facing Swagger URL changes to `http://localhost:8081/swagger-ui.html`.

---

## 7. Start Infrastructure Services

Start the database, messaging, auth, and cache services first. Application services depend on these being healthy.

```bash
cd /path/to/tech-assess-qa/demo-app

# Podman
podman compose -f docker-compose.infra.yml --in-pod false up -d

# Docker (no --in-pod flag needed)
docker compose -f docker-compose.infra.yml up -d
```

> **WHY `--in-pod false`:** Required for Podman. Without it, `podman-compose` places all containers in a shared pod network namespace. Keycloak, Kafka UI, and Claims Service all use internal port 8080, causing a conflict inside the shared pod. `--in-pod false` gives each container its own network namespace (same behaviour as Docker).

### Verify infrastructure is healthy (~30 seconds)

```bash
podman ps --filter "name=demo-"
# Expected: all 6 containers show (healthy) or Up
# demo-postgres    Up (healthy)   0.0.0.0:5432->5432/tcp
# demo-zookeeper   Up (healthy)
# demo-redis       Up (healthy)
# demo-keycloak    Up (healthy)   0.0.0.0:8180->8080/tcp
# demo-kafka       Up (healthy)   0.0.0.0:9092->9092/tcp
# demo-kafka-ui    Up             0.0.0.0:9093->8080/tcp
```

---

## 8. Start Application Services

Build and start the three custom application containers. Infrastructure must be running first.

```bash
# Podman (builds images then starts containers)
podman compose -f docker-compose.apps.yml --in-pod false up -d

# Docker
docker compose -f docker-compose.apps.yml up -d
```

First run will build Docker images (~2–3 minutes). Subsequent runs use cached images and take ~30 seconds.

### Verify application services are healthy (~60–90 seconds)

```bash
podman ps --filter "name=demo-"
# Expected:
# demo-claims-service   Up (healthy)   0.0.0.0:8081->8080/tcp
# demo-bff-service      Up (healthy)   0.0.0.0:8090->8090/tcp
# demo-ui               Up             0.0.0.0:3001->3001/tcp
```

### Health check endpoints

```bash
curl http://localhost:8081/actuator/health   # Claims Service
curl http://localhost:8090/actuator/health   # BFF Service
# Both should return: {"status":"UP"}
```

---

## 9. Access the Application

### Services

| Service | URL | Credentials |
|---|---|---|
| Demo App UI | http://localhost:3001 | See test users below |
| BFF Swagger | http://localhost:8090/swagger-ui.html | — |
| Claims Swagger | http://localhost:8081/swagger-ui.html | — |
| Keycloak Admin | http://localhost:8180 | admin / Admin123! |
| Kafka UI | http://localhost:9093 | — |

### Test Users

| Role | Username | Password |
|---|---|---|
| Admin | admin@demo.com | Admin123! |
| Claimant | claimant@demo.com | Claimant123! |

---

## 10. Rebuilding After Code Changes

### Rebuild everything (Java services)

```bash
# Step 1 - rebuild JARs
env -u JAVA_HOME mvn -f code/claims-service/pom.xml package -Dmaven.test.skip=true
env -u JAVA_HOME mvn -f code/bff-service/pom.xml package -Dmaven.test.skip=true

# Step 2 - rebuild and restart containers
podman compose -f docker-compose.apps.yml --in-pod false up --build -d
```

### Rebuild a single service only

```bash
# Rebuild just claims-service
env -u JAVA_HOME mvn -f code/claims-service/pom.xml package -Dmaven.test.skip=true
podman compose -f docker-compose.apps.yml --in-pod false up --build --pull never claims-service
```

---

## 11. Stopping the Application

```bash
# Stop application services
podman compose -f docker-compose.apps.yml --in-pod false down

# Stop infrastructure services
podman compose -f docker-compose.infra.yml --in-pod false down

# Stop everything and wipe all data volumes (clean slate)
podman compose -f docker-compose.apps.yml --in-pod false down
podman compose -f docker-compose.infra.yml --in-pod false down -v
```

---

## 12. Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Cannot connect to Podman | Machine not running | `podman machine start` |
| WSL `getpwnam(root)` failed on machine start | Corrupted WSL distro after crash or force-stop | `wsl --unregister podman-machine-default` then re-run machine init and start |
| Port 8080 already in use | Windows HTTP.sys (PID 4) reserves port 8080 | Change host port to `8081:8080` in `docker-compose.apps.yml` (see Step 6) |
| External network `demo-network` does not exist | Infra compose not started first, or machine was reset | Always run infra compose before apps compose |
| Maven build fails — COMPILATION ERROR in `UpdateClaimStatusUseCaseTest` | Intentionally broken test ships with the project | Use `-Dmaven.test.skip=true` instead of `-DskipTests` |
| `COPY target/*.jar` — no such file or directory (Docker build fails) | `.mvn/wrapper` directory missing, or Maven not run yet | Run Maven package step (Step 5) before running `compose up --build` |
| Port conflict inside pod (`claims-service` fails to start) | `podman-compose` created a shared pod; Keycloak + Kafka UI + claims-service all use port 8080 inside | Add `--in-pod false` to all `podman compose` commands |
| Services show `Created` but not `Up` after compose up | Stale container from a previous failed run holds the name or port | `podman container rm demo-claims-service demo-bff-service demo-ui` then re-run `compose up` |

---

## 13. Quick Reference — Command Cheat Sheet

| Task | Podman Command |
|---|---|
| Start machine | `podman machine start` |
| Start infrastructure | `podman compose -f docker-compose.infra.yml --in-pod false up -d` |
| Build JARs | `env -u JAVA_HOME mvn -f code/<svc>/pom.xml package -Dmaven.test.skip=true` |
| Start app services | `podman compose -f docker-compose.apps.yml --in-pod false up -d` |
| Check container status | `podman ps --filter "name=demo-"` |
| View service logs | `podman logs -f demo-claims-service` |
| Stop app services | `podman compose -f docker-compose.apps.yml --in-pod false down` |
| Stop infrastructure | `podman compose -f docker-compose.infra.yml --in-pod false down` |
| Wipe all data | `podman compose -f docker-compose.infra.yml --in-pod false down -v` |
| List running machines | `podman machine list` |
| Stop machine | `podman machine stop` |
