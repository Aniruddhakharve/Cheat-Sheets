# 🚀 GitHub Actions & CI/CD — Master Interview Cheat Sheet

## Day 38 → Day 49 | YAML → CI/CD → GitHub Actions → Docker CI/CD → Reusable Workflows → DevSecOps → AWS OIDC Deployment

> **Purpose:** This is my master revision sheet for everything I learned from Day 38 to Day 49.
>
> The goal is not to memorize commands blindly.
>
> For every concept, I should be able to answer:
>
> ```text
> WHAT is it?
> WHY do we use it?
> HOW does it work?
> HOW did I use it in my project?
> WHAT problem did I face?
> HOW did I solve it?
> ```
>
> **Primary final project reference:**
>
> `secure-cicd-pipeline-lab`
>
> GitHub:
>
> `https://github.com/Aniruddhakharve/secure-cicd-pipeline-lab`
>
> Docker Hub:
>
> `aniruddhakharve/secure-cicd-pipeline-lab`
>
> The older OWASP NodeGoat project was also used during the DevSecOps learning process, so some examples below refer to NodeGoat where useful. However, the **final Day 48–49 production-style architecture described in this sheet is based on my Secure CI/CD Pipeline Lab**.

---

# 🧠 MASTER MENTAL MODEL

Before remembering individual GitHub Actions keywords, remember the whole journey:

```text
YAML
  ↓
CI/CD
  ↓
GitHub Actions
  ↓
Workflow
  ↓
Triggers
  ↓
Jobs
  ↓
Steps
  ↓
Runner
  ↓
Conditions
  ↓
Environment Variables
  ↓
Inputs / Outputs
  ↓
Secrets
  ↓
Artifacts
  ↓
Cache
  ↓
Matrix
  ↓
Reusable Workflows
  ↓
Composite Actions
  ↓
Docker CI/CD
  ↓
Git SHA Image Tagging
  ↓
Pull Request Pipeline
  ↓
Main Pipeline
  ↓
Production Environment Approval
  ↓
DevSecOps
  ↓
SAST
  ↓
Secret Scanning
  ↓
Dependency Scanning
  ↓
Dependency Review
  ↓
Dockerfile Linting
  ↓
Trivy Image Scanning
  ↓
Security Gates
  ↓
OIDC
  ↓
AWS STS
  ↓
IAM
  ↓
AWS SSM
  ↓
Secrets Manager
  ↓
EC2
  ↓
Docker Compose
  ↓
Health Check
```

---

# 1. YAML BASICS

## What is YAML?

**YAML = YAML Ain't Markup Language**

YAML is a human-readable, indentation-sensitive configuration format.

DevOps tools commonly use YAML:

```text
GitHub Actions
Docker Compose
Kubernetes
Ansible
CI/CD configuration
```

### Basic structure

```yaml
name: My Workflow

services:
  web:
    image: nginx
    ports:
      - "8080:80"
```

### Important YAML rules

```text
Indentation matters
Use spaces, not tabs
key: value
Lists use -
Nested objects use indentation
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

### Interview answer

> "YAML is a human-readable and indentation-sensitive configuration format. In DevOps, it is commonly used for GitHub Actions, Docker Compose, Kubernetes and Ansible."

---

# 2. CI/CD FUNDAMENTALS

## What is CI?

**CI = Continuous Integration**

The idea is that developers frequently integrate code into a shared repository and automated checks validate the change.

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

Instead of:

```text
Developer writes code
       ↓
Weeks pass
       ↓
Integration
       ↓
Everything breaks
```

we want:

```text
Small change
   ↓
Automated validation
   ↓
Fast feedback
```

---

# 3. CONTINUOUS DELIVERY

Continuous Delivery means the software is continuously built, tested and kept ready for release.

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

Deployment may still require a manual approval.

---

# 4. CONTINUOUS DEPLOYMENT

Continuous Deployment means successful changes are automatically deployed.

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

# 5. CI VS CD

| CI | CD |
|---|---|
| Integrates code | Delivers/deploys code |
| Build | Package |
| Test | Release |
| Quality checks | Deployment |
| Fast feedback | Automated delivery |

### Interview answer

> "CI focuses on automatically building and testing code whenever changes are integrated. CD takes that validated software toward release or deployment."

---

# 6. GITHUB ACTIONS

## What is GitHub Actions?

GitHub Actions is GitHub's automation and CI/CD platform.

The basic hierarchy is:

```text
Workflow
   ↓
Jobs
   ↓
Steps
   ↓
Runner
```

Example:

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

# 7. WORKFLOW

A **workflow** is the complete GitHub Actions YAML file.

Usually stored in:

```text
.github/workflows/
```

Example:

```text
.github/workflows/main-pipeline.yml
```

A workflow defines:

```text
When should it run?
What jobs should run?
What runner should execute them?
What steps should be performed?
What permissions are needed?
```

---

# 8. TRIGGERS

Triggers answer:

> **"When should GitHub Actions start this workflow?"**

---

## `push`

```yaml
on:
  push:
```

Runs when code is pushed.

---

## Push only to `main`

```yaml
on:
  push:
    branches:
      - main
```

Flow:

```text
Push to main
    ↓
Workflow runs
```

A push to another branch does not trigger that workflow.

---

## `pull_request`

```yaml
on:
  pull_request:
    branches:
      - main
```

Runs for pull request activity involving `main`.

---

## `workflow_dispatch`

Manual execution.

```yaml
on:
  workflow_dispatch:
```

Useful for:

```text
Manual testing
Manual deployment
Troubleshooting
Re-running a controlled process
```

---

## `schedule`

Runs according to a cron expression.

```yaml
on:
  schedule:
    - cron: '0 */12 * * *'
```

---

## Pull Request event types

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened
```

Common meanings:

```text
opened
→ PR created

synchronize
→ New commits pushed to PR

reopened
→ Closed PR opened again
```

---

# 9. BRANCH FILTERS

```yaml
on:
  push:
    branches:
      - main
      - 'release/**'
```

Only matching branches trigger the workflow.

---

# 10. PATH FILTERS

```yaml
on:
  push:
    paths:
      - 'app/**'
      - '.github/workflows/**'
```

The workflow runs only when matching paths change.

---

## `paths-ignore`

```yaml
on:
  push:
    paths-ignore:
      - '*.md'
      - 'docs/**'
```

Useful when documentation-only changes should not consume CI resources.

---

# 11. WORKFLOW VS JOB VS STEP

This is one of the most important GitHub Actions hierarchies.

```text
Workflow
   ↓
Job
   ↓
Step
   ↓
Command / Action
```

### Workflow

Complete YAML automation.

### Job

A logical unit executed on a runner.

### Step

One operation inside a job.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: docker build -t app .
```

Mental model:

```text
Workflow
  └── build job
       ├── Checkout step
       └── Build step
```

---

# 12. RUNNERS

A runner is the machine that executes the job.

```text
Workflow
   ↓
Runner
   ↓
Jobs execute
```

Common GitHub-hosted runners:

```text
ubuntu-latest
windows-latest
macos-latest
```

Self-hosted:

```text
GitHub Actions
      ↓
My own server / VM
```

### Interview answer

> "A runner is the execution environment where GitHub Actions runs the job's commands and actions. It can be GitHub-hosted or self-hosted."

---

# 13. `run` VS `uses`

## `run`

Executes a shell command.

```yaml
- name: Show Python version
  run: python --version
```

## `uses`

Uses an existing GitHub Action.

```yaml
- name: Checkout code
  uses: actions/checkout@v4
```

Easy memory:

```text
run
→ Run my command

uses
→ Use an existing action
```

---

# 14. `needs` — JOB DEPENDENCY

`needs` creates an execution dependency between jobs.

Example:

```yaml
test:
  needs: build
```

Meaning:

```text
build
  ↓
test
```

`test` waits for `build`.

---

## Multiple dependencies

```yaml
docker-build:
  needs:
    - lint
    - build-test
    - sast
    - secret-scan
```

Meaning:

```text
lint ─────────┐
build-test ───┤
SAST ─────────┤
secret-scan ──┤
              ↓
          Docker Build
```

---

## Why we used it in the final project

Our image scan must happen after the image exists:

```text
docker-build
     ↓
trivy-scan
```

Then Docker Push must wait for the scan:

```text
docker-build
     ↓
trivy-scan
     ↓
docker-push
```

This is the actual pipeline control.

### Interview answer

> "`needs` defines job dependencies. In my DevSecOps pipeline I used it to ensure that security validation and Docker image scanning happen before the image can be pushed or deployed."

---

# 15. PARALLEL JOBS

Not every security check needs to wait for every other security check.

Example:

```yaml
sast:
  needs: lint

secret-scan:
  needs: lint

dependency-scan:
  needs: lint

dockerfile-lint:
  needs: lint
```

The flow becomes:

```text
             ┌── SAST
             │
             ├── Secret Scan
Lint ────────┼── Dependency Scan
             │
             └── Dockerfile Lint
```

After those finish:

```text
Security checks
      ↓
Docker Build
```

### Why parallelize?

Because independent jobs can run simultaneously.

```text
Sequential:

A → B → C → D
         slower

Parallel:

      ┌→ B
A ────┼→ C
      └→ D
         faster
```

---

# 16. ENVIRONMENT VARIABLES — IMPORTANT

`env:` defines environment variables.

This topic has **three important levels**.

```text
Workflow-level
      ↓
Job-level
      ↓
Step-level
```

The lower level can override the higher-level value.

---

# 17. WORKFLOW-LEVEL `env:`

Available broadly to jobs/steps in the workflow.

```yaml
name: CI

env:
  APP_NAME: secure-cicd-pipeline-lab
  APP_ENV: production

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - run: echo "$APP_NAME"
```

Mental model:

```text
Workflow
   │
   ├── Job A
   │     └── steps
   │
   └── Job B
         └── steps

Workflow env
   ↓
available to them
```

Use it for values that are genuinely common to the whole workflow.

---

# 18. JOB-LEVEL `env:`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    env:
      ENVIRONMENT: production

    steps:
      - run: echo "$ENVIRONMENT"
```

This is available to steps inside that job.

Mental model:

```text
Workflow
   ↓
Build Job
   ↓
Build steps

Job env
→ Available to Build Job steps
```

It does not automatically become a universal value for unrelated jobs.

---

# 19. STEP-LEVEL `env:`

```yaml
steps:

  - name: Build
    env:
      IMAGE_TAG: ${{ github.sha }}
    run: |
      echo "$IMAGE_TAG"
```

This variable is specifically provided to that step.

Mental model:

```text
Workflow
   ↓
Job
   ↓
Specific Step
      ↓
Step env
```

---

# 20. `env:` LEVEL PRIORITY

Think:

```text
Workflow env
      ↓
Job env
      ↓
Step env
```

Example:

```yaml
env:
  APP_ENV: global

jobs:
  build:
    env:
      APP_ENV: job

    steps:

      - name: Test
        env:
          APP_ENV: step
        run: echo "$APP_ENV"
```

The output is:

```text
step
```

because the most specific definition wins.

### Easy memory

```text
Broad
 ↓
Workflow

More specific
 ↓
Job

Most specific
 ↓
Step
```

---

# 21. `env:` VS `vars` VS `secrets`

These are easy to confuse.

## `env`

Environment variables made available to commands.

```yaml
env:
  APP_ENV: production
```

## `vars`

Repository / organization / environment configuration variables.

Example:

```yaml
${{ vars.DOCKER_USERNAME }}
```

Useful for non-sensitive configuration.

## `secrets`

Sensitive values.

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

Easy memory:

```text
env
→ Runtime environment variable

vars
→ Configuration value

secrets
→ Sensitive value
```

---

# 22. NEVER PUT SECRETS IN `env` AS HARDCODED VALUES

Bad:

```yaml
env:
  DB_PASSWORD: my-real-password
```

Better:

```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

Or for production runtime:

```text
AWS Secrets Manager
       ↓
EC2
       ↓
.env
       ↓
Docker Compose
```

---

# 23. GITHUB CONTEXT VARIABLES

GitHub automatically provides information about the workflow.

Examples:

```text
github.sha
github.ref
github.event_name
github.repository
github.actor
```

Example:

```yaml
run: echo "${{ github.sha }}"
```

---

# 24. `github.sha`

`github.sha` identifies the commit associated with the workflow run.

In our final project we use:

```yaml
image_tag: ${{ github.sha }}
```

This becomes:

```text
Docker image tag = exact Git commit SHA
```

Example concept:

```text
Git Commit
   ↓
abc123...
   ↓
Docker Image
   ↓
aniruddhakharve/secure-cicd-pipeline-lab:abc123...
```

---

# 25. CONDITIONS — `if:`

`if:` controls whether something should execute.

Example:

```yaml
if: github.ref == 'refs/heads/main'
```

Meaning:

```text
Is this main branch?
       ↓
     YES
       ↓
Run
```

---

# 26. `always()`

```yaml
if: ${{ always() }}
```

Runs even when earlier steps/jobs fail, subject to the job dependency context.

Useful for:

```text
Final reporting
Cleanup
Result collection
```

Our final deployment workflow uses the concept when determining the final deployment result.

---

# 27. `failure()`

```yaml
if: ${{ failure() }}
```

Useful when you want something to happen after a failure.

Example:

```yaml
- name: Collect failure logs
  if: ${{ failure() }}
  run: ./collect-logs.sh
```

---

# 28. `success()`

```yaml
if: ${{ success() }}
```

Useful when you explicitly want a step to run only after preceding steps succeeded.

---

# 29. `continue-on-error`

```yaml
continue-on-error: true
```

This means:

```text
Step fails
   ↓
Do not immediately stop the job
```

But understand the difference:

```text
continue-on-error
→ tolerate this step's failure

Security gate
→ intentionally fail pipeline when policy is violated
```

Do not use `continue-on-error: true` blindly for security-critical checks.

---

# 30. OUTPUTS — DEEP UNDERSTANDING

This is one of the most important topics from our GitHub Actions work.

An **output** is a value generated by one part of the workflow and made available to another part.

Think:

```text
I calculated something
       ↓
I need to pass it forward
       ↓
Use an output
```

---

# 31. STEP OUTPUT

Example:

```yaml
- name: Generate image tag
  id: meta
  run: |
    IMAGE_TAG="${GITHUB_SHA}"
    echo "image-tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

Here:

```text
id: meta
```

gives the step a name.

Then:

```text
$GITHUB_OUTPUT
```

stores the output.

Output name:

```text
image-tag
```

Value:

```text
GITHUB_SHA
```

---

# 32. WHY `id:` IS REQUIRED

If we write:

```yaml
id: meta
```

we can later use:

```yaml
${{ steps.meta.outputs.image-tag }}
```

Mental model:

```text
Step
 ↓
id = meta
 ↓
output = image-tag
 ↓
steps.meta.outputs.image-tag
```

Without the step ID, there is no `steps.meta...` reference.

---

# 33. `$GITHUB_OUTPUT`

This is how a shell command creates a GitHub Actions step output.

Example:

```bash
echo "image-tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

This does **not** mean:

```text
print something on screen
```

It means:

```text
Create an output variable for this step
```

---

# 34. STEP OUTPUT VS NORMAL SHELL VARIABLE

This is important.

Normal shell variable:

```bash
IMAGE_TAG="abc123"
```

exists inside the shell process.

GitHub step output:

```bash
echo "image-tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

makes it accessible through GitHub Actions expression syntax.

So:

```text
Shell variable
→ shell only

GITHUB_OUTPUT
→ GitHub Actions step output
```

---

# 35. JOB OUTPUTS

A job can expose an output to later jobs.

Example:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.meta.outputs.image-tag }}

    steps:

      - name: Generate image tag
        id: meta
        run: |
          echo "image-tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
```

The job output is now:

```text
image_tag
```

---

# 36. CONSUMING A JOB OUTPUT

Another job uses:

```yaml
deploy:
  needs: build

  steps:
    - run: echo "${{ needs.build.outputs.image_tag }}"
```

Notice the hierarchy:

```text
needs
 ↓
build
 ↓
outputs
 ↓
image_tag
```

Full expression:

```text
${{ needs.build.outputs.image_tag }}
```

---

# 37. `needs` AND OUTPUTS WORK TOGETHER

This is the mental model:

```text
Job A
 │
 ├── Step generates output
 │
 └── Job exposes output
          ↓
       Job B
       needs: Job A
          ↓
       needs.job-a.outputs.name
```

Therefore:

```text
needs
→ controls dependency

outputs
→ passes data
```

A very important distinction.

---

# 38. WORKFLOW INPUTS

Inputs are values passed **into a reusable workflow**.

Definition:

```yaml
on:
  workflow_call:

    inputs:
      image-tag:
        description: "Docker image tag"
        required: true
        type: string
```

Caller:

```yaml
jobs:

  docker-build:
    uses: ./.github/workflows/reusable-docker-build.yml

    with:
      image-tag: ${{ github.sha }}
```

Inside reusable workflow:

```yaml
${{ inputs.image-tag }}
```

---

# 39. INPUTS — DEEP MENTAL MODEL

Think of a reusable workflow as a function.

Example programming idea:

```text
build(image_name, image_tag, username)
```

GitHub Actions equivalent:

```text
Caller
   ↓
with:
   ↓
Reusable Workflow
   ↓
inputs.image-name
inputs.image-tag
inputs.docker-username
```

So:

```text
inputs = parameters
```

---

# 40. INPUT TYPES

For `workflow_call`, inputs can have types.

Common types:

```yaml
type: string
```

```yaml
type: boolean
```

```yaml
type: number
```

Example:

```yaml
on:
  workflow_call:

    inputs:

      image-tag:
        required: true
        type: string

      run-tests:
        required: true
        type: boolean

      retry-count:
        required: false
        type: number
```

---

# 41. INPUT DEFAULTS

An input can have a default.

```yaml
on:
  workflow_call:

    inputs:
      run-tests:
        required: false
        type: boolean
        default: true
```

Caller does not have to pass it.

Then:

```yaml
${{ inputs.run-tests }}
```

returns:

```text
true
```

---

# 42. REQUIRED VS OPTIONAL INPUT

Required:

```yaml
required: true
```

Caller must provide it.

Optional:

```yaml
required: false
```

Caller may omit it if a default or suitable behavior exists.

---

# 43. INPUTS VS SECRETS

A reusable workflow can receive:

```text
inputs
secrets
```

Example:

```yaml
jobs:

  docker-push:
    uses: ./.github/workflows/reusable-docker-push.yml

    with:
      image-ref: ${{ needs.docker-build.outputs.image_ref }}

    secrets:
      docker_password: ${{ secrets.DOCKER_PASSWORD }}
```

Think:

```text
with:
→ normal configuration

secrets:
→ sensitive information
```

---

# 44. WORKFLOW OUTPUTS

A reusable workflow can return outputs to its caller.

Example:

```yaml
on:
  workflow_call:

    outputs:
      image_ref:
        description: "Built Docker image"
        value: ${{ jobs.build.outputs.image_ref }}
```

Caller can consume:

```yaml
${{ needs.docker-build.outputs.image_ref }}
```

---

# 45. COMPLETE INPUT → OUTPUT CHAIN

This is one of the most important concepts in this entire sheet.

```text
Caller Workflow
      │
      │ with:
      ↓
Reusable Workflow Input
      │
      ↓
Reusable Workflow Job
      │
      ↓
Step
      │
      │ $GITHUB_OUTPUT
      ↓
Step Output
      │
      ↓
Job Output
      │
      ↓
Reusable Workflow Output
      │
      ↓
Caller Workflow
      │
      ↓
needs.<job>.outputs.<name>
```

---

# 46. INPUTS VS OUTPUTS — EASY MEMORY

| Inputs | Outputs |
|---|---|
| Information enters | Information leaves |
| Caller → reusable workflow | Reusable workflow → caller |
| Usually passed using `with:` | Usually consumed using `needs...outputs` |
| `inputs.name` | `needs.job.outputs.name` |

Easy memory:

```text
INPUT
→ IN

OUTPUT
→ OUT
```

---

# 47. REAL PROJECT INPUT EXAMPLE

In our production deployment:

```yaml
production-deploy:
  needs: docker-push

  uses: ./.github/workflows/reusable-production-deploy.yml

  with:
    image-tag: ${{ github.sha }}
```

Inside the reusable workflow:

```yaml
${{ inputs.image-tag }}
```

This tells deployment:

```text
"Deploy this exact Git SHA image."
```

---

# 48. REAL PROJECT OUTPUT EXAMPLE

Our reusable build workflow exposes information such as:

```text
image_ref
artifact_name
```

Then later jobs consume them.

For example:

```yaml
trivy-scan:
  needs: docker-build

  with:
    image_ref: ${{ needs.docker-build.outputs.image_ref }}
    artifact_name: ${{ needs.docker-build.outputs.artifact_name }}
```

Mental model:

```text
Docker Build
    ↓
Creates image_ref
    ↓
Creates artifact_name
    ↓
Outputs them
    ↓
Trivy consumes them
```

---

# 49. SECRETS

Secrets store sensitive information.

Examples:

```text
Passwords
API tokens
Private credentials
Docker passwords
SSH keys
Cloud credentials
```

GitHub expression:

```yaml
${{ secrets.DOCKER_PASSWORD }}
```

Never hardcode:

```yaml
password: my-real-password
```

---

# 50. GITHUB SECRETS VS VARIABLES

Think:

```text
Variables
→ Configuration

Secrets
→ Sensitive configuration
```

Example:

```text
DOCKER_USERNAME
→ Repository Variable

DOCKER_PASSWORD
→ Repository Secret
```

---

# 51. ARTIFACTS

Artifacts are files produced by a workflow and stored so they can be used later.

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
SARIF files
JSON reports
Logs
Build packages
Docker image TAR
```

---

# 52. ARTIFACT VS CACHE

These are different.

## Artifact

Used to preserve or transfer a workflow output.

```text
Build Job
   ↓
Artifact
   ↓
Another Job
```

## Cache

Used to speed up repeated work.

```text
Dependencies
   ↓
Cache
   ↓
Future workflow run
```

Easy memory:

```text
Artifact
→ Store/pass output

Cache
→ Reuse dependencies/data to speed up builds
```

---

# 53. ARTIFACT VS DOCKER IMAGE

Very important.

```text
Docker Image
→ Application artifact/package for containers
```

```text
GitHub Artifact
→ File stored by GitHub Actions
```

Our project:

```text
Docker Image
      ↓
docker save
      ↓
docker-image.tar.gz
      ↓
GitHub Actions Artifact
      ↓
Download
      ↓
docker load
      ↓
Same Docker Image
```

---

# 54. WHY DID WE SAVE THE DOCKER IMAGE?

Because we wanted:

```text
Build Once
```

not:

```text
Build
Scan
Build again
Push
```

We want:

```text
Build exact image
      ↓
Save exact image
      ↓
Scan exact image
      ↓
Push exact image
```

---

# 55. CACHE

Cache avoids repeatedly downloading or generating the same data.

Mental model:

```text
First run
   ↓
Cache miss
   ↓
Download dependencies
   ↓
Save cache

Next run
   ↓
Cache hit
   ↓
Reuse
```

---

# 56. MATRIX

Matrix runs one job with multiple configurations.

Example:

```yaml
strategy:
  matrix:
    python-version:
      - '3.11'
      - '3.12'
      - '3.13'
```

GitHub creates multiple job executions:

```text
Python 3.11
Python 3.12
Python 3.13
```

Our reusable lint and build-test workflows use a matrix across Python versions.

---

# 57. MATRIX `fail-fast`

```yaml
strategy:
  fail-fast: false
```

Meaning:

```text
One matrix job fails
       ↓
Other matrix jobs still continue
```

Useful when you want to see compatibility across all versions even if one fails.

---

# 58. REUSABLE WORKFLOWS

A reusable workflow uses:

```yaml
on:
  workflow_call:
```

Caller:

```yaml
jobs:

  lint:
    uses: ./.github/workflows/reusable-lint.yml
```

---

# 59. WHY REUSABLE WORKFLOWS?

Without reusable workflows:

```text
PR Pipeline
   ↓
Copy build logic

Main Pipeline
   ↓
Copy build logic again
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
Consistent behavior
Easier maintenance
Reusable inputs
Reusable outputs
Clear architecture
```

---

# 60. COMPOSITE ACTIONS

Composite actions package reusable **steps**.

Example:

```text
.github/actions/my-action/action.yml
```

Concept:

```yaml
runs:
  using: composite

  steps:
    - name: Hello
      shell: bash
      run: echo "Hello"
```

Use:

```yaml
uses: ./.github/actions/my-action
```

---

# 61. REUSABLE WORKFLOW VS COMPOSITE ACTION

| Reusable Workflow | Composite Action |
|---|---|
| Reuses workflow/job logic | Reuses steps |
| `workflow_call` | `composite` |
| Can contain multiple jobs | Runs inside a job |
| Good for pipeline stages | Good for repeated step sequences |

Easy memory:

```text
Reusable Workflow
→ Reuse jobs

Composite Action
→ Reuse steps
```

---

# 62. `workflow_call` VS `workflow_run`

Very important.

## `workflow_call`

Means:

```text
"Use this workflow as a reusable component."
```

```text
Caller
  ↓
workflow_call
  ↓
Reusable Workflow
```

## `workflow_run`

Means:

```text
"Run this workflow after another workflow finishes."
```

```text
Workflow A
   ↓
completed
   ↓
Workflow B
```

Easy memory:

```text
workflow_call
→ Reuse

workflow_run
→ React after completion
```

---

# 63. `repository_dispatch`

Used when an external system needs to trigger a workflow.

```yaml
on:
  repository_dispatch:
    types:
      - deploy-request
```

Mental model:

```text
External System
      ↓
GitHub API / Dispatch Event
      ↓
GitHub Actions Workflow
```

---

# 64. DOCKER CI/CD

Typical Docker pipeline:

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

---

# 65. DOCKER LOGIN

Example:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v4

  with:
    username: ${{ inputs.docker_username }}
    password: ${{ secrets.docker_password }}
```

Notice:

```text
username
→ normal configuration

password
→ secret
```

---

# 66. DOCKER BUILD

Example:

```yaml
- name: Build image
  run: |
    docker build \
      -t "$DOCKER_USERNAME/secure-cicd-pipeline-lab:$IMAGE_TAG" \
      .
```

---

# 67. DOCKER PUSH

```bash
docker push "$DOCKER_USERNAME/secure-cicd-pipeline-lab:$IMAGE_TAG"
```

---

# 68. WHY USE GIT SHA IMAGE TAGS?

Avoid relying only on:

```text
latest
```

Use:

```text
<full-git-sha>
```

because:

```text
latest
→ Mutable

Git SHA
→ Specific source revision
```

Example:

```text
aniruddhakharve/secure-cicd-pipeline-lab:abc123....
```

---

# 69. IMAGE TRACEABILITY

The full flow:

```text
Git Commit
   ↓
github.sha
   ↓
Docker Image Tag
   ↓
Docker Hub
   ↓
Production
```

This allows us to answer:

> "Which source commit produced the image running in production?"

That is extremely useful for debugging and rollback.

---

# 70. DOCKER IMAGE VS CONTAINER

```text
Image
→ Immutable package/template

Container
→ Running instance of image
```

Pipeline:

```text
Build Image
```

Production:

```text
Run Container from Image
```

---

# 71. PRODUCTION ENVIRONMENT APPROVAL

GitHub Environments can protect deployment jobs.

Our production deployment used:

```text
Environment:
Production
```

with required reviewer/approval.

Flow:

```text
CI
 ↓
Security
 ↓
Docker Push
 ↓
Production Environment
 ↓
Manual Approval
 ↓
AWS Deployment
```

This gives a human approval gate before production.

---

# 72. PULL REQUEST VS MAIN PIPELINE

Our final architecture separates validation from deployment.

## Pull Request

```text
PR
 ↓
Lint
 ↓
Build/Test
 ↓
SAST
 ↓
Secret Scan
 ↓
Dependency Scan
 ↓
Dependency Review
 ↓
Dockerfile Lint
 ↓
PR Validation
```

No Docker push.

No production deployment.

---

## Main

```text
Push main
 ↓
Validation
 ↓
Security
 ↓
Docker Build
 ↓
Trivy
 ↓
Docker Push
 ↓
Production Approval
 ↓
AWS Deployment
```

---

# 73. DEVSECOPS

**DevSecOps = Development + Security + Operations**

The core idea:

> Security becomes part of the software delivery lifecycle rather than a final manual activity.

Without DevSecOps:

```text
Code
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
Security
 ↓
Build
 ↓
Image Security
 ↓
Deploy
 ↓
Runtime Validation
```

---

# 74. SECURITY LAYERS

Think of security by layer:

```text
Source Code
   ↓
Semgrep

Secrets
   ↓
Gitleaks

Dependencies
   ↓
pip-audit
Dependency Review

Dockerfile
   ↓
Hadolint

Container Image
   ↓
Trivy

AWS Authentication
   ↓
OIDC / STS / IAM

Production Secrets
   ↓
Secrets Manager

Runtime / Deployment
   ↓
SSM + Health Check
```

---

# 75. SAST

**SAST = Static Application Security Testing**

Tests source code without running the application.

```text
Source Code
   ↓
Semgrep
   ↓
Security Findings
```

Our project uses Semgrep.

Typical command:

```bash
semgrep scan --config=auto --sarif --output=semgrep-results.sarif
```

---

# 76. DAST

**DAST = Dynamic Application Security Testing**

Tests a running application.

```text
Running Application
       ↓
DAST
       ↓
Security Findings
```

Earlier NodeGoat work used OWASP ZAP for this concept.

Easy memory:

```text
SAST
→ Source

DAST
→ Running application
```

---

# 77. SAST VS DAST

| SAST | DAST |
|---|---|
| Static | Dynamic |
| Source code | Running application |
| Semgrep | OWASP ZAP |
| Before runtime | During/after deployment |
| Finds code patterns | Finds runtime/web behavior |

---

# 78. SECRET SCANNING

Secret scanning looks for:

```text
API keys
Tokens
Passwords
Private keys
Cloud credentials
```

Our project uses Gitleaks.

---

# 79. WHY `fetch-depth: 0`?

Our secret scanning checkout uses:

```yaml
with:
  fetch-depth: 0
```

This fetches complete Git history.

Why?

Imagine:

```text
Commit 1
→ Secret accidentally committed

Commit 2
→ Secret deleted
```

Current files may look clean, but Git history still contains the old secret.

Therefore:

```text
Current source
+
Git history
```

gives the scanner more visibility.

Important:

> Detecting a secret does not automatically make the credential safe again. A real leaked credential should also be revoked/rotated.

---

# 80. SECRET SCANNING VS PUSH PROTECTION

## Secret Scanning

```text
Secret appears
   ↓
Detect
```

## Push Protection

```text
Secret detected before push
   ↓
Prevent push
```

Easy memory:

```text
Secret Scanning
→ Detect

Push Protection
→ Prevent
```

---

# 81. DEPENDENCY SCANNING

Dependencies can contain known vulnerabilities.

For Python:

```text
requirements.txt
       ↓
pip-audit
       ↓
Vulnerability Report
```

Our final application uses:

```text
Flask
pytest
mysql-connector-python
```

---

# 82. DEPENDENCY REVIEW

Dependency Review is focused on dependency changes introduced through pull requests.

Our PR pipeline uses:

```yaml
uses: actions/dependency-review-action@...
with:
  fail-on-severity: critical
```

Concept:

```text
PR adds/changes dependency
       ↓
Dependency Review
       ↓
Critical vulnerability?
    ↙             ↘
  YES             NO
   ↓               ↓
 PR fails       PR continues
```

Easy memory:

```text
pip-audit
→ Audit dependencies

Dependency Review
→ Review dependency changes in PR
```

---

# 83. HADOLINT

Hadolint checks Dockerfiles for common problems and best-practice violations.

```text
Dockerfile
    ↓
Hadolint
    ↓
Lint / findings
```

Easy memory:

```text
Hadolint
→ Dockerfile
```

---

# 84. TRIVY

Trivy scans the built container image for known vulnerabilities.

```text
Docker Image
      ↓
Trivy
      ↓
OS + Library vulnerabilities
```

Our project uses:

```text
CRITICAL
HIGH
```

as the configured security-gate severities.

---

# 85. TRIVY SECURITY GATE

Our production-style configuration concept is:

```yaml
severity: CRITICAL,HIGH
exit-code: 1
```

Meaning:

```text
Build Image
    ↓
Trivy
    ↓
HIGH / CRITICAL?
   ↙          ↘
 YES          NO
  ↓            ↓
FAIL        CONTINUE
               ↓
          Docker Push
```

---

# 86. SCANNER VS SECURITY GATE

This distinction is extremely important.

## Scanner

Answers:

> "What did I find?"

```text
Trivy
 ↓
HIGH vulnerability
```

## Security Gate

Answers:

> "Should the pipeline continue?"

```text
HIGH vulnerability
      ↓
Policy
      ↓
Block release
```

Therefore:

```text
Scanner
= Detection

Security Gate
= Enforcement
```

---

# 87. `exit-code: 0` VS `exit-code: 1`

## `exit-code: 0`

Normally means the command exits successfully.

The scan can still report findings depending on the tool configuration.

```text
Scan
 ↓
Findings
 ↓
Report
 ↓
Command succeeds
```

## `exit-code: 1`

The command exits with failure.

```text
Scan
 ↓
Policy condition met
 ↓
Exit 1
 ↓
Step fails
```

Important:

> "The scanner exited with 0" does not automatically mean "zero vulnerabilities."

Always distinguish:

```text
Scan execution result
vs
Security findings
vs
Pipeline policy
```

---

# 88. BUILD ONCE → SCAN SAME IMAGE → PUSH SAME IMAGE

This is one of the strongest concepts from the final project.

Bad architecture:

```text
Build Image A
   ↓
Scan Image A

Build Image B
   ↓
Push Image B
```

The thing scanned might not be exactly the thing pushed.

Better architecture:

```text
Build Image
   ↓
Save exact image
   ↓
GitHub Artifact
   ↓
Load exact image
   ↓
Trivy
   ↓
Docker Push
```

Therefore:

```text
Build Once
→ Scan Same Image
→ Push Same Image
→ Deploy Same Image
```

---

# 89. WHY BUILD ONCE?

Because the artifact should not silently change between validation and deployment.

Think:

```text
Source
  ↓
Build
  ↓
Artifact
  ↓
Security
  ↓
Promotion
```

The validated artifact is the artifact that gets promoted.

---

# 90. SARIF

**SARIF = Static Analysis Results Interchange Format**

Security tools can produce SARIF results that GitHub can consume.

Flow:

```text
Semgrep / Trivy
       ↓
     SARIF
       ↓
GitHub Security / Code Scanning
```

Security-reporting jobs need:

```yaml
permissions:
  contents: read
  security-events: write
```

---

# 91. TRIVY SARIF LESSON

We encountered an important Trivy reporting issue.

The scan was configured for:

```text
HIGH
CRITICAL
```

but SARIF severity handling needed explicit control.

The solution was:

```yaml
limit-severities-for-sarif: true
```

Mental lesson:

```text
Scan configuration
+
Output format
+
Report handling
```

all need to be understood.

---

# 92. ACTION PINNING

Instead of only:

```yaml
uses: actions/checkout@v4
```

we pin important actions to a specific commit SHA:

```yaml
uses: actions/checkout@<commit-sha> # version comment
```

---

# 93. WHY PIN ACTIONS?

A mutable tag can move.

```text
@v4
 ↓
Tag may point to different commit later
```

A commit SHA points to a specific revision.

```text
@<SHA>
 ↓
Exact action commit
```

This helps reduce supply-chain risk from unexpected action changes.

---

# 94. GITHUB ACTIONS PERMISSIONS

Use least privilege.

Default pattern in our workflows:

```yaml
permissions:
  contents: read
```

If SARIF upload is required:

```yaml
permissions:
  contents: read
  security-events: write
```

For AWS OIDC:

```yaml
permissions:
  id-token: write
  contents: read
```

---

# 95. WHY LEAST PRIVILEGE?

Imagine a third-party action is compromised.

Broad permissions:

```text
Compromised Action
       ↓
Lots of GitHub permissions
       ↓
Large blast radius
```

Restricted permissions:

```text
Compromised Action
       ↓
Only required permission
       ↓
Smaller blast radius
```

Interview phrase:

> "I follow the principle of least privilege by granting each workflow or job only the permissions it actually needs."

---

# 96. GITHUB ENVIRONMENT

We used:

```text
Production
```

as a GitHub Environment.

The deployment job uses:

```yaml
environment: Production
```

This environment can contain:

```text
Approval rules
Environment variables
Environment secrets
```

Our production deployment required manual approval before deployment.

---

# 97. DAY 49 — FINAL PROJECT

The final project is:

```text
secure-cicd-pipeline-lab
```

Application:

```text
Flask
+
MySQL
```

Containerization:

```text
Docker
+
Docker Compose
```

CI/CD:

```text
GitHub Actions
```

Security:

```text
Semgrep
Gitleaks
pip-audit
Dependency Review
Hadolint
Trivy
SARIF
```

Production:

```text
AWS EC2
AWS SSM
AWS IAM
GitHub OIDC
AWS STS
AWS Secrets Manager
```

---

# 98. FINAL PROJECT ARCHITECTURE

```text
                         GitHub
                            │
              ┌─────────────┴─────────────┐
              │                           │
          Pull Request                Push main
              │                           │
              ▼                           ▼
         PR Pipeline                 Main Pipeline
              │                           │
       ┌──────┴──────┐             ┌──────┴──────┐
       │             │             │             │
     Build         Security      Build         Security
      Test           Checks       Test           Checks
       │             │             │             │
       └──────┬──────┘             └──────┬──────┘
              │                           │
              ▼                           ▼
          PR Gate                   Docker Build
                                          │
                                          ▼
                                     Save Image
                                          │
                                          ▼
                                    GitHub Artifact
                                          │
                                          ▼
                                     Trivy Scan
                                          │
                                  ┌───────┴───────┐
                                  │               │
                                FAIL            PASS
                                  │               │
                                  X               ▼
                                            Docker Push
                                                │
                                                ▼
                                      Production Approval
                                                │
                                                ▼
                                           GitHub OIDC
                                                │
                                                ▼
                                           AWS STS
                                                │
                                                ▼
                                          IAM Role
                                                │
                                                ▼
                                             AWS SSM
                                                │
                                                ▼
                                             EC2
                                                │
                           ┌────────────────────┴─────────────────┐
                           │                                      │
                         Flask                                  MySQL
                           │                                      │
                           └────────────────────┬─────────────────┘
                                                │
                                                ▼
                                           Health Check
```

---

# 99. OIDC — WHAT IS IT?

This is one of the most important Day 49 concepts.

**OIDC = OpenID Connect**

In this project, GitHub Actions uses OIDC to prove its identity to AWS.

The goal:

> Do not store long-lived AWS access keys in GitHub when a short-lived identity-based authentication flow can be used.

---

# 100. WITHOUT OIDC

Traditional approach:

```text
GitHub Secrets
    │
    ├── AWS_ACCESS_KEY_ID
    └── AWS_SECRET_ACCESS_KEY
             │
             ▼
           AWS
```

Problem:

```text
Long-lived credentials
```

If compromised, they may remain usable until rotated/revoked.

---

# 101. WITH OIDC

Our project:

```text
GitHub Actions
      ↓
OIDC Identity Token
      ↓
AWS STS
      ↓
Assume IAM Role
      ↓
Temporary AWS Credentials
      ↓
AWS API calls
```

No long-lived AWS access key is required for this deployment.

---

# 102. OIDC DOES NOT DIRECTLY GIVE AWS PERMISSIONS

This is an important conceptual point.

OIDC answers:

> "Who is this workload?"

IAM answers:

> "What is this workload allowed to do?"

STS connects the two by issuing temporary credentials after the token is trusted.

So:

```text
OIDC
= Identity proof

IAM
= Authorization policy

STS
= Temporary credential service
```

---

# 103. WHAT IS STS?

**STS = AWS Security Token Service**

STS provides temporary AWS security credentials.

Common operation:

```text
AssumeRoleWithWebIdentity
```

In our architecture:

```text
GitHub OIDC token
       ↓
AWS STS
       ↓
Assume IAM role
       ↓
Temporary credentials
```

---

# 104. OIDC → STS → IAM — STEP BY STEP

This is the interview explanation to memorize.

```text
1. GitHub Actions starts deployment
```

↓

```text
2. Workflow has id-token: write
```

↓

```text
3. GitHub provides an OIDC identity token
```

↓

```text
4. AWS trusts GitHub's OIDC provider
```

↓

```text
5. AWS STS validates the token
```

↓

```text
6. STS allows the workflow to assume the IAM role
```

↓

```text
7. Temporary AWS credentials are issued
```

↓

```text
8. GitHub Actions uses those credentials to call AWS APIs
```

---

# 105. WHAT IS THE AWS OIDC PROVIDER?

AWS needs to know:

```text
"I trust tokens issued by GitHub's OIDC provider."
```

The provider is:

```text
https://token.actions.githubusercontent.com
```

Audience:

```text
sts.amazonaws.com
```

This provider establishes trust between AWS and GitHub Actions.

---

# 106. OIDC TRUST POLICY

The IAM role has a trust policy.

Conceptually:

```text
Who may assume this role?
```

Our trust policy restricted access to:

```text
GitHub repository
+
Production environment
```

This is stronger than:

```text
Any GitHub repository can assume role
```

Mental model:

```text
Trust Policy
→ Who can assume the role?

Permission Policy
→ What can the role do?
```

Very important distinction.

---

# 107. TRUST POLICY VS PERMISSION POLICY

## Trust Policy

Controls:

```text
WHO can assume the role?
```

Example concept:

```text
GitHub OIDC
+
Specific repository
+
Specific Production environment
```

## Permission Policy

Controls:

```text
WHAT can the role do?
```

Example:

```text
ssm:SendCommand
ssm:GetCommandInvocation
```

Easy memory:

```text
Trust
→ Who?

Permissions
→ What?
```

---

# 108. GITHUB OIDC IAM ROLE

Our GitHub deployment role:

```text
GitHubActions-Production-Deploy
```

Purpose:

```text
GitHub Actions
      ↓
Assume this role through OIDC
      ↓
Use temporary AWS credentials
```

---

# 109. WHY DID WE NEED THIS ROLE?

GitHub needed AWS permissions to perform deployment operations such as:

```text
SSM SendCommand
SSM command status/read operations
```

But GitHub itself should not have permanent AWS credentials.

Therefore:

```text
OIDC
 ↓
GitHubActions-Production-Deploy
 ↓
Temporary AWS identity
```

---

# 110. SECOND IAM ROLE — EC2 ROLE

The EC2 instance had:

```text
ProductionEC2-SSM-Role
```

This is completely different from the GitHub Actions role.

It belongs to:

```text
EC2
```

not GitHub.

---

# 111. WHY DOES EC2 NEED AN IAM ROLE?

The EC2 instance needs AWS permissions for actions performed from the server.

For example:

```text
EC2
 ↓
AWS Secrets Manager
```

The instance should use an IAM role instead of storing AWS access keys on the server.

---

# 112. FINAL IAM ARCHITECTURE

```text
GitHub Actions
      │
      │ OIDC
      ↓
GitHubActions-Production-Deploy
      │
      │ SSM
      ↓
Production EC2
      │
      │ Instance Role
      ↓
ProductionEC2-SSM-Role
```

There are **two roles**.

```text
1. GitHubActions-Production-Deploy
2. ProductionEC2-SSM-Role
```

No third role is required in our final design.

---

# 113. WHAT IS AWS SSM?

**SSM = AWS Systems Manager**

In our project we mainly used SSM to execute commands on the EC2 instance remotely.

Think:

```text
GitHub Actions
      ↓
AWS SSM
      ↓
EC2
```

Instead of:

```text
GitHub Actions
      ↓
SSH private key
      ↓
EC2
```

---

# 114. WHAT DOES SSM DO FOR US?

GitHub Actions sends a command:

```text
docker compose pull
```

or:

```text
docker compose up -d
```

through AWS Systems Manager.

SSM then executes that command on the managed EC2 instance.

---

# 115. SSM COMMUNICATION FLOW

```text
GitHub Actions
      ↓
AWS API
      ↓
ssm:SendCommand
      ↓
SSM
      ↓
EC2 SSM Agent
      ↓
Shell Command
```

The EC2 machine has the SSM Agent and an IAM role allowing it to communicate with Systems Manager.

---

# 116. WHY SSM INSTEAD OF SSH?

Traditional:

```text
GitHub
 ↓
SSH private key
 ↓
EC2
```

Our project:

```text
GitHub
 ↓
OIDC
 ↓
AWS IAM
 ↓
SSM
 ↓
EC2
```

Benefits:

```text
No long-lived SSH private key in GitHub
AWS IAM controls authorization
Central AWS management
```

---

# 117. WHAT IS AWS SECRETS MANAGER?

AWS Secrets Manager is a managed service for storing sensitive values securely.

Our secret:

```text
secure-cicd-pipeline-lab/production
```

Contained:

```text
MYSQL_DATABASE
MYSQL_USER
MYSQL_PASSWORD
MYSQL_ROOT_PASSWORD
```

---

# 118. WHY NOT STORE DATABASE PASSWORD IN GITHUB?

Because the database password belongs to the production environment.

Instead of:

```text
Git Repository
   ↓
Password
```

we use:

```text
AWS Secrets Manager
   ↓
EC2 deployment
   ↓
.env
   ↓
Docker Compose
```

---

# 119. OIDC VS SECRETS MANAGER

Very important interview question.

They solve different problems.

## OIDC

```text
How does GitHub authenticate to AWS?
```

## Secrets Manager

```text
Where do production application secrets live?
```

Easy memory:

```text
OIDC
→ Authentication / identity

Secrets Manager
→ Secret storage
```

---

# 120. STS VS SECRETS MANAGER

Also different.

```text
STS
→ Issues temporary AWS credentials

Secrets Manager
→ Stores application secrets
```

Example:

```text
GitHub OIDC
   ↓
STS
   ↓
Temporary AWS credentials
```

while:

```text
EC2
   ↓
Secrets Manager
   ↓
Database password
```

---

# 121. WHAT IS EC2 IN THIS ARCHITECTURE?

EC2 is the production compute server.

Think:

```text
AWS
 ↓
EC2
 ↓
Docker Engine
 ↓
Docker Compose
 ↓
Application + Database
```

Our production server ran:

```text
Flask
+
MySQL
```

---

# 122. PRODUCTION DEPLOYMENT — COMPLETE STEP-BY-STEP

This is the exact story to learn for interviews.

## Step 1 — Developer pushes to `main`

```text
git push origin main
```

↓

GitHub Actions starts the main pipeline.

---

## Step 2 — CI validation

```text
Lint
Build/Test
SAST
Secret Scan
Dependency Scan
Dockerfile Lint
```

These run before image promotion.

---

## Step 3 — Docker Build

The pipeline builds:

```text
aniruddhakharve/secure-cicd-pipeline-lab:<full-git-sha>
```

---

## Step 4 — Save the exact image

```bash
docker save "$IMAGE" | gzip > docker-image.tar.gz
```

The TAR file is uploaded as a GitHub Actions artifact.

---

## Step 5 — Trivy loads the same image

```text
GitHub Artifact
      ↓
docker load
      ↓
Exact Docker Image
```

---

## Step 6 — Trivy scans it

```text
Trivy
 ↓
OS packages
 ↓
Libraries
 ↓
HIGH / CRITICAL
```

If policy fails:

```text
Pipeline stops
```

---

## Step 7 — Docker Push

Only after Trivy passes:

```text
Docker Hub
```

receives the same image.

---

## Step 8 — Production approval

GitHub Environment:

```text
Production
```

requires approval.

---

## Step 9 — GitHub requests OIDC identity

The deployment workflow requests:

```yaml
permissions:
  id-token: write
```

---

## Step 10 — AWS trusts GitHub OIDC

AWS validates the GitHub token against the configured OIDC provider and trust policy.

---

## Step 11 — STS assumes the IAM role

```text
AWS STS
   ↓
GitHubActions-Production-Deploy
```

Temporary credentials are provided.

---

## Step 12 — GitHub uses SSM

The workflow calls:

```text
aws ssm send-command
```

against the production EC2 instance.

---

## Step 13 — EC2 uses its own IAM role

EC2 has:

```text
ProductionEC2-SSM-Role
```

This gives the instance the required AWS permissions.

---

## Step 14 — Retrieve production secrets

The EC2 deployment runs:

```bash
aws secretsmanager get-secret-value \
  --secret-id secure-cicd-pipeline-lab/production
```

The database configuration is used to create the production `.env`.

---

## Step 15 — Download exact Compose file

The deployment downloads the `docker-compose.yml` from the exact commit being deployed.

Mental model:

```text
Git SHA
   ↓
Exact Compose file
   ↓
EC2
```

This keeps code version and deployment configuration aligned.

---

## Step 16 — Verify Compose configuration

```bash
docker compose config --images
```

This confirms which images Compose resolves.

---

## Step 17 — Pull exact production images

```bash
docker compose pull
```

The EC2 server pulls the exact Git-SHA application image.

---

## Step 18 — Recreate application

```bash
docker compose down
docker compose up -d --force-recreate
```

Important:

```text
down
→ Stops/removes containers/network

down -v
→ Also removes named volumes
```

We intentionally did **not** use:

```bash
docker compose down -v
```

because MySQL data was stored in a named volume.

---

## Step 19 — Health check

The deployment checks:

```bash
curl -fsS http://localhost:5000/health
```

Expected:

```json
{
  "database": "connected",
  "status": "healthy"
}
```

---

# 123. EXACT AWS DEPLOYMENT MEMORY DIAGRAM

```text
                        GitHub Actions
                              │
                              ▼
                        OIDC Identity
                              │
                              ▼
                         AWS STS
                              │
                              ▼
               GitHubActions-Production-Deploy
                              │
                         SSM SendCommand
                              │
                              ▼
                         EC2 Instance
                              │
                   ProductionEC2-SSM-Role
                              │
                 ┌────────────┴────────────┐
                 │                         │
                 ▼                         ▼
          Secrets Manager             Docker Hub
                 │                         │
                 ▼                         ▼
          Production .env           Exact image pull
                 │                         │
                 └────────────┬────────────┘
                              ▼
                        Docker Compose
                              │
                       ┌──────┴──────┐
                       ▼             ▼
                     Flask         MySQL
                       │             │
                       └──────┬──────┘
                              ▼
                         /health
```

---

# 124. WHAT EXACTLY DID GITHUB PERMISSION `id-token: write` MEAN?

This is often confusing.

It does **not** mean:

```text
GitHub can write to my AWS resources
```

Instead, it allows the workflow to request a GitHub OIDC identity token.

Then AWS decides whether to trust that identity based on the IAM trust policy.

So:

```text
id-token: write
→ Obtain identity token

IAM permission policy
→ AWS authorization
```

---

# 125. OIDC AUTHENTICATION VS IAM AUTHORIZATION

This is a great interview explanation.

```text
Authentication
→ Who are you?

Authorization
→ What are you allowed to do?
```

In our project:

```text
GitHub OIDC
→ Authentication / identity

IAM role policy
→ Authorization
```

---

# 126. SSM COMMAND EXECUTION

We sent commands like:

```text
aws ssm send-command
```

The returned command ID is stored:

```bash
echo "COMMAND_ID=$COMMAND_ID" >> "$GITHUB_ENV"
```

Later steps use:

```bash
aws ssm get-command-invocation
```

This gave us:

```text
Status
ResponseCode
StandardOutputContent
StandardErrorContent
```

---

# 127. SSM `STATUS` VS `ERROR`

This caused confusion during our deployment work.

Docker Compose can write normal progress information to stderr.

Therefore:

```text
StandardErrorContent != automatically failed
```

We checked:

```text
Status
+
ResponseCode
```

The actual logic became:

```text
Status == Success
AND
ResponseCode == 0
```

then:

```text
Deployment step succeeded
```

---

# 128. WHY DID WE USE `|| true` WITH SSM WAIT?

Example:

```bash
aws ssm wait command-executed ... || true
```

The wait command itself was allowed to return without immediately failing the GitHub step.

Then we separately inspected:

```text
Status
ResponseCode
```

to make the actual success/failure decision.

This separates:

```text
Waiting for completion
```

from:

```text
Validating command result
```

---

# 129. PRODUCTION `.env` LESSON

One important issue we understood:

This:

```text
${{ github.sha }}
```

is evaluated by GitHub Actions.

But:

```text
${IMAGE_TAG}
```

is evaluated by Docker Compose.

They are different systems.

Production eventually receives an actual value:

```text
IMAGE_TAG=<real-full-sha>
```

So:

```text
GitHub expression
→ produces value

Compose variable
→ consumes value
```

---

# 130. PRODUCTION DOCKER COMPOSE

Application service:

```yaml
services:

  app:
    image: ${DOCKER_USERNAME}/secure-cicd-pipeline-lab:${IMAGE_TAG}

    ports:
      - "5000:5000"

    depends_on:
      mysql:
        condition: service_healthy
```

MySQL:

```yaml
  mysql:
    image: mysql:8.0
```

---

# 131. WHY `depends_on` HEALTH CONDITION?

We do not only want:

```text
MySQL container exists
```

We want:

```text
MySQL is healthy
```

Then:

```text
MySQL
 ↓
Health Check
 ↓
Healthy
 ↓
Flask startup dependency
```

---

# 132. MYSQL HEALTHCHECK

Example:

```yaml
healthcheck:
  test:
    - CMD
    - mysqladmin
    - ping
    - "-h"
    - localhost

  interval: 5s
  timeout: 5s
  retries: 10
```

---

# 133. MYSQL PERSISTENCE

Named volume:

```yaml
volumes:
  - mysql-data:/var/lib/mysql
```

Mental model:

```text
Container
→ Disposable

Named Volume
→ Persistent data
```

---

# 134. WHY NOT `docker compose down -v`?

Because:

```bash
docker compose down -v
```

removes named volumes.

That could delete database data.

Therefore:

```bash
docker compose down
```

preserves the named volume.

---

# 135. EXACT IMAGE VS `latest`

The final project uses:

```text
full Git SHA
```

instead of relying on:

```text
latest
```

The production Compose file uses:

```yaml
image: ${DOCKER_USERNAME}/secure-cicd-pipeline-lab:${IMAGE_TAG}
```

and:

```text
IMAGE_TAG = full Git SHA
```

---

# 136. DOCKER HUB'S ROLE

Docker Hub is the image registry.

Our flow:

```text
GitHub Actions
      ↓
Docker Build
      ↓
Trivy
      ↓
Docker Push
      ↓
Docker Hub
```

Production:

```text
EC2
 ↓
docker compose pull
 ↓
Docker Hub
 ↓
Exact image
```

---

# 137. PRODUCTION DEPLOYMENT IS NOT A BUILD

CI:

```text
Build image
Test image
Scan image
Push image
```

Production:

```text
Pull image
Run image
```

This separation is important.

---

# 138. SECURITY HEADERS

The Flask application also implements security headers.

Examples:

```text
Content-Security-Policy
X-Content-Type-Options
X-Frame-Options
Referrer-Policy
Permissions-Policy
```

Example:

```python
response.headers["X-Content-Type-Options"] = "nosniff"
response.headers["X-Frame-Options"] = "DENY"
```

This demonstrates that application security is not limited to CI scanners.

---

# 139. APPLICATION HEALTH VS SECURITY

These are different.

Health check:

```text
Is the application running correctly?
```

Security scan:

```text
Does the application/image/code violate security checks?
```

Example:

```text
/health
→ Application health

Trivy
→ Image security
```

Passing one does not imply passing the other.

---

# 140. FINAL PR PIPELINE

```text
                 Pull Request
                      │
                      ▼
                    Lint
                      │
                 Build/Test
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      SAST       Secret Scan    Dependency Scan
        │             │             │
        └─────────────┼─────────────┘
                      ↓
              Dependency Review
                      ↓
               Dockerfile Lint
                      ↓
                  PR Result
```

---

# 141. FINAL MAIN PIPELINE

```text
                    Push to main
                         │
                         ▼
                    ┌─────────┐
                    │  Lint   │
                    └────┬────┘
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   Build/Test          SAST          Secret Scan
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                Dependency Scan
                         │
                   Dockerfile Lint
                         │
                         ▼
                   Docker Build
                         │
                         ▼
                   Save Image
                         │
                         ▼
                 GitHub Artifact
                         │
                         ▼
                    Trivy Scan
                         │
                 ┌───────┴───────┐
                 │               │
               FAIL             PASS
                 │               │
                 X               ▼
                         Docker Push
                               │
                               ▼
                      Production Approval
                               │
                               ▼
                           OIDC → STS
                               │
                               ▼
                              IAM
                               │
                               ▼
                              SSM
                               │
                               ▼
                              EC2
                               │
                               ▼
                         Secrets Manager
                               │
                               ▼
                         Docker Compose
                               │
                               ▼
                           Health Check
```

---

# 142. MOST IMPORTANT GITHUB ACTIONS CONCEPTS TABLE

| Concept | Easy Meaning |
|---|---|
| Workflow | Complete automation |
| Job | Group of steps |
| Step | One operation |
| Runner | Machine executing job |
| `on` | Trigger |
| `needs` | Job dependency |
| `if` | Condition |
| `env` | Environment variable |
| `vars` | Configuration variable |
| `secrets` | Sensitive value |
| `id` | Step identifier |
| `$GITHUB_OUTPUT` | Create step output |
| Job output | Pass value to another job |
| `inputs` | Data entering reusable workflow |
| `outputs` | Data leaving reusable workflow |
| `with` | Pass action/workflow inputs |
| Artifact | Stored workflow file/output |
| Cache | Reuse data to speed builds |
| Matrix | Multiple configurations |
| `workflow_call` | Reuse workflow |
| Composite Action | Reuse steps |
| `workflow_run` | Trigger after workflow |
| `repository_dispatch` | External trigger |
| `workflow_dispatch` | Manual trigger |
| `permissions` | GitHub token access |
| Environment | Protected deployment target |

---

# 143. MOST IMPORTANT DEVSECOPS TOOL TABLE

| Layer | Tool | Purpose |
|---|---|---|
| Source code | Semgrep | SAST |
| Secrets | Gitleaks | Secret detection |
| Python dependencies | pip-audit | Dependency vulnerabilities |
| PR dependencies | Dependency Review | Review dependency changes |
| Dockerfile | Hadolint | Dockerfile linting |
| Container image | Trivy | OS/library vulnerabilities |
| Security results | SARIF | GitHub security reporting |
| Runtime web app | OWASP ZAP | DAST |

---

# 144. WHAT EACH TOOL DOES — ONE-LINER

```text
Semgrep
→ Is my code insecure?

Gitleaks
→ Did I expose a secret?

pip-audit
→ Is a Python dependency vulnerable?

Dependency Review
→ Did this PR introduce a risky dependency?

Hadolint
→ Is my Dockerfile written safely?

Trivy
→ Is my final container image vulnerable?

ZAP
→ Is my running web application vulnerable?
```

---

# 145. MOST IMPORTANT AWS CONCEPT TABLE

| Concept | Easy Meaning |
|---|---|
| OIDC | Proves GitHub workload identity |
| STS | Issues temporary AWS credentials |
| IAM Role | Defines AWS permissions/trust |
| Trust Policy | Defines who can assume role |
| Permission Policy | Defines what role can do |
| SSM | Remotely executes commands on managed EC2 |
| Secrets Manager | Stores production secrets |
| EC2 | Production compute server |
| Docker Hub | Image registry |
| GitHub Environment | Deployment protection/approval |

---

# 146. AWS INTERVIEW MEMORY

Remember these three questions:

```text
OIDC
→ Who are you?

IAM
→ What are you allowed to do?

SSM
→ How do I execute the deployment command on EC2?
```

And:

```text
STS
→ Give me temporary AWS credentials after I prove my identity.
```

```text
Secrets Manager
→ Where do I safely store the production password?
```

---

# 147. COMPLETE AWS INTERVIEW ANSWER

If interviewer asks:

> "How did you deploy your application to EC2 using OIDC?"

Answer:

> "I used GitHub Actions with OIDC instead of storing long-lived AWS access keys. The deployment workflow had `id-token: write` permission, which allowed it to request a GitHub OIDC token. AWS had an OIDC provider configured for GitHub and an IAM trust policy restricted to my repository and Production environment. AWS STS validated the web identity and allowed GitHub Actions to assume the `GitHubActions-Production-Deploy` role, which provided temporary credentials. That role had permission to use Systems Manager against the production EC2 instance. I then used SSM to execute the deployment commands on EC2. The EC2 instance had its own `ProductionEC2-SSM-Role`, allowing the SSM agent to operate and retrieve the production secret from AWS Secrets Manager. The deployment pulled the exact Git-SHA Docker image from Docker Hub, used Docker Compose to recreate the application, preserved the MySQL volume, and finally verified `/health`."

---

# 148. EVEN SHORTER AWS ANSWER

```text
GitHub Actions
    ↓
OIDC
    ↓
AWS STS
    ↓
GitHubActions-Production-Deploy
    ↓
SSM
    ↓
EC2
    ↓
ProductionEC2-SSM-Role
    ↓
Secrets Manager
    ↓
Docker Compose
    ↓
Health Check
```

Say:

> "OIDC handled identity, STS issued temporary credentials, IAM authorized access, SSM executed commands on EC2, Secrets Manager supplied production secrets, and Docker Compose deployed the validated image."

---

# 149. TWO IAM ROLES — INTERVIEW ANSWER

> "I used two IAM roles. The first, `GitHubActions-Production-Deploy`, was assumed by GitHub Actions through OIDC and allowed the deployment workflow to use the required SSM operations. The second, `ProductionEC2-SSM-Role`, was attached to the EC2 instance and provided the permissions required by the SSM agent and the production application, including access to the production secret. They solve different sides of the deployment."

---

# 150. TRUST POLICY VS PERMISSION POLICY — INTERVIEW ANSWER

> "The trust policy controls who can assume an IAM role. In my project it restricted the GitHub OIDC identity to my repository and Production environment. The permission policy controls what the assumed role can do, such as the required Systems Manager operations."

---

# 151. WHY OIDC OVER AWS ACCESS KEYS?

> "OIDC avoids storing long-lived AWS access keys in GitHub. The workflow receives an identity token, AWS STS validates it, and temporary credentials are issued through the IAM role. This reduces the lifetime of credentials and follows a more identity-based authentication model."

---

# 152. WHY SSM OVER SSH?

> "I used AWS Systems Manager instead of storing SSH private keys in GitHub. GitHub Actions authenticated to AWS through OIDC and then used SSM to execute commands on the EC2 instance."

---

# 153. WHY SECRETS MANAGER?

> "I stored production database credentials in AWS Secrets Manager instead of committing them to Git or hardcoding them in Docker Compose. During deployment the EC2 instance retrieved the secret and used it to generate the runtime environment configuration."

---

# 154. WHY GIT SHA?

> "I use the full Git commit SHA as the Docker image tag so every image is traceable to an exact source revision. It also prevents ambiguity that comes with mutable tags such as `latest`."

---

# 155. WHY BUILD ONCE?

> "I wanted the artifact that passed security scanning to be exactly the artifact that was pushed and deployed. So I build the image once, save it as an artifact, load that same image for Trivy, and then push the same image."

---

# 156. WHAT IF TRIVY FAILS?

```text
Trivy
 ↓
HIGH / CRITICAL
 ↓
exit-code 1
 ↓
Job fails
 ↓
docker-push needs Trivy
 ↓
Docker Push does not run
 ↓
Production deployment does not start
```

This is the security gate.

---

# 157. WHAT IF THE IMAGE BUILD FAILS?

```text
Docker Build
   ↓
FAIL
   ↓
Trivy cannot run
   ↓
Push cannot run
   ↓
Deployment cannot run
```

Because of:

```yaml
needs:
```

---

# 158. WHAT IF SAST FAILS?

Our main Docker build has:

```yaml
needs:
  - lint
  - build-test
  - sast
  - secret-scan
  - dependency-scan
  - dockerfile-lint
```

So if SAST fails:

```text
SAST
 ↓
FAIL
 ↓
Docker Build cannot proceed
```

assuming the normal `needs` success behavior.

---

# 159. WHY SEPARATE PR AND MAIN PIPELINES?

Because a PR should validate the change without publishing or deploying it.

```text
PR
→ Validate

main
→ Build + Promote + Deploy
```

This is safer and easier to reason about.

---

# 160. FINAL PROJECT FILE STRUCTURE

```text
secure-cicd-pipeline-lab/
│
├── app/
│   ├── app.py
│   ├── templates/
│   │   └── index.html
│   └── static/
│       ├── style.css
│       └── app.js
│
├── tests/
│   └── test_app.py
│
├── .github/
│   └── workflows/
│       ├── main-pipeline.yml
│       ├── pr-pipeline.yml
│       ├── reusable-lint.yml
│       ├── reusable-build-test.yml
│       ├── reusable-sast.yml
│       ├── reusable-secret-scan.yml
│       ├── reusable-dependency-scan.yml
│       ├── reusable-dependency-review.yml
│       ├── reusable-dockerfile-lint.yml
│       ├── reusable-docker-build.yml
│       ├── reusable-trivy-scan.yml
│       ├── reusable-docker-push.yml
│       └── reusable-production-deploy.yml
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
└── .dockerignore
```

---

# 161. IMPORTANT PRODUCTION DOCKERFILE

Our application uses:

```dockerfile
FROM python:3.12-alpine

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ ./app/

EXPOSE 5000

CMD ["python", "app/app.py"]
```

Mental flow:

```text
Base Image
 ↓
Workdir
 ↓
Copy dependencies
 ↓
Install dependencies
 ↓
Copy application
 ↓
Expose port
 ↓
Start Flask
```

---

# 162. `.dockerignore`

Our project ignores:

```text
.venv/
.git/
.pytest_cache/
__pycache__/
*.pyc
.env
requiredcmd.txt
tests/
```

Why?

```text
Smaller build context
Fewer unnecessary files
Reduced accidental secret inclusion
Cleaner image build
```

---

# 163. DOCKERIGNORE VS DOCKERFILE `COPY`

A common confusion:

`.dockerignore` prevents files from entering the **Docker build context**.

But Dockerfile commands can create new content during the build.

For example:

```dockerfile
RUN pip install -r requirements.txt
```

can create:

```text
site-packages
```

even though there was no `site-packages` directory in the original build context.

Mental model:

```text
.dockerignore
→ What enters build context

Dockerfile RUN
→ What gets created during image build
```

---

# 164. ENVIRONMENT CONFIGURATION MENTAL MODEL

Do not mix these up:

```text
GitHub expression
${{ github.sha }}

Shell variable
$IMAGE_TAG

GitHub environment
env:
  IMAGE_TAG: ...

Docker Compose variable
${IMAGE_TAG}
```

They belong to different evaluation systems.

---

# 165. COMMON CONFUSION — `$GITHUB_ENV` VS `$GITHUB_OUTPUT`

This is very important.

## `$GITHUB_ENV`

Used to make an environment variable available to later steps in the same job.

Example:

```bash
echo "COMMAND_ID=$COMMAND_ID" >> "$GITHUB_ENV"
```

Later step:

```bash
echo "$COMMAND_ID"
```

Mental model:

```text
Step A
 ↓
GITHUB_ENV
 ↓
Later Step B
```

## `$GITHUB_OUTPUT`

Used to create a step output.

Example:

```bash
echo "image-tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

Later:

```text
steps.meta.outputs.image-tag
```

Mental model:

```text
Step A
 ↓
GITHUB_OUTPUT
 ↓
Step output
```

---

# 166. `$GITHUB_ENV` VS `$GITHUB_OUTPUT`

| `$GITHUB_ENV` | `$GITHUB_OUTPUT` |
|---|---|
| Environment variable | Step output |
| Used by later steps | Referenced through `steps.<id>.outputs` |
| Same job | Can flow into job/workflow outputs |
| Example: `COMMAND_ID` | Example: `image-tag` |

Easy memory:

```text
ENV
→ Environment

OUTPUT
→ Data passed as output
```

---

# 167. COMMON CONFUSION — `needs` VS `outputs`

```text
needs
→ Dependency

outputs
→ Data
```

Example:

```yaml
deploy:
  needs: build
```

means:

```text
Wait for build
```

while:

```yaml
${{ needs.build.outputs.image_tag }}
```

means:

```text
Get the image_tag produced by build
```

Together:

```text
needs
+
outputs
```

means:

```text
Wait for Job A
then retrieve data from Job A
```

---

# 168. COMMON CONFUSION — `env` VS `inputs`

```text
env
→ Runtime environment variable

inputs
→ Parameter passed into reusable workflow/action
```

Example:

```yaml
with:
  image-tag: ${{ github.sha }}
```

becomes:

```text
inputs.image-tag
```

while:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

becomes:

```text
$IMAGE_TAG
```

inside the shell step.

---

# 169. COMMON CONFUSION — `vars` VS `env`

```text
vars
→ Stored GitHub configuration value

env
→ Environment variable made available to a job/step/workflow
```

Example:

```yaml
DOCKER_USERNAME: ${{ vars.DOCKER_USERNAME }}
```

Here:

```text
vars.DOCKER_USERNAME
```

is read from GitHub configuration.

Then:

```text
DOCKER_USERNAME
```

can be exposed through `env`.

---

# 170. COMMON CONFUSION — OIDC VS IAM

Do not say:

> "OIDC gives permission."

Better:

```text
OIDC
→ Proves identity

IAM
→ Grants permission
```

---

# 171. COMMON CONFUSION — IAM ROLE VS IAM USER

Our deployment did not require an IAM user with permanent access keys.

Instead:

```text
GitHub
→ IAM Role via OIDC

EC2
→ IAM Role
```

Roles are assumed by trusted identities/services.

---

# 172. COMMON CONFUSION — STS VS IAM

```text
IAM
→ Defines trust + permissions

STS
→ Provides temporary credentials
```

Think:

```text
IAM = rules

STS = temporary credential issuer
```

---

# 173. COMMON CONFUSION — SSM VS SSH

```text
SSH
→ Network login using SSH credentials

SSM
→ AWS Systems Manager managed command execution
```

Our deployment uses SSM.

---

# 174. COMMON CONFUSION — SECRETS MANAGER VS GITHUB SECRETS

```text
GitHub Secrets
→ Secrets needed by GitHub Actions

AWS Secrets Manager
→ Runtime/production secrets managed inside AWS
```

Our production database credentials were stored in:

```text
AWS Secrets Manager
```

---

# 175. COMMON CONFUSION — IMAGE SCAN VS DEPENDENCY SCAN

They overlap but are not identical.

```text
Dependency scan
→ Application dependencies

Image scan
→ Final container image
→ OS packages + libraries + other image contents
```

This is why we use multiple security layers.

---

# 176. COMMON CONFUSION — HEALTH CHECK VS SECURITY CHECK

```text
Health check
→ Is application functioning?

Security check
→ Is application/image/code meeting security policy?
```

A healthy vulnerable application is still vulnerable.

---

# 177. COMMON PROJECT PROBLEMS AND LESSONS

## Trivy vulnerability results

Lesson:

```text
A successful build does not mean a secure image.
```

---

## SARIF severity handling

Problem:

```text
SARIF results did not initially behave exactly as expected.
```

Solution:

```yaml
limit-severities-for-sarif: true
```

Lesson:

```text
Always validate the actual security report format.
```

---

## Build Once issue

Lesson:

```text
Do not rebuild after scanning.
```

Use:

```text
Build
 ↓
Artifact
 ↓
Scan same image
 ↓
Push same image
```

---

## SSM stderr confusion

Problem:

```text
Docker Compose output appeared in stderr.
```

Lesson:

```text
stderr content alone does not prove failure.
```

Check:

```text
Status
ResponseCode
```

---

## Production `.env` confusion

Lesson:

```text
GitHub expressions are evaluated by GitHub.
Compose variables are evaluated by Compose.
```

---

## MySQL persistence

Lesson:

```text
Never blindly use docker compose down -v
when the database volume must persist.
```

---

# 178. ERROR-DEBUGGING FORMULA

When a workflow fails, do not randomly change commands.

Think:

```text
1. Which job failed?
2. Which step failed?
3. What exact error?
4. Is it YAML?
5. Is it permission?
6. Is it authentication?
7. Is it environment?
8. Is it artifact?
9. Is it Docker?
10. Is it AWS?
11. Is it application runtime?
```

---

# 179. GITHUB ACTIONS DEBUGGING MAP

```text
Workflow doesn't start
→ Check trigger

Workflow starts but syntax error
→ Check YAML

Job doesn't run
→ Check needs / if

Command not found
→ Check runner/environment

Permission denied
→ Check permissions

Output empty
→ Check id + GITHUB_OUTPUT

Value missing between jobs
→ Check job outputs + needs

Artifact missing
→ Check upload/download names

Docker image missing
→ Check build/tag/artifact

Trivy wrong image
→ Check docker load + image_ref

AWS authentication fails
→ Check OIDC / trust policy

AWS API denied
→ Check IAM permission policy

SSM fails
→ Check instance role / managed status / command response

Secret unavailable
→ Check Secrets Manager + IAM

Compose fails
→ Check .env + config + image

Application unhealthy
→ Check container + DB + health endpoint
```

---

# 180. INTERVIEW QUESTION — WHAT IS A CI/CD PIPELINE?

> "A CI/CD pipeline is an automated sequence that takes source-code changes through validation, building, testing, packaging, security checks and delivery or deployment. In my project I implemented separate pull-request validation and main-branch delivery pipelines using GitHub Actions."

---

# 181. INTERVIEW QUESTION — WHAT IS A RUNNER?

> "A runner is the execution environment where GitHub Actions runs a job. In my project I primarily used GitHub-hosted Ubuntu runners."

---

# 182. INTERVIEW QUESTION — WHAT IS `needs`?

> "`needs` defines job dependencies. For example, my Trivy scan needs the Docker build because the image must exist first, and Docker Push needs the Trivy scan so the image is not pushed before the security gate passes."

---

# 183. INTERVIEW QUESTION — WHAT IS AN ARTIFACT?

> "An artifact is a file generated by one workflow stage and stored so it can be consumed later. In my pipeline I saved the exact Docker image as a compressed TAR artifact so the Trivy job could load and scan the same image without rebuilding it."

---

# 184. INTERVIEW QUESTION — INPUTS VS OUTPUTS?

> "Inputs are values passed into a reusable workflow, usually through `with`. Outputs are values produced by a reusable workflow and consumed by the caller. Internally I create step outputs through `$GITHUB_OUTPUT`, expose them as job outputs, and then consume them through `needs.<job>.outputs.<name>`."

---

# 185. INTERVIEW QUESTION — `env` LEVELS?

> "GitHub Actions environment variables can be defined at workflow, job and step scope. Workflow scope is broadest, job scope applies to that job, and step scope applies only to that step. A more specific scope overrides a broader value with the same name."

---

# 186. INTERVIEW QUESTION — `workflow_call`?

> "`workflow_call` makes a workflow reusable. I used reusable workflows to separate linting, testing, security scanning, Docker build, image scanning, Docker push and production deployment."

---

# 187. INTERVIEW QUESTION — WHY REUSABLE WORKFLOWS?

> "They reduce duplication and make a large pipeline easier to maintain. My PR and main pipelines can call the same reusable security and build workflows while passing different inputs."

---

# 188. INTERVIEW QUESTION — WHAT IS DEVSECOPS?

> "DevSecOps means integrating security into the DevOps lifecycle. In my project I added SAST, secret scanning, dependency scanning, Dockerfile linting, container image scanning, least-privilege permissions and secure AWS authentication rather than treating security as a final separate step."

---

# 189. INTERVIEW QUESTION — WHAT IS SAST?

> "SAST analyzes source code without running the application. I used Semgrep."

---

# 190. INTERVIEW QUESTION — WHAT IS DAST?

> "DAST tests the running application dynamically. During the earlier NodeGoat work I used OWASP ZAP for this."

---

# 191. INTERVIEW QUESTION — WHY GITLEAKS?

> "Gitleaks scans the repository for exposed secrets such as tokens, API keys and credentials. I also used full Git history checkout so historical commits could be scanned."

---

# 192. INTERVIEW QUESTION — WHY PIP-AUDIT?

> "The final application is Python-based, so I used pip-audit to identify known vulnerabilities in Python dependencies."

---

# 193. INTERVIEW QUESTION — WHY DEPENDENCY REVIEW?

> "Dependency Review is useful on pull requests because it evaluates dependency changes introduced by that PR before they are merged."

---

# 194. INTERVIEW QUESTION — WHY HADOLINT?

> "Hadolint checks the Dockerfile for common problems and Docker best-practice violations."

---

# 195. INTERVIEW QUESTION — WHY TRIVY?

> "Trivy scans the final container image for known vulnerabilities in OS packages and application libraries. I configured the production security gate around HIGH and CRITICAL findings."

---

# 196. INTERVIEW QUESTION — WHAT IS A SECURITY GATE?

> "A security gate is the policy that determines whether the pipeline can continue based on a security result. The scanner finds the issue; the gate enforces the release decision."

---

# 197. INTERVIEW QUESTION — WHAT IS OIDC?

> "OIDC is an identity mechanism that allowed my GitHub Actions workflow to authenticate to AWS without storing long-lived AWS access keys. AWS trusts GitHub's OIDC identity, STS exchanges that trust for temporary credentials, and the workflow assumes an IAM role."

---

# 198. INTERVIEW QUESTION — WHAT IS STS?

> "AWS STS, or Security Token Service, provides temporary AWS credentials. In my deployment, STS validated the GitHub web identity and allowed the workflow to assume the production deployment IAM role."

---

# 199. INTERVIEW QUESTION — WHAT IS SSM?

> "AWS Systems Manager allowed my GitHub Actions workflow to remotely execute deployment commands on the EC2 instance without storing an SSH private key in GitHub."

---

# 200. INTERVIEW QUESTION — WHAT IS SECRETS MANAGER?

> "AWS Secrets Manager stores sensitive production values such as database credentials. My deployment retrieved those values on the EC2 instance rather than storing them in the Git repository."

---

# 201. INTERVIEW QUESTION — HOW DID YOU DEPLOY TO EC2?

> "After the exact image passed CI and Trivy, GitHub Actions waited for Production environment approval. It then authenticated to AWS with OIDC, assumed the GitHubActions-Production-Deploy role through STS, used SSM to execute commands on EC2, retrieved production credentials from Secrets Manager, downloaded the exact Compose file for the Git SHA, pulled the exact Docker image from Docker Hub, recreated the containers with Docker Compose, and verified the `/health` endpoint."

---

# 202. INTERVIEW QUESTION — WHY TWO IAM ROLES?

> "The GitHub Actions role and EC2 role have different trust boundaries. GitHubActions-Production-Deploy is assumed by GitHub through OIDC and controls what the CI/CD workflow can do. ProductionEC2-SSM-Role is attached to EC2 and controls what the server itself can do."

---

# 203. INTERVIEW QUESTION — WHY NOT ONE ROLE?

Because:

```text
GitHub workload
and
EC2 workload
```

are different identities.

Using separate roles supports:

```text
Least privilege
Clear trust boundaries
Easier auditing
```

---

# 204. INTERVIEW QUESTION — WHY NOT AWS ACCESS KEYS?

> "I wanted to avoid long-lived credentials in CI/CD. OIDC with STS provides temporary credentials tied to the GitHub workload and IAM trust policy."

---

# 205. INTERVIEW QUESTION — WHY NOT SSH?

> "SSM gave me an AWS-managed deployment channel without storing an SSH private key in GitHub. Authorization is controlled through IAM."

---

# 206. INTERVIEW QUESTION — HOW DO YOU STORE DB PASSWORD?

> "The production database credentials were stored in AWS Secrets Manager. The EC2 instance accessed the secret through its IAM role, and deployment generated the runtime `.env` configuration on the server."

---

# 207. INTERVIEW QUESTION — WHAT IF TRIVY FINDS HIGH?

```text
Trivy
 ↓
HIGH
 ↓
exit-code 1
 ↓
Job fails
 ↓
Docker Push waits on Trivy
 ↓
Push blocked
 ↓
Production deployment never starts
```

---

# 208. INTERVIEW QUESTION — WHAT IS THE DIFFERENCE BETWEEN `latest` AND SHA TAG?

> "`latest` is mutable, while a Git SHA tag identifies a specific source revision. I use the full Git SHA so the deployed image can be traced directly to the commit that created it."

---

# 209. INTERVIEW QUESTION — WHY ARTIFACT?

> "The artifact allows me to transfer the exact Docker image between GitHub Actions jobs. That lets the Trivy job scan the same image that the build job produced without rebuilding."

---

# 210. INTERVIEW QUESTION — WHAT IS THE DIFFERENCE BETWEEN `GITHUB_ENV` AND `GITHUB_OUTPUT`?

> "`GITHUB_ENV` exposes an environment variable to later steps in the same job. `GITHUB_OUTPUT` creates a step output that can be referenced through the step ID and can then be promoted to job and workflow outputs."

---

# 211. INTERVIEW QUESTION — `needs` VS `if`

> "`needs` controls job dependency and ordering, while `if` controls whether a job or step should execute based on a condition."

---

# 212. INTERVIEW QUESTION — `workflow_call` VS `workflow_run`

> "`workflow_call` is used to reuse another workflow. `workflow_run` is used to trigger a workflow after another workflow completes."

---

# 213. INTERVIEW QUESTION — `run` VS `uses`

> "`run` executes a shell command. `uses` invokes an existing GitHub Action."

---

# 214. 30-SECOND PROJECT ANSWER

> "I built a secure CI/CD pipeline for a containerized Flask and MySQL application using GitHub Actions. I separated PR validation from the main deployment pipeline and used reusable workflows for linting, testing, SAST, secret scanning, dependency scanning, Dockerfile linting, Docker build, Trivy scanning, Docker push and production deployment. The image is built once, tagged with the full Git SHA, saved as an artifact, scanned with Trivy and then the same image is pushed and deployed. For AWS, I implemented OIDC-based authentication with STS and IAM, used SSM for EC2 deployment and Secrets Manager for production database credentials, followed by a health check."

---

# 215. 60-SECOND PROJECT ANSWER

> "My Secure CI/CD Pipeline Lab is a Flask and MySQL application containerized with Docker and deployed through GitHub Actions. I created reusable workflows so PRs run linting, tests, Semgrep SAST, Gitleaks, pip-audit, Dependency Review and Hadolint, while the main pipeline additionally builds the Docker image, saves the exact image as an artifact, loads it for Trivy scanning and pushes the same Git-SHA-tagged image to Docker Hub only after the security gate passes.
>
> For production deployment I used a GitHub Production environment approval followed by OIDC-based AWS authentication. GitHub gets an OIDC token, AWS STS validates it and allows the workflow to assume a dedicated IAM role. The workflow then uses SSM to execute commands on EC2. The EC2 instance has its own IAM role and retrieves production MySQL credentials from Secrets Manager. Docker Compose pulls the exact image, preserves the MySQL volume and starts the application. Finally, the pipeline verifies the `/health` endpoint."

---

# 216. 90-SECOND "EXPLAIN YOUR AWS DEPLOYMENT"

Use this structure:

```text
1. Build
2. Scan
3. Push
4. Approval
5. OIDC
6. STS
7. IAM
8. SSM
9. Secrets Manager
10. Docker Compose
11. Health Check
```

Spoken answer:

> "First, GitHub Actions validates and builds the application image. I tag it with the full Git commit SHA and save the exact image as an artifact. Trivy scans that same image for HIGH and CRITICAL vulnerabilities. Only if the security gate passes do I push the image to Docker Hub. Then the Production GitHub Environment requires approval.
>
> After approval, the deployment workflow requests an OIDC token. AWS trusts GitHub's OIDC provider, STS validates the token and allows the workflow to assume the GitHubActions-Production-Deploy IAM role. That role is authorized to use Systems Manager for the production EC2 instance.
>
> SSM executes the deployment commands on EC2. The EC2 instance has the ProductionEC2-SSM-Role, which allows the server to interact with AWS and retrieve the production secret from Secrets Manager. The deployment generates the runtime `.env`, downloads the exact Compose file for the Git SHA, pulls the exact Docker image, runs Docker Compose, and checks `/health` to confirm that the Flask application can communicate with MySQL."

---

# 217. MASTER PROJECT FLOW

```text
Developer
   │
   ▼
Pull Request
   │
   ├── Lint
   ├── Test
   ├── Semgrep
   ├── Gitleaks
   ├── pip-audit
   ├── Dependency Review
   └── Hadolint
   │
   ▼
Merge to main
   │
   ▼
Main Pipeline
   │
   ├── Lint
   ├── Test
   ├── SAST
   ├── Secret Scan
   ├── Dependency Scan
   └── Dockerfile Lint
   │
   ▼
Docker Build
   │
   ▼
Full Git SHA Image
   │
   ▼
docker save
   │
   ▼
GitHub Artifact
   │
   ▼
docker load
   │
   ▼
Trivy
   │
   ├── FAIL → STOP
   │
   └── PASS
        │
        ▼
    Docker Push
        │
        ▼
Production Approval
        │
        ▼
      OIDC
        │
        ▼
      STS
        │
        ▼
      IAM
        │
        ▼
      SSM
        │
        ▼
      EC2
        │
        ├── Secrets Manager
        │       ↓
        │    DB credentials
        │
        └── Docker Compose
                │
          ┌─────┴─────┐
          ▼           ▼
        Flask        MySQL
          │           │
          └─────┬─────┘
                ▼
            /health
                │
                ▼
          Deployment Success
```

---

# 218. DAY 38 → DAY 49 LEARNING STORY

Remember the progression like this:

```text
YAML
 ↓
Understand configuration

CI/CD
 ↓
Understand automation

GitHub Actions
 ↓
Understand Workflow / Job / Step / Runner

Triggers
 ↓
Control WHEN pipeline runs

Jobs + needs
 ↓
Control dependency

env
 ↓
Pass runtime configuration

if
 ↓
Control behavior

Secrets
 ↓
Protect sensitive data

Artifacts
 ↓
Move outputs between jobs

Cache
 ↓
Speed up repeated work

Matrix
 ↓
Test multiple configurations

Inputs
 ↓
Pass data INTO reusable workflows

Outputs
 ↓
Pass data OUT

Reusable Workflows
 ↓
Avoid duplication

Composite Actions
 ↓
Reuse steps

workflow_run
 ↓
React to completed workflows

repository_dispatch
 ↓
External triggering

Docker Build/Push
 ↓
Package application

Git SHA
 ↓
Traceability

PR Pipeline
 ↓
Validate before merge

Main Pipeline
 ↓
Build → Scan → Push → Deploy

DevSecOps
 ↓
Security everywhere

Semgrep
 ↓
Source security

Gitleaks
 ↓
Secret security

pip-audit / Dependency Review
 ↓
Dependency security

Hadolint
 ↓
Dockerfile security

Trivy
 ↓
Image security

OIDC
 ↓
GitHub identity to AWS

STS
 ↓
Temporary AWS credentials

IAM
 ↓
AWS authorization

SSM
 ↓
Remote EC2 execution

Secrets Manager
 ↓
Production secret storage

Docker Compose
 ↓
Production runtime

Health Check
 ↓
Deployment verification
```

---

# 219. MASTER CHEAT SHEET — WHAT TO REMEMBER FIRST

When revising quickly, remember these relationships:

```text
Workflow
→ Automation

Job
→ Logical unit

Step
→ Individual action

Runner
→ Execution machine

Trigger
→ When to run

needs
→ Dependency

if
→ Condition

env
→ Runtime configuration

inputs
→ Data entering reusable workflow

outputs
→ Data leaving reusable workflow

secrets
→ Sensitive values

artifact
→ Stored output

cache
→ Reuse for speed

matrix
→ Multiple configurations

workflow_call
→ Reuse workflow

workflow_run
→ Trigger after workflow

repository_dispatch
→ External trigger
```

---

# 220. SECURITY MASTER MEMORY

```text
Semgrep
→ Code security

Gitleaks
→ Secret security

pip-audit
→ Python dependency security

Dependency Review
→ PR dependency changes

Hadolint
→ Dockerfile

Trivy
→ Docker image

ZAP
→ Running web application
```

---

# 221. AWS MASTER MEMORY

```text
OIDC
→ GitHub identity

STS
→ Temporary AWS credentials

IAM
→ Authorization

Trust Policy
→ Who can assume role?

Permission Policy
→ What can role do?

SSM
→ Execute commands on EC2

Secrets Manager
→ Store production secrets

EC2
→ Production server
```

---

# 222. THE 5 MOST IMPORTANT FLOWS

## Flow 1 — GitHub Actions

```text
Trigger
 ↓
Workflow
 ↓
Job
 ↓
Step
 ↓
Runner
```

## Flow 2 — Outputs

```text
Step
 ↓
GITHUB_OUTPUT
 ↓
Step Output
 ↓
Job Output
 ↓
needs.job.outputs
```

## Flow 3 — DevSecOps

```text
Code
 ↓
SAST
 ↓
Secrets
 ↓
Dependencies
 ↓
Dockerfile
 ↓
Image
 ↓
Runtime
```

## Flow 4 — Docker Promotion

```text
Build
 ↓
Save
 ↓
Artifact
 ↓
Load
 ↓
Scan
 ↓
Push
 ↓
Deploy
```

## Flow 5 — AWS Deployment

```text
GitHub
 ↓
OIDC
 ↓
STS
 ↓
IAM
 ↓
SSM
 ↓
EC2
 ↓
Secrets Manager
 ↓
Docker Compose
 ↓
Health Check
```

---

# 223. FINAL INTERVIEW FORMULA

For almost every DevOps question, think:

```text
WHAT?
→ Definition

WHY?
→ Problem it solves

HOW?
→ Implementation

WHERE?
→ Where I used it

PROBLEM?
→ What went wrong

FIX?
→ How I solved it

RESULT?
→ What the final design achieved
```

Example:

```text
Question:
Why did you use Trivy?

WHAT?
→ Container image vulnerability scanner

WHY?
→ Detect known vulnerabilities before release

HOW?
→ Scan exact image after Docker build

WHERE?
→ Main GitHub Actions pipeline

PROBLEM?
→ Build and scan could become different artifacts

FIX?
→ Build once → save artifact → load same image → scan → push

RESULT?
→ The pushed image is the same image that passed security scanning
```

---

# 224. FINAL PROJECT INTERVIEW FORMULA

When explaining the entire project:

```text
Application
    ↓
Docker
    ↓
GitHub Actions
    ↓
Reusable Workflows
    ↓
CI Validation
    ↓
DevSecOps Security
    ↓
Build Once
    ↓
Scan Same Image
    ↓
Push Same Image
    ↓
Production Approval
    ↓
OIDC
    ↓
STS
    ↓
IAM
    ↓
SSM
    ↓
EC2
    ↓
Secrets Manager
    ↓
Docker Compose
    ↓
Health Check
```

Then explain the WHY:

```text
Reusable Workflows
→ Maintainability

Git SHA
→ Traceability

Artifacts
→ Same build artifact across jobs

Trivy
→ Image security

Security Gate
→ Prevent insecure promotion

OIDC
→ Avoid long-lived AWS credentials

IAM
→ Least privilege

SSM
→ Secure AWS-managed remote execution

Secrets Manager
→ Secure production secrets

Health Check
→ Verify real application state
```

---

# 225. MASTER ONE-LINER

> **"I built a reusable GitHub Actions CI/CD pipeline and evolved it into a DevSecOps pipeline for a containerized Flask-MySQL application. I added source-code, secret, dependency, Dockerfile and image security checks; implemented Build Once → Scan Same Image → Push Same Image using Git-SHA tags and artifacts; and deployed the validated image to AWS EC2 using GitHub OIDC, AWS STS, IAM, SSM and Secrets Manager, followed by a production health check."**

---

# 🔥 FINAL MEMORY MAP

```text
                    ┌─────────────────────┐
                    │     GITHUB ACTIONS  │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
             TRIGGERS        JOBS          RUNNERS
                │              │
                │           needs / if
                │              │
                └───────┬──────┘
                        │
                     STEPS
                        │
          ┌─────────────┼─────────────┐
          │             │             │
         env         inputs        outputs
          │             │             │
          │          reusable       GITHUB_OUTPUT
          │          workflow           │
          │                             │
          │                           jobs
          │                             │
          └─────────────┬───────────────┘
                        │
                  Docker CI/CD
                        │
            ┌───────────┴───────────┐
            │                       │
          Build                   Push
            │
         Git SHA
            │
         Artifact
            │
          Trivy
            │
       Security Gate
            │
            ▼
       Docker Hub
            │
            ▼
   Production Approval
            │
           OIDC
            │
           STS
            │
           IAM
            │
           SSM
            │
           EC2
            │
     ┌──────┴──────┐
     │             │
 Secrets Manager  Docker Compose
     │             │
     │        ┌────┴────┐
     │        │         │
     │      Flask     MySQL
     │        │         │
     └────────┴────┬────┘
                   │
               Health Check
                   │
                   ▼
              PRODUCTION
```

---

# 🏁 FINAL REVISION RULE

Do not try to memorize this cheat sheet from top to bottom every time.

Use this order:

```text
1. GitHub Actions hierarchy
   Workflow → Job → Step → Runner

2. Execution control
   on → needs → if

3. Data movement
   env → inputs → outputs → secrets → artifacts

4. Reuse
   workflow_call → composite action

5. Docker CI/CD
   build → tag → artifact → scan → push

6. DevSecOps
   SAST → secrets → dependencies → Dockerfile → image → runtime

7. AWS authentication
   OIDC → STS → IAM

8. AWS deployment
   SSM → EC2

9. Production secrets
   Secrets Manager

10. Final deployment
   Docker Compose → health check
```

---

# 🚀 THE ONE STORY I SHOULD BE ABLE TO EXPLAIN WITHOUT LOOKING AT NOTES

```text
Developer
   ↓
Pull Request
   ↓
Lint + Test + Security
   ↓
Merge
   ↓
Push main
   ↓
Lint + Test + Security
   ↓
Docker Build
   ↓
Git SHA Tag
   ↓
Save Image as Artifact
   ↓
Load Same Image
   ↓
Trivy
   ↓
Security Gate
   ↓
Docker Hub
   ↓
Production Approval
   ↓
GitHub OIDC
   ↓
AWS STS
   ↓
IAM Role
   ↓
SSM
   ↓
EC2
   ↓
EC2 IAM Role
   ↓
Secrets Manager
   ↓
Production .env
   ↓
Docker Compose
   ↓
Flask + MySQL
   ↓
Health Check
   ↓
Deployment Verified
```

> **If I understand this flow and can explain WHY every arrow exists, I understand the core of what I built from Day 38 to Day 49.**

---

# 🔥 FINAL MASTER STATEMENT

> **"My approach is not just to remember GitHub Actions syntax. I understand the pipeline as a system: triggers decide when it starts, jobs define logical stages, needs controls dependencies, env and inputs move configuration into stages, outputs move generated data between stages, artifacts preserve exact build outputs, reusable workflows prevent duplication, security tools inspect different layers, security gates enforce policy, Git SHA tags provide traceability, OIDC authenticates GitHub to AWS without long-lived credentials, STS provides temporary credentials, IAM authorizes actions, SSM executes deployment commands on EC2, Secrets Manager protects runtime secrets, Docker Compose runs the production stack, and the health check verifies that the deployment actually works."**

# 🎯 MASTER INTERVIEW GOAL

For any topic in this sheet, I should be able to explain:

```text
WHAT IS IT?
      ↓
WHY DO WE NEED IT?
      ↓
HOW DOES IT WORK?
      ↓
WHERE DID I USE IT?
      ↓
WHAT PROBLEM DID I FACE?
      ↓
HOW DID I FIX IT?
```

That is the level of understanding I am aiming for as a fresher DevOps Engineer.
