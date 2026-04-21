---
title: "From commit to running service: your first deployment"
slug: "from-commit-to-running-service-your-first-deployment"
request_issue: "#1"
level: "beginner"
format: ["written", "lab"]
duration: "135 minutes"
owner_sme: "tbd"
status: "draft"
created: "2026-04-21"
last_updated: "2026-04-21"
tags: ["devops", "deployment", "github-actions", "azure-app-service", "rollback", "oidc"]
---

# From commit to running service: your first deployment

> Links back to request: [#1](../../../issues/1)
>
> **Status:** draft — pending SME content verification. Approved plan:
> [Plan Acceptance Form comment](../../../issues/1#issuecomment-4290351538).

## Learning outcomes

After completing this training, you will be able to:

1. **Explain** the difference between *build*, *release*, and *deploy*
   stages in a deployment pipeline in your own words, naming what each
   stage produces, what promotes between stages, and what can revert.
2. **Configure** a minimal GitHub Actions workflow that builds a Node.js
   service **once** and deploys the **same** artifact to Azure App
   Service, tagged with the commit SHA.
3. **Roll back** a bad deployment by redeploying the prior artifact
   without rebuilding from an older commit.
4. **Diagnose** an ambiguous, multi-cause deployment failure by
   interpreting raw logs and distinguishing signal from noise.

## Prerequisites

Required:

- Daily Git usage (commit, branch, PR, merge).
- Basic JavaScript / Node.js — you can read and modify a small Express handler.
- Basic Linux shell (`cd`, `ls`, `cat`, piping).
- An Azure subscription with permission to create an App Service and an
  Entra ID app registration — or access to a sandbox your SME has
  pre-provisioned using the [one-time setup appendix](#appendix-a--one-time-azure-setup).

Not required (we'll cover what you need):

- Prior GitHub Actions authoring experience.
- Prior Azure Portal experience beyond signing in.
- Docker / containers.
- Terraform / Bicep.

## Out of scope

To protect your 135-minute time budget, the following are explicitly
**not** covered here. Each is named so you know what you still have left
to learn:

- Multi-environment promotion (dev → staging → prod) with approval gates.
- Blue/green, canary, or deployment-slot swap strategies.
- Database migrations (expand/contract, Flyway, EF migrations, …).
- Infrastructure as Code (Bicep, Terraform) — the App Service is created
  by hand in Appendix A.
- Observability beyond the post-deploy smoke check (SLOs, Application
  Insights dashboards, alert rules, distributed tracing). Look for the
  future "Production observability basics" training.
- Multi-region / disaster recovery.
- Container-based deploys (ACR → App Service Containers). That's the
  recommended next training.

## Outline

| # | Section | Time |
|---|---------|------|
| 1 | [Build vs release vs deploy — mental model](#section-1--build-vs-release-vs-deploy) | 20 min |
| 2 | [Your first pipeline with GitHub Actions](#section-2--your-first-pipeline-with-github-actions) | 45 min |
| 3 | [Hands-on lab: deploy, break, roll back, diagnose](#section-3--hands-on-lab) | 55 min |
| 4 | [Reading deployment logs without panic](#section-4--reading-deployment-logs-without-panic) | 15 min |
|   | **Total** | **135 min** |

---

## Content

### Section 1 — Build vs release vs deploy

A "deployment pipeline" is three distinct stages people often smush into
one word. Keeping them separate is the single highest-leverage habit you
can build as a new deployer.

| Stage | Produces | Promoted by | Reverted by |
|-------|----------|-------------|-------------|
| **Build** | An immutable artifact (a zip, a container image, a jar) identified by a version — typically the commit SHA. | Passing tests and packaging. | You don't "revert" a build; you build a new one. |
| **Release** | A decision: "this artifact is a candidate for environment X." Often manifests as a tag, a GitHub release, or a manual approval. | Human or automated gate. | Marking the release as withdrawn. |
| **Deploy** | A running instance of that artifact in a specific environment. | Uploading the same artifact to the target and routing traffic to it. | Re-deploying the **previous** artifact. |

Three consequences you should tattoo somewhere:

1. **Never rebuild from a tag to roll back.** A rebuild can pick up a
   different dependency, a different base image, a different build
   environment. It is a *different* artifact wearing the same name.
2. **The artifact must be immutable.** If your "artifact" is "whatever
   `main` produces today," you don't have artifacts, you have a
   recurring coincidence.
3. **Rollback is a deploy, not a build.** If your mental model can't
   tell the difference, your rollback drill will silently fail under
   pressure.

#### Checkpoint 1

In your own words, write one paragraph (3–5 sentences) answering:

> If I merge a change and tests pass, why is redeploying the resulting
> artifact tomorrow not the same as "promoting today's release to
> production tomorrow"?

A model answer is in [solutions/section-1.md](solutions/section-1.md).

---

### Section 2 — Your first pipeline with GitHub Actions

We will build a **single** workflow file that does three things:

1. Builds a tiny Express service once per commit.
2. Uploads the built output as a pipeline artifact tagged with the
   first 7 chars of the commit SHA.
3. Deploys the same artifact to an Azure App Service.

#### 2.1 Anatomy of a workflow

```yaml
# .github/workflows/deploy.yml
name: build-and-deploy

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  id-token: write   # required for OIDC to Azure
  contents: read
  actions: read

concurrency:
  group: deploy-${{ github.ref }}
  cancel-in-progress: false
```

Three things to notice before we even have jobs:

- `permissions:` is scoped down from the default. `id-token: write` is
  the key that unlocks OIDC to Azure — we'll use that instead of a
  long-lived publish profile.
- `concurrency:` prevents two deploys overlapping, but does **not**
  cancel an in-flight one. You do not want to cancel a half-finished
  deployment.
- `workflow_dispatch` gives you a manual trigger for the rollback drill
  later.

#### 2.2 The `build_and_package` job

```yaml
jobs:
  build_and_package:
    runs-on: ubuntu-latest
    outputs:
      sha_short: ${{ steps.sha.outputs.short }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - run: npm ci
      - run: npm run build --if-present
      - run: npm test --if-present

      - id: sha
        run: echo "short=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

      - name: Package deploy artifact
        run: |
          mkdir -p dist
          cp -r package.json package-lock.json src dist/
          (cd dist && zip -r ../app-${{ steps.sha.outputs.short }}.zip .)

      - uses: actions/upload-artifact@v4
        with:
          name: app-${{ steps.sha.outputs.short }}
          path: app-${{ steps.sha.outputs.short }}.zip
          retention-days: 30
          if-no-files-found: error
```

This job is the **build-once** half of "build once, deploy same." Notice
that nothing in this job talks to Azure — build jobs should not have
cloud credentials.

#### 2.3 The `deploy` job

```yaml
  deploy:
    needs: build_and_package
    runs-on: ubuntu-latest
    environment: production
    env:
      SHA_SHORT: ${{ needs.build_and_package.outputs.sha_short }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: app-${{ env.SHA_SHORT }}

      - uses: azure/login@v2
        with:
          client-id:       ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id:       ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Set version label on App Service
        run: |
          az webapp config appsettings set \
            --name "${{ vars.APP_SERVICE_NAME }}" \
            --resource-group "${{ vars.APP_RESOURCE_GROUP }}" \
            --settings WEBSITE_LABEL="${SHA_SHORT}" >/dev/null

      - uses: azure/webapps-deploy@v3
        with:
          app-name: ${{ vars.APP_SERVICE_NAME }}
          package: app-${{ env.SHA_SHORT }}.zip

      - name: Smoke check
        run: |
          URL="https://${{ vars.APP_SERVICE_NAME }}.azurewebsites.net/healthz"
          for i in 1 2 3 4 5 6; do
            body=$(curl -fsS --max-time 5 "$URL") && echo "$body" | grep -q '"status":"ok"' && exit 0
            sleep 5
          done
          echo "::error::Smoke check failed"; exit 1
```

Three design choices worth understanding:

- **No secrets in the repo.** The three `AZURE_*` ids are not secrets in
  the dangerous sense — they only become credentials when combined with
  a valid OIDC token from this specific workflow, on this repo, on the
  configured branch. See Appendix A for the trust relationship.
- **The smoke check has a ceiling.** It looks for HTTP 200 **and** a
  specific body shape within 30 seconds. It does **not** wait 10 minutes.
  If the service isn't up in 30s, the deploy is failed; that is a feature.
- **`WEBSITE_LABEL` is a receipt.** You can look at the App Service
  configuration blade in the Portal and see which SHA is running, without
  trusting what Git says was deployed.

#### 2.4 Secrets hygiene — what to avoid

| Anti-pattern | Why it bites |
|--------------|--------------|
| Hardcoded tokens in `.env` committed to the repo | Scanners find them in minutes; rotation is painful; blast radius is huge. |
| Long-lived broad-scope PATs shared across pipelines | One leak compromises every project that uses the token. |
| `echo $AZURE_CLIENT_SECRET` for "debugging" | GitHub's secret masker is not a security boundary — fragments leak via shell expansion, timing, and partial matches. |
| Deploy credential scoped to the whole resource group | If the workflow is subverted, it can delete every resource in the group. Scope to the single site. |

The federated identity you create in Appendix A is scoped to the single
App Service resource, role `Contributor`. That is deliberate.

---

### Section 3 — Hands-on lab

You will run three drills back to back. Each has a declared intent so
you don't slip into "debug the lab" mode.

#### Lab 3a — Happy-path deploy *(15 min)*

**Intent:** prove the pipeline works.

1. Fork the starter repo (link in Appendix B).
2. Follow Appendix A to provision your App Service and OIDC trust.
3. Add the three `AZURE_*` repo secrets and the two `APP_SERVICE_NAME` /
   `APP_RESOURCE_GROUP` repo variables.
4. Push a trivial commit to `main`.
5. Watch the Actions run go green. Open
   `https://<your-site>.azurewebsites.net/` and confirm you see
   `Hello from commit <sha>`.

**Expected pass signals:** green run, URL responds, `WEBSITE_LABEL` in
the Portal matches your commit SHA.

#### Lab 3b — Rollback drill *(15 min)*

**Intent:** prove you can recover without rebuilding.

1. Push a commit that breaks the service. (The starter repo's
   `src/handler.js` has a comment pointing at a one-character change
   that returns a 500. Make that change.)
2. Watch the deploy go green (the pipeline cannot know your code is
   broken from unit tests alone — that's the point).
3. Visit the URL; confirm you see `500 Internal Server Error`.
4. Run the **Rollback** workflow (`workflow_dispatch`) with
   input `sha = <prior 7-char sha>`.
5. Confirm the URL recovers, and confirm in the Actions log that the
   rollback job downloaded the **prior** artifact and did **not** run
   `npm ci` or `npm run build`.

**Expected pass signals:** URL recovered in under 60 seconds, no build
step in the rollback run, `WEBSITE_LABEL` reflects the prior SHA.

> **Explain-a-decision checkpoint.** Before moving on, write one
> paragraph in `solutions/section-3-decision.md` answering:
>
> *Suppose the failing change also renamed an environment variable from
> `DATABASE_URL` to `DATABASE_URI`. Why would "redeploy the same
> `GITHUB_SHA`" still be a safe rollback only if the env-var rename is
> also reverted together with the code? Why is this why config must be
> pinned alongside the artifact?*
>
> A model answer is in [solutions/section-3-decision.md](solutions/section-3-decision.md).

#### Lab 3c — The ambiguous 502 *(25 min)*

**Intent:** teach you to separate signal from noise in real logs.

Switch to the `lab-3c-start` branch of the starter repo and deploy. The
deploy will succeed. The site will return **intermittent 502**. The
App Service log stream will show:

- ~12 lines of `WARN: auth client unconfigured, continuing without auth`.
- ~6 lines of `ERROR: DNS lookup failed for telemetry.example.invalid`.
- Scrolling back, **one** line at startup that reads
  `WARN: DATABASE_URI not set; falling back to the sample database`.
- Exactly one line per request that reads
  `ERROR: upstream ECONNREFUSED 127.0.0.1:5432`.

The auth warnings are real but irrelevant — the service does not
require auth in this lab.
The DNS timeouts are real but irrelevant — the telemetry endpoint is a
deliberate dead host, and telemetry is fire-and-forget.
The real cause is that the workflow sets `DATABASE_URL` but the code
reads `DATABASE_URI` (typo). The code silently falls back to a
localhost placeholder, which on App Service has nothing listening.

**Your job:**

1. In `solutions/section-3c.md`, answer two questions:
   - Which of the three log patterns is the actual cause, and how did
     you rule out the other two?
   - What **one** line change to the workflow fixes the service?

A walk-through is in [solutions/section-3c.md](solutions/section-3c.md).

---

### Section 4 — Reading deployment logs without panic

Classify each excerpt **before** you reveal the answer. Cover the page
below the excerpt with your hand.

#### Excerpt 1

```
Run npm ci
added 312 packages in 4s
Run npm run build
> tsc
src/handler.ts(14,22): error TS1005: ')' expected.
Error: Process completed with exit code 2.
```

> **Class:** build-time / source error. You can read this straight off
> the compiler diagnostic. No deploy happened; no rollback needed.

#### Excerpt 2

```
Starting app...
Error: Failed to fetch secret 'db-password': status 403 Forbidden
Error: App failed to start in 30 seconds.
Container exited with code 1.
```

> **Class:** runtime / missing-or-unauthorized secret. The deploy
> "succeeded" (the artifact shipped) but the app can't start. Fix is in
> identity/secret configuration, not in code. Smoke check correctly
> failed the deploy for you.

#### Excerpt 3

```
Deployment complete.
Health probe GET /healthz returned 503 (body: {"status":"starting"})
Health probe GET /healthz returned 503 (body: {"status":"starting"})
Health probe GET /healthz returned 503 (body: {"status":"starting"})
::error::Smoke check failed
```

> **Class:** legitimate slow-start **or** a genuinely unhealthy app.
> First response is "is 30 seconds the right ceiling for my app?"
> Second response, only after confirming the ceiling: look at the app's
> own startup logs for why `/healthz` is stuck at `starting`.

#### The four failure classes you'll meet 95% of the time

1. **Build / source** — the compiler or tests tell you exactly where.
2. **Configuration / secret** — deploy ships, app can't start or can't
   call a dependency; fix is in config, not code.
3. **Runtime crash** — app started, then died; look for the stack trace
   in the **first** crash, not the 200th.
4. **Health-check mismatch** — app is alive but your probe doesn't
   agree; align the probe with the app's actual readiness signal.

## Definition of Done

You are done with this training when you can, unaided:

- [ ] Trigger a successful deploy via `git push` and reach your
      App Service URL.
- [ ] Execute the rollback workflow to restore the prior artifact
      **without** rebuilding.
- [ ] Diagnose the ambiguous-502 scenario from Lab 3c and name the
      root cause in writing.
- [ ] Classify all three raw log excerpts in Section 4 correctly on
      the first try.
- [ ] **Explain** in one paragraph why redeploying the same
      `GITHUB_SHA` is not, by itself, a safe rollback when the failing
      change included a config-shape change (e.g. a renamed env var).

## Hands-on lab

The starter repo, the three lab branches (`main`,
`lab-3a-broken-change`, `lab-3c-start`), and a sample `solutions/` tree
live at **tbd — to be added by SME**. Until that link lands, the lab
instructions are self-contained against a fresh Node.js + Express repo
following the `src/handler.js` / `package.json` layout referenced
throughout Section 2.

### Setup

```bash
# After Appendix A is complete:
gh repo fork <starter-repo> --clone --remote
cd <starter-repo>

gh secret   set AZURE_CLIENT_ID       --body "<app-reg client id>"
gh secret   set AZURE_TENANT_ID       --body "<tenant id>"
gh secret   set AZURE_SUBSCRIPTION_ID --body "<subscription id>"
gh variable set APP_SERVICE_NAME      --body "<site name>"
gh variable set APP_RESOURCE_GROUP    --body "<rg name>"

git commit --allow-empty -m "chore: trigger first deploy"
git push
```

### Exercises

Covered in Section 3 (Labs 3a, 3b, 3c above).

### Solution

See `solutions/` in the starter repo once published, or the inline
answer keys referenced in each section.

---

## Appendix A — One-time Azure setup

> SME: fill in exact commands for your org's subscription / region /
> naming policy. The shape below is a template; values marked with
> `<...>` must be substituted.

```bash
az group create -n <rg>      -l <region>
az appservice plan create -g <rg> -n <plan> --is-linux --sku B1
az webapp create          -g <rg> -p <plan> -n <site> --runtime "NODE|20-lts"

# Entra app registration + federated credential for this repo
az ad app create --display-name "<site>-github-oidc"
APP_ID=$(az ad app list --display-name "<site>-github-oidc" --query "[0].appId" -o tsv)

az ad sp create --id "$APP_ID"

az ad app federated-credential create --id "$APP_ID" --parameters '{
  "name": "<site>-main",
  "issuer": "https://token.actions.githubusercontent.com",
  "subject": "repo:<org>/<repo>:ref:refs/heads/main",
  "audiences": ["api://AzureADTokenExchange"]
}'

# Least-privilege role assignment — scoped to the single site, not the RG
SITE_ID=$(az webapp show -g <rg> -n <site> --query id -o tsv)
az role assignment create --assignee "$APP_ID" --role "Contributor" --scope "$SITE_ID"
```

## Appendix B — Starter repo layout

```
.
├─ .github/
│  └─ workflows/
│     ├─ deploy.yml
│     └─ rollback.yml
├─ src/
│  └─ handler.js
├─ package.json
└─ README.md
```

The `rollback.yml` workflow takes a SHA input, downloads the artifact
from the matching prior run, and uploads it to App Service — with no
`npm ci`, no `npm run build`, no source checkout-and-rebuild.

## References

- GitHub Actions: [Deploying to Azure App Service](https://learn.microsoft.com/azure/app-service/deploy-github-actions)
- GitHub Actions: [About OpenID Connect](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- Azure: [Configure a federated identity credential](https://learn.microsoft.com/entra/workload-id/workload-identity-federation-create-trust)
- Azure: [Set app settings on App Service](https://learn.microsoft.com/cli/azure/webapp/config/appsettings)
- Conceptual: [Build once, deploy many](https://12factor.net/build-release-run) (12-Factor — *Build, release, run*)

## Feedback

- Feedback issues: please open a new `training-request` issue and
  reference this training's slug in the body; the intake workflow
  will thread it back here.
- Known limitations / follow-ups:
  - Starter repo link is `tbd` — SME to publish.
  - Solutions files (`solutions/section-1.md`,
    `solutions/section-3-decision.md`, `solutions/section-3c.md`) to be
    added alongside this training when the starter repo lands.
  - Container-based deploy path (ACR → App Service Containers) is
    referenced as the next training and has no asset yet.
