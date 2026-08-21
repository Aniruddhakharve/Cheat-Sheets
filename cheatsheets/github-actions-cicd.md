# GitHub Actions CI/CD Cheat Sheet

## Quick Concepts
- Workflow: `.github/workflows/*.yml`
- Trigger: `on: [push, pull_request]`
- Jobs run in parallel by default
- Steps run sequentially

## Common Snippets
- Checkout code: `uses: actions/checkout@v4`
- Setup runtime: `uses: actions/setup-node@v4`
- Cache dependencies for faster builds

## Interview Pointers
- CI vs CD
- Matrix builds
- Secrets management
- Required status checks
