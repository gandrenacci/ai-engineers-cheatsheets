# Docker Complete Cheat Sheet

> Modern Docker CLI, Dockerfile, BuildKit/Buildx, Docker Compose, networking, storage, security, troubleshooting, and cleanup.
>
> **Syntax used:** current `docker compose` plugin syntax rather than the legacy `docker-compose` executable.

---

## Table of Contents

1. [Core concepts](#1-core-concepts)
2. [Installation checks and system information](#2-installation-checks-and-system-information)
3. [Command structure and help](#3-command-structure-and-help)
4. [Images](#4-images)
5. [Run containers](#5-run-containers)
6. [Container lifecycle](#6-container-lifecycle)
7. [Inspect, monitor, and debug](#7-inspect-monitor-and-debug)
8. [Execute commands and copy files](#8-execute-commands-and-copy-files)
9. [Ports and networking](#9-ports-and-networking)
10. [Volumes, bind mounts, and temporary filesystems](#10-volumes-bind-mounts-and-temporary-filesystems)
11. [Environment variables and configuration](#11-environment-variables-and-configuration)
12. [Registries and image transfer](#12-registries-and-image-transfer)
13. [Dockerfile reference](#13-dockerfile-reference)
14. [Production-ready Dockerfile example](#14-production-ready-dockerfile-example)
15. [BuildKit and Buildx](#15-buildkit-and-buildx)
16. [Docker Compose](#16-docker-compose)
17. [Security hardening](#17-security-hardening)
18. [Docker Scout and vulnerability scanning](#18-docker-scout-and-vulnerability-scanning)
19. [Resource limits and restart policies](#19-resource-limits-and-restart-policies)
20. [Cleanup and disk usage](#20-cleanup-and-disk-usage)
21. [Contexts and remote Docker hosts](#21-contexts-and-remote-docker-hosts)
22. [Docker Swarm essentials](#22-docker-swarm-essentials)
23. [Troubleshooting](#23-troubleshooting)
24. [Practical recipes](#24-practical-recipes)
25. [Dangerous commands](#25-dangerous-commands)
26. [Legacy syntax to avoid](#26-legacy-syntax-to-avoid)

---

## 1. Core Concepts

| Concept | Meaning |
|---|---|
| **Image** | Immutable, layered application package used to create containers. |
| **Container** | Running or stopped instance of an image. |
| **Registry** | Service that stores and distributes images, such as Docker Hub or a private registry. |
| **Repository** | Collection of related image tags in a registry. |
| **Tag** | Human-readable image version, such as `app:1.4.0`. |
| **Digest** | Immutable content identifier, such as `image@sha256:...`. |
| **Dockerfile** | Declarative instructions used to build an image. |
| **Build context** | Files available to a build, normally the directory passed to `docker build`. |
| **Volume** | Docker-managed persistent storage. |
| **Bind mount** | Host path mounted directly into a container. |
| **Network** | Virtual network connecting containers and other endpoints. |
| **Compose project** | Multi-container application defined in `compose.yaml`. |

### Image → container lifecycle

```text
Dockerfile
    │
    ├── docker build
    ▼
Docker image
    │
    ├── docker run
    ▼
Docker container
```

---

## 2. Installation Checks and System Information

```bash
# Docker CLI version
docker --version

# Client, Engine, API, containerd, and related component versions
docker version

# Engine configuration, storage driver, runtimes, resources, and plugins
docker info

# Verify that the Engine can pull and run an image
docker run --rm hello-world

# Show Docker disk usage
docker system df
docker system df -v
```

### Linux daemon status

```bash
sudo systemctl status docker
sudo systemctl start docker
sudo systemctl restart docker
sudo journalctl -u docker --since "30 minutes ago"
```

### Use Docker without `sudo` on Linux

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

> Membership in the `docker` group generally grants root-equivalent control over the host. Use it only for trusted users. Consider rootless Docker for stronger isolation.

---

## 3. Command Structure and Help

```bash
docker [GLOBAL_OPTIONS] COMMAND [COMMAND_OPTIONS] [ARGUMENTS]

docker help
docker COMMAND --help
docker container --help
docker image --help
docker compose --help
```

Docker supports both short aliases and explicit object commands:

```bash
docker ps                  # alias
docker container ls        # explicit form

docker images              # alias
docker image ls            # explicit form

docker rmi IMAGE           # alias
docker image rm IMAGE      # explicit form
```

### Useful global options

```bash
docker --context prod ps
docker --host ssh://user@server ps
docker --log-level debug info
```

---

## 4. Images

### List and inspect

```bash
docker image ls
docker image ls -a
docker image ls --digests
docker image inspect nginx:alpine
docker image history nginx:alpine
```

### Pull

```bash
docker pull nginx
docker pull nginx:alpine
docker pull --platform linux/amd64 nginx:alpine
docker pull nginx@sha256:DIGEST
```

### Build

```bash
docker build .
docker build -t myapp:latest .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker build --pull -t myapp:1.0 .
docker build --no-cache -t myapp:1.0 .
docker build --progress=plain -t myapp:1.0 .
```

### Tag

```bash
docker tag myapp:1.0 myregistry.example.com/team/myapp:1.0
docker tag myapp:1.0 myregistry.example.com/team/myapp:latest
```

### Remove

```bash
docker image rm myapp:1.0
docker image rm -f IMAGE_ID
docker image prune
docker image prune -a
```

### Filter and format

```bash
docker image ls --filter dangling=true
docker image ls --filter reference='myapp:*'
docker image ls --format 'table {{.Repository}}\t{{.Tag}}\t{{.ID}}\t{{.Size}}'
```

---

## 5. Run Containers

### Basic run

```bash
docker run IMAGE
docker run --name web nginx:alpine
docker run --rm alpine echo "hello"
```

### Interactive shell

```bash
docker run --rm -it ubuntu:24.04 bash
docker run --rm -it alpine sh
```

### Detached service

```bash
docker run -d --name web nginx:alpine
```

### Publish ports

```bash
# HOST_PORT:CONTAINER_PORT
docker run -d --name web -p 8080:80 nginx:alpine

# Bind only to localhost
docker run -d --name web -p 127.0.0.1:8080:80 nginx:alpine

# Publish all EXPOSE ports to random host ports
docker run -d -P nginx:alpine
```

### Environment variables

```bash
docker run --rm -e APP_ENV=production myapp:1.0
docker run --rm --env-file .env myapp:1.0
```

### Working directory and user

```bash
docker run --rm -w /app myapp:1.0 pwd
docker run --rm --user 10001:10001 myapp:1.0 id
```

### Override command or entrypoint

```bash
docker run --rm myapp:1.0 python --version
docker run --rm --entrypoint sh myapp:1.0
```

### Complete example

```bash
docker run -d \
  --name myapp \
  --restart unless-stopped \
  -p 127.0.0.1:8080:8000 \
  --env-file .env \
  --mount source=myapp-data,target=/app/data \
  --memory 512m \
  --cpus 1.0 \
  --pids-limit 200 \
  --read-only \
  --tmpfs /tmp \
  --security-opt no-new-privileges=true \
  --cap-drop ALL \
  myregistry.example.com/team/myapp:1.0
```

---

## 6. Container Lifecycle

```bash
# List running containers
docker ps
docker container ls

# List all containers
docker ps -a

# Create without starting
docker create --name web nginx:alpine

# Start
docker start web

# Start and attach
docker start -a web

# Graceful stop
docker stop web

# Stop with a custom timeout
docker stop --time 30 web

# Restart
docker restart web

# Pause and resume processes
docker pause web
docker unpause web

# Send a signal
docker kill --signal SIGHUP web

# Force stop using the default kill signal
docker kill web

# Rename
docker rename web web-old

# Wait for termination and print exit code
docker wait web

# Remove stopped container
docker rm web

# Force-remove a running container
docker rm -f web

# Remove a container and its anonymous volumes
docker rm -v web
```

### Operate on multiple containers

```bash
docker stop web api worker
docker rm web api worker

# Stop every running container
docker stop $(docker ps -q)

# Remove every stopped container
docker container prune
```

---

## 7. Inspect, Monitor, and Debug

### Logs

```bash
docker logs web
docker logs -f web
docker logs --tail 100 web
docker logs --since 10m web
docker logs --timestamps web
docker logs -f --tail 100 web
```

### Inspect metadata

```bash
docker inspect web
docker inspect --type container web
docker inspect --format '{{.State.Status}}' web
docker inspect --format '{{.State.ExitCode}}' web
docker inspect --format '{{json .NetworkSettings.Networks}}' web
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web
```

### Processes and resource usage

```bash
docker top web
docker stats
docker stats web api
docker stats --no-stream
```

### Ports and filesystem changes

```bash
docker port web
docker diff web
```

`docker diff` output:

| Prefix | Meaning |
|---|---|
| `A` | Added |
| `C` | Changed |
| `D` | Deleted |

### Events

```bash
docker events
docker events --since 30m
docker events --filter container=web
docker events --filter event=die
```

### Useful formatted container list

```bash
docker ps --format \
  'table {{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}'
```

### Filters

```bash
docker ps --filter status=exited
docker ps --filter name=web
docker ps --filter ancestor=nginx:alpine
docker ps --filter health=unhealthy
docker ps -q --filter status=running
```

---

## 8. Execute Commands and Copy Files

### Run a command inside an existing container

```bash
docker exec web nginx -t
docker exec -it web sh
docker exec -it api bash
docker exec -u root -it api sh
docker exec -w /app api pwd
docker exec -e DEBUG=true api env
```

### Attach to the main process

```bash
docker attach web
```

Detach without stopping the container:

```text
Ctrl-p Ctrl-q
```

> Prefer `docker exec` for debugging. `docker attach` connects directly to the container's main process and can affect it.

### Copy files

```bash
# Host → container
docker cp ./config.yaml api:/app/config.yaml

# Container → host
docker cp api:/app/output.json ./output.json
```

### Export and import a container filesystem

```bash
docker export web > web-filesystem.tar
docker import web-filesystem.tar myimage:imported
```

> `export/import` handles a flattened container filesystem and does not preserve image history, metadata, or named volumes.

### Create an image from container changes

```bash
docker commit web myimage:debug-snapshot
```

> `docker commit` is useful for investigation but should not replace a reproducible Dockerfile.

---

## 9. Ports and Networking

### Important port rule

```text
-p HOST_PORT:CONTAINER_PORT
```

Example:

```bash
docker run -d -p 8080:80 nginx:alpine
```

The application must listen on the container interface, normally `0.0.0.0`, not only `127.0.0.1`.

### List networks

```bash
docker network ls
docker network inspect bridge
```

### Create a user-defined bridge network

```bash
docker network create app-net
```

### Attach containers

```bash
docker run -d --name db --network app-net postgres:17
docker run -d --name api --network app-net myapi:1.0
```

On a user-defined network, containers can reach one another using container or service names:

```text
postgresql://db:5432/app
```

### Connect or disconnect an existing container

```bash
docker network connect app-net web
docker network disconnect app-net web
```

### Remove and prune

```bash
docker network rm app-net
docker network prune
```

### Common network drivers

| Driver | Use |
|---|---|
| `bridge` | Default single-host container networking. |
| `host` | Container shares the host network namespace on supported platforms. |
| `none` | No external networking. |
| `overlay` | Multi-host networking in Docker Swarm. |
| `macvlan` | Container appears as a device on the physical network. |

### Host access from a container

Docker Desktop commonly provides:

```text
host.docker.internal
```

On Linux Engine, explicitly add the host gateway when needed:

```bash
docker run --add-host host.docker.internal:host-gateway myapp:1.0
```

### DNS troubleshooting

```bash
docker exec api getent hosts db
docker exec api cat /etc/resolv.conf
docker network inspect app-net
```

---

## 10. Volumes, Bind Mounts, and Temporary Filesystems

### Storage choices

| Type | Best use |
|---|---|
| **Named volume** | Persistent application data managed by Docker. |
| **Bind mount** | Source code, host configuration, or files intentionally shared with the host. |
| **tmpfs** | Ephemeral, memory-backed sensitive or temporary data on Linux. |

### Named volumes

```bash
docker volume create app-data
docker volume ls
docker volume inspect app-data
docker volume rm app-data
docker volume prune
```

Mount with the recommended `--mount` syntax:

```bash
docker run -d \
  --name db \
  --mount source=postgres-data,target=/var/lib/postgresql/data \
  postgres:17
```

Equivalent short syntax:

```bash
docker run -d -v postgres-data:/var/lib/postgresql/data postgres:17
```

### Bind mounts

```bash
docker run --rm \
  --mount type=bind,source="$PWD",target=/workspace \
  -w /workspace \
  python:3.13 python app.py
```

Read-only bind mount:

```bash
docker run --rm \
  --mount type=bind,source="$PWD/config",target=/app/config,readonly \
  myapp:1.0
```

Short syntax:

```bash
docker run --rm -v "$PWD:/workspace:ro" alpine ls /workspace
```

### Temporary filesystem

```bash
docker run --rm --tmpfs /tmp:rw,noexec,nosuid,size=64m myapp:1.0
```

### Back up a named volume

```bash
docker run --rm \
  -v app-data:/data:ro \
  -v "$PWD:/backup" \
  alpine \
  tar czf /backup/app-data-backup.tgz -C /data .
```

### Restore a named volume

```bash
docker run --rm \
  -v app-data:/data \
  -v "$PWD:/backup:ro" \
  alpine \
  sh -c 'rm -rf /data/* && tar xzf /backup/app-data-backup.tgz -C /data'
```

### Inspect a volume's mount point

```bash
docker volume inspect \
  --format '{{.Mountpoint}}' app-data
```

> Avoid manually editing Docker-managed volume directories on the host.

---

## 11. Environment Variables and Configuration

### Pass one variable

```bash
docker run --rm -e APP_ENV=production myapp:1.0
```

### Forward an existing host variable

```bash
export API_URL=https://api.example.com
docker run --rm -e API_URL myapp:1.0
```

### Load a file

```bash
docker run --rm --env-file .env myapp:1.0
```

Example `.env`:

```dotenv
APP_ENV=production
LOG_LEVEL=info
DATABASE_HOST=db
DATABASE_PORT=5432
```

### Show environment variables in a container

```bash
docker exec api env
docker inspect --format '{{json .Config.Env}}' api
```

> Environment variables are visible through inspection and can leak through process or diagnostic output. Do not use them as the default mechanism for high-value secrets.

---

## 12. Registries and Image Transfer

### Authenticate

```bash
docker login
docker login registry.example.com
docker logout registry.example.com
```

Prefer token input through standard input:

```bash
printf '%s' "$REGISTRY_TOKEN" |
  docker login registry.example.com \
    --username "$REGISTRY_USER" \
    --password-stdin
```

### Push

```bash
docker tag myapp:1.0 registry.example.com/team/myapp:1.0
docker push registry.example.com/team/myapp:1.0
```

### Pull

```bash
docker pull registry.example.com/team/myapp:1.0
```

### Save and load images without a registry

```bash
docker save -o myapp.tar myapp:1.0
docker load -i myapp.tar
```

Compressed transfer:

```bash
docker save myapp:1.0 | gzip > myapp.tar.gz
gunzip -c myapp.tar.gz | docker load
```

Transfer over SSH:

```bash
docker save myapp:1.0 |
  gzip |
  ssh user@server 'gunzip | docker load'
```

### `save/load` versus `export/import`

| Pair | Operates on | Preserves image metadata and layers |
|---|---|---|
| `docker save` / `docker load` | Images | Yes |
| `docker export` / `docker import` | Container filesystem | No |

---

## 13. Dockerfile Reference

### Main instructions

| Instruction | Purpose |
|---|---|
| `# syntax=` | Select Dockerfile frontend features. |
| `ARG` | Build-time variable. |
| `FROM` | Select base image and start a build stage. |
| `LABEL` | Add image metadata. |
| `ENV` | Set runtime environment variable. |
| `WORKDIR` | Set working directory for subsequent instructions. |
| `COPY` | Copy files from build context or another stage. |
| `ADD` | Copy with extra behavior such as local archive extraction; use only when needed. |
| `RUN` | Execute a build step and create a layer. |
| `USER` | Set default runtime user. |
| `EXPOSE` | Document the intended container port. |
| `VOLUME` | Declare a mount point. |
| `HEALTHCHECK` | Define a container health probe. |
| `STOPSIGNAL` | Set the signal used to stop the container. |
| `ENTRYPOINT` | Configure the main executable. |
| `CMD` | Set default arguments or command. |
| `SHELL` | Override the default shell. |
| `ONBUILD` | Add trigger instructions for downstream builds. |

### Exec form versus shell form

Preferred exec form:

```dockerfile
CMD ["python", "-m", "myapp"]
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
```

Shell form:

```dockerfile
CMD python -m myapp
```

Exec form avoids an unnecessary shell and generally handles signals more predictably.

### `ENTRYPOINT` and `CMD` together

```dockerfile
ENTRYPOINT ["python", "-m", "myapp"]
CMD ["--host", "0.0.0.0", "--port", "8000"]
```

Runtime override:

```bash
docker run myapp:1.0 --port 9000
```

### `.dockerignore`

```gitignore
.git
.github
.env
.env.*
!.env.example
__pycache__/
*.py[cod]
.pytest_cache/
.mypy_cache/
.ruff_cache/
.venv/
venv/
node_modules/
dist/
build/
coverage/
*.log
Dockerfile*
compose*.yaml
README.md
docs/
```

Adjust the exclusions when files such as documentation or Compose definitions are intentionally required by the build.

### Build-time arguments

```dockerfile
ARG PYTHON_VERSION=3.13
FROM python:${PYTHON_VERSION}-slim
ARG APP_VERSION=dev
LABEL org.opencontainers.image.version="${APP_VERSION}"
```

```bash
docker build \
  --build-arg PYTHON_VERSION=3.13 \
  --build-arg APP_VERSION=1.4.0 \
  -t myapp:1.4.0 .
```

> Do not pass secrets through `ARG` or `ENV`; they can persist in image metadata, layers, or build records.

---

## 14. Production-Ready Dockerfile Example

Python example using a multi-stage build, dependency cache, non-root user, and health check:

```dockerfile
# syntax=docker/dockerfile:1

ARG PYTHON_VERSION=3.13

FROM python:${PYTHON_VERSION}-slim AS builder

ENV PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /build

COPY requirements.txt .

RUN --mount=type=cache,target=/root/.cache/pip \
    pip wheel --wheel-dir /wheels -r requirements.txt


FROM python:${PYTHON_VERSION}-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

RUN useradd \
      --create-home \
      --uid 10001 \
      --shell /usr/sbin/nologin \
      appuser

COPY --from=builder /wheels /wheels
COPY requirements.txt .

RUN pip install --no-cache-dir /wheels/* \
    && rm -rf /wheels

COPY --chown=appuser:appuser src/ ./src/

USER 10001:10001

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s --start-period=20s --retries=3 \
  CMD python -c \
  "import urllib.request; urllib.request.urlopen('http://127.0.0.1:8000/health')"

ENTRYPOINT ["python", "-m", "src.app"]
CMD ["--host", "0.0.0.0", "--port", "8000"]
```

Build and run:

```bash
docker build --pull -t myapp:1.0 .
docker run --rm -p 127.0.0.1:8000:8000 myapp:1.0
```

### Dockerfile best-practice checklist

- Use trusted, maintained base images.
- Pin production images by immutable version or digest where appropriate.
- Use multi-stage builds.
- Keep the final runtime image minimal.
- Create and use a non-root user.
- Copy dependency manifests before source code to improve cache reuse.
- Keep build context small with `.dockerignore`.
- Combine package installation and cleanup in the same `RUN` step.
- Avoid embedding credentials.
- Use BuildKit secret or SSH mounts for build credentials.
- Prefer `COPY` over `ADD` unless `ADD` behavior is specifically required.
- Use exec-form `ENTRYPOINT` and `CMD`.
- Add a meaningful health check when the application supports one.
- Rebuild regularly to pick up operating-system and dependency fixes.
- Scan the final image.

---

## 15. BuildKit and Buildx

### Buildx status

```bash
docker buildx version
docker buildx ls
docker buildx inspect
```

### Create a builder

```bash
docker buildx create \
  --name multiarch \
  --driver docker-container \
  --use

docker buildx inspect --bootstrap
```

### Multi-platform build

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry.example.com/team/myapp:1.0 \
  --push \
  .
```

Single-platform build loaded into the local image store:

```bash
docker buildx build \
  --platform linux/amd64 \
  -t myapp:local \
  --load \
  .
```

### Build cache

```bash
docker buildx build \
  --cache-from type=registry,ref=registry.example.com/team/myapp:buildcache \
  --cache-to type=registry,ref=registry.example.com/team/myapp:buildcache,mode=max \
  -t registry.example.com/team/myapp:1.0 \
  --push \
  .
```

Local cache:

```bash
docker buildx build \
  --cache-from type=local,src=.buildx-cache \
  --cache-to type=local,dest=.buildx-cache-new,mode=max \
  -t myapp:1.0 \
  .
```

### Build secrets

```bash
docker build \
  --secret id=pypi_token,src=.pypi-token \
  -t myapp:1.0 .
```

Dockerfile:

```dockerfile
# syntax=docker/dockerfile:1

RUN --mount=type=secret,id=pypi_token \
    TOKEN="$(cat /run/secrets/pypi_token)" \
    && pip install \
       --index-url "https://token:${TOKEN}@packages.example.com/simple" \
       private-package
```

Secret from an environment variable:

```bash
export PYPI_TOKEN='...'

docker build \
  --secret id=pypi_token,env=PYPI_TOKEN \
  -t myapp:1.0 .
```

### SSH mount

```bash
docker buildx build \
  --ssh default \
  -t myapp:1.0 .
```

```dockerfile
# syntax=docker/dockerfile:1

RUN --mount=type=ssh \
    git clone git@github.com:organization/private-repository.git
```

### Cache mount

```dockerfile
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt
```

### Build one stage

```bash
docker build --target builder -t myapp:builder .
```

### Build metadata

```bash
docker buildx build \
  --sbom=true \
  --provenance=true \
  -t registry.example.com/team/myapp:1.0 \
  --push \
  .
```

### Builder cleanup

```bash
docker buildx du
docker builder prune
docker builder prune -a
docker buildx prune
```

---

## 16. Docker Compose

Compose manages services, networks, volumes, configs, and secrets for a multi-container application.

### Modern `compose.yaml` example

```yaml
name: example-app

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: runtime
    image: example-api:local
    init: true
    restart: unless-stopped
    env_file:
      - .env
    environment:
      DATABASE_HOST: db
      DATABASE_PORT: "5432"
    ports:
      - "127.0.0.1:8000:8000"
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - app-data:/app/data
    networks:
      - frontend
      - backend
    healthcheck:
      test:
        - CMD
        - python
        - -c
        - >-
          import urllib.request;
          urllib.request.urlopen('http://127.0.0.1:8000/health')
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 20s
    read_only: true
    tmpfs:
      - /tmp
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL

  db:
    image: postgres:17
    restart: unless-stopped
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

volumes:
  app-data:
  postgres-data:

networks:
  frontend:
  backend:
    internal: true

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

> Do not commit `./secrets/db_password.txt` or real `.env` files.

### Lifecycle commands

```bash
# Validate and render the effective configuration
docker compose config

# Build, create, and start
docker compose up

# Start in background
docker compose up -d

# Build before starting
docker compose up -d --build

# Recreate even when configuration appears unchanged
docker compose up -d --force-recreate

# Start only selected services
docker compose up -d db api

# Stop without removing
docker compose stop

# Start existing stopped services
docker compose start

# Restart
docker compose restart

# Stop and remove project containers and networks
docker compose down

# Also remove named project volumes
docker compose down -v

# Also remove project images
docker compose down --rmi local
```

### Inspect and debug

```bash
docker compose ps
docker compose ps -a
docker compose images
docker compose top

docker compose logs
docker compose logs -f
docker compose logs -f --tail 100 api

docker compose exec api sh
docker compose exec db psql -U app -d app

docker compose run --rm api python -m pytest
docker compose run --rm --no-deps api python -m src.cli
```

### Build and pull

```bash
docker compose build
docker compose build --no-cache
docker compose build --pull
docker compose pull
docker compose push
```

### Configuration and discovery

```bash
docker compose config
docker compose config --services
docker compose config --images
docker compose ls
```

### Scale a service

```bash
docker compose up -d --scale worker=3
```

Do not set a fixed `container_name` for services that need to scale.

### Multiple Compose files

```bash
docker compose \
  -f compose.yaml \
  -f compose.prod.yaml \
  up -d
```

Later files override or extend earlier configuration.

### Project name

```bash
docker compose -p myproject up -d
```

Or:

```bash
export COMPOSE_PROJECT_NAME=myproject
docker compose up -d
```

### Profiles

```yaml
services:
  api:
    image: myapi:1.0

  adminer:
    image: adminer
    profiles: ["debug"]
```

```bash
docker compose up -d
docker compose --profile debug up -d
```

### Environment interpolation

```yaml
services:
  api:
    image: "${REGISTRY:-docker.io}/myteam/api:${APP_TAG:-latest}"
```

```bash
APP_TAG=1.2.0 docker compose config
APP_TAG=1.2.0 docker compose up -d
```

### Compose networking

- Compose creates a project-level default network when none is declared.
- Services resolve one another by service name.
- Use the **container port**, not the published host port, for service-to-service communication.
- Use `internal: true` for networks that should not have external connectivity.

### Compose health and startup order

`depends_on` controls dependency order. For readiness-aware startup, combine:

```yaml
depends_on:
  db:
    condition: service_healthy
```

with a real `healthcheck` on the dependency.

---

## 17. Security Hardening

### Safer runtime example

```bash
docker run -d \
  --name api \
  --user 10001:10001 \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=64m \
  --cap-drop ALL \
  --security-opt no-new-privileges=true \
  --pids-limit 200 \
  --memory 512m \
  --cpus 1 \
  -p 127.0.0.1:8000:8000 \
  myapi:1.0
```

### Security checklist

- Run the application as a non-root user.
- Prefer rootless Docker where compatible.
- Avoid `--privileged`.
- Drop Linux capabilities and add back only those required.
- Keep the default seccomp profile enabled.
- Do not use `seccomp=unconfined` as a routine workaround.
- Use `no-new-privileges`.
- Make the root filesystem read-only where possible.
- Use `tmpfs` for writable temporary paths.
- Apply CPU, memory, PID, and file-size limits.
- Bind internal services to `127.0.0.1` or private networks.
- Never expose an unauthenticated Docker daemon TCP socket.
- Prefer SSH or mutual TLS for remote Engine access.
- Do not mount `/var/run/docker.sock` into untrusted containers.
- Treat Docker socket access as host-level administrative access.
- Do not store secrets in a Dockerfile, image, repository, `ARG`, or ordinary environment variable.
- Use BuildKit secrets for builds.
- Use Compose or Swarm secrets where appropriate at runtime.
- Pin important production dependencies by digest.
- Rebuild images regularly.
- Scan images and dependencies.
- Keep Docker Engine, Docker Desktop, plugins, base images, and the host kernel patched.
- Use a private registry policy and least-privilege credentials.
- Sign and verify images where your supply-chain tooling supports it.

### Drop and add capabilities

```bash
docker run --cap-drop ALL myapp:1.0

docker run \
  --cap-drop ALL \
  --cap-add NET_BIND_SERVICE \
  myapp:1.0
```

### Read-only root filesystem

```bash
docker run \
  --read-only \
  --tmpfs /tmp \
  --mount source=app-data,target=/app/data \
  myapp:1.0
```

### Rootless mode checks

```bash
docker info | grep -i rootless
docker context ls
```

### Avoid direct Docker socket mounts

High-risk pattern:

```bash
docker run -v /var/run/docker.sock:/var/run/docker.sock ...
```

A process with broad Docker API access can normally create privileged containers, mount host paths, and take control of the host.

---

## 18. Docker Scout and Vulnerability Scanning

```bash
# Quick image overview
docker scout quickview myapp:1.0

# List known vulnerabilities
docker scout cves myapp:1.0

# Filter by severity
docker scout cves --only-severity critical,high myapp:1.0

# Compare two image versions
docker scout compare \
  --to myapp:1.0 \
  myapp:1.1

# Export SARIF for CI or code-scanning integration
docker scout cves \
  --format sarif \
  --output scout.sarif.json \
  myapp:1.0
```

Scanning does not replace patching, secure configuration, runtime isolation, or application-level testing.

---

## 19. Resource Limits and Restart Policies

### CPU and memory

```bash
docker run \
  --memory 512m \
  --memory-swap 512m \
  --cpus 1.5 \
  myapp:1.0
```

### PID limit

```bash
docker run --pids-limit 200 myapp:1.0
```

### CPU pinning

```bash
docker run --cpuset-cpus 0-3 myapp:1.0
```

### Shared memory

```bash
docker run --shm-size 1g myapp:1.0
```

### Ulimits

```bash
docker run \
  --ulimit nofile=65535:65535 \
  myapp:1.0
```

### Restart policies

| Policy | Behavior |
|---|---|
| `no` | Never restart automatically. |
| `on-failure[:N]` | Restart after non-zero exit, optionally up to `N` retries. |
| `always` | Restart regardless of exit status. |
| `unless-stopped` | Restart unless explicitly stopped. |

```bash
docker run -d \
  --restart unless-stopped \
  --name api \
  myapi:1.0
```

Update an existing container:

```bash
docker update \
  --restart unless-stopped \
  --memory 768m \
  --cpus 2 \
  api
```

### Out-of-memory investigation

```bash
docker inspect --format '{{.State.OOMKilled}}' api
docker inspect --format '{{.HostConfig.Memory}}' api
docker stats --no-stream api
```

---

## 20. Cleanup and Disk Usage

### Inspect usage first

```bash
docker system df
docker system df -v
docker buildx du
```

### Targeted cleanup

```bash
docker container prune
docker image prune
docker image prune -a
docker network prune
docker volume prune
docker builder prune
docker buildx prune
```

### System cleanup

```bash
docker system prune
docker system prune -a
docker system prune -a --volumes
```

> `docker system prune -a --volumes` is destructive. Review active containers, image requirements, and persistent data before running it.

### Remove resources older than a threshold

```bash
docker container prune --filter until=24h
docker image prune -a --filter until=168h
docker builder prune --filter until=168h
```

### Remove dangling images

```bash
docker image ls --filter dangling=true
docker image prune
```

### Remove exited containers

```bash
docker ps -a --filter status=exited
docker container prune
```

---

## 21. Contexts and Remote Docker Hosts

### List contexts

```bash
docker context ls
docker context show
docker context inspect default
```

### Create an SSH context

```bash
docker context create prod \
  --docker "host=ssh://user@server"
```

### Use a context

```bash
docker context use prod
docker ps

# One command only
docker --context prod ps
```

### Return to local Engine

```bash
docker context use default
```

### Remove

```bash
docker context rm prod
```

### Direct SSH host

```bash
docker --host ssh://user@server ps
```

> Prefer SSH or mutual TLS for remote access. Do not expose an unprotected Docker API endpoint over TCP.

---

## 22. Docker Swarm Essentials

### Initialize

```bash
docker swarm init --advertise-addr MANAGER_IP
```

### Join tokens

```bash
docker swarm join-token worker
docker swarm join-token manager
```

### Nodes

```bash
docker node ls
docker node inspect self
docker node update --availability drain NODE
docker node update --availability active NODE
```

### Services

```bash
docker service create \
  --name web \
  --publish 8080:80 \
  --replicas 3 \
  nginx:alpine

docker service ls
docker service ps web
docker service logs -f web
docker service inspect web
docker service scale web=5
docker service update --image nginx:stable-alpine web
docker service rollback web
docker service rm web
```

### Stack deployment

```bash
docker stack deploy -c compose.yaml myapp
docker stack ls
docker stack services myapp
docker stack ps myapp
docker stack rm myapp
```

### Leave swarm

```bash
docker swarm leave
docker swarm leave --force
```

---

## 23. Troubleshooting

### Container exits immediately

```bash
docker ps -a
docker logs CONTAINER
docker inspect --format '{{.State.ExitCode}}' CONTAINER
docker inspect --format '{{.State.Error}}' CONTAINER
docker inspect --format '{{.State.OOMKilled}}' CONTAINER
```

Common causes:

- Main process exits normally.
- Invalid command or entrypoint.
- Missing file or dependency.
- Permission error.
- Required environment variable missing.
- Port already in use.
- Health check failure.
- Out-of-memory termination.
- Image architecture does not match the host.

### Debug an image with an alternate entrypoint

```bash
docker run --rm -it --entrypoint sh myapp:1.0
```

For images that contain Bash:

```bash
docker run --rm -it --entrypoint bash myapp:1.0
```

### Debug a running container

```bash
docker exec -it api sh
docker top api
docker stats --no-stream api
docker inspect api
docker logs -f --tail 100 api
```

### Port already in use

Linux:

```bash
sudo ss -ltnp | grep ':8080'
sudo lsof -iTCP:8080 -sTCP:LISTEN
```

Use another host port:

```bash
docker run -p 8081:80 nginx:alpine
```

### Application is unreachable

Check:

```bash
docker ps
docker port web
docker logs web
docker inspect --format '{{json .NetworkSettings.Ports}}' web
```

Inside the container:

```bash
docker exec web sh -c \
  'wget -qO- http://127.0.0.1:80 || true'
```

Verify that the application listens on `0.0.0.0` inside the container.

### Permission denied on Docker socket

```bash
ls -l /var/run/docker.sock
id
getent group docker
```

Then either use `sudo`, configure rootless Docker, or deliberately grant trusted users membership in the Docker group.

### Permission problems on mounted files

```bash
docker exec api id
ls -ln ./data
docker inspect --format '{{json .Mounts}}' api
```

Match host and container UID/GID where appropriate:

```bash
docker run --user "$(id -u):$(id -g)" ...
```

### DNS or service discovery problem

```bash
docker network ls
docker network inspect app-net
docker exec api getent hosts db
docker exec api cat /etc/resolv.conf
```

### Image architecture mismatch

```bash
docker image inspect \
  --format '{{.Os}}/{{.Architecture}}' myapp:1.0

docker version --format '{{.Server.Os}}/{{.Server.Arch}}'
```

Run a selected platform when emulation is available:

```bash
docker run --platform linux/amd64 myapp:1.0
```

### Build cache behaves unexpectedly

```bash
docker build --progress=plain .
docker build --no-cache .
docker builder prune
```

Inspect build context size and `.dockerignore`.

### Disk full

```bash
docker system df -v
docker buildx du
df -h
df -i
```

Clean selectively rather than deleting all resources blindly.

### Compose configuration problem

```bash
docker compose config
docker compose config --services
docker compose ps -a
docker compose logs --tail 200
```

### Engine daemon problem

```bash
sudo systemctl status docker
sudo journalctl -u docker --since "1 hour ago"
docker info
```

---

## 24. Practical Recipes

### Run a temporary Python environment

```bash
docker run --rm -it \
  -v "$PWD:/work" \
  -w /work \
  python:3.13-slim \
  python script.py
```

### Run a temporary Node.js environment

```bash
docker run --rm -it \
  -v "$PWD:/work" \
  -w /work \
  node:22-alpine \
  npm test
```

### Start PostgreSQL

```bash
docker run -d \
  --name postgres \
  --restart unless-stopped \
  -e POSTGRES_DB=app \
  -e POSTGRES_USER=app \
  -e POSTGRES_PASSWORD='change-me' \
  -p 127.0.0.1:5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:17
```

For non-demo environments, use a proper secret-management mechanism instead of placing the password in shell history.

### Connect to PostgreSQL

```bash
docker exec -it postgres \
  psql -U app -d app
```

### Dump PostgreSQL

```bash
docker exec postgres \
  pg_dump -U app -d app \
  > app.sql
```

### Restore PostgreSQL

```bash
cat app.sql |
  docker exec -i postgres \
    psql -U app -d app
```

### Start Redis

```bash
docker run -d \
  --name redis \
  --restart unless-stopped \
  -p 127.0.0.1:6379:6379 \
  -v redis-data:/data \
  redis:7-alpine \
  redis-server --appendonly yes
```

### Start Nginx with a local configuration

```bash
docker run -d \
  --name nginx \
  -p 127.0.0.1:8080:80 \
  -v "$PWD/nginx.conf:/etc/nginx/nginx.conf:ro" \
  nginx:alpine
```

### Check image contents without starting the normal application

```bash
docker run --rm \
  --entrypoint sh \
  myapp:1.0 \
  -c 'find /app -maxdepth 2 -type f | sort'
```

### Print the image default command

```bash
docker image inspect \
  --format 'Entrypoint={{json .Config.Entrypoint}} Cmd={{json .Config.Cmd}}' \
  myapp:1.0
```

### Follow logs for every Compose service

```bash
docker compose logs -f --tail 100
```

### Rebuild one Compose service

```bash
docker compose build --no-cache api
docker compose up -d --no-deps api
```

### Copy a file from a Compose service

```bash
CONTAINER_ID="$(docker compose ps -q api)"
docker cp "${CONTAINER_ID}:/app/report.json" ./report.json
```

### Wait for a container to become healthy

```bash
until [ \
  "$(docker inspect --format='{{.State.Health.Status}}' api 2>/dev/null)" \
  = "healthy" \
]; do
  sleep 2
done
```

### Show container IP addresses

```bash
docker ps -q |
  xargs -r docker inspect \
    --format '{{.Name}} {{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}'
```

### Remove containers created from a specific image

```bash
docker ps -aq --filter ancestor=myapp:old |
  xargs -r docker rm -f
```

---

## 25. Dangerous Commands

Review carefully before running:

```bash
# Stops every running container
docker stop $(docker ps -q)

# Deletes every container
docker rm -f $(docker ps -aq)

# Deletes all images
docker image rm -f $(docker image ls -aq)

# Deletes unused containers, networks, images, build cache, and volumes
docker system prune -a --volumes

# Deletes unused volumes
docker volume prune

# Force-leaves a Swarm manager
docker swarm leave --force
```

Also high risk:

```bash
docker run --privileged ...
docker run -v /:/host ...
docker run -v /var/run/docker.sock:/var/run/docker.sock ...
```

These can expose or control the host.

---

## 26. Legacy Syntax to Avoid

| Legacy or discouraged form | Current recommendation |
|---|---|
| `docker-compose up` | `docker compose up` |
| Compose top-level `version: "2"` or `"3"` | Use the current Compose Specification without a `version` field. |
| Dockerfile `MAINTAINER` | Use OCI-compatible `LABEL` metadata. |
| Compose `links` for normal discovery | Use service names on user-defined or Compose networks. |
| Passing secrets through `ARG` or `ENV` | Use BuildKit secret mounts or an external secret manager. |
| Building by manually changing a running container | Use a reproducible Dockerfile. |
| Broad use of `ADD` | Prefer `COPY`; use `ADD` only for its intentional extra behavior. |
| Exposing the daemon on unauthenticated TCP | Use SSH or mutual TLS. |
| Routine use of `--privileged` | Grant the minimum capabilities and devices required. |
| `latest` as the only production version | Use controlled semantic tags and/or immutable digests. |

---

## Quick Reference

```bash
# Build
docker build -t myapp:1.0 .

# Run
docker run -d --name myapp -p 127.0.0.1:8080:8000 myapp:1.0

# List
docker ps
docker ps -a
docker image ls

# Logs and shell
docker logs -f --tail 100 myapp
docker exec -it myapp sh

# Inspect and resources
docker inspect myapp
docker stats --no-stream myapp

# Stop and remove
docker stop myapp
docker rm myapp
docker image rm myapp:1.0

# Compose
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose exec api sh
docker compose down

# Cleanup
docker system df
docker container prune
docker image prune
docker builder prune
```

---

## References

This cheat sheet modernizes and expands the command categories in the supplied Docker Labs cheat sheet, especially image management, container inspection, image transfer, Compose, cleanup, and Docker Scout.

Official Docker documentation:

- [Docker CLI reference](https://docs.docker.com/reference/cli/docker/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Docker build best practices](https://docs.docker.com/build/building/best-practices/)
- [Multi-stage builds](https://docs.docker.com/build/building/multi-stage/)
- [Build secrets](https://docs.docker.com/build/building/secrets/)
- [Build cache optimization](https://docs.docker.com/build/cache/optimize/)
- [Buildx reference](https://docs.docker.com/reference/cli/docker/buildx/)
- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker Engine security](https://docs.docker.com/engine/security/)
- [Rootless mode](https://docs.docker.com/engine/security/rootless/)
- [Seccomp profiles](https://docs.docker.com/engine/security/seccomp/)
- [Protect Docker daemon access](https://docs.docker.com/engine/security/protect-access/)
- [Docker Scout CLI](https://docs.docker.com/reference/cli/docker/scout/)
