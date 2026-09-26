# Hello CI/CD Starter Demo

A simple end-to-end CI/CD demonstration using **GitHub Actions** to validate source code, run automated checks, and deploy a static website to **Cloudflare Pages**.

## Table of Contents

- [Architecture](https://github.com/ang-jason/hello-cicd-starter-demo#architecture)
- [GitHub Actions](https://github.com/ang-jason/hello-cicd-starter-demo#github-actions)
- [Project Structure](https://github.com/ang-jason/hello-cicd-starter-demo#project-structure)
- [CI Checks](https://github.com/ang-jason/hello-cicd-starter-demo#ci-checks)
- [First-time Setup](https://github.com/ang-jason/hello-cicd-starter-demo#first-time-setup)
- [Git Setup](https://github.com/ang-jason/hello-cicd-starter-demo#git-setup)
- [GitHub Repository Secrets](https://github.com/ang-jason/hello-cicd-starter-demo#github-repository-secrets)
- [CI Pipeline](https://github.com/ang-jason/hello-cicd-starter-demo#ci-pipeline)
- [CI Failure Demo](https://github.com/ang-jason/hello-cicd-starter-demo#ci-failure-demo)
- [Optional Hardening](https://github.com/ang-jason/hello-cicd-starter-demo#optional-hardening-after-package-lockjson-exists)
- [Cloudflare Pages vs Workers](https://github.com/ang-jason/hello-cicd-starter-demo#cloudflare-pages-vs-workers)
- [Deployment URLs](https://github.com/ang-jason/hello-cicd-starter-demo#deployment-urls)
- [Summary](https://github.com/ang-jason/hello-cicd-starter-demo#summary)

---

## Architecture

```text
Developer
    │
    │ git push / pull request
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    │
    ├── CI
    │    ├── Checkout Source Code
    │    ├── Install Dependencies
    │    ├── Check Source Files
    │    ├── Test 5 + 4 = 9
    │    └── Validate Source Code
    │
    └── CD
         └── Deploy to Cloudflare Pages
                    │
                    ▼
             Production Website
```

### Pipeline behaviour

| Event | CI | Production Deployment |
|---|---:|---:|
| Pull Request to `main` | ✅ | ❌ |
| Push to `main` | ✅ | ✅ |
| Merge to `main` | ✅ | ✅ |
| CI failure | ❌ | ❌ |

---

## GitHub Actions

**GitHub Actions is GitHub's built-in automation platform for creating workflows that automatically build, test, and deploy code based on repository events such as pushes and pull requests.**

The workflow is stored at:

```text
.github/workflows/cicd.yml
```

Main concepts:

| Concept | Description |
|---|---|
| Workflow | The complete automation pipeline |
| Trigger | Event that starts the workflow, such as `push` or `pull_request` |
| Job | A group of related tasks, such as `test` or `deploy` |
| Step | One individual task inside a job |
| Runner | The machine that executes the workflow |
| Action | A reusable automation component |

---

## Project Structure

```text
hello-cicd-starter-demo/
├── .github/
│   └── workflows/
│       └── cicd.yml
├── public/
│   └── index.html
├── .gitignore
├── .htmlhintrc
├── package.json
├── package-lock.json
├── wrangler.toml
└── README.md
```

The project now uses **Cloudflare Pages**.

`wrangler.toml` contains the Pages configuration:

```toml
name = "hello-cicd"
pages_build_output_dir = "./public"
compatibility_date = "2026-09-26"
```

---

## CI Checks

The GitHub Actions CI job runs three simple checks.

### 1. Check Source Files

```yaml
- name: Check Source Files
  run: |
    echo "Checking public/index.html..."

    if [ ! -f "public/index.html" ]; then
      echo "❌ Required source file public/index.html does not exist"
      exit 1
    fi

    echo "✅ Required source file exists"
```

### 2. Test 5 + 4

The calculation test is deliberately defined directly in the CI pipeline.

```yaml
- name: Test 5 + 4
  run: |
    RESULT=$((5 + 4))

    echo "Testing: 5 + 4 = $RESULT"

    if [ "$RESULT" -ne 9 ]; then
      echo "❌ Calculation test failed"
      exit 1
    fi

    echo "✅ Calculation test passed"
```

### 3. Validate Source Code

```yaml
- name: Validate Source Code
  run: npm run test:html
```

If any check fails:

```text
CI FAILED
    ↓
Deployment blocked
```

---

## First-time Setup

Install the project dependencies:

```bash
npm install
```

This creates or updates:

```text
package-lock.json
```

Run validation locally:

```bash
npm test
```

Run the Cloudflare Pages project locally:

```bash
npx wrangler pages dev
```

Because `wrangler.toml` contains:

```toml
pages_build_output_dir = "./public"
```

Wrangler knows which directory to serve.

---

## Git Setup

Initialise Git:

```bash
git init
git branch -M main
```

Add and commit the project:

```bash
git add .
git commit -m "Initial CI/CD project"
```

Add the GitHub remote:

```bash
git remote add origin https://github.com/ang-jason/hello-cicd-starter-demo.git
```

Push:

```bash
git push -u origin main
```

---

## GitHub Repository Secrets

Create these repository secrets:

```text
CLOUDFLARE_API_TOKEN
CLOUDFLARE_ACCOUNT_ID
```

Location:

```text
GitHub Repository
→ Settings
→ Secrets and variables
→ Actions
→ Repository secrets
```

For the Cloudflare API token, use:

```text
Account
→ Cloudflare Pages
→ Edit
```

Restrict the token to the Cloudflare account used by this deployment.

---

## CI Pipeline

The workflow triggers on both Pull Requests and pushes to `main`:

```yaml
on:
  pull_request:
    branches:
      - main

  push:
    branches:
      - main
```

The CI job:

```text
Checkout Source Code
        ↓
Setup Node.js
        ↓
Install Dependencies
        ↓
Check Source Files
        ↓
Test 5 + 4 = 9
        ↓
Validate Source Code
        ↓
CI Passed
```

The deployment job runs only after CI succeeds:

```yaml
needs:
  - test
```

and only for a push to `main`:

```yaml
if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

The Pages deployment step is:

```yaml
- name: Deploy to Cloudflare Pages
  uses: cloudflare/wrangler-action@v4
  with:
    apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
    accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
    command: pages deploy
```

Because the Pages project name and output directory are defined in `wrangler.toml`, the command can remain simple:

```bash
wrangler pages deploy
```

---

## CI Failure Demo

To demonstrate how CI protects production, temporarily change:

```bash
if [ "$RESULT" -ne 9 ]; then
```

to:

```bash
if [ "$RESULT" -ne 10 ]; then
```

The pipeline calculates:

```text
5 + 4 = 9
```

but now expects `10`.

The result:

```text
Source Code Push
       ↓
CI
       ↓
✅ Source File Check
       ↓
❌ Calculation Test
       ↓
CI FAILED
       ↓
Deployment BLOCKED
```

Change the value back to `9` and push again to restore the successful pipeline.

---

## Optional Hardening After package-lock.json Exists

Once `package-lock.json` is committed, use deterministic dependency installation in CI.

Change:

```yaml
- name: Install Dependencies
  run: npm install
```

to:

```yaml
- name: Install Dependencies
  run: npm ci
```

You can also enable npm caching:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v7
  with:
    node-version: 24
    cache: npm
```

This works because the repository now contains the lock file that `setup-node` expects for npm caching.

---

## Cloudflare Pages vs Workers

### Cloudflare Pages

**Cloudflare Pages is a platform for deploying and hosting static or frontend websites.**

Typical uses:

```text
HTML
CSS
JavaScript
Static framework builds
```

Pages is appropriate when the main requirement is simply:

> **Host the website.**

### Cloudflare Workers

**Cloudflare Workers is a serverless compute platform that runs application logic at Cloudflare's edge.**

Typical uses:

```text
APIs
Authentication
Backend logic
Request routing
Scheduled processing
Full-stack applications
```

Workers is appropriate when the requirement is:

> **Run application or backend logic.**

### Comparison

| Cloudflare Pages | Cloudflare Workers |
|---|---|
| Static/frontend focused | Serverless compute focused |
| Simple website hosting | APIs and backend logic |
| Supports Pages Functions | Worker logic is native |
| Uses `pages.dev` | Uses `workers.dev` |
| Best fit for this demo | Better when backend logic is required |

This repository uses **Cloudflare Pages** because it is a simple static-site CI/CD demonstration.

---

## Deployment URLs

### Cloudflare Pages

Pages projects use:

```text
https://<project-name>.pages.dev
```

For this project:

```text
https://hello-cicd.pages.dev
```

The exact URL depends on the final Pages project name.

### Cloudflare Workers

Workers normally use:

```text
https://<worker-name>.<account-subdomain>.workers.dev
```

Example:

```text
https://hello-cicd.<account-subdomain>.workers.dev
```

Both Pages and Workers can also be mapped to custom domains.

---

## Summary

The complete CI/CD workflow is:

```text
Source Code
    ↓
GitHub
    ↓
GitHub Actions
    ↓
CI
    ├── Check Source Files
    ├── Test 5 + 4 = 9
    └── Validate Source Code
    ↓
Quality Gate
    ↓
Cloudflare Pages
    ↓
Production Website
```

For Pull Requests:

```text
Pull Request
     ↓
CI Tests
     ↓
PASS
     ↓
No Production Deployment
```

For `main`:

```text
Push / Merge to main
     ↓
CI Tests
     ↓
PASS
     ↓
Cloudflare Pages
     ↓
https://hello-cicd.pages.dev
```
