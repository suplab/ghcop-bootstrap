---
applyTo: "**/Dockerfile*, **/docker-compose*.yml, **/docker-compose*.yaml, **/*.k8s.yaml, **/k8s/**/*.yaml, **/kubernetes/**/*.yaml, **/*.helm.yaml, **/helm/**/*.yaml"
description: "Container and Kubernetes standards: multi-stage Dockerfiles, security hardening, K8s manifest patterns, resource limits, and liveness/readiness probe configuration."
---

## Context

This instruction file applies to Docker and Kubernetes configuration files. All production containers must be non-root, resource-bounded, and include health probes. Container images must be built with minimal attack surface using multi-stage builds.

---

## Dockerfile Standards

### Multi-Stage Builds (Required)

Always use multi-stage builds. Never ship build tools in the runtime image.

```dockerfile
# Stage 1: Build (JDK)
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .
RUN ./mvnw dependency:go-offline -q
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Stage 2: Runtime (JRE only — ~150MB vs ~500MB for JDK)
FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar
USER appuser
EXPOSE 8080
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "org.springframework.boot.loader.launch.JarLauncher"]
```

### Security Rules

- **Never run as root** — create a non-root user in the Dockerfile
- **Use specific image tags** — never `FROM openjdk:latest`; pin to `eclipse-temurin:21.0.3_9-jre-alpine`
- **Minimal base image** — prefer Alpine variants; consider Distroless for production
- **No secrets in Dockerfile** — never `ARG` or `ENV` for credentials; use runtime secrets injection
- **`.dockerignore`** must exist and exclude: `.git`, `target/`, `node_modules/`, `*.env`, `*.key`

### JVM Container Flags

```dockerfile
# CORRECT: Container-aware JVM settings
ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "-XX:+ExitOnOutOfMemoryError", \
  "-Djava.security.egd=file:/dev/./urandom", \
  "org.springframework.boot.loader.launch.JarLauncher"]
```

---

## Kubernetes Manifest Standards

### Resource Requests and Limits (Required)

All containers must declare resource requests and limits. No limits = noisy neighbour risk.

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

### Liveness and Readiness Probes (Required)

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 5
  failureThreshold: 3
```

### Security Context (Required)

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000

containers:
  - securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

### Pod Disruption Budget (Production)

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: app-service
```

---

## Docker Compose Standards

- Use named volumes, not bind mounts, for database data directories
- Define `healthcheck` for all services that other services depend on
- Use `depends_on: { condition: service_healthy }` — not just `depends_on`
- Use `.env` file for local variables; never commit credentials in `docker-compose.yml`

---

## Image Tagging Strategy

| Tag Pattern | When to Use |
|-------------|------------|
| `{registry}/{image}:{git-sha}` | CI builds (immutable) |
| `{registry}/{image}:{env}-latest` | Environment-pinned (mutable) |
| `{registry}/{image}:{semver}` | Release artefacts |

Never deploy `latest` to production — use the git SHA or semver tag.
