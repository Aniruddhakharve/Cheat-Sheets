# Docker Cheat Sheet

> Comprehensive Docker reference built from hands-on practice during Days 29–37 of the 90 Days of DevOps challenge.

---

# Table of Contents

1. [Docker Fundamentals](#1-docker-fundamentals)
2. [Docker Architecture](#2-docker-architecture)
3. [Installation & Verification](#3-installation--verification)
4. [Container Commands](#4-container-commands)
5. [Running Containers](#5-running-containers)
6. [Container Lifecycle](#6-container-lifecycle)
7. [Logs & Exec](#7-logs--exec)
8. [Port Mapping](#8-port-mapping)
9. [Docker Images](#9-docker-images)
10. [Image History & Layers](#10-image-history--layers)
11. [Docker Build Cache](#11-docker-build-cache)
12. [Dockerfile Instructions](#12-dockerfile-instructions)
13. [COPY vs ADD](#13-copy-vs-add)
14. [CMD vs ENTRYPOINT](#14-cmd-vs-entrypoint)
15. [.dockerignore](#15-dockerignore)
16. [Docker Volumes](#16-docker-volumes)
17. [Bind Mounts](#17-bind-mounts)
18. [Docker Networking](#18-docker-networking)
19. [Docker Compose](#19-docker-compose)
20. [Compose Environment Variables](#20-compose-environment-variables)
21. [Compose Healthchecks](#21-compose-healthchecks)
22. [depends_on](#22-depends_on)
23. [Docker Hub](#23-docker-hub)
24. [Cleanup Commands](#24-cleanup-commands)
25. [Multi-Stage Builds](#25-multi-stage-builds)
26. [Java + Maven + WAR + Tomcat](#26-java--maven--war--tomcat)
27. [jlink Java Runtime Optimization](#27-jlink-java-runtime-optimization)
28. [Non-Root Containers](#28-non-root-containers)
29. [Day-36 Project Architecture](#29-day-36-project-architecture)
30. [Day-36 Troubleshooting](#30-day-36-troubleshooting)
31. [Common Docker Errors](#31-common-docker-errors)
32. [Interview Questions](#32-interview-questions)
33. [Important Docker Interview Answers](#33-important-docker-interview-answers)
34. [Quick Daily Reference](#34-quick-daily-reference)

---

# 1. Docker Fundamentals

## What is Docker?

Docker is a platform for building, packaging, distributing, and running applications in containers.

The basic idea is:

```text
Application
    +
Dependencies
    +
Runtime
    +
Configuration
        ↓
    Docker Image
        ↓
   Docker Container
```

Docker helps reduce the classic:

```text
"It works on my machine."
```

problem by making the application runtime environment reproducible.

Docker does not magically make software work on every machine. The target machine still needs a compatible Docker runtime, architecture, resources, networking, etc.

---

# 2. Docker Architecture

Basic Docker architecture:

```text
Developer
    │
    │ docker build/run/pull
    ▼
Docker CLI
    │
    ▼
Docker Engine / Daemon
    │
    ├── Images
    ├── Containers
    ├── Networks
    ├── Volumes
    └── Build Cache
          │
          ▼
       Containers
```

Docker also uses registries such as Docker Hub:

```text
Docker Hub
    │
    │ docker pull
    ▼
Local Docker Engine
    │
    ▼
Docker Image
    │
    ▼
Container
```

---

# 3. Installation & Verification

Check Docker version:

```bash
docker --version
```

Example:

```text
Docker version 29.6.2
```

Check Compose:

```bash
docker compose version
```

Example:

```text
Docker Compose version v5.3.1
```

Detailed Docker information:

```bash
docker info
```

Check Docker installation with:

```bash
docker run hello-world
```

---

# 4. Container Commands

## List running containers

```bash
docker ps
```

## List all containers

```bash
docker ps -a
```

## Start a container

```bash
docker start CONTAINER
```

## Stop a container

```bash
docker stop CONTAINER
```

## Restart a container

```bash
docker restart CONTAINER
```

## Force remove a container

```bash
docker rm -f CONTAINER
```

## Remove a stopped container

```bash
docker rm CONTAINER
```

## Run a container

```bash
docker run IMAGE
```

## Give container a name

```bash
docker run --name my-nginx nginx
```

## Run detached

```bash
docker run -d nginx
```

## Run interactive

```bash
docker run -it ubuntu bash
```

`-it` combines:

```text
-i = interactive
-t = pseudo-terminal
```

---

# 5. Running Containers

## Nginx

```bash
docker run -d --name nginx-web -p 8080:80 nginx
```

Access:

```text
http://localhost:8080
```

## Ubuntu interactive

```bash
docker run -it ubuntu bash
```

Inside the container:

```bash
ls
pwd
cat /etc/os-release
```

Exit:

```bash
exit
```

## Custom command

```bash
docker run --rm alpine echo "Hello Docker"
```

---

# 6. Container Lifecycle

Typical lifecycle:

```text
docker create
     ↓
Created
     ↓
docker start
     ↓
Running
     ↓
docker pause
     ↓
Paused
     ↓
docker unpause
     ↓
Running
     ↓
docker stop
     ↓
Stopped
     ↓
docker restart
     ↓
Running
     ↓
docker kill
     ↓
Stopped
     ↓
docker rm
     ↓
Removed
```

## Create without starting

```bash
docker create --name nginx-test nginx
```

## Start

```bash
docker start nginx-test
```

## Pause

```bash
docker pause nginx-test
```

## Unpause

```bash
docker unpause nginx-test
```

## Stop gracefully

```bash
docker stop nginx-test
```

## Restart

```bash
docker restart nginx-test
```

## Kill immediately

```bash
docker kill nginx-test
```

## Remove

```bash
docker rm nginx-test
```

---

# 7. Logs & Exec

## View logs

```bash
docker logs CONTAINER
```

## Follow logs

```bash
docker logs -f CONTAINER
```

## Limit output

```bash
docker logs --tail 100 CONTAINER
```

## Execute command inside running container

```bash
docker exec CONTAINER command
```

Example:

```bash
docker exec nginx-web ls /usr/share/nginx/html
```

## Open shell

```bash
docker exec -it nginx-web sh
```

or:

```bash
docker exec -it nginx-web bash
```

depending on the image.

---

# 8. Port Mapping

Syntax:

```bash
docker run -p HOST_PORT:CONTAINER_PORT IMAGE
```

Example:

```bash
docker run -p 8080:80 nginx
```

Meaning:

```text
Host                         Container
8080  ────────────────────>  80
```

Then:

```text
http://localhost:8080
```

reaches port `80` inside the container.

## Important

```dockerfile
EXPOSE 80
```

does not publish the port to the host.

It documents the intended container port.

Actual publishing requires:

```bash
-p 8080:80
```

---

# 9. Docker Images

## List images

```bash
docker images
```

or:

```bash
docker image ls
```

## Pull image

```bash
docker pull nginx
```

Specific tag:

```bash
docker pull nginx:1.27
```

## Build image

```bash
docker build -t myapp:v1 .
```

The final `.` means:

```text
Current directory = build context
```

## Tag image

```bash
docker tag myapp:v1 username/myapp:v1
```

## Remove image

```bash
docker rmi IMAGE
```

Force:

```bash
docker rmi -f IMAGE
```

## Inspect image

```bash
docker image inspect IMAGE
```

## Image size

```bash
docker image inspect IMAGE --format '{{.Size}}'
```

## Inspect image user

```bash
docker image inspect IMAGE --format '{{.Config.User}}'
```

---

# 10. Image History & Layers

Inspect image layers:

```bash
docker history IMAGE
```

Each Dockerfile build instruction can contribute to an image layer.

Example:

```dockerfile
FROM alpine

RUN echo "step 1"

RUN echo "step 2"

COPY app.txt /app/
```

Conceptually:

```text
Layer 1 → FROM
Layer 2 → RUN step 1
Layer 3 → RUN step 2
Layer 4 → COPY
```

Some Dockerfile instructions may show:

```text
0B
```

because they modify image metadata/configuration rather than adding filesystem content.

---

# 11. Docker Build Cache

Docker caches build steps when possible.

Example:

```dockerfile
FROM alpine:3.22

RUN echo "Installing application dependencies..."

RUN echo "Setting up application..."

COPY app.txt /app/app.txt

CMD ["cat", "/app/app.txt"]
```

First build:

```bash
docker build -t cache-demo:v1 .
```

Second build without changes:

```bash
docker build -t cache-demo:v1 .
```

You should see:

```text
CACHED
```

for reusable steps.

If:

```bash
echo "Version 2" > app.txt
```

then:

```bash
docker build -t cache-demo:v2 .
```

may show:

```text
CACHED [2/4]
CACHED [3/4]
[4/4] COPY app.txt
```

because the `COPY` instruction changed.

## Important caching rule

When a layer changes, subsequent instructions may also need to be rebuilt.

Example:

```dockerfile
FROM ubuntu:24.04

COPY . /app

RUN apt-get update && apt-get install -y python3
```

If only `app.py` changes:

```text
FROM
 ↓
CACHED

COPY
 ↓
CHANGED

RUN
 ↓
REBUILT
```

Better:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y python3

COPY . /app
```

Now changes to application source do not invalidate the package installation layer.

## General rule

Put:

```text
Stable instructions
        ↓
Frequently changing instructions
```

Example:

```text
FROM
OS/system dependencies
Application dependency files
Source code
```

---

# 12. Dockerfile Instructions

## FROM

Select the base image.

```dockerfile
FROM ubuntu:24.04
```

Multi-stage:

```dockerfile
FROM maven:3.9-eclipse-temurin-11 AS builder
```

---

## RUN

Execute a command during image build.

```dockerfile
RUN apt-get update
```

Example:

```dockerfile
RUN apt-get update && \
    apt-get install -y curl
```

---

## COPY

Copy files into the image.

```dockerfile
COPY app.py /app/
```

Copy from another build stage:

```dockerfile
COPY --from=builder /build/target/app.war /opt/tomcat/webapps/ROOT.war
```

---

## WORKDIR

Set working directory.

```dockerfile
WORKDIR /app
```

Equivalent conceptually to:

```bash
cd /app
```

for subsequent Dockerfile instructions.

---

## ENV

Set environment variable.

```dockerfile
ENV APP_ENV=production
```

---

## EXPOSE

Document the intended container port.

```dockerfile
EXPOSE 8080
```

It does not publish the port to the host.

---

## CMD

Define the default command.

```dockerfile
CMD ["python3", "app.py"]
```

or:

```dockerfile
CMD ["catalina.sh", "run"]
```

---

## ENTRYPOINT

Define the main executable.

```dockerfile
ENTRYPOINT ["python3"]
```

Running:

```bash
docker run image app.py
```

results in the equivalent of:

```text
python3 app.py
```

---

## USER

Specify the runtime user.

```dockerfile
USER appuser
```

Useful for non-root container execution.

---

## ARG

Build-time variable.

```dockerfile
ARG VERSION=1.0
```

Use:

```bash
docker build --build-arg VERSION=2.0 -t myapp .
```

---

# 13. COPY vs ADD

## COPY

Straightforward copying:

```dockerfile
COPY source.txt /app/
```

## ADD

Provides COPY functionality plus additional behavior such as local tar extraction and URL sources.

Example:

```dockerfile
ADD app.tar.gz /app/
```

Hands-on result:

```text
COPY
→ copies file

ADD
→ copies + automatically extracts local tar archive
```

## Rule

Prefer:

```dockerfile
COPY
```

for normal file copying because it is explicit and predictable.

---

# 14. CMD vs ENTRYPOINT

## CMD

Default command:

```dockerfile
CMD ["echo", "hello"]
```

Runtime command can replace CMD:

```bash
docker run image echo hi
```

## ENTRYPOINT

Main executable:

```dockerfile
ENTRYPOINT ["echo"]
```

Arguments can be appended:

```bash
docker run image hello
```

Conceptually:

```text
CMD
→ default command

ENTRYPOINT
→ main executable
```

---

# 15. .dockerignore

`.dockerignore` prevents unnecessary files from entering the Docker build context.

Example:

```text
.git
.gitignore
README.md
*.md
.env
node_modules/
target/
```

Be careful not to accidentally exclude required application source.

We encountered this during Day 36 when `src/` was excluded and Docker failed at:

```dockerfile
COPY src ./src
```

Error:

```text
CopyIgnoredFile: Attempting to Copy file "src" that is excluded by .dockerignore
```

Fix:

Remove `src/` from `.dockerignore`.

---

# 16. Docker Volumes

## Create volume

```bash
docker volume create my-volume
```

## List

```bash
docker volume ls
```

## Inspect

```bash
docker volume inspect my-volume
```

## Remove

```bash
docker volume rm my-volume
```

## Mount

```bash
docker run \
  -v my-volume:/data \
  alpine
```

Use volumes for persistent data such as PostgreSQL.

---

# 17. Bind Mounts

Syntax:

```bash
-v /host/path:/container/path
```

Example:

```bash
docker run -d \
  --name nginx-bind \
  -p 8080:80 \
  -v /home/user/website:/usr/share/nginx/html \
  nginx
```

The host files appear directly inside the container.

Difference:

```text
Named Volume
→ Docker manages storage location

Bind Mount
→ You specify exact host directory
```

---

# 18. Docker Networking

## List networks

```bash
docker network ls
```

## Create network

```bash
docker network create my-app-net
```

## Inspect

```bash
docker network inspect my-app-net
```

## Connect container

```bash
docker network connect my-app-net container
```

## Disconnect

```bash
docker network disconnect my-app-net container
```

## Remove

```bash
docker network rm my-app-net
```

---

## Container-to-container communication

User-defined network:

```text
my-app-net
    │
    ├── app
    │
    └── db
```

The application can use:

```text
db
```

as the hostname.

Docker's embedded DNS resolves the service/container name.

Avoid hard-coding container IPs because container IPs can change.

---

# 19. Docker Compose

## Start

```bash
docker compose up
```

## Detached

```bash
docker compose up -d
```

## Build and start

```bash
docker compose up --build
```

## Stop/remove

```bash
docker compose down
```

## Stop/remove including volumes

```bash
docker compose down -v
```

## List services

```bash
docker compose ps
```

## Logs

```bash
docker compose logs
```

## Follow logs

```bash
docker compose logs -f
```

## Service-specific logs

```bash
docker compose logs app
```

## Build

```bash
docker compose build
```

## Recreate services

```bash
docker compose up -d --force-recreate
```

## Execute command inside service

```bash
docker compose exec db psql -U appuser -d appdb
```

---

# 20. Compose Environment Variables

`.env` example:

```env
POSTGRES_DB=appdb
POSTGRES_USER=appuser
POSTGRES_PASSWORD=appsecret

DB_HOST=db
DB_NAME=appdb
DB_USER=appuser
DB_PASSWORD=appsecret
```

Compose:

```yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}

  app:
    environment:
      DB_HOST: ${DB_HOST}
      DB_NAME: ${DB_NAME}
      DB_USER: ${DB_USER}
      DB_PASSWORD: ${DB_PASSWORD}
```

Validate:

```bash
docker compose config
```

Do not commit real secrets into Git.

---

# 21. Compose Healthchecks

Example PostgreSQL healthcheck:

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U appuser -d appdb"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 10s
```

Check:

```bash
docker compose ps
```

Expected:

```text
Up (healthy)
```

Healthchecks verify service readiness rather than merely container existence.

---

# 22. depends_on

Basic:

```yaml
depends_on:
  - db
```

This gives startup ordering.

Better:

```yaml
depends_on:
  db:
    condition: service_healthy
```

This waits for the database healthcheck.

Use this when the application depends on a database or another service being ready.

---

# 23. Docker Hub

## Login

```bash
docker login
```

## Tag

```bash
docker tag myapp:v1 username/myapp:v1
```

## Push

```bash
docker push username/myapp:v1
```

## Pull

```bash
docker pull username/myapp:v1
```

## Multiple tags

Example:

```bash
docker tag three-tier-java-app:v3-jlink \
  aniruddhakharve/three-tier-java-app:v3-jlink
```

```bash
docker tag three-tier-java-app:v3-jlink \
  aniruddhakharve/three-tier-java-app:latest
```

Push:

```bash
docker push aniruddhakharve/three-tier-java-app:v3-jlink
docker push aniruddhakharve/three-tier-java-app:latest
```

---

# 24. Cleanup Commands

## Remove stopped containers

```bash
docker container prune
```

## Remove unused images

```bash
docker image prune
```

## Remove unused volumes

```bash
docker volume prune
```

## Remove unused networks

```bash
docker network prune
```

## General cleanup

```bash
docker system prune
```

## Aggressive image cleanup

```bash
docker system prune -a
```

Be careful.

## Docker disk usage

```bash
docker system df
```

Detailed:

```bash
docker system df -v
```

---

# 25. Multi-Stage Builds

Basic structure:

```dockerfile
FROM build-image AS builder

# Build application

FROM runtime-image

COPY --from=builder /path/to/artifact /app/
```

Benefits:

- Smaller final images
- Separates build and runtime
- Excludes compilers and build tools
- Reduces unnecessary components
- Can reduce attack surface

Important:

Multi-stage builds do not automatically guarantee a smaller image.

If the runtime base image is large, the final image can still be large.

---

# 26. Java + Maven + WAR + Tomcat

The Day-36 Java application flow:

```text
Java Source Code
        │
        ▼
      Maven
        │
        ├── Download dependencies
        ├── Compile Java code
        └── Package application
                │
                ▼
              app.war
                │
                ▼
             Tomcat
                │
                ▼
        Running Web Application
                │
                │ JDBC
                ▼
           PostgreSQL
```

## Maven

Maven is the Java build/dependency management tool.

`pom.xml` defines:

- Dependencies
- Build plugins
- Java version
- Packaging
- Project metadata

Build:

```bash
mvn clean package
```

Output:

```text
target/app.war
```

---

## WAR

WAR means:

```text
Web Application Archive
```

Contains:

- Compiled Java classes
- Java dependencies
- HTML/static files
- `WEB-INF`
- `web.xml`

Inspect:

```bash
jar tf target/app.war
```

---

## Tomcat

Tomcat is the servlet container that runs the WAR.

Typical flow:

```text
Tomcat
   ↓
ROOT.war
   ↓
Servlets
   ↓
Application
```

Start Tomcat:

```bash
catalina.sh run
```

In our Dockerfile:

```dockerfile
CMD ["catalina.sh", "run"]
```

---

# 27. jlink Java Runtime Optimization

A full JDK contains much more than a running application needs.

`jlink` creates a custom Java runtime containing only selected modules.

Example:

```bash
jlink \
  --module-path "$JAVA_HOME/jmods" \
  --add-modules java.base,java.sql,java.xml,java.logging \
  --strip-debug \
  --no-man-pages \
  --no-header-files \
  --compress=2 \
  --output /opt/java-minimal
```

Concept:

```text
Full JDK
   │
   │ jlink
   ▼
Selected Java modules
   │
   ▼
Custom Runtime
```

Day-36 result:

```text
~209 MB
   ↓
~81 MB
```

Approximate reduction:

```text
61%
```

---

## Important jlink lesson

A runtime that is too minimal may fail.

We encountered missing classes/modules such as:

```text
org/ietf/jgss/GSSException
```

and:

```text
java.lang.instrument.IllegalClassFormatException
```

The solution was to add the required modules to the runtime.

This demonstrates that `jlink` requires understanding the runtime dependencies of the application server/application.

---

# 28. Non-Root Containers

Create user:

```dockerfile
RUN groupadd --system appgroup \
    && useradd --system --gid appgroup appuser
```

Change ownership:

```dockerfile
RUN chown -R appuser:appgroup /app
```

Run as user:

```dockerfile
USER appuser
```

Verify:

```bash
docker inspect IMAGE --format '{{.Config.User}}'
```

Example:

```text
User: tomcat
```

---

# 29. Day-36 Project Architecture

The actual Day-36 project used:

```text
Browser
   │
   │ :8081
   ▼
Java/Tomcat Container
   │
   │ JDBC
   ▼
PostgreSQL Container
   │
   ▼
Named Volume
```

Network:

```text
three-tier-network
```

Application image:

```text
three-tier-java-app:v3-jlink
```

Docker Hub:

```text
aniruddhakharve/three-tier-java-app
```

Tags:

```text
latest
v3-jlink
```

---

# 30. Day-36 Troubleshooting

## Error 1 — `.dockerignore` excluded source

Error:

```text
CopyIgnoredFile: Attempting to Copy file "src" that is excluded by .dockerignore
```

Cause:

```text
src/
```

was ignored.

Fix:

Remove `src/` from `.dockerignore`.

---

## Error 2 — PostgreSQL password mismatch

We observed:

```text
FATAL: password authentication failed for user "appuser"
```

Cause:

The PostgreSQL password and application password did not match.

Example:

```text
PostgreSQL:
POSTGRES_PASSWORD=appsecret
```

while application default configuration used:

```text
apppassword
```

Fix:

Align the application environment variables with the PostgreSQL credentials.

Better practice:

Use environment variables:

```text
DB_PASSWORD
```

instead of hard-coding passwords.

---

## Error 3 — Java container stopped

Command:

```bash
docker exec day36-java-app getent hosts db-primary-service
```

failed because:

```text
container is not running
```

Fix:

Check:

```bash
docker ps -a
docker logs day36-java-app
```

Never blindly restart a failing container without checking logs.

---

## Error 4 — Missing jlink module

Error:

```text
NoClassDefFoundError: org/ietf/jgss/GSSException
```

Fix:

Add the required Java runtime module:

```text
java.security.jgss
```

---

## Error 5 — Another missing jlink module

Error:

```text
NoClassDefFoundError:
java/lang/instrument/IllegalClassFormatException
```

Fix:

Add:

```text
java.instrument
```

The lesson:

A minimal Java runtime must still contain every module required by Tomcat and the application.

---

## Error 6 — Docker Compose network label conflict

Error:

```text
network three-tier-network was found but has incorrect label
com.docker.compose.network
```

Cause:

A manually-created network had the same name as the Compose-managed network but was not owned by that Compose project.

Fix:

Remove/recreate the network under Compose management.

Example:

```bash
docker compose down
docker network rm three-tier-network
docker compose up -d
```

Only remove the network after checking that no required containers still use it.

---

## Error 7 — Optimized image was accidentally deleted

The local image:

```text
three-tier-java-app:v3-jlink
```

was accidentally removed before it had been pushed to Docker Hub.

Recovery:

Rebuild from:

```text
Dockerfile.jlink
```

Then verify:

```bash
docker image inspect three-tier-java-app:v3-jlink
```

and republish:

```bash
docker push aniruddhakharve/three-tier-java-app:v3-jlink
```

Lesson:

Reproducible Dockerfiles and registry backups are important.

---

# 31. Common Docker Errors

## Port already allocated

Typical error:

```text
Bind for 0.0.0.0:80 failed: port is already allocated
```

Find process:

Linux:

```bash
ss -lntp
```

or:

```bash
lsof -i :80
```

Windows:

```bash
netstat -ano | findstr :80
```

Then either stop the process or use another host port:

```bash
docker run -p 8080:80 nginx
```

---

## Container exits immediately

Check:

```bash
docker ps -a
docker logs CONTAINER
```

A container stops when its main process exits.

Remember:

```text
Container lifetime
=
main process lifetime
```

---

## Image not found

Check:

```bash
docker images
```

Pull:

```bash
docker pull IMAGE:TAG
```

Or check the tag carefully.

---

## Cannot connect to database

Check:

```bash
docker ps
docker logs db
docker logs app
docker network inspect NETWORK
```

Verify:

```text
Hostname
Port
Database
Username
Password
Network
```

---

## DNS resolution failure between containers

Example:

```text
UnknownHostException
```

Check:

```bash
docker network inspect NETWORK
```

Ensure both containers are attached to the same user-defined network.

---

# 32. Interview Questions

## Fundamentals

### 1. What is Docker?

Docker is a platform for building, packaging, distributing, and running applications as containers.

### 2. What is a Docker image?

An immutable template used to create containers.

### 3. What is a container?

A running or stopped instance of a Docker image with an isolated execution environment.

### 4. Image vs container?

```text
Image = template
Container = instance
```

---

## Containers

### 5. Why does a container stop?

Usually because its main process exits.

### 6. How do you troubleshoot an exited container?

```bash
docker ps -a
docker logs CONTAINER
docker inspect CONTAINER
```

### 7. Difference between `docker stop` and `docker kill`?

`docker stop` attempts graceful shutdown.

`docker kill` sends an immediate kill signal.

---

## Images

### 8. What are Docker image layers?

Layers are filesystem changes generated by Dockerfile instructions and reused through caching.

### 9. Why is Docker build cache useful?

It avoids rebuilding unchanged layers.

### 10. How do you inspect image layers?

```bash
docker history IMAGE
```

---

## Dockerfile

### 11. Difference between RUN and CMD?

```text
RUN → build time
CMD → container runtime
```

### 12. Difference between CMD and ENTRYPOINT?

```text
CMD → default command
ENTRYPOINT → main executable
```

### 13. Difference between COPY and ADD?

`COPY` is straightforward file copying.

`ADD` provides additional functionality such as local tar extraction and URL sources.

### 14. What does WORKDIR do?

Sets the working directory for subsequent Dockerfile instructions and runtime commands.

### 15. What does EXPOSE do?

Documents the port the application is expected to listen on.

---

## Networking

### 16. How do containers communicate?

Through Docker networks.

### 17. Why use custom networks?

They provide isolated communication and Docker DNS/service discovery.

### 18. Why avoid hard-coding container IPs?

Container IPs can change.

---

## Volumes

### 19. Why do databases need volumes?

Container writable storage is ephemeral. Volumes provide persistent storage.

### 20. Difference between volume and bind mount?

```text
Volume → Docker-managed
Bind mount → explicit host directory
```

---

## Compose

### 21. What is Docker Compose?

A tool for defining and managing multi-container applications using YAML.

### 22. Difference between `docker compose down` and `down -v`?

`down` removes containers/network but preserves named volumes.

`down -v` also removes Compose-managed volumes.

### 23. What is `depends_on`?

Defines service startup dependency/order.

### 24. Why use healthchecks with depends_on?

Startup order alone does not guarantee service readiness.

---

## Docker Hub

### 25. Why tag images?

Tags provide identifiable versions:

```text
myapp:v1
myapp:v2
myapp:latest
```

### 26. How do you publish an image?

```bash
docker tag
docker push
```

### 27. Why use version tags instead of only latest?

Version tags allow predictable deployments and rollbacks.

---

## Security

### 28. Why should containers avoid root?

Running as non-root reduces the impact of a container compromise.

### 29. How do you run a container as a specific user?

Dockerfile:

```dockerfile
USER appuser
```

---

# 33. Important Docker Interview Answers

## "Explain your Day-36 Docker project."

> I Dockerized a three-tier Java web application consisting of a Java Servlet application running on Tomcat and a PostgreSQL database. I created a multi-stage Docker build where Maven and JDK 11 were used only during the build stage. I then created a custom Java runtime using `jlink`, reducing the application image from approximately 209 MB to about 81 MB. The final runtime runs as a non-root Tomcat user. I used Docker Compose for orchestration, a custom Docker network for application-to-database communication, a named volume for PostgreSQL persistence, healthchecks, environment variables, and `depends_on`. Finally, I pushed versioned images to Docker Hub and verified deployment from the registry.

---

## "Why didn't your first multi-stage build reduce the image size?"

> The original image was already a runtime image containing Tomcat and the JDK. The multi-stage build removed Maven and other build-time dependencies, but the runtime stage still contained the full JDK. I analyzed `docker history` and then optimized the runtime using `jlink` to create a custom Java runtime containing only the required modules.

---

## "What is jlink?"

> `jlink` is a Java tool used to create a custom runtime image containing only selected Java modules. It allows us to avoid shipping the full JDK when the application only needs a subset of the Java runtime.

---

## "How did you reduce the image size?"

```text
Multi-stage build
       +
Debian Slim
       +
jlink custom Java runtime
       +
Remove unnecessary Tomcat webapps
       +
Remove package/cache files
       +
Non-root runtime
```

Result:

```text
~209 MB → ~81 MB
```

---

## "How does your Java application connect to PostgreSQL?"

```text
Java Application
      │
      │ JDBC
      ▼
PostgreSQL JDBC Driver
      │
      ▼
PostgreSQL
```

Docker provides the network path between the containers.

---

## "Why use service/container names instead of container IPs?"

> Container IP addresses can change. Docker's embedded DNS on user-defined networks lets the application communicate using stable service/container names.

---

## "What happens if you run docker compose down -v?"

> Compose removes the project's containers and networks and also removes its named volumes. If PostgreSQL data is stored in that Compose-managed volume, the database data will be deleted.

---

# 34. Quick Daily Reference

## Container

```bash
docker run -d --name app nginx
docker ps
docker ps -a
docker logs app
docker exec -it app sh
docker stop app
docker rm app
```

## Image

```bash
docker images
docker pull nginx
docker build -t myapp:v1 .
docker tag myapp:v1 user/myapp:v1
docker push user/myapp:v1
docker history myapp:v1
docker image inspect myapp:v1
```

## Volume

```bash
docker volume create data
docker volume ls
docker volume inspect data
```

## Network

```bash
docker network create app-net
docker network ls
docker network inspect app-net
```

## Compose

```bash
docker compose config
docker compose up -d
docker compose ps
docker compose logs -f
docker compose exec db sh
docker compose down
docker compose down -v
```

## Disk usage

```bash
docker system df
docker system df -v
```

## Cleanup

```bash
docker container prune
docker image prune
docker volume prune
docker network prune
docker system prune
```

---

# Final Docker Mental Model

```text
                         Docker
                           │
        ┌──────────────────┼───────────────────┐
        │                  │                   │
        ▼                  ▼                   ▼
      Image            Container             Registry
        │                  │                   │
        │                  ├── Network         │
        │                  ├── Volume          │
        │                  └── Process         │
        │                                      │
        └──────── docker push/pull ────────────┘
```

For a real application:

```text
Source Code
    │
    ▼
Dockerfile
    │
    ▼
Docker Image
    │
    ▼
Container
    │
    ├── Port
    ├── Network
    └── Volume
```

For multi-container applications:

```text
                Docker Compose
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
       App           DB          Cache
        │            │            │
        └────────────┼────────────┘
                     │
                Docker Network
```

For the Day-36 Java application:

```text
Java Source
     │
     ▼
   Maven
     │
     ▼
   app.war
     │
     ▼
   Tomcat
     │
     │ Java Runtime
     ▼
   jlink
     │
     ▼
Minimal Java Runtime
     │
     ▼
Docker Runtime Image
     │
     ▼
Docker Compose
     │
     ├── Java/Tomcat
     │
     └── PostgreSQL
             │
             ▼
        Named Volume
```

---

# Most Important Things to Remember

```text
1. IMAGE ≠ CONTAINER
   Image = immutable template
   Container = instance of image

2. CONTAINER STORAGE IS EPHEMERAL
   Use volumes for persistent data.

3. CUSTOM NETWORKS PROVIDE DNS
   app can communicate with db by name.

4. EXPOSE ≠ PORT PUBLISHING
   EXPOSE documents.
   -p actually publishes.

5. MULTI-STAGE ≠ AUTOMATICALLY SMALL
   Runtime image still matters.

6. jlink ≠ multi-stage
   Multi-stage removes build tools.
   jlink removes unnecessary Java runtime modules.

7. COPY IS PREFERRED FOR NORMAL COPYING
   ADD has additional behavior.

8. CACHE FAVORS GOOD DOCKERFILE ORDER
   Stable layers first.
   Frequently changing files later.

9. docker compose down ≠ down -v
   -v removes Compose-managed volumes.

10. NEVER HARD-CODE CONTAINER IPs
    Use service/container names on custom networks.

11. CHECK LOGS BEFORE RESTARTING
    docker logs CONTAINER

12. RUN PRODUCTION CONTAINERS AS NON-ROOT
    USER appuser

13. VERSION YOUR IMAGES
    app:v1
    app:v2
    app:latest

14. PUBLISH IMPORTANT IMAGES
    Docker Hub / registry

15. BUILD REPRODUCIBLY
    Dockerfile + source + dependencies + configuration
```

---

# End of Docker Cheat Sheet

Built from hands-on Docker practice completed during Days 29–37 of the 90 Days of DevOps challenge.

Primary project used for advanced Docker practice:

https://github.com/Aniruddhakharve/three-tier-java-app-dockerize

Docker Hub:

https://hub.docker.com/r/aniruddhakharve/three-tier-java-app

90 Days of DevOps Day 36:

https://github.com/Aniruddhakharve/90DaysOfDevOps-shubham-londe/tree/master/2026/day-36
