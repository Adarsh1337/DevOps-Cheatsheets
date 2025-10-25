# GitHub Actions Cheatsheet

## Table of Contents
- Overview
- Quick Start
- Workflow Syntax
- Common Jobs and Reusable Workflows
- Caching and Artifacts
- Secrets and Security
- Matrix Builds
- Best Practices
- Troubleshooting

## Overview
GitHub Actions is a CI/CD platform to automate builds, tests, and deployments using YAML workflows in .github/workflows.

## Quick Start
```yaml
# .github/workflows/ci.yaml
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test --silent
```

## Workflow Syntax
- name, on, jobs, runs-on, steps
- uses for actions, run for shell commands, with for inputs
- env at job or step level

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    env:
      NODE_ENV: test
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint
```

## Common Jobs and Reusable Workflows
```yaml
# Reusable workflow: .github/workflows/reusable-ci.yaml
name: Reusable CI
on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci && npm test
```
```yaml
# Caller
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable-ci.yaml
    with:
      node-version: '20'
```

## Caching and Artifacts
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

- uses: actions/upload-artifact@v4
  with:
    name: build
    path: dist/
```

## Secrets and Security
- Store secrets in Settings > Secrets and variables > Actions
- Use environments and required reviewers for prod
- Principle of least privilege for tokens
```yaml
env:
  NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
permissions:
  contents: read
  packages: write
```

## Matrix Builds
```yaml
strategy:
  matrix:
    node: [18, 20]
    os: [ubuntu-latest, windows-latest]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with:
      node-version: ${{ matrix.node }}
```

## Best Practices
1. Pin action versions (use major versions or SHAs)
2. Cache dependencies to speed up builds
3. Use reusable workflows for DRY pipelines
4. Limit default GITHUB_TOKEN permissions
5. Fail fast and surface test results

## Troubleshooting
- Check Actions logs and step summaries
- Enable step debug: set secret ACTIONS_STEP_DEBUG=true
- Rerun jobs with SSH for self-hosted

---
Last Updated: October 2025
Version: Actions runner 2.x
