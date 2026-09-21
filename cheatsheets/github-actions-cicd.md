# 🚀 GitHub Actions & CI/CD — Interview Cheat Sheet
## Day 38 → Day 49 | YAML → CI/CD → GitHub Actions → Docker CI/CD → DevSecOps → OWASP NodeGoat

> **Purpose:** This cheat sheet is designed for **fast interview revision**.  
> Read a section once and you should be able to recall **what it is, why it is used, how it works, and a practical example from the projects built during Day 38–49**.
>
> **Project references used here:**  
> Day 48 → `github-actions-capstone`  
> Day 49 → `devsecops-capstone-project` using OWASP NodeGoat

---

# 1. YAML BASICS

## What is YAML?

**YAML = YAML Ain't Markup Language**

YAML is a human-readable configuration format used heavily in DevOps tools.

Common examples:

```text
GitHub Actions
Docker Compose
Kubernetes
Ansible
CI/CD configuration
```

### Basic Structure

```yaml
name: My Workflow

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
List → -
Nested configuration → indentation
```

Example:

```yaml
server:
  name: web-server
  port: 8080
  enabled: true

packages:
  - git
  - docker
  - nginx
```

### Interview Answer

> "YAML is a human-readable, indentation-sensitive configuration format. In DevOps it is commonly used for tools like GitHub Actions, Docker Compose, Kubernetes and Ansible."

---

# 2. CI/CD FUNDAMENTALS

## What is CI?

**CI = Continuous Integration**

Developers frequently integrate code into a shared repository and automated checks run.

```text
Code Push
   ↓
Build
   ↓
Test
   ↓
Quality Checks
```

### Why CI?

Catch problems early.

---

## What is Continuous Delivery?

The application is automatically built, tested and prepared for release.

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

## What is Continuous Deployment?

Successful changes are automatically deployed.

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

## CI vs CD

| CI | CD |
|---|---|
| Integrates code | Delivers/deploys code |
| Build | Package |
| Test | Release |
| Quality checks | Deployment |
| Detects problems early | Automates delivery |

### Interview Answer

> "CI automates the process of building and testing code whenever changes are integrated. CD automates delivering that validated application to an environment."

---

# 3. GITHUB ACTIONS

## What is GitHub Actions?

GitHub Actions is a CI/CD and automation platform integrated with GitHub repositories.

The basic structure is:

```text
Workflow
   ↓
Jobs
   ↓
Steps
   ↓
Runner
```

### Example

```yaml
name: CI

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: echo "Building application"
```

---

# 4. WORKFLOW

A **workflow** is the complete YAML automation definition.

Usually stored in:

```text
.github/workflows/
```

Example:

```text
.github/workflows/main.yml
```

A workflow defines:

```text
When should it run?
What jobs should run?
Which runner should execute them?
What steps should be performed?
```

---

# 5. TRIGGERS

## Push

```yaml
on:
  push:
```

Runs when code is pushed.

---

## Pull Request

```yaml
on:
  pull_request:
```

Runs when a pull request event occurs.

---

## Manual Trigger

```yaml
on:
  workflow_dispatch:
```

Useful for:

```text
Manual deployment
Testing a workflow
Health checks
Emergency jobs
```

---

## Scheduled Trigger

```yaml
on:
  schedule:
    - cron: '0 */12 * * *'
```

Runs every 12 hours.

---

## PR Event Types

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened
      - closed
```

Useful when different PR lifecycle events need different behavior.

---

# 6. BRANCH AND PATH FILTERS

## Branch Filter

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'
```

Only those branches trigger the workflow.

---

## Path Filter

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'app/**'
```

The workflow runs only when relevant paths change.

---

## `paths-ignore`

```yaml
paths-ignore:
  - '*.md'
  - 'docs/**'
```

Useful when documentation-only changes should not trigger expensive jobs.

---

# 7. JOBS

A job is a collection of steps executed on a runner.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "Build"
```

Multiple jobs:

```yaml
jobs:
  build:
    ...

  test:
    needs: build
    ...
```

Flow:

```text
build
  ↓
test
```

---

# 8. `needs` — JOB DEPENDENCY

`needs` creates a dependency between jobs.

```yaml
test:
  needs: build
```

Means:

```text
build must complete successfully
        ↓
test can start
```

### Example from NodeGoat

```yaml
image-scan:
  needs: docker-build
```

The image cannot be scanned until the image has been built.

### Interview Answer

> "`needs` defines the execution dependency between jobs. I used it in my DevSecOps pipeline so image scanning happens only after Docker build completes."

---

# 9. RUNNERS

A runner is the machine that executes the workflow.

```text
Workflow
   ↓
Runner
   ↓
Commands / Tests / Build
```

Common GitHub-hosted runners:

```text
ubuntu-latest
windows-latest
macos-latest
```

Self-hosted runner:

```text
GitHub
   ↓
Self-hosted Server
```

### Interview Answer

> "A runner is the execution environment used by GitHub Actions to run jobs. It can be GitHub-hosted or self-hosted."

---

# 10. `run` VS `uses`

## `run`

Executes a shell command.

```yaml
- name: Show version
  run: python --version
```

## `uses`

Uses an existing action.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

### Easy Memory Trick

```text
run  → Run command
uses → Use action
```

---

# 11. ENVIRONMENT VARIABLES

Global:

```yaml
env:
  APP_NAME: nodegoat
```

Job-level:

```yaml
jobs:
  build:
    env:
      ENVIRONMENT: production
```

Step-level:

```yaml
steps:
  - name: Test
    env:
      API_URL: http://localhost:4000
    run: echo "$API_URL"
```

Environment variables are useful for non-secret configuration.

---

# 12. GITHUB ACTIONS CONDITIONS

## `if`

Run a step/job only when a condition is true.

```yaml
if: github.ref == 'refs/heads/main'
```

Example:

```yaml
- name: Push image
  if: github.ref == 'refs/heads/main'
  run: docker push username/app:latest
```

This prevents feature branches from publishing production images.

---

## `failure()`

Run when an earlier step failed.

```yaml
if: failure()
```

---

## `always()`

Run regardless of previous result.

```yaml
if: always()
```

Useful for:

```text
Cleanup
Reports
Logs
Container removal
```

---

## `continue-on-error`

```yaml
continue-on-error: true
```

The step can fail but the workflow continues.

Important:

```text
continue-on-error
→ Do not stop pipeline because of this step
```

---

# 13. OUTPUTS — VERY IMPORTANT

Outputs are values produced by one step/job/workflow and consumed later.

This is one of the most useful GitHub Actions concepts for understanding your Day 48 and Day 49 projects.

---

## Step Output

Give a step an ID:

```yaml
- name: Generate version
  id: version
  run: |
    echo "version=v1.0" >> "$GITHUB_OUTPUT"
```

Use it later:

```yaml
${{ steps.version.outputs.version }}
```

Flow:

```text
Step
 ↓
GITHUB_OUTPUT
 ↓
steps.version.outputs.version
```

### Why `id`?

Without:

```yaml
id: version
```

you cannot reference:

```yaml
steps.version.outputs.version
```

---

# 14. JOB OUTPUTS

A job can expose outputs to another job.

Example:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:
      - name: Generate version
        id: version
        run: |
          echo "version=v1.0" >> "$GITHUB_OUTPUT"
```

Another job can consume it:

```yaml
deploy:
  needs: build
  runs-on: ubuntu-latest

  steps:
    - run: echo "${{ needs.build.outputs.version }}"
```

Flow:

```text
Step Output
     ↓
Job Output
     ↓
needs.<job>.outputs.<name>
```

---

# 15. WORKFLOW INPUTS — VERY IMPORTANT

Inputs are values passed **into a reusable workflow**.

Example:

```yaml
on:
  workflow_call:
    inputs:
      image-tag:
        required: true
        type: string
```

Caller:

```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image-tag: sha-c49c556
```

Inside reusable workflow:

```yaml
${{ inputs.image-tag }}
```

### Easy Memory

```text
inputs
   ↓
Value enters reusable workflow
```

---

# 16. WORKFLOW OUTPUTS — VERY IMPORTANT

Outputs are values returned **from a reusable workflow**.

Example:

```yaml
on:
  workflow_call:
    outputs:
      image_url:
        description: "Docker image path"
        value: ${{ jobs.docker.outputs.image_url }}
```

Inside the reusable workflow:

```yaml
jobs:
  docker:
    outputs:
      image_url: ${{ steps.image.outputs.image_url }}
```

Step:

```yaml
- name: Set image output
  id: image
  run: |
    IMAGE="aniruddhakharve/nodegoat:sha-c49c556"
    echo "image_url=$IMAGE" >> "$GITHUB_OUTPUT"
```

Caller:

```yaml
${{ needs.docker.outputs.image_url }}
```

### Complete Flow

```text
Step
 ↓
GITHUB_OUTPUT
 ↓
Step Output
 ↓
Job Output
 ↓
Reusable Workflow Output
 ↓
Caller Workflow
```

This exact concept became important in the NodeGoat and GitHub Actions projects.

---

# 17. INPUTS VS OUTPUTS

| Inputs | Outputs |
|---|---|
| Data entering workflow | Data leaving workflow |
| Caller → reusable workflow | Reusable workflow → caller |
| `with:` | `needs...outputs` |
| `${{ inputs.name }}` | `${{ needs.job.outputs.name }}` |

### Easy Memory

```text
INPUT
→ In

OUTPUT
→ Out
```

---

# 18. SECRETS

Secrets store sensitive values.

Examples:

```text
DOCKER_TOKEN
EC2_SSH_KEY
EC2_HOST
API_TOKEN
PASSWORD
```

Use:

```yaml
${{ secrets.DOCKER_TOKEN }}
```

Never:

```yaml
password: mypassword
```

---

## Secrets vs Environment Variables

```text
Normal configuration
→ env

Sensitive credentials
→ secrets
```

Example:

```yaml
env:
  APP_ENV: production

secrets:
  DOCKER_TOKEN
```

---

# 19. ARTIFACTS

Artifacts are files generated by a workflow that need to be stored or passed to another job.

Example:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: reports
    path: reports/
```

Download:

```yaml
- uses: actions/download-artifact@v4
  with:
    name: reports
```

Common artifacts:

```text
Test reports
Security reports
Logs
Docker image TAR
Build outputs
```

---

# 20. ARTIFACT VS DOCKER IMAGE

This distinction is important.

```text
Artifact
→ File stored by GitHub Actions

Docker Image
→ Application package used to create containers
```

In the NodeGoat project:

```text
Docker Image
   ↓
docker save
   ↓
nodegoat.tar
   ↓
GitHub Artifact
```

Later:

```text
nodegoat.tar
   ↓
docker load
   ↓
nodegoat:ci
```

This allowed the already-built image to be scanned without rebuilding it.

---

# 21. CACHE

Caching avoids repeatedly downloading the same dependencies.

```text
First Run
 ↓
Cache Miss
 ↓
Download dependencies
 ↓
Save cache

Next Run
 ↓
Cache Hit
 ↓
Reuse dependencies
```

### Interview Answer

> "Caching improves CI performance by reusing previously downloaded dependencies instead of fetching them on every run."

---

# 22. MATRIX

Matrix lets one job run with multiple configurations.

```yaml
strategy:
  matrix:
    node-version: [20, 22]
```

Creates:

```text
Node 20 → Job
Node 22 → Job
```

Can also be used for:

```text
Operating systems
Python versions
Node versions
Database versions
```

---

# 23. MATRIX `fail-fast`

```yaml
strategy:
  fail-fast: false
```

Other matrix jobs continue even if one fails.

---

# 24. REUSABLE WORKFLOWS

Reusable workflow:

```yaml
on:
  workflow_call:
```

Caller:

```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

### Why use reusable workflows?

Without reusable workflows:

```text
PR Workflow
   ↓
Duplicate build logic

Main Workflow
   ↓
Duplicate build logic
```

With reusable workflows:

```text
PR ────────┐
           ├── Reusable Build
Main ──────┘
```

Benefits:

```text
Less duplication
Easier maintenance
Consistent pipeline logic
Reusable inputs/outputs
```

---

# 25. COMPOSITE ACTIONS

Composite actions package reusable **steps**.

Example location:

```text
.github/actions/setup-and-greet/action.yml
```

Basic structure:

```yaml
runs:
  using: composite

  steps:
    - name: Greeting
      shell: bash
      run: echo "Hello"
```

Use:

```yaml
uses: ./.github/actions/setup-and-greet
```

---

# 26. REUSABLE WORKFLOW VS COMPOSITE ACTION

| Reusable Workflow | Composite Action |
|---|---|
| Reuses complete workflow logic | Reuses steps |
| `workflow_call` | `composite` |
| Can contain jobs | Used inside a job |
| Good for pipeline stages | Good for repeated commands |

### Easy Memory

```text
Reusable Workflow
→ Reuse jobs/workflows

Composite Action
→ Reuse steps
```

---

# 27. `workflow_run`

Runs a workflow after another workflow completes.

```yaml
on:
  workflow_run:
    workflows: ["Run Tests"]
    types:
      - completed
```

Flow:

```text
Run Tests
   ↓
Completed
   ↓
Deploy Workflow
```

---

# 28. `workflow_call` VS `workflow_run`

```text
workflow_call
→ Reuse another workflow

workflow_run
→ React to another workflow finishing
```

This distinction is commonly asked in interviews.

---

# 29. `repository_dispatch`

Used to trigger a workflow from an external system.

```yaml
on:
  repository_dispatch:
    types:
      - deploy-request
```

Example payload concept:

```text
environment = production
```

Useful for external automation.

---

# 30. DOCKER CI/CD

Typical flow:

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

Example:

```yaml
- name: Login
  uses: docker/login-action@v4
  with:
    username: ${{ inputs.docker_username }}
    password: ${{ secrets.docker_token }}

- name: Build
  run: docker build -t username/app:latest .

- name: Push
  run: docker push username/app:latest
```

---

# 31. WHY USE GIT SHA IMAGE TAGS?

Instead of only:

```text
latest
```

use:

```text
sha-c49c556
```

because:

```text
latest
→ Mutable tag

sha-c49c556
→ Exact source revision
```

Flow:

```text
Git Commit
   ↓
Docker Image
   ↓
Docker Hub
   ↓
Deployment
```

This provides:

```text
Traceability
Version identification
Rollback capability
```

---

# 32. DEVSECOPS

## What is DevSecOps?

**DevSecOps = Development + Security + Operations**

The important idea:

> Security is integrated into the DevOps lifecycle instead of being treated as a separate final step.

Without DevSecOps:

```text
Develop
 ↓
Build
 ↓
Deploy
 ↓
Security Review
```

With DevSecOps:

```text
Code
 ↓
Test
 ↓
Security Checks
 ↓
Build
 ↓
Image Scan
 ↓
Deploy
 ↓
DAST
```

### Easy Memory

```text
DevOps + Security everywhere
= DevSecOps
```

---

# 33. WHY DEVSECOPS?

The goal is to detect security problems as early and automatically as practical.

Examples:

```text
Bad code
   ↓
SAST

Leaked secret
   ↓
Secret Scanner

Vulnerable dependency
   ↓
Dependency Scanner

Bad Dockerfile
   ↓
Hadolint

Vulnerable image
   ↓
Trivy

Running web vulnerability
   ↓
ZAP
```

---

# 34. SECURITY GATE — VERY IMPORTANT

A **security gate** is a pipeline rule that decides whether the pipeline is allowed to continue based on a security result.

Example:

```text
Security Scan
      ↓
Critical vulnerability?
      ↓
YES → FAIL → Stop deployment

NO
 ↓
Continue pipeline
```

This is different from simply running a scanner.

### Scanner

```text
Find vulnerability
```

### Security Gate

```text
Decide pipeline behavior
```

This distinction is extremely important.

---

# 35. TRIVY SECURITY GATE

Trainer example:

```yaml
- name: Scan Docker Image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: username/app:latest
    format: table
    exit-code: '1'
    severity: CRITICAL,HIGH
```

Meaning:

```text
Scan image
 ↓
Check HIGH + CRITICAL
 ↓
Found?
 ↓
exit-code 1
 ↓
Step fails
 ↓
Pipeline can stop
```

---

# 36. `exit-code: 0` VS `exit-code: 1`

## `exit-code: 0`

```text
Scan finds vulnerabilities
        ↓
Report
        ↓
Pipeline continues
```

## `exit-code: 1`

```text
Scan finds selected vulnerability
        ↓
Step fails
        ↓
Pipeline blocked
```

### NodeGoat Example

The NodeGoat project is intentionally vulnerable, so the current capstone uses report-oriented behavior for some scans.

```yaml
exit-code: 0
```

That allows us to demonstrate the whole pipeline.

### Production Concept

For a normal production image, the policy could be:

```yaml
severity: CRITICAL,HIGH
exit-code: 1
```

Then:

```text
Critical/High finding
→ Security Gate fails
→ No push/deployment
```

### Interview Answer

> "The scanner detects the vulnerability, while the exit code and pipeline condition define the security gate. In my NodeGoat learning project I used report-oriented behavior because the application is intentionally vulnerable. In production I would define severity thresholds and block release when the policy requires it."

---

# 37. SAST

**SAST = Static Application Security Testing**

Analyzes source code without running the application.

```text
Source Code
    ↓
SAST
    ↓
Security Findings
```

In NodeGoat:

```text
Tool → Semgrep
```

Example:

```bash
semgrep scan \
  --config=auto \
  --json \
  --output=semgrep-report.json
```

---

# 38. DAST

**DAST = Dynamic Application Security Testing**

Tests the running application.

```text
Running Application
       ↓
DAST
       ↓
Security Findings
```

In NodeGoat:

```text
Tool → OWASP ZAP
```

### Easy Memory

```text
SAST → Source

DAST → Running application
```

---

# 39. SAST VS DAST

| SAST | DAST |
|---|---|
| Static testing | Dynamic testing |
| Source code | Running application |
| Semgrep | OWASP ZAP |
| Finds insecure coding patterns | Finds runtime/web issues |

---

# 40. SECRET SCANNING

Secret scanning looks for credentials such as:

```text
API keys
Tokens
Passwords
Private keys
Cloud credentials
```

NodeGoat project:

```text
Tool → Gitleaks
```

Example:

```yaml
- name: Run Gitleaks
  uses: gitleaks/gitleaks-action@v2
```

---

# 41. WHY `fetch-depth: 0` FOR SECRET SCANNING?

Default checkout may be shallow.

```yaml
with:
  fetch-depth: 0
```

gets the complete Git history.

Why important?

Because:

```text
Secret committed
 ↓
Secret removed from current file
 ↓
Old commit still contains secret
```

A history-aware scan can still detect it.

---

# 42. SECRET SCANNING VS PUSH PROTECTION

## Secret Scanning

```text
Secret enters repository/history
        ↓
Scanner detects it
```

## Push Protection

```text
Secret detected before push
        ↓
Push blocked
```

### Easy Memory

```text
Secret Scanning
→ Detect

Push Protection
→ Prevent
```

---

# 43. REAL NODEGOAT SECRET ISSUE

During the project, secret-like content was detected:

```text
Private key
Hardcoded ZAP API key
```

The application configuration was changed to read the ZAP API key from the environment:

```javascript
process.env.ZAP_API_KEY || ""
```

The private key file was removed.

### Important Security Rule

```text
Detect
 ↓
Revoke / Rotate
 ↓
Remove from source
 ↓
Clean history when required
```

Removing a secret from the current file does not automatically remove it from Git history.

---

# 44. DEPENDENCY SCANNING

Dependencies can have known vulnerabilities.

For Node.js:

```text
package.json
    ↓
npm audit
    ↓
Known vulnerabilities
```

Example:

```bash
npm audit --json > npm-audit-report.json || true
```

---

# 45. `npm audit` VS DEPENDENCY REVIEW

## npm audit

Audits the dependency tree.

```text
Project dependencies
      ↓
npm audit
      ↓
Vulnerability report
```

## Dependency Review

Checks dependency changes introduced by a pull request.

Example:

```yaml
- name: Dependency Review
  uses: actions/dependency-review-action@v4
  with:
    fail-on-severity: critical
```

### Easy Memory

```text
npm audit
→ Audit dependencies

Dependency Review
→ Review dependency changes in PR
```

---

# 46. NODEGOAT DEPENDENCY FINDINGS

NodeGoat uses an intentionally old dependency tree.

Observed audit:

```text
145 vulnerabilities
```

Approximately:

```text
8 low
33 moderate
66 high
38 critical
```

This is useful for demonstrating security scanning.

Important lesson:

```text
Detection ≠ Remediation
```

Scanning identifies the problem.

Fixing the dependency is a separate task.

---

# 47. WHY NOT `npm audit fix --force`?

Because:

```bash
npm audit fix --force
```

can introduce breaking dependency changes.

For an old/intentionally vulnerable training application:

```text
Force upgrade
   ↓
Dependency changes
   ↓
Application may break
```

So the project used the audit primarily for:

```text
Detection
 ↓
Reporting
 ↓
Review
```

---

# 48. HADOLINT

Hadolint checks Dockerfiles for common mistakes and best-practice violations.

Example:

```yaml
- name: Run Hadolint
  uses: hadolint/hadolint-action@v3.3.0
  with:
    dockerfile: Dockerfile
```

Easy memory:

```text
Hadolint
→ Dockerfile
```

---

# 49. TRIVY

Trivy scans container images for known vulnerabilities.

Flow:

```text
Docker Image
     ↓
Trivy
     ↓
Vulnerability Report
```

NodeGoat:

```text
nodegoat:ci
```

Example:

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

# 50. IMPORTANT TRIVY LESSON

A successful scanner execution does **not automatically mean zero vulnerabilities**.

For example:

```text
exit-code: 0
```

can simply mean:

```text
Scanner completed successfully
```

It does not necessarily mean:

```text
0 HIGH
0 CRITICAL
```

Always distinguish:

```text
Scan result
vs
Pipeline policy
```

---

# 51. NODEGOAT TRIVY FINDINGS

The image contained findings from:

```text
Alpine Linux packages
Node.js application dependencies
```

Examples included:

```text
kind-of
body-parser
brace-expansion
marked
minimatch
```

The point of the exercise was to detect and document these vulnerabilities.

---

# 52. DOCKER IMAGE SECURITY FLOW

The NodeGoat project used:

```text
Docker Build
      ↓
Docker Save
      ↓
GitHub Artifact
      ↓
Docker Load
      ↓
Trivy Scan
      ↓
Docker Push
```

This is useful because the image is:

```text
Built once
Scanned as the same image
Published after scanning
```

---

# 53. WHY BUILD ONCE AND SCAN THE SAME IMAGE?

Imagine:

```text
Build Image A
 ↓
Scan Image A

Build Image B
 ↓
Push Image B
```

Now the scanned image and pushed image could differ.

Better:

```text
Build Image
 ↓
Save exact image
 ↓
Scan exact image
 ↓
Push exact image
```

This improves traceability and consistency.

---

# 54. DOCKER COMPOSE NETWORKING — NODEGOAT

NodeGoat and MongoDB run in separate containers.

```text
web
 ↓
mongo
```

Connection:

```text
mongodb://mongo:27017/nodegoat
```

Not:

```text
mongodb://localhost:27017/nodegoat
```

Why?

Inside a container:

```text
localhost
→ This same container
```

Docker Compose service name:

```text
mongo
→ MongoDB container
```

---

# 55. NODEGOAT `localhost` ERROR

### Problem

Application tried:

```text
mongodb://localhost:27017/nodegoat
```

### Why it failed

The NodeGoat container was looking for MongoDB inside itself.

### Fix

```text
mongodb://mongo:27017/nodegoat
```

because Docker Compose provides internal DNS.

### Interview Answer

> "The application and MongoDB were running in different containers, so localhost pointed to the NodeGoat container itself. I used the Compose service name `mongo`, which Docker resolves through its internal DNS."

---

# 56. NODEGOAT APPLICATION

Project:

```text
OWASP NodeGoat
```

Application:

```text
Node.js
Express
MongoDB
```

Port:

```text
4000
```

Start:

```bash
npm start
```

MongoDB:

```text
mongodb://mongo:27017/nodegoat
```

---

# 57. WHY NODEGOAT?

NodeGoat is intentionally vulnerable.

That makes it useful for learning:

```text
Secure coding
SAST
Secret scanning
Dependency scanning
Docker security
Image scanning
DAST
```

Important:

> Vulnerabilities in NodeGoat are expected as part of the training scenario; the goal is to detect and understand them.

---

# 58. SECURITY AT MULTIPLE LAYERS

A vulnerability can exist in different places:

```text
Application Code
       ↓
Dependencies
       ↓
Dockerfile
       ↓
Base Image
       ↓
Container
       ↓
Running Application
```

Therefore one security tool is not enough for broad coverage.

---

# 59. FINAL NODEGOAT SECURITY TOOL MAP

```text
JSHint
 ↓
Code Quality

Semgrep
 ↓
SAST

Gitleaks
 ↓
Secrets

npm audit
 ↓
Dependencies

Hadolint
 ↓
Dockerfile

Trivy
 ↓
Container Image

OWASP ZAP
 ↓
DAST
```

---

# 60. PARALLEL SECURITY JOBS

Independent checks do not need to wait for each other.

Example:

```yaml
sast:
  needs: [lint]

secret-scan:
  needs: [lint]

dependency-scan:
  needs: [lint]

dockerfile-lint:
  needs: [lint]
```

Flow:

```text
             ┌── SAST
             │
             ├── Secret Scan
Lint ────────┼── Dependency Scan
             │
             └── Dockerfile Lint
```

Then:

```text
Security Jobs
      ↓
Docker Build
```

### Why?

Parallel execution reduces pipeline runtime.

Observed NodeGoat pipeline improvement:

```text
Before → ~7m 43s
After  → ~6m 29s
```

Approximately:

```text
1m 14s improvement
```

---

# 61. REUSABLE NODEGOAT WORKFLOW STRUCTURE

The capstone separated the pipeline into logical reusable workflows.

```text
main.yml
lint.yml
test.yml
sast.yml
secret-scan.yml
dependency-scan.yml
dockerfile-lint.yml
docker-build.yml
image-scan.yml
docker-push.yml
deploy-ec2.yml
health-check.yml
zap-dast.yml
```

Caller:

```yaml
jobs:
  sast:
    needs: [lint]
    uses: ./.github/workflows/sast.yml
```

This makes the main pipeline easier to read.

---

# 62. NODEGOAT PIPELINE

```text
Push / Pull Request
        ↓
      Lint
        ↓
 ┌──────┼────────┬───────────────┐
 ↓      ↓        ↓               ↓
Test   SAST   Secret Scan   Dependency Scan
                     │
                     └──────┬──────────┘
                            ↓
                     Dockerfile Lint
                            ↓
                       Docker Build
                            ↓
                      Image Scan
                            ↓
                       Docker Hub
                            ↓
                        AWS EC2
                            ↓
                      Health Check
                            ↓
                       OWASP ZAP
```

---

# 63. PULL REQUEST VS PUSH

A common DevSecOps design:

```text
Pull Request
   ↓
Build
   ↓
Test
   ↓
Security Checks
   ↓
Validation
   ↓
No Production Deployment
```

After merge/push to main:

```text
Push to main
   ↓
Build
   ↓
Security Checks
   ↓
Docker Build
   ↓
Image Scan
   ↓
Docker Hub
   ↓
EC2
```

Example deployment condition:

```yaml
if: github.event_name == 'push' &&
    github.ref == 'refs/heads/master'
```

---

# 64. GITHUB ACTIONS PERMISSIONS

Least privilege means giving workflows only the permissions they need.

Example:

```yaml
permissions:
  contents: read
```

If a workflow needs PR comments:

```yaml
permissions:
  contents: read
  pull-requests: write
```

### Why?

Suppose a third-party action is compromised.

Broad permissions:

```text
Compromised Action
      ↓
More GitHub access
      ↓
Greater impact
```

Restricted permissions:

```text
Compromised Action
      ↓
Limited permissions
      ↓
Smaller impact
```

### Easy Memory

```text
Least privilege
→ Minimum required access
```

---

# 65. SECURITY SCANNING VS SECURITY GATE

Very important interview distinction.

## Scanning

```text
Find the problem
```

## Gate

```text
Decide whether pipeline continues
```

Example:

```text
Trivy
 ↓
Find HIGH vulnerability
 ↓
exit-code 1
 ↓
Job fails
 ↓
Docker Push does not run
```

So:

```text
Scanner = Detection

Gate = Enforcement
```

---

# 66. REPORT-ONLY VS BLOCKING

## Report-only

```text
Finding
 ↓
Report
 ↓
Continue
```

## Blocking

```text
Finding
 ↓
Policy threshold reached
 ↓
Fail job
 ↓
Stop deployment
```

NodeGoat learning environment:

```text
Mostly report-oriented
```

Typical production policy:

```text
Critical → Block
High     → Possibly block
Medium   → Depending on policy
Low      → Usually report
```

The exact policy should be defined by the organization.

---

# 67. NODEGOAT SAST RESULTS

Observed Semgrep findings:

```text
WARNING: 25
ERROR:    7
INFO:     1
```

Examples:

```text
Dangerous eval()
Private key material
Open redirect-related findings
Other insecure coding patterns
```

Important:

```text
NodeGoat is intentionally vulnerable
```

so findings are expected.

---

# 68. NODEGOAT ZAP

OWASP ZAP:

```text
OWASP Zed Attack Proxy
```

Purpose:

```text
Dynamic web application security testing
```

Target:

```text
Running NodeGoat
```

Example:

```yaml
uses: zaproxy/action-baseline@v0.15.0

with:
  target: http://${{ secrets.EC2_HOST }}:4000
```

Because the application is running, this is DAST.

---

# 69. WHY ZAP COMES AFTER DEPLOYMENT

DAST needs a live application.

Therefore:

```text
Build
 ↓
Deploy
 ↓
Application Running
 ↓
ZAP
```

Not:

```text
Source Code
 ↓
ZAP
```

That would be the wrong mental model for DAST.

---

# 70. ZAP FULL SCAN VS BASELINE

A full ZAP scan was initially tested.

Observed:

```text
42+ minutes
```

That was too slow for a regular CI/CD pipeline.

The pipeline was switched to:

```yaml
zaproxy/action-baseline@v0.15.0
```

Observed:

```text
~7 minutes
```

### Interview Answer

> "I initially tested a full ZAP scan, but it was too slow for regular CI/CD. I switched to a baseline scan so that dynamic security checks could run more practically within the pipeline."

---

# 71. ZAP FINDINGS

The scan identified observations including:

```text
Missing security headers
Directory browsing
Vulnerable JavaScript libraries
Cross-domain JavaScript concerns
XSS-related findings
Other web security observations
```

Important:

> A finding is an observation from the scanner; whether it is release-blocking depends on the security policy.

---

# 72. ZAP 403 ERROR

The ZAP scan completed, but the workflow later failed while trying to create a GitHub issue.

Error:

```text
403 Resource not accessible by integration
```

Flow:

```text
ZAP Scan
   ↓
Completed
   ↓
Create GitHub Issue
   ↓
Permission denied
   ↓
HTTP 403
```

Important interview lesson:

```text
Scan failure
≠
Post-scan automation failure
```

The scan had completed.

The issue was with GitHub integration permissions.

---

# 73. NODEGOAT DEPLOYMENT

Final flow:

```text
GitHub Actions
      ↓
Docker Image
      ↓
Docker Hub
      ↓
AWS EC2
      ↓
Docker Compose
      ↓
NodeGoat + MongoDB
```

Deployment used:

```text
SSH
Docker Compose
Environment variables
Docker Hub authentication
```

---

# 74. APPLICATION HEALTH CHECK

After deployment:

```text
EC2
 ↓
NodeGoat running
 ↓
HTTP request
 ↓
Application responds
```

Example:

```bash
curl -fsS http://${EC2_HOST}:4000/login > /dev/null
```

Health verification answers:

```text
Did deployment actually start the application?
```

---

# 75. HTTP 302 IN NODEGOAT

The deployed application responded to:

```text
/login
```

with:

```text
HTTP 302
```

A `302` indicates a redirect.

Important:

```text
Application is reachable
```

does not necessarily mean:

```text
Application is vulnerability-free
```

Health and security are separate checks.

---

# 76. IMAGE TRACEABILITY IN NODEGOAT

Image tag:

```text
<username>/nodegoat:<Git SHA>
```

and:

```text
<username>/nodegoat:latest
```

Flow:

```text
Git Commit
   ↓
Step Output
   ↓
Job Output
   ↓
Reusable Workflow Output
   ↓
Docker Push
   ↓
EC2 Deployment
```

The SHA tag lets you identify exactly which commit produced the deployed image.

---

# 77. `id` + `GITHUB_OUTPUT` + `needs` — COMPLETE EXAMPLE

This is a very useful interview example.

Step:

```yaml
- name: Generate image tag
  id: meta
  run: |
    IMAGE_TAG="${GITHUB_SHA}"
    echo "image-tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

Job output:

```yaml
outputs:
  image_tag: ${{ steps.meta.outputs.image-tag }}
```

Another job:

```yaml
deploy:
  needs: docker-build
  runs-on: ubuntu-latest

  steps:
    - run: echo "${{ needs.docker-build.outputs.image_tag }}"
```

Memory:

```text
id
 ↓
GITHUB_OUTPUT
 ↓
steps.<id>.outputs
 ↓
jobs.<job>.outputs
 ↓
needs.<job>.outputs
```

---

# 78. NODEGOAT `.ENV` / SECRET CONCEPT

Application configuration should not hardcode secrets.

Instead of:

```javascript
const apiKey = "actual-secret";
```

use:

```javascript
process.env.ZAP_API_KEY || ""
```

and pass the value securely through the environment.

The important DevSecOps rule:

```text
Code
→ No credentials

Environment / Secret Store
→ Sensitive values
```

---

# 79. DOCKER SECURITY IMPROVEMENTS IN NODEGOAT

The final Dockerfile included concepts such as:

```text
Alpine-based image
Multi-stage build
Production dependencies
.dockerignore
Non-root runtime user
Required application port only
```

Running as a non-root user reduces the impact of some container compromises.

Example:

```dockerfile
USER $USER
```

with the runtime user set to:

```text
node
```

---

# 80. IMAGE SECURITY VS DOCKERFILE SECURITY

These are different.

## Dockerfile Security

```text
Dockerfile
   ↓
Hadolint
```

Checks:

```text
Best practices
Common mistakes
Dockerfile issues
```

## Image Security

```text
Built Image
   ↓
Trivy
```

Checks:

```text
OS packages
Libraries
Known vulnerabilities
```

### Easy Memory

```text
Hadolint
→ Dockerfile

Trivy
→ Image
```

---

# 81. CODE QUALITY VS SAST

Also different.

## Code Quality

Example:

```text
JSHint
```

Looks for:

```text
Style
Syntax
Code-quality issues
```

## SAST

Example:

```text
Semgrep
```

Looks for:

```text
Security-related coding patterns
```

### Easy Memory

```text
JSHint
→ Code quality

Semgrep
→ Security
```

---

# 82. COMPLETE NODEGOAT SECURITY MATRIX

| Stage | Tool | What it checks |
|---|---|---|
| Code Quality | JSHint | JavaScript quality |
| SAST | Semgrep | Source-code security |
| Secret Scan | Gitleaks | Leaked secrets |
| Dependency Scan | npm audit | Vulnerable dependencies |
| Dockerfile | Hadolint | Dockerfile practices |
| Image Scan | Trivy | Image vulnerabilities |
| DAST | OWASP ZAP | Running web application |

---

# 83. DEVSECOPS PIPELINE — FINAL MEMORY DIAGRAM

```text
                     Git Push / Pull Request
                              │
                              ▼
                             Lint
                              │
       ┌──────────────────────┼────────────────────────┐
       │                      │                        │
       ▼                      ▼                        ▼
     Test                   SAST                Secret Scan
       │                      │                        │
       └───────────────┬──────┴──────────────┬─────────┘
                       │                     │
                       ▼                     ▼
                 Dependency Scan      Dockerfile Lint
                       │                     │
                       └──────────┬──────────┘
                                  ▼
                            Docker Build
                                  │
                                  ▼
                              Trivy Scan
                                  │
                                  ▼
                             Docker Hub
                                  │
                                  ▼
                              AWS EC2
                                  │
                           Docker Compose
                                  │
                      ┌───────────┴───────────┐
                      ▼                       ▼
                  NodeGoat                MongoDB
                      │
                      ▼
                 Health Check
                      │
                      ▼
                 OWASP ZAP
                    DAST
```

---

# 84. DAY 48 + DAY 49 COMBINED STORY

Your learning progression can be remembered like this:

```text
YAML
 ↓
CI/CD Fundamentals
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
Conditions
 ↓
Secrets
 ↓
Artifacts
 ↓
Caching
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
Secret Scanning
 ↓
Dependency Scanning
 ↓
Dockerfile Scanning
 ↓
Image Scanning
 ↓
Docker Hub
 ↓
EC2
 ↓
Health Check
 ↓
DAST
```

---

# 85. MOST IMPORTANT INTERVIEW QUESTIONS

## What is CI?

> "Continuous Integration means automatically building and testing code whenever changes are integrated into the shared repository."

---

## What is CD?

> "Continuous Delivery or Deployment automates the process of taking validated code toward a release or deployment environment."

---

## What is DevSecOps?

> "DevSecOps integrates security into the DevOps lifecycle so security checks are automated as part of CI/CD rather than being treated only as a final manual activity."

---

## What is a security gate?

> "A security gate is a policy or pipeline rule that determines whether the pipeline can continue based on security results. For example, a Trivy scan can fail the job when HIGH or CRITICAL vulnerabilities are detected."

---

## What is SAST?

> "SAST analyzes source code without running the application. I used Semgrep in the NodeGoat project."

---

## What is DAST?

> "DAST tests a running application. I used OWASP ZAP against the deployed NodeGoat application."

---

## Why use both SAST and DAST?

> "They examine different layers. SAST analyzes source code, while DAST tests the running application."

---

## Why use Gitleaks?

> "To detect accidentally committed secrets such as API keys, tokens and private keys."

---

## Why use `fetch-depth: 0`?

> "To make the complete Git history available so secret scanning can inspect previous commits as well as the current source."

---

## Why use npm audit?

> "To identify known vulnerabilities in npm dependencies."

---

## Why use Hadolint?

> "To detect common Dockerfile issues and Docker best-practice violations."

---

## Why use Trivy?

> "To scan container images for known vulnerabilities in OS packages and application dependencies."

---

## Why use ZAP?

> "ZAP performs dynamic security testing against the running web application."

---

## Why use `needs`?

> "To control job dependencies and ensure that a job runs only after required jobs complete successfully."

---

## What is `workflow_call`?

> "It allows one workflow to be reused by another workflow."

---

## What is `workflow_run`?

> "It triggers a workflow after another workflow has completed."

---

## What is `repository_dispatch`?

> "It allows an external system to send an event that triggers a GitHub Actions workflow."

---

## What are inputs?

> "Inputs are values passed into a reusable workflow, typically using the `with` block."

Example:

```yaml
with:
  image-tag: sha-c49c556
```

---

## What are outputs?

> "Outputs are values produced by a step, job or reusable workflow and consumed later in the pipeline."

---

## How do you pass a value between jobs?

> "I create a step output using `$GITHUB_OUTPUT`, expose it as a job output, and then consume it using `needs.<job>.outputs.<name>`."

---

## Why use artifacts?

> "Artifacts allow workflow-generated files to be stored and shared between jobs. In NodeGoat I used an artifact to transfer the Docker image TAR between build and scanning stages."

---

## Why not use only `latest`?

> "`latest` is mutable. A Git SHA tag gives an exact relationship between the Docker image and the source commit."

---

## Why run security scans in parallel?

> "Independent scans can run concurrently, reducing total pipeline execution time."

---

## Why use reusable workflows?

> "To split a large pipeline into reusable logical stages and avoid duplicating CI/CD logic."

---

## What is the difference between a reusable workflow and a composite action?

> "A reusable workflow reuses jobs or complete workflow logic, while a composite action packages reusable steps."

---

# 86. NODEGOAT ERROR QUESTIONS

## Why did MongoDB fail with localhost?

> "Because localhost inside the NodeGoat container referred to that same container rather than the MongoDB container. I changed the connection to the Compose service name `mongo`."

---

## Why were there so many dependency vulnerabilities?

> "NodeGoat uses an intentionally old and vulnerable dependency tree because it is a security training application."

---

## Why didn't you blindly run `npm audit fix --force`?

> "Because aggressive dependency upgrades can introduce breaking changes, especially in an intentionally old application. I wanted to preserve the application while using the audit to detect and report vulnerabilities."

---

## Why did Trivy not fail the NodeGoat pipeline?

> "Because the project intentionally contains vulnerabilities. I configured the scan in report-oriented mode so the pipeline could demonstrate the complete lifecycle. In production I would configure a security gate with a defined severity policy."

---

## What was the ZAP 403 error?

> "The ZAP scan itself completed. The failure happened afterward when the workflow attempted to create a GitHub issue and the GitHub integration returned `403 Resource not accessible by integration`."

---

# 87. THE MOST IMPORTANT SECURITY CONCEPT

Remember this chain:

```text
Scanner
   ↓
Detect
   ↓
Result
   ↓
Policy
   ↓
Security Gate
   ↓
Allow / Block
```

Example:

```text
Trivy
 ↓
HIGH vulnerability
 ↓
Policy says HIGH blocks release
 ↓
exit-code 1
 ↓
Pipeline fails
 ↓
No deployment
```

This is much better than simply saying:

> "Trivy scans Docker."

---

# 88. THE MOST IMPORTANT INPUT/OUTPUT CONCEPT

Remember:

```text
Caller
   ↓
with:
   ↓
inputs
   ↓
Reusable Workflow
   ↓
Step
   ↓
GITHUB_OUTPUT
   ↓
Step Output
   ↓
Job Output
   ↓
Workflow Output
   ↓
Caller
```

Example:

```text
Git SHA
 ↓
Generate short SHA
 ↓
Output
 ↓
Docker tag
 ↓
Push
 ↓
Deploy same version
```

This is one of the strongest GitHub Actions concepts to explain in an interview.

---

# 89. THE MOST IMPORTANT DEVSECOPS CONCEPT

Do not memorize only:

```text
Semgrep
Gitleaks
Trivy
ZAP
```

Remember **what layer each tool protects**:

```text
Code
 ↓
Semgrep
 ↓
Source security

Git History
 ↓
Gitleaks
 ↓
Secrets

Dependencies
 ↓
npm audit
 ↓
Known vulnerabilities

Dockerfile
 ↓
Hadolint
 ↓
Container build practices

Image
 ↓
Trivy
 ↓
Image vulnerabilities

Running Application
 ↓
ZAP
 ↓
Runtime/web security
```

---

# 90. 30-SECOND DEVSECOPS ANSWER

> "DevSecOps means integrating security into the CI/CD pipeline. In my NodeGoat project I added multiple layers: Semgrep for SAST, Gitleaks for secret detection, npm audit for dependency vulnerabilities, Hadolint for Dockerfile checks, Trivy for container image scanning and OWASP ZAP for DAST. The pipeline builds the image, scans it, publishes the approved image, deploys it to EC2, verifies the application and then performs dynamic testing."

---

# 91. 60-SECOND NODEGOAT PROJECT ANSWER

> "I built a DevSecOps pipeline around OWASP NodeGoat, an intentionally vulnerable Node.js application using MongoDB. I used GitHub Actions and reusable workflows to separate linting, testing, SAST, secret scanning, dependency scanning, Dockerfile linting, Docker build, image scanning, deployment and DAST.
>
> The Docker image is built once, saved as an artifact and then loaded for Trivy scanning so the image being scanned is the same image that gets published. The image is tagged using the Git commit SHA and pushed to Docker Hub.
>
> After that, the exact image is deployed to AWS EC2 using Docker Compose. The pipeline verifies the application and then runs OWASP ZAP against the live application.
>
> One issue I solved was the MongoDB connection using localhost. Since MongoDB was in another container, I changed it to the Compose service name `mongo`. I also handled secret-detection findings and a GitHub 403 integration error after the ZAP scan."

---

# 92. FINAL QUICK REVISION TABLE

| Concept | Remember |
|---|---|
| YAML | Configuration |
| CI | Build + Test |
| CD | Deliver + Deploy |
| Workflow | Complete automation |
| Job | Group of steps |
| Step | Individual task |
| Runner | Execution machine |
| `needs` | Job dependency |
| `if` | Condition |
| `failure()` | Previous failure |
| `always()` | Run regardless |
| Secret | Sensitive data |
| Artifact | Stored workflow file |
| Cache | Reuse dependencies |
| Matrix | Multiple configurations |
| Input | Data entering reusable workflow |
| Output | Data leaving reusable workflow |
| `id` | Step identifier |
| `$GITHUB_OUTPUT` | Create step output |
| `workflow_call` | Reuse workflow |
| Composite Action | Reuse steps |
| `workflow_run` | Trigger after workflow |
| `repository_dispatch` | External trigger |
| SAST | Source-code security |
| DAST | Running-app security |
| Gitleaks | Secrets |
| npm audit | Dependencies |
| Hadolint | Dockerfile |
| Trivy | Image vulnerabilities |
| ZAP | Web DAST |
| Security Gate | Decide allow/block |
| Docker Hub | Image registry |
| EC2 | Deployment server |
| `mongo` | Docker Compose service |
| `localhost` | Same container |

---

# 93. FINAL MEMORY FLOW

```text
                GITHUB ACTIONS
                      │
        ┌─────────────┴─────────────┐
        │                           │
      INPUTS                     OUTPUTS
        │                           │
   Data enters                 Data leaves
        │                           │
        └─────────────┬─────────────┘
                      │
                    JOBS
                      │
                    needs
                      │
                  CONDITIONS
                      │
                    DOCKER
                      │
               ┌──────┴──────┐
               │             │
            Build          Image
                           Scan
                             │
                          TRIVY
                             │
                        SECURITY GATE
                             │
                        Push / Block
                             │
                            EC2
                             │
                         HEALTH
                             │
                           ZAP
                             │
                           DAST
```

---

# 🏁 FINAL INTERVIEW FORMULA

For almost every DevOps/DevSecOps question, think:

```text
WHAT?
→ What is it?

WHY?
→ Why do we use it?

HOW?
→ How did I implement it?

PROBLEM?
→ What problem did I face?

FIX?
→ How did I solve it?

RESULT?
→ What did the pipeline achieve?
```

### Example — Trivy

```text
WHAT?
→ Container image vulnerability scanner

WHY?
→ Detect known vulnerabilities

HOW?
→ Scan nodegoat:ci after Docker build

PROBLEM?
→ NodeGoat intentionally contains vulnerabilities

FIX?
→ Use report-oriented mode for learning

SECURITY GATE?
→ Production can use exit-code 1 for selected severities

RESULT?
→ Vulnerabilities are detected before release/deployment
```

### Example — GitHub Actions Outputs

```text
WHAT?
→ Values passed from one stage to another

WHY?
→ Share dynamic information

HOW?
→ id + GITHUB_OUTPUT + job/workflow outputs

EXAMPLE?
→ Git SHA → image tag → Docker push → EC2 deployment
```

### Example — MongoDB

```text
WHAT?
→ Container-to-container communication

PROBLEM?
→ localhost

WHY?
→ localhost means the current container

FIX?
→ mongodb://mongo:27017/nodegoat

RESULT?
→ NodeGoat communicates with MongoDB through Docker Compose DNS
```

---

# 🔥 MASTER ONE-LINER

> **"I learned to build a CI/CD pipeline with GitHub Actions and then extend it into DevSecOps by adding automated security checks at the source-code, dependency, secret, Dockerfile, container-image and running-application layers, with security gates controlling whether the pipeline can continue to deployment."**
