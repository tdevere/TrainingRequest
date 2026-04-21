---
title: "From commit to running service: your first deployment"
slug: "from-commit-to-running-service-your-first-deployment"
request_issue: 1
level: "beginner"
format: ["written", "lab"]
duration: "~2 hours"
owner_sme: "tbd"
status: "fixture"
created: "2026-04-21"
last_updated: "2026-04-21"
tags: ["github-actions", "nodejs", "deployment", "ci-cd", "beginner"]
---

# From commit to running service: your first deployment

> Links back to request: https://github.com/tdevere/TrainingRequest/issues/1

## Learning outcomes

After completing this training, you will be able to:

- Explain how **build**, **release**, and **deploy** are different stages and why teams separate them.
- Configure a minimal GitHub Actions workflow that builds/tests a Node.js service and deploys it to a non-production target.
- Roll back a bad deployment by redeploying the last known-good version.
- Read deployment logs and classify common failure types (build error, config/secret error, runtime/startup error, and environment connectivity error).

## Prerequisites

- Daily Git usage (commit, branch, PR, merge)
- Basic JavaScript / Node.js
- Basic Linux shell commands
- A GitHub repository where you can enable Actions

## Outline

1. **Mental model: build vs release vs deploy (20 min)**
   - stage definitions
   - promotion and rollback basics
2. **Build your first pipeline (40 min)**
   - minimal Node CI with GitHub Actions
   - deployment job structure and required secrets
3. **Hands-on deployment + rollback drill (45 min)**
   - perform one good deployment
   - simulate and recover from one bad deployment
4. **Reading logs without panic (15 min)**
   - where to look first
   - failure-class triage checklist

## Content

### 1) Build, release, deploy: one sentence each

- **Build**: turn source code into a runnable artifact (for Node, this is often tested source + dependency lockfile + build output if applicable).
- **Release**: create a versioned candidate artifact and metadata (for example, image tag `v1.2.3` or commit SHA) that is approved for use.
- **Deploy**: put that released artifact into an environment and start serving traffic.

Why this matters: if deploy fails, you want to redeploy a previous *release* without rebuilding from scratch.

### 2) Minimal pipeline shape for a small Node service

Use two jobs:

1. `build_and_test`
   - checkout
   - setup Node
   - `npm ci`
   - `npm test`
2. `deploy_staging` (depends on build)
   - authenticate to target
   - deploy current commit artifact
   - smoke check endpoint

Starter workflow:

```yaml
name: ci-deploy

on:
  push:
    branches: [main]

jobs:
  build_and_test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm
      - run: npm ci
      - run: npm test

  deploy_staging:
    needs: build_and_test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: |
          echo "Deploying ${GITHUB_SHA} to staging"
          # replace with your real deploy command
```

> Keep production deploys manual at first (`workflow_dispatch` or required reviewers on protected environments).

### 3) Rollback pattern you can use immediately

- Always deploy a traceable version (commit SHA or semver tag).
- Store "last known good" in your deployment history/tooling.
- Rollback = redeploy that exact previous version.

Simple rollback runbook:

1. Confirm incident scope (who is affected, since when).
2. Find last healthy deployment version in deploy history.
3. Redeploy previous version.
4. Validate health checks and key user path.
5. Open a follow-up issue for root cause; do not hot-edit in panic.

### 4) Log-reading checklist (first 5 minutes)

1. **Workflow run summary**: which step failed?
2. **Build logs**: test failure, missing dependency, syntax/type error.
3. **Deploy logs**: auth failure, missing secret, CLI/config mismatch.
4. **Runtime logs** (after deploy): app boot errors, port mismatch, DB/network reachability.

Common failure classes:

- **Build-time**: failing tests, lint/type failure.
- **Configuration/secret**: missing env var, invalid credentials.
- **Runtime/startup**: process exits, health check fails, wrong port.
- **Environment/dependency**: DB endpoint blocked, DNS/network issues.

## Hands-on lab

### Setup

1. Create a small Node API (or use an existing hello-world service).
2. Ensure `npm test` runs (even one smoke test is enough for this lab).
3. Add `.github/workflows/ci-deploy.yml` using the starter workflow.
4. In GitHub repo settings, create a **staging** environment.
5. Add required deployment secrets (if your deploy command needs them).

### Exercises

1. **Happy path deploy**
   - Push a small change to `main`.
   - Confirm `build_and_test` passes.
   - Confirm `deploy_staging` runs and logs the deployed SHA.

2. **Simulated bad deploy + rollback**
   - Intentionally break startup (for example, require a missing env var).
   - Push to `main` and observe failed health check/runtime behavior.
   - Roll back by redeploying the previous known-good SHA.

3. **Failure classification drill**
   - Capture one failed run log.
   - Label the failure class from the checklist.
   - Write one prevention action (example: add startup validation test).

### Solution

Expected evidence of completion:

- One successful deployment run with commit SHA visible in logs.
- One failed deployment or runtime event diagnosed by failure class.
- One successful rollback to previous SHA.
- A short notes file (or issue comment) with: failure class, root cause, and prevention step.

## References

- GitHub Docs — Understanding GitHub Actions: https://docs.github.com/actions/learn-github-actions/understanding-github-actions
- GitHub Docs — Deploying with GitHub Actions: https://docs.github.com/actions/deployment/about-deployments/deploying-with-github-actions
- GitHub Docs — Managing environments for deployment: https://docs.github.com/actions/deployment/targeting-different-environments/using-environments-for-deployment
- Node.js Docs — Getting started: https://nodejs.org/en/docs

## Feedback

- Feedback issues: <!-- auto-linked by workflow -->
- Known limitations / follow-ups:
  - This fixture keeps deployment target generic; adapt commands to your platform (container host, VM, or PaaS).
