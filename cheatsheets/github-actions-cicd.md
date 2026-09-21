# 🚀 GitHub Actions & CI/CD — Interview Cheat Sheet

> **Purpose:** Quick interview revision — not a replacement for hands-on practice.
>
> This cheat sheet covers GitHub Actions, CI/CD, Docker-based pipelines, reusable workflows, artifacts, secrets, deployment concepts, and DevSecOps concepts from **Day 38 → Day 49**.

> **Project Reference:**  
> My end-to-end DevSecOps implementation is available here:  
> https://github.com/Aniruddhakharve/devsecops-capstone-project

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
  - docker            # List
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

---

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

Uses an existing GitHub Action.

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

### Step-Level Environment Variable

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
- Docker image archives

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

Reusable workflow:

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

---

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

---

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

Project URL:

```text
https://github.com/Aniruddhakharve/devsecops-capstone-project
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

# What is DevSecOps?

## Definition

> DevSecOps integrates security into the DevOps lifecycle instead of treating security as a separate final step.

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

### Main Principle

```text
DevOps
   +
Security
   =
DevSecOps
```

Security is integrated into the same automated delivery lifecycle.

---

# 🔐 DevSecOps Key Principles

## 1. Catch Problems Early

A security issue detected during development or pull request validation can be addressed before deployment.

```text
Developer
   ↓
Security Check
   ↓
Early Detection
```

---

## 2. Automate Security Checks

Security checks should run automatically instead of depending entirely on manual review.

```text
Git Push
   ↓
Pipeline
   ↓
Security Scans
```

---

## 3. Security Throughout the Lifecycle

Security is not only one final step.

```text
Source Code
     ↓
Dependencies
     ↓
Dockerfile
     ↓
Container Image
     ↓
Running Application
```

---

## 4. Never Put Secrets in Code

Examples of sensitive values:

```text
Passwords
API Keys
Tokens
Private Keys
Cloud Credentials
```

Use:

```text
GitHub Secrets
Environment Variables
AWS Secrets Manager
AWS Systems Manager Parameter Store
```

as appropriate.

---

## 5. Least Privilege

Give workflows only the permissions they actually need.

Example:

```yaml
permissions:
  contents: read
```

Concept:

```text
Minimum Required Permissions
          ↓
Reduced Attack Surface
```

---

# 🧪 Security Testing Types

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

Example tool:

```text
Semgrep
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

Example tool:

```text
OWASP ZAP
```

This is why DAST belongs after application deployment/startup.

---

## SCA / Dependency Scanning

Checks third-party dependencies for known vulnerabilities.

```text
package.json
    ↓
Dependency Scanner
    ↓
Known CVEs
```

Example:

```text
npm audit
```

---

## Secret Scanning

Looks for accidentally committed secrets.

Examples:

```text
API Keys
Passwords
Tokens
Private Keys
```

Example:

```text
Gitleaks
```

---

## Dockerfile Linting

Checks Dockerfile best practices and potential problems.

Example:

```text
Hadolint
```

---

## Container Image Scanning

Checks the built image for known vulnerabilities.

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

# 🛡️ DevSecOps Tools Used in NodeGoat

| Tool | Category | Purpose |
|---|---|---|
| JSHint | Linting | Code quality |
| Semgrep | SAST | Source-code security |
| Gitleaks | Secret Scanning | Detect leaked credentials |
| npm audit | Dependency Scan | Detect vulnerable npm packages |
| Hadolint | Dockerfile Lint | Dockerfile best practices |
| Trivy | Image Scan | Container vulnerabilities |
| OWASP ZAP | DAST | Running web application security |

---

# 🧱 NodeGoat Architecture

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

Application port:

```text
4000
```

MongoDB connection:

```text
mongodb://mongo:27017/nodegoat
```

---

# 🌐 Docker Networking in NodeGoat

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

# 💾 Docker Compose Volume

MongoDB uses a named volume:

```yaml
volumes:
  mongo_data:
```

and:

```yaml
volumes:
  - mongo_data:/data/db
```

Purpose:

```text
MongoDB Container
      ↓
Persistent Volume
      ↓
Data survives container recreation
```

---

# ❤️ Docker Health Checks

NodeGoat health check:

```yaml
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:4000"]
  interval: 30s
  timeout: 10s
  retries: 3
```

MongoDB health check:

```yaml
healthcheck:
  test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
  interval: 10s
  timeout: 5s
  retries: 5
```

---

# ⏳ `depends_on`

```yaml
depends_on:
  mongo:
    condition: service_healthy
```

This means the web application waits for MongoDB to become healthy before startup.

Important:

```text
Container started
≠
Service ready
```

Health checks help verify actual readiness.

---

# 🧠 Important MongoDB Confusion

This:

```yaml
user: mongodb
```

does not mean:

```text
MongoDB database username
```

It refers to the operating-system/container user.

Database authentication is a separate concept.

---

# 🐳 Dockerfile Security Improvements

The final Dockerfile uses:

```dockerfile
FROM node:20-alpine
```

and a multi-stage build.

The runtime stage uses:

```dockerfile
USER $USER
```

where:

```text
USER=node
```

Other improvements include:

```text
.dockerignore
Production dependencies
Multi-stage build
Non-root runtime user
```

---

# 🔍 SAST — Semgrep

Semgrep scans source code for insecure patterns.

Command:

```bash
semgrep scan \
  --config=auto \
  --json \
  --output=semgrep-report.json
```

The report is uploaded as an artifact.

One observed scan produced approximately:

```text
WARNING: 25
ERROR:    7
INFO:     1
```

Examples included:

```text
Dangerous eval() usage
Private key material
Open redirect-related findings
Other insecure coding patterns
```

---

# 🔑 Secret Scan — Gitleaks

Workflow:

```yaml
- name: checkout code
  uses: actions/checkout@v7
  with:
    fetch-depth: 0

- name: Run Gitleaks
  uses: gitleaks/gitleaks-action@v2
```

`fetch-depth: 0` is important because it makes the complete Git history available.

---

# 🚨 Secret Leak Lesson

During development, Gitleaks detected secret-like values.

The project contained:

```text
Private key
Hardcoded ZAP API key
```

The ZAP API key configuration was changed to:

```javascript
process.env.ZAP_API_KEY || ""
```

The private key file was removed.

Important rule:

```text
Removing a secret from the current file
does not automatically remove it from Git history.
```

For a real exposed secret:

```text
Detect
  ↓
Rotate / Revoke
  ↓
Remove from source
  ↓
Clean history when appropriate
```

---

# 📦 Dependency Scanning — npm audit

The workflow runs:

```bash
npm ci
```

and:

```bash
npm audit --json > npm-audit-report.json || true
```

One initial audit reported approximately:

```text
145 vulnerabilities

8 low
33 moderate
66 high
38 critical
```

This is expected to some extent because NodeGoat is intentionally vulnerable and uses an old dependency tree.

I did not run:

```bash
npm audit fix --force
```

because it could introduce breaking changes.

---

# 🐳 Dockerfile Linting — Hadolint

Workflow:

```yaml
- name: Run Hadolint
  uses: hadolint/hadolint-action@v3.3.0
  with:
    dockerfile: Dockerfile
```

Purpose:

```text
Dockerfile
   ↓
Hadolint
   ↓
Best-practice checks
```

---

# 🛡️ Trivy Image Scanning

The image is built:

```bash
docker build -t nodegoat:ci .
```

Then saved:

```bash
docker save nodegoat:ci -o nodegoat.tar
```

The next workflow loads it:

```bash
docker load -i nodegoat.tar
```

Then Trivy scans:

```yaml
- name: Run Trivy image scan
  uses: aquasecurity/trivy-action@v0.36.0
  with:
    image-ref: nodegoat:ci
    format: table
    severity: CRITICAL,HIGH
    exit-code: 0
```

---

# ⚠️ Trivy Exit Code

Important interview concept:

```text
exit-code: 0
```

does not automatically mean:

```text
0 vulnerabilities
```

It can simply mean the scan completed without using vulnerability findings as a failure condition.

Always inspect the actual scan results.

The trainer example used:

```yaml
exit-code: '1'
```

to block the pipeline for selected severities.

In the NodeGoat capstone, I intentionally used report-only behavior because the application is intentionally vulnerable.

---

# 📦 Why Save Docker Image as an Artifact?

The image is built once.

```text
Docker Build
     ↓
nodegoat:ci
     ↓
docker save
     ↓
nodegoat.tar
     ↓
GitHub Artifact
```

Later:

```text
Download Artifact
     ↓
docker load
     ↓
Trivy Scan
     ↓
Docker Hub Push
```

This ensures the same built image can be scanned and published without rebuilding it.

---

# 🏷️ SHA-Based Docker Tags

The image tag is generated from:

```text
github.sha
```

Example:

```text
aniruddhakharve/nodegoat:264585bd...
```

The flow is:

```text
Git Commit
    ↓
Docker Build
    ↓
SHA Tag
    ↓
Docker Hub
    ↓
EC2 Deployment
```

This provides traceability.

---

# 🔗 GitHub Actions Output Chain

The image tag moves through:

```text
Step Output
     ↓
Job Output
     ↓
Reusable Workflow Output
     ↓
Main Pipeline
     ↓
Docker Push
     ↓
EC2 Deployment
```

The step creates the output:

```bash
echo "image-tag=${{ github.sha }}" >> "$GITHUB_OUTPUT"
```

The step has:

```yaml
id: meta
```

The job exposes:

```yaml
outputs:
  image-tag: ${{ steps.meta.outputs.image-tag }}
```

The reusable workflow exposes the workflow output.

The main workflow consumes it with:

```yaml
${{ needs.docker-build.outputs.image-tag }}
```

---

# 🔐 `secrets: inherit`

Reusable workflows that need repository secrets can be called using:

```yaml
secrets: inherit
```

Example:

```yaml
docker-push:
  uses: ./.github/workflows/docker-push.yml
  secrets: inherit
```

This allows the called workflow to access the repository secrets it requires.

---

# 🔀 Parallel Security Jobs

Independent security jobs can run in parallel.

Example:

```text
                 Lint
                   │
       ┌───────────┼─────────────┬──────────────┐
       ▼           ▼             ▼              ▼
      SAST      Gitleaks      npm audit     Hadolint
       │           │             │              │
       └───────────┴─────────────┴──────────────┘
                            │
                            ▼
                       Docker Build
```

Configured using:

```yaml
needs: [lint]
```

This reduced the observed pipeline execution time.

Observed:

```text
Before:
7m 43s

After:
6m 29s
```

Improvement:

```text
~1m 14s
```

---

# 🔄 Full NodeGoat DevSecOps Pipeline

```text
Developer
    ↓
Git Push / Pull Request
    ↓
GitHub Actions
    ↓
Lint
    ↓
 ┌──────────────┬──────────────┬──────────────┐
 ▼              ▼              ▼              ▼
Test           SAST        Secret Scan   Dependency Scan
                                             
 └──────────────┬──────────────┬──────────────┘
                ▼
         Dockerfile Lint
                ↓
          Docker Build
                ↓
         Docker Image Artifact
                ↓
           Trivy Scan
                ↓
           Docker Hub
                ↓
              EC2
                ↓
         Docker Compose
                ↓
         Health Check
                ↓
           OWASP ZAP
                ↓
              DAST
```

---

# ☁️ AWS EC2 Deployment

The exact image version is deployed to EC2.

Concept:

```text
GitHub Actions
      ↓
Docker Hub
      ↓
EC2
      ↓
Docker Compose
      ↓
NodeGoat + MongoDB
```

The deployment workflow:

```text
Connect to EC2
      ↓
Check Docker
      ↓
Check Docker Compose
      ↓
Create ~/devops
      ↓
Copy docker-compose.yml
      ↓
Create .env
      ↓
Login to Docker Hub
      ↓
docker compose pull
      ↓
docker compose down
      ↓
docker compose up -d --force-recreate
```

---

# 🔐 EC2 `.env`

The deployment generates:

```env
DOCKER_USERNAME=<Docker Hub username>
IMAGE_TAG=<Git SHA>
```

Permissions:

```bash
chmod 600 ~/devops/.env
```

This configuration allows Compose to pull the correct Docker image.

---

# ❤️ Application Health Check

After deployment:

```bash
curl -fsS http://${{ secrets.EC2_HOST }}:4000/login > /dev/null
```

Expected result:

```text
Checking application health...
Application is healthy.
```

The important point:

```text
Container running
≠
Application healthy
```

The health check verifies the application is reachable.

---

# 🕷️ OWASP ZAP

## What is ZAP?

OWASP ZAP = **OWASP Zed Attack Proxy**

It performs dynamic web application security testing.

```text
Running NodeGoat
       ↓
OWASP ZAP
       ↓
Security Findings
```

---

# 🧪 ZAP Baseline vs Full Scan

I initially tested a full ZAP scan.

It took approximately:

```text
42+ minutes
```

The full scan was therefore too slow for the regular CI/CD pipeline.

I switched to:

```yaml
zaproxy/action-baseline@v0.15.0
```

The baseline scan completed in approximately:

```text
7 minutes
```

as part of the pipeline.

---

# 🔎 ZAP Findings

The DAST scan identified observations including:

```text
Missing security headers
Directory browsing
Vulnerable JavaScript libraries
Cross-domain JavaScript concerns
XSS-related findings
Other web security observations
```

---

# ⚠️ ZAP 403 Troubleshooting

During initial implementation, ZAP completed the scan but the workflow then attempted GitHub issue creation.

The workflow returned:

```text
403 Resource not accessible by integration
```

The important distinction is:

```text
ZAP Scan
   ↓
Completed
   ↓
GitHub Issue Creation
   ↓
403 Permission Error
```

The scan itself had completed; the failure happened during the later GitHub issue-writing operation.

The workflow was changed to:

```yaml
allow_issue_writing: false
```

---

# 🔐 Security Policy in This Learning Project

NodeGoat is intentionally vulnerable.

Therefore security scans are primarily configured for reporting:

```text
Semgrep
   ↓
Findings

Gitleaks
   ↓
Secret Detection

npm audit
   ↓
Dependency Findings

Trivy
   ↓
Image Findings

ZAP
   ↓
DAST Findings
```

This does not mean vulnerabilities are acceptable in production.

A production pipeline could define security gates such as:

```text
Critical Vulnerability
        ↓
Pipeline Failure
        ↓
Deployment Blocked
```

---

# 🧠 Security Gate vs Security Scanner

A scanner answers:

```text
"What vulnerabilities exist?"
```

A security gate answers:

```text
"Should this result block the pipeline?"
```

Example:

```text
Trivy
  ↓
Finds CRITICAL CVE
  ↓
Policy decides
  ↓
Fail / Warn
```

This distinction is important in interviews.

---

# 🔐 GitHub Actions Security — Least Privilege

Recommended workflow-level permission when read access is sufficient:

```yaml
permissions:
  contents: read
```

If a workflow needs to write pull-request comments:

```yaml
permissions:
  contents: read
  pull-requests: write
```

General principle:

```text
Grant only what the workflow requires.
```

If a third-party action is compromised, unnecessary write permissions could increase its potential impact.

---

# 🚀 Production Improvements to Remember

The capstone is a learning project.

A production implementation could additionally use:

```text
OIDC authentication
AWS Secrets Manager
AWS Systems Manager Parameter Store
Image digest pinning
Dependency update automation
Stricter vulnerability gates
HTTPS
Application Load Balancer
Blue/Green Deployment
Rolling Deployment
ECS / EKS
Centralized logging
Monitoring
Alerting
```

---

# 🔥 OIDC — Important Interview Concept

## What is OIDC?

OIDC = **OpenID Connect**

In GitHub Actions, OIDC can be used for cloud authentication without storing long-lived cloud credentials in GitHub Secrets.

Concept:

```text
GitHub Actions
      ↓
OIDC Token
      ↓
Cloud Identity Provider
      ↓
Short-Lived Credentials
      ↓
AWS
```

### Traditional Approach

```text
AWS Access Key
AWS Secret Key
       ↓
GitHub Secrets
       ↓
GitHub Actions
```

### OIDC Approach

```text
GitHub Actions
       ↓
OIDC
       ↓
Assume IAM Role
       ↓
Temporary Credentials
```

### Why Is OIDC Important?

It reduces the need to store long-lived AWS access keys.

It is widely useful for production CI/CD authentication.

### Interview Answer

> "GitHub Actions OIDC allows the workflow to obtain short-lived cloud credentials through an identity trust relationship instead of storing long-lived AWS access keys in GitHub Secrets."

---

# 🔗 OIDC AWS Concept

Typical architecture:

```text
GitHub Actions
      ↓
OIDC Identity Token
      ↓
AWS IAM OIDC Provider
      ↓
IAM Role
      ↓
Temporary AWS Credentials
      ↓
AWS Resources
```

Important keywords:

```text
OIDC
IAM Role
Trust Policy
Short-Lived Credentials
Federation
```

---

# 🧱 Action Pinning — Supply Chain Security

Using:

```yaml
uses: actions/checkout@v4
```

is convenient.

A stronger supply-chain security practice is pinning an action to a specific commit SHA:

```yaml
uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11
```

Concept:

```text
Tag
 ↓
Can potentially move

Commit SHA
 ↓
Fixed version
```

This reduces the risk of silently changing action code through a moved tag.

---

# 📊 SARIF and GitHub Security Tab

Security tools can generate SARIF reports.

SARIF:

```text
Static Analysis Results Interchange Format
```

Example Trivy output:

```yaml
- uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'username/app:latest'
    format: 'sarif'
    output: 'trivy-results.sarif'

- uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```

This allows compatible security findings to be uploaded to GitHub's code-scanning/security interfaces.

---

# 🔍 Dependency Review Action

Trainer Day 49 introduced:

```yaml
- name: Check Dependencies for Vulnerabilities
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical
```

Important concept:

```text
Pull Request
     ↓
Dependency Changes
     ↓
Dependency Review
     ↓
Check New Dependencies
```

This is different from auditing the entire existing dependency tree.

```text
Dependency Review
→ Review dependency changes in a PR

npm audit
→ Audit project npm dependencies
```

---

# 🧠 Day 49 Concepts Not Directly Used in the Capstone

The trainer introduced several additional concepts that are important for interview preparation even though they were not all required in the final NodeGoat implementation.

Remember these separately:

```text
GitHub Secret Scanning
Push Protection
Dependency Review Action
Workflow Permissions
OIDC
SARIF
Action Pinning
```

---

# 🔐 Secret Scanning vs Push Protection

```text
Secret Scanning
      ↓
Detect secret exposure

Push Protection
      ↓
Prevent secret from being pushed
```

Interview answer:

> "Secret scanning detects exposed credentials, while push protection tries to stop a detected secret before it reaches the repository."

---

# 🔑 GitHub Secret Scanning

GitHub can scan repositories for supported secret patterns.

Basic idea:

```text
Developer Push
      ↓
GitHub
      ↓
Secret Detection
      ↓
Alert
```

This can complement pipeline-based secret scanners such as Gitleaks.

---

# 🛡️ Push Protection

Push protection acts earlier:

```text
Developer Push
      ↓
Secret Detected
      ↓
Push Blocked
```

It is therefore a prevention mechanism rather than only a detection mechanism.

---

# 📦 Dependency Review Action

Used for pull-request dependency changes.

Example:

```yaml
uses: actions/dependency-review-action@v4
```

Concept:

```text
PR
 ↓
Dependency Change
 ↓
Dependency Review
 ↓
Pass / Fail
```

---

# 📜 SARIF

SARIF is a standard format used to exchange static analysis/security scan results.

Useful flow:

```text
Security Tool
     ↓
SARIF Report
     ↓
GitHub
     ↓
Security / Code Scanning Interface
```

---

# 🔐 Action Pinning

Basic:

```yaml
uses: actions/checkout@v4
```

Pinned:

```yaml
uses: actions/checkout@<commit-sha>
```

Interview idea:

> "Pinning actions to commit SHAs provides stronger supply-chain control because the referenced code cannot silently change just because a tag moves."

---

# 🔄 `workflow_call` vs `workflow_run` vs `repository_dispatch`

| Feature | Purpose |
|---|---|
| `workflow_call` | Reuse another workflow |
| `workflow_run` | Trigger after another workflow completes |
| `repository_dispatch` | Trigger workflow from an external event |

Quick memory:

```text
workflow_call
→ Reuse

workflow_run
→ React after workflow

repository_dispatch
→ External trigger
```

---

# 🧠 GitHub Actions Inputs / Outputs / Artifacts

## Input

Value passed into a reusable workflow.

```yaml
with:
  image-tag: abc123
```

## Output

Value produced by a workflow.

```yaml
${{ needs.build.outputs.image-tag }}
```

## Artifact

File transferred between workflow jobs.

```text
nodegoat.tar
semgrep-report.json
trivy-report.json
npm-audit-report.json
```

Quick memory:

```text
Input
→ Goes IN

Output
→ Comes OUT

Artifact
→ Carries FILES
```

---

# 🧠 `id` vs `outputs`

Example:

```yaml
- name: Set image tag
  id: meta
  run: echo "image-tag=${{ github.sha }}" >> "$GITHUB_OUTPUT"
```

Here:

```text
id: meta
→ Identifies the step

$GITHUB_OUTPUT
→ Creates the step output

steps.meta.outputs.image-tag
→ Reads the output
```

Job output:

```yaml
outputs:
  image-tag: ${{ steps.meta.outputs.image-tag }}
```

Then another job can use:

```yaml
${{ needs.build.outputs.image-tag }}
```

---

# ⚡ `always()` vs `failure()` vs `continue-on-error`

## `always()`

```yaml
if: always()
```

Run regardless of the previous result.

Good for:

```text
Reports
Logs
Cleanup
Artifact upload
```

## `failure()`

```yaml
if: failure()
```

Run when a previous step/job has failed.

## `continue-on-error`

```yaml
continue-on-error: true
```

Allow that step/job to continue without failing the whole execution in the normal way.

Quick memory:

```text
always()
→ Always execute

failure()
→ Execute after failure

continue-on-error
→ Don't let this failure stop execution normally
```

---

# 🧠 CI/CD vs DevSecOps

## CI/CD

```text
Code
 ↓
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
Code
 ↓
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

## Interview Answer

> "CI/CD automates building, testing and delivery. DevSecOps extends that process by integrating security checks throughout the lifecycle."

---

# 🎯 Most Important Interview Questions — GitHub Actions

## What is GitHub Actions?

> "GitHub Actions is a CI/CD automation platform integrated with GitHub. It allows workflows to execute automatically in response to repository events."

## What is a runner?

> "A runner is the execution environment where GitHub Actions jobs run."

## What is a workflow?

> "A workflow is a YAML-defined automation process containing triggers, jobs and steps."

## What is a job?

> "A job is a group of steps that runs on a runner."

## What is a step?

> "A step is an individual task inside a job, implemented using either `run` or `uses`."

## What is `needs`?

> "`needs` defines dependencies between jobs and controls execution order."

## What is `if`?

> "`if` evaluates a condition and determines whether a job or step should run."

## What is `workflow_dispatch`?

> "It allows a workflow to be manually triggered from GitHub."

## What is `workflow_call`?

> "It allows one workflow to call another reusable workflow."

## What is `workflow_run`?

> "It triggers a workflow after another workflow completes."

## What is `repository_dispatch`?

> "It allows an external system to send an event that triggers a GitHub Actions workflow."

---

# 🎯 Most Important Interview Questions — DevSecOps

## What is DevSecOps?

> "DevSecOps integrates security into the DevOps lifecycle so security checks are automated throughout CI/CD."

## What is SAST?

> "Static Application Security Testing analyzes source code without running the application."

## What is DAST?

> "Dynamic Application Security Testing tests the running application."

## What is dependency scanning?

> "Dependency scanning identifies known vulnerabilities in third-party packages used by the application."

## What is secret scanning?

> "Secret scanning detects credentials such as API keys, passwords, tokens and private keys that may have been exposed."

## Why use Trivy?

> "Trivy scans container images and other targets for known security vulnerabilities."

## Why use Gitleaks?

> "Gitleaks detects secrets in source code and Git history."

## Why use Semgrep?

> "Semgrep performs static code analysis and can identify insecure coding patterns."

## Why use Hadolint?

> "Hadolint checks Dockerfiles for common mistakes and best-practice violations."

## Why use ZAP?

> "OWASP ZAP performs dynamic web application security testing against a running application."

---

# 🎯 Most Important Interview Questions — Docker + DevSecOps

## Why scan Docker images?

> "A container image can contain vulnerabilities in base OS packages or application dependencies, so scanning helps identify issues before deployment."

## Why use a SHA-based tag?

> "A SHA-based tag provides a traceable link between the Docker image and the exact Git commit used to build it."

## Why not only use `latest`?

> "`latest` is mutable and does not uniquely identify a build. A commit-specific tag gives deterministic versioning."

## Why use artifacts for Docker images?

> "The Docker image can be built once, stored as an artifact, then scanned and published without rebuilding the image."

---

# 🎯 Most Important Interview Questions — Security

## What is least privilege?

> "Least privilege means granting only the permissions required to perform the task and no more."

## What is OIDC?

> "OIDC allows GitHub Actions to authenticate with cloud providers using short-lived credentials instead of storing long-lived cloud keys."

## What is SARIF?

> "SARIF is a standardized format for security and static-analysis results that can be consumed by platforms such as GitHub."

## Why pin GitHub Actions?

> "Pinning actions to commit SHAs helps protect against supply-chain risks associated with mutable tags."

---

# 🧠 Real-World Scenario Questions

## Scenario: A developer accidentally commits an AWS access key. What should happen?

```text
Detect
 ↓
Block / Alert
 ↓
Revoke / Rotate Credential
 ↓
Remove Secret
 ↓
Review Git History
```

Tools/features that may help:

```text
GitHub Push Protection
GitHub Secret Scanning
Gitleaks
```

---

## Scenario: Trivy finds a critical vulnerability. What happens?

The scanner reports the vulnerability.

Then pipeline policy decides whether:

```text
Fail
```

or:

```text
Warn / Report
```

For production environments, organizations commonly define severity thresholds according to their security policy.

---

## Scenario: PR introduces a vulnerable package. Which concept is useful?

```text
Dependency Review
```

This can evaluate dependency changes introduced by the pull request.

---

## Scenario: Workflow needs AWS access but you do not want long-lived AWS keys.

Use:

```text
GitHub Actions OIDC
        ↓
AWS IAM Role
        ↓
Temporary Credentials
```

---

## Scenario: Independent scans are taking too long.

Check whether they can run in parallel.

Example:

```text
SAST
Secret Scan
Dependency Scan
Dockerfile Lint
```

can often run simultaneously when they do not depend on one another.

Use:

```yaml
needs:
```

only for actual dependencies.

---

# 🧪 Common NodeGoat Problems & Fixes

| Problem | Reason | Fix / Lesson |
|---|---|---|
| Docker container exits | Missing/incorrect command | Check `docker ps -a`, logs and inspect |
| MongoDB connection refused | Application used `localhost` | Use Docker Compose service name |
| MongoDB health check failed | Wrong shell command | Use available `mongo` command |
| Docker permission error on EC2 | User not in Docker group | Add user to Docker group |
| Secret scanner misses history | Shallow checkout | Use `fetch-depth: 0` |
| ZAP workflow returns 403 | GitHub issue-writing permission | Disable issue writing if not required |
| Pipeline too slow | Independent jobs were sequential | Parallelize with `needs` |
| Wrong Docker version deployed | Only `latest` used | Use SHA-based image tag |
| Image scanned differs from pushed image | Image rebuilt between stages | Build once and reuse artifact |
| Scanner passes but vulnerabilities exist | Exit code policy misunderstood | Inspect scan results separately |

---

# ⚡ Quick Revision — DevSecOps

```text
DevSecOps
→ Security integrated into DevOps

SAST
→ Source code security

DAST
→ Running application security

SCA
→ Dependency security

Secret Scan
→ Detect credentials

Hadolint
→ Dockerfile

Trivy
→ Container image

ZAP
→ Web application DAST

OIDC
→ Short-lived cloud authentication

SARIF
→ Standard security report format

Push Protection
→ Prevent secret from being pushed

Least Privilege
→ Minimum permissions
```

---

# ⚡ Quick Revision — GitHub Actions

```text
Workflow
→ Complete automation definition

Trigger
→ Starts workflow

Runner
→ Executes jobs

Job
→ Group of steps

Step
→ Individual action/command

uses
→ Existing action

run
→ Shell command

needs
→ Dependency

if
→ Condition

always()
→ Execute regardless of previous result

failure()
→ Execute after failure

continue-on-error
→ Allow failure without normal pipeline stop

env
→ Environment variables

secrets
→ Sensitive values

inputs
→ Values passed into reusable workflow

outputs
→ Values passed out

id
→ Identify a step

GITHUB_OUTPUT
→ Create step output

Artifacts
→ Share workflow files

Matrix
→ Multiple configurations

workflow_call
→ Reusable workflow

workflow_run
→ After another workflow completes

repository_dispatch
→ External trigger
```

---

# ⚡ Quick Revision — Docker CI/CD

```text
Git Push
   ↓
GitHub Actions
   ↓
Docker Build
   ↓
Docker Image
   ↓
Image Scan
   ↓
Docker Hub
   ↓
Deployment
```

With SHA tagging:

```text
Git Commit
   ↓
SHA
   ↓
Docker Image Tag
   ↓
Registry
   ↓
Deployment
```

---

# 🏆 Final Day 38 → Day 49 Story

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
OWASP NodeGoat
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

The learning progression is:

```text
Configuration
    ↓
CI/CD Fundamentals
    ↓
GitHub Actions
    ↓
Reusable Automation
    ↓
Docker CI/CD
    ↓
Cloud Deployment
    ↓
DevSecOps
    ↓
Application Security
```

---

# 🎤 60-Second Interview Story — Day 38 → Day 49

> "I started by learning YAML and CI/CD fundamentals, then built GitHub Actions workflows covering triggers, runners, jobs, steps, matrix strategies, conditions, secrets, artifacts and caching. I then learned reusable workflows using `workflow_call`, workflow inputs and outputs, and composite actions.
>
> I applied those concepts to an end-to-end CI/CD project where code was tested, Docker images were built and published, and deployments were controlled using GitHub Actions.
>
> For the DevSecOps stage, I extended the work using OWASP NodeGoat. I integrated JSHint, Semgrep, Gitleaks, npm audit, Hadolint, Trivy and OWASP ZAP. The pipeline builds the Docker image once, scans it, pushes a SHA-tagged image to Docker Hub, deploys that exact image to AWS EC2, performs a health check, and then runs ZAP DAST against the live application.
>
> I also learned important security concepts such as least privilege, secret scanning, dependency review, action pinning, SARIF and OIDC."

---

# 🏁 Final Interview Checklist

Before an interview, make sure you can explain these in your own words:

## GitHub Actions

```text
✓ Workflow
✓ Trigger
✓ Job
✓ Step
✓ Runner
✓ uses
✓ run
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
✓ inputs
✓ outputs
✓ id
✓ GITHUB_OUTPUT
✓ workflow_call
✓ composite actions
✓ workflow_run
✓ repository_dispatch
✓ permissions
```

## Docker

```text
✓ Image
✓ Container
✓ Dockerfile
✓ Docker Build
✓ Docker Push
✓ Docker Hub
✓ Image Tags
✓ SHA Tags
✓ Multi-stage builds
✓ Non-root containers
✓ Docker Compose
✓ Networks
✓ Service Names
✓ Volumes
✓ Healthchecks
✓ depends_on
```

## DevSecOps

```text
✓ DevSecOps
✓ SAST
✓ DAST
✓ SCA
✓ Secret Scanning
✓ Push Protection
✓ Dependency Review
✓ Dockerfile Scanning
✓ Image Scanning
✓ Security Gates
✓ Trivy
✓ Semgrep
✓ Gitleaks
✓ Hadolint
✓ OWASP ZAP
✓ SARIF
✓ OIDC
✓ Least Privilege
✓ Action Pinning
```

## AWS / Deployment

```text
✓ EC2
✓ SSH Deployment
✓ Docker on EC2
✓ Docker Compose on EC2
✓ Docker Hub Authentication
✓ SHA-based Deployment
✓ Health Check
```

---

# 💡 One-Line Memory Tricks

```text
YAML
→ Configuration

CI
→ Build + Test

CD
→ Deliver + Deploy

Runner
→ Machine executing workflow

Job
→ Group of steps

Step
→ Individual task

needs
→ Dependency

if
→ Condition

Secrets
→ Sensitive values

Artifact
→ Workflow-generated files

Matrix
→ Multiple configurations

workflow_call
→ Reusable workflow

Composite
→ Reusable steps

workflow_run
→ After workflow completion

repository_dispatch
→ External trigger

SAST
→ Source code security

SCA
→ Dependency security

DAST
→ Running application security

Gitleaks
→ Secrets

Semgrep
→ SAST

npm audit
→ Dependencies

Hadolint
→ Dockerfile

Trivy
→ Container Image

ZAP
→ Web Application DAST

OIDC
→ Short-lived cloud authentication

SARIF
→ Security report format

Docker Hub
→ Container registry

EC2
→ Deployment server
```

---

# 🎯 Final Mental Model

When asked to design a secure GitHub Actions pipeline, think:

```text
1. Trigger
   ↓
2. Checkout
   ↓
3. Lint
   ↓
4. Test
   ↓
5. SAST
   ↓
6. Secret Scan
   ↓
7. Dependency Scan
   ↓
8. Dockerfile Scan
   ↓
9. Docker Build
   ↓
10. Image Scan
   ↓
11. Push Image
   ↓
12. Deploy
   ↓
13. Health Check
   ↓
14. DAST
```

Then ask:

```text
Can independent jobs run in parallel?
Are secrets protected?
Are workflow permissions minimal?
Is the deployed image traceable to a commit?
Can the same build artifact be reused?
Should vulnerabilities block deployment?
Can cloud authentication use OIDC?
```

That thought process is more important than memorizing individual YAML snippets.

---

# 🔥 Final Project Reference

Complete NodeGoat DevSecOps implementation:

https://github.com/Aniruddhakharve/devsecops-capstone-project

Use the repository to review:

```text
.github/workflows/
Dockerfile
.dockerignore
docker-compose.yml
Security workflows
Deployment workflow
Health check
OWASP ZAP
Docker configuration
Project README
```

> **Core interview message:**  
> "I progressed from understanding YAML and CI/CD concepts to building GitHub Actions workflows, reusable workflows and an end-to-end Docker CI/CD pipeline. I then extended that pipeline into a DevSecOps pipeline using OWASP NodeGoat, integrating SAST, secret scanning, dependency scanning, Dockerfile linting, container image scanning, AWS EC2 deployment and OWASP ZAP DAST."
