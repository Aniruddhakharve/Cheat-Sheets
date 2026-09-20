# 🚀 90 Days of DevOps — Day 38 to Day 49 Interview Cheat Sheet

> **Purpose:** Quick interview revision — not a replacement for hands-on practice.
>
> This cheat sheet covers the important concepts, commands, examples, errors, fixes, and interview explanations learned from **Day 38 → Day 49**, including the **NodeGoat DevSecOps Capstone Project**.

---

# 📌 Day 38 — YAML Basics

## What is YAML?

YAML = **YAML Ain't Markup Language**

It is a human-readable data serialization format commonly used for:

- GitHub Actions
- Docker Compose
- Kubernetes
- Ansible
- CI/CD configuration

### Basic Example

```yaml
name: My Application

version: 1.0

services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

### Important YAML Rules

```text
Indentation matters
Use spaces, not tabs
Key: value
Lists use -
Nested values use indentation
```

### Example

```yaml
server:
  name: web-server
  port: 8080
  enabled: true

packages:
  - nginx
  - git
  - docker
```

## YAML Data Types

```yaml
name: Aniruddha       # String
port: 8080            # Integer
enabled: true         # Boolean
version: 1.0          # Number
packages:
  - git
  - docker             # List
```

## Interview Answer

> "YAML is a human-readable configuration format. It is indentation-sensitive and is widely used in DevOps tools such as GitHub Actions, Docker Compose, Kubernetes and Ansible."

---

# 📌 Day 39 — CI/CD Concepts

## CI — Continuous Integration

Developers frequently push code and automated systems:

```text
Code Push
   ↓
Build
   ↓
Test
   ↓
Quality Checks
```

### Goal

Catch problems early.

---

## CD — Continuous Delivery

Code is automatically prepared for deployment.

```text
Code
 ↓
Build
 ↓
Test
 ↓
Package
 ↓
Ready for Deployment
```

Deployment may still require approval.

---

## Continuous Deployment

Deployment happens automatically after successful checks.

```text
Code
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Deploy Automatically
```

## CI/CD Example

```text
Developer
    ↓
Git Push
    ↓
GitHub Actions
    ↓
Build
    ↓
Test
    ↓
Security Scan
    ↓
Docker Build
    ↓
Docker Registry
    ↓
EC2
    ↓
Application
```

## CI vs CD

| CI | CD |
|---|---|
| Integrates code | Delivers/deploys code |
| Build | Package |
| Test | Deploy |
| Code quality | Environment deployment |
| Finds bugs early | Makes releases repeatable |

## Interview Answer

> "CI automatically builds and tests code whenever changes are integrated. CD takes the validated application and prepares or deploys it to an environment in a repeatable way."

---

# 📌 Day 40 — First GitHub Actions Workflow

## GitHub Actions Structure

```yaml
name: Example

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run command
        run: echo "Hello"
```

## Important Components

```text
name       → Workflow name
on         → Trigger
jobs       → Jobs to execute
runs-on    → Runner
steps      → Individual tasks
uses       → Existing action
run        → Shell command
```

## `uses` vs `run`

```yaml
uses: actions/checkout@v4
```

Uses a reusable GitHub Action.

```yaml
run: echo "Hello"
```

Runs a shell command.

## Interview Answer

> "A GitHub Actions workflow is defined in YAML. It contains triggers, jobs, runners and steps. `uses` executes an existing action while `run` executes shell commands."

---

# 📌 Day 41 — Triggers & Matrix

## Common Triggers

### Push

```yaml
on:
  push:
```

Runs when code is pushed.

### Pull Request

```yaml
on:
  pull_request:
```

Runs when a PR event occurs.

### Manual

```yaml
on:
  workflow_dispatch:
```

Allows manually starting the workflow.

### Scheduled

```yaml
on:
  schedule:
    - cron: '0 */6 * * *'
```

Runs according to a cron schedule.

---

## Matrix Strategy

Matrix allows the same job to run with different combinations.

```yaml
strategy:
  matrix:
    python-version: ["3.10", "3.11", "3.12"]
```

This creates:

```text
Python 3.10 → Job
Python 3.11 → Job
Python 3.12 → Job
```

### Matrix Exclude

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest]
    python: ["3.11", "3.12"]
    exclude:
      - os: windows-latest
        python: "3.11"
```

## `fail-fast`

```yaml
strategy:
  fail-fast: false
```

Other matrix jobs continue even if one fails.

## Interview Answer

> "A matrix strategy allows one GitHub Actions job to run across multiple versions, operating systems or configurations without duplicating the workflow."

---

# 📌 Day 42 — GitHub Actions Runners

## What is a Runner?

A runner is the machine that executes GitHub Actions jobs.

```text
GitHub Workflow
       ↓
Runner
       ↓
Commands / Tests / Build
```

## GitHub-Hosted Runners

Examples:

```text
ubuntu-latest
windows-latest
macos-latest
```

GitHub manages them.

## Self-Hosted Runner

Your own machine/server executes the workflow.

```text
GitHub
   ↓
Self-hosted Runner
   ↓
Your Server
```

Useful when:

- Special software is required
- Private network access is needed
- Custom hardware/environment is required

## Labels

A self-hosted runner can have labels.

Example:

```yaml
runs-on: [self-hosted, linux]
```

This tells GitHub to select a matching runner.

## Interview Answer

> "A runner is the execution environment for a GitHub Actions job. GitHub-hosted runners are managed by GitHub, while self-hosted runners are managed by the organization."

---

# 📌 Day 43 — Jobs, Steps, Variables & Conditionals

## Job

A job is a collection of steps.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Build"
```

## Multiple Jobs

```yaml
jobs:
  build:
    ...

  test:
    needs: build
    ...
```

`needs` creates dependency.

```text
build
  ↓
test
```

---

## Environment Variables

```yaml
env:
  APP_NAME: my-app

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "$APP_NAME"
```

Step-level:

```yaml
steps:
  - name: Test
    env:
      ENVIRONMENT: production
    run: echo "$ENVIRONMENT"
```

---

## Conditions

### `if`

```yaml
if: github.ref == 'refs/heads/main'
```

Run only on main.

### `failure()`

```yaml
if: failure()
```

Runs when a previous step fails.

### `always()`

```yaml
if: always()
```

Runs regardless of previous result.

### `continue-on-error`

```yaml
continue-on-error: true
```

Allows the job to continue after that step fails.

---

## Outputs

Set output:

```bash
echo "version=v1.0" >> "$GITHUB_OUTPUT"
```

Use output:

```yaml
${{ steps.version.outputs.version }}
```

Job output:

```yaml
outputs:
  version: ${{ steps.version.outputs.version }}
```

Use from another job:

```yaml
${{ needs.build.outputs.version }}
```

## Interview Answer

> "Steps execute commands, jobs group steps, `needs` controls job dependencies, environment variables store configuration, and conditions such as `if`, `failure()` and `always()` control execution."

---

# 📌 Day 44 — Secrets, Artifacts, Tests & Cache

## GitHub Secrets

Secrets store sensitive values.

Examples:

```text
DOCKER_USERNAME
DOCKER_TOKEN
SSH_PRIVATE_KEY
```

Usage:

```yaml
${{ secrets.DOCKER_TOKEN }}
```

### Important

Never hardcode:

```yaml
password: mypassword
```

Use:

```yaml
password: ${{ secrets.DOCKER_TOKEN }}
```

---

## Artifacts

Artifacts store files generated by a workflow.

Example:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: reports/
```

Download:

```yaml
- uses: actions/download-artifact@v4
  with:
    name: test-results
```

### Why Artifacts?

Useful for:

- Test reports
- Logs
- Build output
- Security reports
- Generated files

---

## Real Tests

Example:

```yaml
- name: Run tests
  run: pytest
```

If tests fail:

```text
pytest
  ↓
Failure
  ↓
Workflow fails
```

Fix the application/test and rerun.

---

## Cache

Caching avoids downloading the same dependencies repeatedly.

Concept:

```text
First Run
   ↓
Cache Miss
   ↓
Download dependencies
   ↓
Save Cache

Next Run
   ↓
Cache Hit
   ↓
Reuse dependencies
```

## Interview Answer

> "Secrets protect sensitive credentials, artifacts preserve workflow-generated files, tests validate the application, and caching improves workflow performance by reusing dependencies."

---

# 📌 Day 45 — Docker Build & Push with GitHub Actions

## Basic Flow

```text
Git Push
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Docker Login
   ↓
Docker Push
   ↓
Docker Hub
```

## Example

```yaml
- name: Login
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_TOKEN }}

- name: Build
  run: docker build -t username/app:latest .

- name: Push
  run: docker push username/app:latest
```

## Why Tag Images?

Instead of only:

```text
latest
```

Use version/commit tags:

```text
latest
sha-c49c556
v1.0.0
```

This makes versions traceable.

## Feature Branch Rule

A useful pattern:

```yaml
if: github.ref == 'refs/heads/main'
```

Build can happen on multiple branches, while Docker push happens only on main.

## Interview Answer

> "I used GitHub Actions to authenticate with Docker Hub, build the Docker image and push controlled tags. I kept pushing restricted to the main branch so feature branches don't publish production images."

---

# 📌 Day 46 — Reusable Workflows & Composite Actions

## Reusable Workflow

Uses:

```yaml
on:
  workflow_call:
```

Example:

```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

### Why?

Avoid duplicating CI/CD logic.

```text
PR Pipeline ─────┐
                 ├── Reusable Build
Main Pipeline ───┘
```

---

## Inputs

```yaml
inputs:
  app_name:
    required: true
    type: string
```

Caller:

```yaml
with:
  app_name: my-app
```

---

## Secrets

```yaml
secrets:
  docker_token:
    required: true
```

Caller:

```yaml
secrets:
  docker_token: ${{ secrets.DOCKER_TOKEN }}
```

---

## Outputs

Reusable workflow:

```yaml
outputs:
  build_version:
    value: ${{ jobs.build.outputs.build_version }}
```

Inside job:

```bash
echo "build_version=v1.0" >> "$GITHUB_OUTPUT"
```

Caller:

```yaml
${{ needs.build.outputs.build_version }}
```

---

## Composite Action

Composite actions combine multiple steps into one custom action.

Location:

```text
.github/actions/my-action/action.yml
```

Example:

```yaml
runs:
  using: composite
  steps:
    - shell: bash
      run: echo "Hello"
```

Use:

```yaml
uses: ./.github/actions/my-action
```

## Reusable Workflow vs Composite Action

| Reusable Workflow | Composite Action |
|---|---|
| Reuses complete jobs/workflows | Reuses steps |
| Uses `workflow_call` | Uses `composite` |
| Can contain jobs | Used inside a job |
| Good for CI/CD pipelines | Good for repeated step logic |

## Interview Answer

> "I use reusable workflows when I want to reuse complete CI/CD jobs and composite actions when I want to package repeated steps into a custom action."

---

# 📌 Day 47 — Advanced GitHub Actions Triggers

## PR Lifecycle

Important PR events:

```yaml
types:
  - opened
  - synchronize
  - reopened
  - closed
```

Useful for different stages of a PR lifecycle.

---

## Branch Filters

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'
```

Only selected branches trigger the workflow.

---

## Path Filters

```yaml
paths:
  - 'src/**'
  - 'app/**'
```

Workflow runs only when those paths change.

### `paths-ignore`

```yaml
paths-ignore:
  - '*.md'
  - 'docs/**'
```

Useful when documentation-only changes should not trigger expensive workflows.

---

## `workflow_run`

Runs a workflow after another workflow completes.

```yaml
on:
  workflow_run:
    workflows: ["Run Tests"]
    types:
      - completed
```

Example:

```text
Run Tests
   ↓
completed
   ↓
Deploy Workflow
```

Important distinction:

```text
workflow_call
→ Reuse a workflow

workflow_run
→ React after another workflow completes
```

---

## `repository_dispatch`

Allows an external system to trigger a GitHub Actions workflow.

```yaml
on:
  repository_dispatch:
    types:
      - deploy-request
```

Payload can contain data such as:

```text
environment = production
```

Useful for external automation systems.

---

## Interview Answer

> "`workflow_call` is for reusable workflows, `workflow_run` is for reacting to another workflow's completion, and `repository_dispatch` allows external systems to trigger GitHub Actions."

---

# 📌 Day 48 — GitHub Actions End-to-End CI/CD Project

## Project

Repository:

```text
github-actions-capstone
```

Application:

```text
Python Flask
```

Endpoints:

```text
/
 /health
```

Technologies:

```text
Python
Flask
Pytest
Docker
GitHub Actions
Docker Hub
```

---

# Day 48 Architecture

```text
Pull Request
     ↓
Build + Test
     ↓
PR Validation
     ↓
Merge to main
     ↓
Build + Test
     ↓
Prepare Short SHA
     ↓
 ┌───────────────┐
 ↓               ↓
Docker latest   Docker SHA
 ↓               ↓
 └───────┬───────┘
         ↓
 Production Deployment
         ↓
 Environment Approval

Every 12 Hours
       ↓
 Health Check
       ↓
 Pull Image
       ↓
 Run Container
       ↓
 /health
       ↓
 HTTP 200
```

## Reusable Build/Test

Important concepts:

```text
workflow_call
inputs
boolean input
Python version
pytest
job outputs
GITHUB_OUTPUT
```

Example:

```yaml
on:
  workflow_call:
    inputs:
      python-version:
        type: string
        default: "3.12"
```

---

## Reusable Docker Workflow

Important concepts:

```text
Docker login
Docker build
Docker push
workflow inputs
workflow secrets
workflow outputs
```

Image:

```text
aniruddhakharve/github-actions-capstone:latest
```

Commit-specific image:

```text
aniruddhakharve/github-actions-capstone:sha-c49c556
```

---

## PR Pipeline

```text
Pull Request
     ↓
Reusable Build/Test
     ↓
PR Validation
```

Important point:

> Docker publishing was not performed for the PR pipeline.

This prevents every PR from publishing images.

---

## Main Pipeline

```text
Push main
   ↓
Build + Test
   ↓
Prepare Short SHA
   ↓
 ┌─────────────┐
 ↓             ↓
Docker latest  Docker SHA
 └──────┬──────┘
        ↓
Production Deploy
```

Short SHA:

```bash
SHORT_SHA=$(echo "${GITHUB_SHA}" | cut -c1-7)
```

Output:

```text
c49c556
```

---

## Production Environment

GitHub Environment:

```text
production
```

Used for deployment control and approval.

Concept:

```text
Pipeline
   ↓
Production Environment
   ↓
Required Approval
   ↓
Deployment
```

---

## Health Check

Workflow runs every 12 hours.

```text
Pull Docker image
       ↓
Run container
       ↓
Wait
       ↓
curl /health
       ↓
HTTP 200
       ↓
PASS
```

Example:

```bash
HTTP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" \
  http://localhost:5000/health)
```

---

## GitHub Step Summary

Can create a readable workflow report:

```bash
echo "## Health Check Report" >> "$GITHUB_STEP_SUMMARY"
```

Useful for:

- Health reports
- Test summaries
- Deployment information
- Human-readable workflow results

---

## Important Day 48 Issue

### Full SHA vs Short SHA

Initial tag:

```text
sha-<full Git SHA>
```

Requirement:

```text
sha-<7-character SHA>
```

Solution:

```bash
SHORT_SHA=$(echo "${GITHUB_SHA}" | cut -c1-7)
```

Then pass it between jobs using:

```text
GITHUB_OUTPUT
```

## Interview Answer

> "In my GitHub Actions capstone, I created reusable build/test and Docker workflows. PRs run validation, while merges to main build and push both latest and commit-specific Docker images, followed by production deployment. I also added a scheduled health check that runs the Docker image and validates the `/health` endpoint."

---

# 🔥 Day 49 — DevSecOps Capstone: OWASP NodeGoat

## Project

Repository:

```text
devsecops-capstone-project
```

Application:

```text
OWASP NodeGoat
```

Technology:

```text
Node.js
Express
MongoDB
Docker
Docker Compose
GitHub Actions
Docker Hub
AWS EC2
OWASP ZAP
DevSecOps security tools
```

NodeGoat is intentionally vulnerable and is useful for learning application security and DevSecOps.

---

# NodeGoat Application

Main server:

```text
server.js
```

Application port:

```text
4000
```

Start command:

```bash
npm start
```

MongoDB connection in Compose:

```text
mongodb://mongo:27017/nodegoat
```

Architecture:

```text
NodeGoat
   ↓
Express Application
   ↓
MongoDB
```

Docker Compose:

```text
web container
     ↓
mongo container
```

---

# Why NodeGoat?

The purpose is not simply to deploy an application.

The project demonstrates:

```text
Development
    ↓
Testing
    ↓
Security
    ↓
Docker
    ↓
Image Security
    ↓
Registry
    ↓
Deployment
    ↓
DAST
    ↓
Health Verification
```

This is the practical difference between ordinary CI/CD and **DevSecOps**.

---

# DevSecOps

## Definition

> "DevSecOps integrates security into the DevOps lifecycle instead of treating security as a separate final step."

Traditional:

```text
Develop
  ↓
Build
  ↓
Deploy
  ↓
Security
```

DevSecOps:

```text
Code
 ↓
Lint
 ↓
SAST
 ↓
Secret Scan
 ↓
Dependency Scan
 ↓
Dockerfile Scan
 ↓
Build
 ↓
Image Scan
 ↓
Deploy
 ↓
DAST
```

---

# Security Testing Types

## SAST

**Static Application Security Testing**

Analyzes source code without running the application.

```text
Source Code
    ↓
SAST
    ↓
Security Findings
```

Example concept:

```text
Detect insecure code patterns
```

---

## DAST

**Dynamic Application Security Testing**

Tests a running application.

```text
Running Application
       ↓
     ZAP
       ↓
Security Findings
```

This is why ZAP belongs after deployment/application startup.

---

## SCA / Dependency Scanning

Checks third-party dependencies for known vulnerabilities.

Example:

```text
package.json
    ↓
Dependency Scanner
    ↓
Known CVEs
```

---

## Secret Scanning

Looks for accidentally committed secrets.

Examples:

```text
API keys
Passwords
Tokens
Private keys
```

---

## Dockerfile Linting

Checks Dockerfile best practices and potential problems.

Example tool:

```text
Hadolint
```

---

## Container Image Scanning

Checks the built image for vulnerabilities.

Example:

```text
Trivy
```

Flow:

```text
Dockerfile
   ↓
Docker Build
   ↓
Docker Image
   ↓
Trivy
   ↓
Vulnerability Report
```

---

# NodeGoat DevSecOps Pipeline

The actual project chain was designed around:

```text
SAST
 ↓
Secret Scan
 ↓
Dependency Scan
 ↓
Dockerfile Lint
 ↓
Docker Build
 ↓
nodegoat:ci
 ↓
nodegoat.tar Artifact
 ↓
Trivy Image Scan
 ↓
Docker Hub Push
 ↓
EC2 Deployment
 ↓
Health Check
 ↓
OWASP ZAP DAST
```

Important optimization:

> The same Docker build artifact was reused for scanning and publishing instead of unnecessarily rebuilding the image.

---

# Why Scan Before Push?

Preferred security flow:

```text
Build Image
    ↓
Scan Image
    ↓
If acceptable
    ↓
Push to Docker Hub
```

This prevents an unvalidated image from being published.

---

# Docker Artifact Concept

The CI pipeline created:

```text
nodegoat:ci
```

and stored the image as:

```text
nodegoat.tar
```

Concept:

```text
Build
 ↓
Docker Image
 ↓
Save as Artifact
 ↓
Next Job
 ↓
Load Image
 ↓
Scan
 ↓
Push
```

This avoids rebuilding the exact same image.

---

# Docker Compose

Important configuration:

```yaml
services:
  web:
    ...
    depends_on:
      - mongo

  mongo:
    ...
```

MongoDB hostname from the application:

```text
mongo
```

Not:

```text
localhost
```

Because containers communicate through the Compose network.

---

# Important Docker Networking Concept

Inside the NodeGoat container:

```text
mongodb://mongo:27017/nodegoat
```

works because:

```text
web container
     ↓
Docker DNS
     ↓
mongo
     ↓
MongoDB container
```

`localhost` would mean:

```text
NodeGoat container itself
```

not the MongoDB container.

## Interview Answer

> "In Docker Compose, services can communicate using service names because Docker provides internal DNS. Therefore NodeGoat connects to MongoDB using `mongo:27017`, not `localhost:27017`."

---

# Important MongoDB Confusion

This was an important concept:

```text
user: mongodb
```

does **not** mean:

```text
MongoDB database username
```

It refers to the OS/container user.

Database authentication is a separate concept.

In this Compose setup MongoDB authentication was not enabled, so the connection did not require database credentials.

---

# NodeGoat Dockerfile

Original NodeGoat Docker setup used an older Node image:

```dockerfile
FROM node:12-alpine
```

Important interview observation:

> The application contains older/deprecated dependencies because NodeGoat is intentionally vulnerable and designed for security training.

Therefore vulnerability/security findings are expected to be part of the learning exercise.

---

# DevSecOps Tool Categories

Remember the purpose instead of memorizing only tool names:

| Tool Category | Purpose |
|---|---|
| Linter | Code quality/style |
| SAST | Source-code security |
| Secret scanner | Detect leaked credentials |
| Dependency scanner | Detect vulnerable packages |
| Hadolint | Dockerfile best practices |
| Trivy | Container/image vulnerabilities |
| ZAP | Running application security testing |

---

# GitHub Actions DevSecOps Concepts

Important concepts used:

```text
workflow_call
inputs
secrets
env
if
needs
id
outputs
artifacts
matrix
```

Example dependency:

```yaml
needs:
  - security-scan
  - dockerfile-lint
```

Means:

```text
Security checks
      ↓
Docker build
```

---

# Parallel Jobs

Security checks that don't depend on each other can run in parallel.

Concept:

```text
             ┌── SAST
             │
             ├── Secret Scan
Code ────────┼── Dependency Scan
             │
             └── Hadolint
                    ↓
                 Docker Build
```

This reduces pipeline runtime.

In the NodeGoat project, parallelization reduced the runtime from approximately:

```text
7m 43s
```

to:

```text
6m 29s
```

Approximately:

```text
1m 14s improvement
```

---

# Important Trivy Concept

A successful Trivy command does **not automatically mean zero vulnerabilities**.

For example:

```text
exit-code: 0
```

can mean:

> The scan completed successfully and the report was generated.

It does not necessarily mean:

```text
0 HIGH
0 CRITICAL
```

This distinction is important in interviews.

---

# EC2 Deployment

The NodeGoat application was deployed to AWS EC2.

Concept:

```text
GitHub Actions
      ↓
Docker Image
      ↓
EC2
      ↓
Docker
      ↓
NodeGoat
      ↓
MongoDB
```

Application port:

```text
4000
```

---

# Deployment Verification

The deployed application was verified with:

```text
MongoDB healthy
NodeGoat healthy
EC2 port 4000 accessible
```

External request reached:

```text
/login
```

with:

```text
HTTP 302
```

The `302` was a redirect to the login route, so it demonstrated that the application was reachable and responding.

---

# OWASP ZAP

## What is ZAP?

OWASP ZAP = **OWASP Zed Attack Proxy**

It is used for dynamic application security testing.

It tests the running application.

```text
EC2 NodeGoat
     ↓
Running Web Application
     ↓
OWASP ZAP
     ↓
Security Findings
```

---

# ZAP Scan Result

The completed ZAP full scan tested:

```text
71 URLs
```

Results:

```text
FAIL-NEW: 0
WARN-NEW: 16
PASS: 125
```

Findings included examples such as:

```text
Directory browsing
Vulnerable JavaScript libraries
Missing security headers
Cookie SameSite issues
Source disclosure
Dangerous JavaScript
Reflected/DOM XSS
```

Important:

> These findings are expected to be useful in this project because NodeGoat is intentionally vulnerable.

---

# ZAP Workflow Error

The ZAP scan itself completed, but the workflow later failed when it attempted automatic GitHub issue creation.

Error:

```text
403 Resource not accessible by integration
```

Meaning:

```text
ZAP scan
   ↓
Completed
   ↓
Attempt GitHub issue creation
   ↓
GitHub permission restriction
   ↓
403
```

This is an important distinction:

> The security scan completed; the failure occurred in the post-scan GitHub issue creation step.

---

# Common NodeGoat Problems & Interview Explanation

## 1. MongoDB `localhost` Problem

### Problem

Application container tries:

```text
mongodb://localhost:27017/nodegoat
```

### Why?

Inside Docker, localhost refers to the same container.

### Fix

Use Compose service name:

```text
mongodb://mongo:27017/nodegoat
```

---

## 2. Docker Compose Service Communication

### Problem

Containers cannot communicate using hostnames you invent.

### Fix

Use the actual Compose service name:

```text
mongo
```

Docker's internal DNS resolves it.

---

## 3. Old Node.js Dependencies

NodeGoat uses an intentionally old/vulnerable application stack.

This can produce:

```text
deprecated package warnings
security vulnerabilities
npm warnings
```

Do not automatically assume every warning is a CI/CD failure.

Interview explanation:

> "NodeGoat is intentionally vulnerable, so deprecated dependencies and known vulnerabilities are expected. The purpose of the project is to detect and demonstrate these issues through DevSecOps tooling."

---

## 4. Docker Image Scanning

### Problem

A successful scanner execution can be misunderstood as zero vulnerabilities.

### Correct interpretation

```text
Scanner exit-code 0
≠
Zero vulnerabilities
```

You must inspect the actual report/findings and configure severity thresholds separately.

---

## 5. ZAP 403

### Problem

ZAP completed but workflow failed during GitHub issue creation.

```text
403 Resource not accessible by integration
```

### Reason

The GitHub Actions token did not have sufficient permission for that operation.

### Interview Explanation

> "The DAST scan itself completed successfully. The workflow failure happened afterward when the automation attempted to create a GitHub issue and GitHub rejected the operation because of token permissions."

---

# CI/CD vs DevSecOps — Important Interview Question

## CI/CD

```text
Build
 ↓
Test
 ↓
Package
 ↓
Deploy
```

## DevSecOps

```text
Build
 ↓
Test
 ↓
SAST
 ↓
Secret Scan
 ↓
Dependency Scan
 ↓
Dockerfile Scan
 ↓
Image Scan
 ↓
Deploy
 ↓
DAST
```

### Interview Answer

> "CI/CD automates building, testing and delivery. DevSecOps extends this process by integrating security checks throughout the lifecycle, including source-code scanning, secret detection, dependency scanning, container scanning and dynamic application testing."

---

# 🔥 Most Important Interview Concepts from Day 38–49

## YAML

```text
Indentation-sensitive configuration language
```

## CI

```text
Build + Test code continuously
```

## CD

```text
Deliver/deploy validated code
```

## Workflow

```text
Trigger → Jobs → Steps
```

## Runner

```text
Machine that executes GitHub Actions
```

## Job

```text
Collection of steps
```

## `needs`

```text
Creates job dependency
```

## `if`

```text
Conditional execution
```

## `always()`

```text
Run even after failure
```

## `failure()`

```text
Run when previous execution failed
```

## Secrets

```text
Sensitive credentials/configuration
```

## Artifacts

```text
Files produced/stored by workflow
```

## Matrix

```text
Run same job across multiple configurations
```

## `workflow_call`

```text
Reusable workflow
```

## Composite Action

```text
Reusable collection of steps
```

## `workflow_run`

```text
Trigger after another workflow completes
```

## `repository_dispatch`

```text
External event trigger
```

## SAST

```text
Analyze source code
```

## DAST

```text
Test running application
```

## Dependency Scan

```text
Find vulnerable third-party packages
```

## Secret Scan

```text
Find leaked credentials
```

## Hadolint

```text
Dockerfile linting
```

## Trivy

```text
Container/filesystem vulnerability scanning
```

## ZAP

```text
Dynamic web application security testing
```

---

# 🧠 Interview Scenario Questions

## Q1. Why use reusable workflows?

**Answer:**

> "To avoid duplicating CI/CD logic. I can define build, test or Docker logic once and call it from multiple workflows using `workflow_call`."

---

## Q2. What is the difference between `workflow_call` and `workflow_run`?

**Answer:**

> "`workflow_call` is used to reuse another workflow. `workflow_run` triggers a workflow after another workflow has completed."

---

## Q3. Why use Docker image tags based on Git SHA?

**Answer:**

> "A commit-specific tag makes the image traceable to the exact source revision. If `sha-c49c556` is deployed, I can identify exactly which commit produced that image."

---

## Q4. Why not use only `latest`?

**Answer:**

> "`latest` is mutable and does not uniquely identify a build. A commit-specific tag provides traceability and makes rollback easier."

---

## Q5. Why use GitHub Secrets?

**Answer:**

> "Secrets prevent sensitive credentials such as Docker tokens and SSH keys from being hardcoded in the repository or workflow."

---

## Q6. What happens if a GitHub Actions job fails?

**Answer:**

> "By default, dependent jobs using `needs` will not run. Conditions such as `always()` can be used when a cleanup or reporting step must execute regardless of failure."

---

## Q7. What is the difference between an artifact and a Docker image?

**Answer:**

> "A workflow artifact is a file or collection of files stored by GitHub Actions. A Docker image is a container package containing the application and its runtime environment."

---

## Q8. Why scan Docker images?

**Answer:**

> "The base image and installed packages can contain known vulnerabilities. Image scanning helps identify those vulnerabilities before the image is deployed."

---

## Q9. Why is ZAP used after deployment?

**Answer:**

> "ZAP performs dynamic testing against a running application, so the application must be accessible before DAST can be performed."

---

## Q10. Why does NodeGoat use `mongo` instead of `localhost`?

**Answer:**

> "Because NodeGoat and MongoDB run in separate containers. Docker Compose provides internal DNS, so the application connects to MongoDB using the service name `mongo`."

---

## Q11. What is the difference between SAST and DAST?

**Answer:**

> "SAST analyzes application source code without executing it, while DAST tests the running application from the outside."

---

## Q12. What is the purpose of dependency scanning?

**Answer:**

> "It identifies known vulnerabilities in third-party libraries used by the application."

---

## Q13. Why scan secrets?

**Answer:**

> "To detect accidentally committed credentials such as API keys, passwords, tokens or private keys before they are misused."

---

## Q14. What is the purpose of Hadolint?

**Answer:**

> "Hadolint analyzes Dockerfiles for common mistakes and best-practice violations."

---

## Q15. What does Trivy do?

**Answer:**

> "Trivy can scan container images and filesystems for known vulnerabilities and security issues."

---

## Q16. Why run security checks in parallel?

**Answer:**

> "Independent checks can execute simultaneously, reducing overall pipeline execution time."

---

## Q17. What happened with the ZAP 403 error?

**Answer:**

> "The ZAP scan completed, but the workflow failed afterward while attempting to create a GitHub issue. GitHub returned `403 Resource not accessible by integration`, indicating insufficient permission for that operation."

---

# 🎯 How to Explain the NodeGoat Project in an Interview

Use this answer:

> "I built a DevSecOps capstone project using OWASP NodeGoat, which is an intentionally vulnerable Node.js application with MongoDB. I containerized the application using Docker and Docker Compose and created a GitHub Actions pipeline that integrates security into the CI/CD lifecycle.
>
> The pipeline performs source and security checks such as SAST, secret scanning, dependency scanning and Dockerfile linting. After that, the application is built into a Docker image, scanned with Trivy and then published to Docker Hub.
>
> The image is deployed to an AWS EC2 instance where NodeGoat and MongoDB run as containers. After deployment, I perform a health check and run OWASP ZAP as a DAST tool against the live application.
>
> One important issue I solved was the MongoDB connection. The application initially used localhost, but inside Docker localhost refers to the application container itself. I changed the connection to the Compose service name `mongo`.
>
> I also encountered a GitHub Actions 403 error after the ZAP scan when the workflow attempted to create a GitHub issue. The scan itself had completed; the failure was related to GitHub integration permissions."

---

# ⚡ 60-Second Revision — Day 38 → Day 49

```text
YAML
 ↓
CI/CD Concepts
 ↓
GitHub Actions
 ↓
Triggers
 ↓
Matrix
 ↓
Runners
 ↓
Jobs / Steps
 ↓
Variables / Conditions
 ↓
Secrets
 ↓
Artifacts
 ↓
Cache
 ↓
Docker Build & Push
 ↓
Reusable Workflows
 ↓
Composite Actions
 ↓
Advanced Triggers
 ↓
End-to-End CI/CD
 ↓
DevSecOps
 ↓
SAST
 ↓
Secret Scan
 ↓
Dependency Scan
 ↓
Hadolint
 ↓
Docker Build
 ↓
Trivy Image Scan
 ↓
Docker Hub
 ↓
EC2
 ↓
Health Check
 ↓
OWASP ZAP / DAST
```

---

# 📝 Quick Command Revision

## GitHub Actions Output

```bash
echo "value=test" >> "$GITHUB_OUTPUT"
```

## Git SHA

```bash
echo "${GITHUB_SHA}"
```

## Short SHA

```bash
echo "${GITHUB_SHA}" | cut -c1-7
```

## Docker Build

```bash
docker build -t username/app:latest .
```

## Docker Push

```bash
docker push username/app:latest
```

## Docker Run

```bash
docker run -d -p 4000:4000 username/app:latest
```

## Docker Compose

```bash
docker compose up -d
```

## Check Containers

```bash
docker ps
```

## Check Logs

```bash
docker logs <container>
```

## Test Health Endpoint

```bash
curl http://localhost:4000/health
```

## Check HTTP Status

```bash
curl -o /dev/null -s -w "%{http_code}" http://localhost:4000/health
```

---

# 🔥 Important Errors to Remember

| Problem | Reason | Fix / Lesson |
|---|---|---|
| YAML workflow not behaving | Wrong indentation/syntax | YAML indentation matters |
| Job running unexpectedly | Missing `if` | Add condition |
| Job running before dependency | Missing `needs` | Define dependency |
| Secret exposed in workflow | Hardcoded credential | Use GitHub Secrets |
| Docker push from feature branch | No branch condition | Restrict push using `if` |
| Wrong image version | Only `latest` used | Add commit-based tag |
| Full SHA instead of short SHA | `${{ github.sha }}` is full SHA | Generate 7-character SHA |
| Repeated workflow logic | Duplicated jobs | Use `workflow_call` |
| Repeated steps | Duplicated commands | Use composite action |
| Docker containers can't reach MongoDB | Used `localhost` | Use Compose service name |
| NodeGoat dependency warnings | Old/intentionally vulnerable dependencies | Expected in security-training project |
| Image scan misunderstood | Scanner exit code interpreted incorrectly | Inspect actual vulnerability results |
| ZAP workflow failed | GitHub issue creation returned 403 | Permission/integration problem |
| Deployment unavailable | Service/port/security-group issue | Verify container, port and EC2 access |

---

# 🏆 Final Interview Checklist

Before an interview, make sure you can explain these without looking them up:

## GitHub Actions

```text
✓ Workflow
✓ Trigger
✓ Job
✓ Step
✓ Runner
✓ needs
✓ if
✓ always()
✓ failure()
✓ continue-on-error
✓ env
✓ secrets
✓ artifacts
✓ cache
✓ matrix
✓ outputs
✓ GITHUB_OUTPUT
✓ workflow_call
✓ composite actions
✓ workflow_run
✓ repository_dispatch
```

## Docker

```text
✓ Image
✓ Container
✓ Dockerfile
✓ Docker Build
✓ Docker Push
✓ Docker Hub
✓ Tags
✓ SHA-based tags
✓ Docker Compose
✓ Container networking
✓ Service names
```

## DevSecOps

```text
✓ SAST
✓ DAST
✓ Secret scanning
✓ Dependency scanning
✓ Dockerfile scanning
✓ Image scanning
✓ Security gates
✓ OWASP ZAP
✓ Trivy
✓ Hadolint
```

## AWS / Deployment

```text
✓ EC2
✓ Application deployment
✓ Port 4000
✓ Docker containers
✓ MongoDB container
✓ Health check
✓ External application verification
```

---

# 💡 One-Line Memory Tricks

```text
YAML       → Configuration

CI         → Build + Test

CD         → Deliver + Deploy

Runner     → Machine executing workflow

Job        → Group of steps

needs      → Dependency

if         → Condition

Secrets    → Sensitive values

Artifact   → Workflow-generated files

Matrix     → Multiple configurations

workflow_call
           → Reusable workflow

Composite  → Reusable steps

SAST       → Source code security

SCA        → Dependency security

DAST       → Running application security

Hadolint   → Dockerfile

Trivy      → Image vulnerabilities

ZAP        → Web application DAST

Docker Hub → Image registry

EC2        → Deployment server
```

---

# 🚀 Final Project Story

```text
Day 38
YAML
 ↓
Day 39
CI/CD Fundamentals
 ↓
Day 40–47
GitHub Actions
 ↓
Day 48
End-to-End CI/CD
 ↓
Day 49
DevSecOps
 ↓
NodeGoat
 ↓
Security Scanning
 ↓
Docker
 ↓
Trivy
 ↓
Docker Hub
 ↓
AWS EC2
 ↓
Health Check
 ↓
OWASP ZAP
```

> **Core interview message:**  
> "I progressed from understanding YAML and CI/CD concepts to building GitHub Actions workflows, reusable workflows and an end-to-end CI/CD pipeline. I then extended that pipeline into a DevSecOps pipeline using OWASP NodeGoat, integrating security checks, container scanning, Docker, Docker Hub, AWS EC2 deployment and OWASP ZAP DAST."

---

# ⭐ Most Important Things to Remember

1. **CI/CD automates software delivery.**
2. **GitHub Actions workflows are YAML files.**
3. **Jobs run on runners.**
4. **`needs` creates dependencies between jobs.**
5. **`if` controls conditional execution.**
6. **Secrets protect credentials.**
7. **Artifacts store workflow-generated files.**
8. **Matrix avoids duplicating workflows for multiple configurations.**
9. **Reusable workflows prevent duplicated CI/CD logic.**
10. **Composite actions package reusable steps.**
11. **Docker packages the application and runtime environment.**
12. **Commit-based image tags provide traceability.**
13. **DevSecOps integrates security throughout CI/CD.**
14. **SAST checks source code.**
15. **Dependency scanning checks third-party packages.**
16. **Secret scanning detects leaked credentials.**
17. **Hadolint checks Dockerfiles.**
18. **Trivy checks images/filesystems for vulnerabilities.**
19. **DAST tests the running application.**
20. **ZAP is used for web application DAST.**
21. **Docker Compose service names provide container-to-container DNS.**
22. **`localhost` inside a container means that same container.**
23. **NodeGoat is intentionally vulnerable, so security findings are part of the learning exercise.**
24. **A successful scanner execution does not automatically mean zero vulnerabilities.**
25. **A pipeline failure must be traced to the exact stage — for example, the NodeGoat ZAP scan completed, while the later GitHub issue creation step failed with HTTP 403.**

---

# 🎯 Final Interview Revision Formula

```text
WHAT?
→ What technology/tool is this?

WHY?
→ Why did we use it?

HOW?
→ How did we implement it?

PROBLEM?
→ What problem/error did we face?

FIX?
→ How did we solve it?

RESULT?
→ What did the pipeline/application achieve?
```

### Example — NodeGoat

```text
WHAT?
→ OWASP NodeGoat DevSecOps project.

WHY?
→ To practice integrating security into CI/CD.

HOW?
→ GitHub Actions + Docker + security scanning + EC2 + ZAP.

PROBLEM?
→ MongoDB connection using localhost.

FIX?
→ Use Docker Compose service name "mongo".

RESULT?
→ NodeGoat deployed on EC2 and tested with health checks and ZAP.
```

**If you can explain every section above in your own words, you have the Day 38 → Day 49 interview revision covered.** 🚀
