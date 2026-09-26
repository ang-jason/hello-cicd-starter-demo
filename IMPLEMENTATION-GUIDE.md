# Simple End-to-End CI/CD Tutorial

# Agenda

1. CI/CD Overview
2. Target Architecture
3. Project Setup
4. Create the Source Code
5. Configure Local Validation
6. Configure Git and GitHub
7. Build the GitHub Actions CI Pipeline
8. Add Automated CI Tests
9. Configure Cloudflare Deployment
10. Add GitHub Secrets
11. Run the End-to-End Pipeline
12. Demonstrate a Failed CI Test
13. Fix the Test and Redeploy
14. Recommended Feature-Branch Workflow
15. Key DevOps Concepts and Summary

---

# Table of Contents

- [Objective](#objective)
- [1. Target Architecture](#1-target-architecture)
- [2. Project Structure](#2-project-structure)
- [3. Prerequisites](#3-prerequisites)
- [4. Create the Project](#4-create-the-project)
- [5. Create the Source Code](#5-create-the-source-code)
- [6. Automated Test Strategy](#6-automated-test-strategy)
- [7. Initialise Node.js](#7-initialise-nodejs)
- [8. Configure package.json](#8-configure-packagejson)
- [9. Configure Source-Code Validation](#9-configure-source-code-validation)
- [10. Configure Cloudflare](#10-configure-cloudflare)
- [11. Create .gitignore](#11-create-gitignore)
- [12. Test Locally](#12-test-locally)
- [13. Initialise Git](#13-initialise-git)
- [14. Create the GitHub Repository](#14-create-the-github-repository)
- [15. Create the GitHub Actions Pipeline](#15-create-the-github-actions-pipeline)
- [16. Understand the CI Stage](#16-understand-the-ci-stage)
- [17. Understand the CD Stage](#17-understand-the-cd-stage)
- [18. Why Pull Requests Do Not Deploy](#18-why-pull-requests-do-not-deploy)
- [19. Commit the CI/CD Pipeline](#19-commit-the-cicd-pipeline)
- [20. Configure Cloudflare Credentials](#20-configure-cloudflare-credentials)
- [21. Add GitHub Secrets](#21-add-github-secrets)
- [22. Trigger a Production Deployment](#22-trigger-a-production-deployment)
- [23. Expected Production URL](#23-expected-production-url)
- [24. Demonstrate a Failed Test](#24-demonstrate-a-failed-test)
- [25. Fix the Failed Test](#25-fix-the-failed-test)
- [26. Recommended Feature-Branch Workflow](#26-recommended-feature-branch-workflow)
- [27. Final Repository](#27-final-repository)
- [28. Complete CI/CD Flow](#28-complete-cicd-flow)
- [29. DevOps Concepts Demonstrated](#29-devops-concepts-demonstrated)
- [30. Key Message](#30-key-message)
- [Summary](#summary)

---


## Objective

Build a simple application and demonstrate a complete CI/CD workflow:

```text
Source Code
    ↓
Git
    ↓
GitHub
    ↓
GitHub Actions
    ↓
CI Tests
    ├── Check source files exist
    ├── Test 5 + 4 = 9
    └── Validate source code
    ↓
All tests pass?
    ↓ YES
CD - Deploy
    ↓
Cloudflare
    ↓
Production Website
```

This lab demonstrates:

- Source control with Git and GitHub
- Continuous Integration with GitHub Actions
- Automated tests
- Static source-code validation
- Quality gates
- Secrets management
- Continuous Deployment to Cloudflare

---

# 1. Target Architecture

```text
Developer
   │
   │ Write / Update Source Code
   │
   │ git push
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
   │    ├── Run Automated Test
   │    └── Validate Source Code
   │
   └── CD
        └── Deploy
             │
             ▼
        Cloudflare
             │
             ▼
         Production
```

Recommended pipeline behaviour:

| Event | CI | Deploy |
|---|---:|---:|
| Pull Request to `main` | ✅ | ❌ |
| Push / Merge to `main` | ✅ | ✅ |
| CI test fails | ❌ | ❌ |
| CI test passes | ✅ | ✅ |

---

# 2. Project Structure

The final repository will look like this:

```text
hello-cicd/
│
├── public/
│   └── index.html
│
├── .github/
│   └── workflows/
│       └── cicd.yml
│
├── .htmlhintrc
├── .gitignore
├── package.json
├── package-lock.json
└── wrangler.jsonc
```

Although this lab uses a simple HTML page, **Source Code** is used as the general DevOps terminology.

---

# 3. Prerequisites

Install or prepare:

| Tool | Purpose |
|---|---|
| Git | Source control |
| GitHub account | Repository and GitHub Actions |
| Node.js | Runtime and testing |
| npm | Dependency management |
| Cloudflare account | Deployment target |
| VS Code | Optional source-code editor |

Check Git:

```bash
git --version
```

Check Node.js:

```bash
node --version
```

Check npm:

```bash
npm --version
```

---

# 4. Create the Project

Create a new folder:

```bash
mkdir hello-cicd
cd hello-cicd
```

macOS / Linux:

```bash
mkdir -p public .github/workflows
```

PowerShell:

```powershell
mkdir public
mkdir .github
mkdir .github/workflows
```

Result:

```text
hello-cicd/
├── public/
└── .github/
    └── workflows/
```

---

# 5. Create the Source Code

Create:

```text
public/index.html
```

Add:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hello CI/CD</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background: #f5f5f5;
        }

        .card {
            background: white;
            padding: 40px;
            border-radius: 16px;
            text-align: center;
            box-shadow: 0 8px 30px rgba(0, 0, 0, 0.08);
        }

        h1 {
            margin-bottom: 10px;
        }
    </style>
</head>

<body>

    <main class="card">
        <h1>Hello World!</h1>
        <p>My first CI/CD deployment.</p>
    </main>

</body>
</html>
```

The application is intentionally simple because the objective is to understand the **CI/CD pipeline**.

---

# 6. Automated Test Strategy

For this lab, the simple calculation test will run **directly inside the GitHub Actions CI pipeline**.

There is no separate JavaScript test file.

The CI pipeline will execute:

```bash
RESULT=$((5 + 4))

echo "Testing: 5 + 4 = $RESULT"

if [ "$RESULT" -ne 9 ]; then
  echo "❌ Calculation test failed"
  exit 1
fi

echo "✅ Calculation test passed"
```

Expected CI output:

```text
Testing: 5 + 4 = 9
✅ Calculation test passed
```

The important concept is:

```text
Command returns exit code 0
        ↓
CI Step Passed

Command returns exit code 1
        ↓
CI Step Failed
        ↓
GitHub Actions Failed
        ↓
Deployment Blocked
```

This makes it very clear that the automated test is part of the **CI pipeline itself**.

---

# 7. Initialise Node.js

Run:

```bash
npm init -y
```

Install the required development dependencies:

```bash
npm install --save-dev htmlhint wrangler
```

This creates:

```text
package.json
package-lock.json
```

Commit `package-lock.json` into Git because GitHub Actions will use:

```bash
npm ci
```

---

# 8. Configure package.json

Update the `scripts` section of `package.json`:

```json
{
  "name": "hello-cicd",
  "version": "1.0.0",
  "description": "Simple CI/CD demonstration",
  "main": "index.js",
  "scripts": {
    "test:html": "htmlhint public/index.html",
    "test": "npm run test:html",
    "dev": "wrangler dev",
    "deploy": "wrangler deploy"
  },
  "private": true,
  "devDependencies": {
    "htmlhint": "^1.9.2",
    "wrangler": "^4.0.0"
  }
}
```

Test everything locally:

```bash
npm test
```

Expected:

```text
Scanned 1 files, no errors found
```

The calculation test will be executed later by GitHub Actions as part of CI.

---

# 9. Configure Source-Code Validation

Create:

```text
.htmlhintrc
```

Add:

```json
{
  "tagname-lowercase": true,
  "attr-lowercase": true,
  "attr-value-double-quotes": true,
  "doctype-first": true,
  "tag-pair": true,
  "id-unique": true,
  "src-not-empty": true,
  "attr-no-duplication": true,
  "title-require": true
}
```

Run:

```bash
npm run test:html
```

Expected:

```text
Scanned 1 files, no errors found
```

This is a simple static-analysis check against the source code.

---

# 10. Configure Cloudflare

Create:

```text
wrangler.jsonc
```

Add:

```jsonc
{
  "$schema": "./node_modules/wrangler/config-schema.json",

  "name": "hello-cicd",

  "compatibility_date": "2026-09-11",

  "assets": {
    "directory": "./public"
  }
}
```

This tells Cloudflare to serve the source files in:

```text
./public
```

---

# 11. Create .gitignore

Create:

```text
.gitignore
```

Add:

```gitignore
node_modules/
.wrangler/
.DS_Store
```

Do **not** ignore:

```text
package-lock.json
```

---

# 12. Test Locally

Run source-code validation:

```bash
npm run test:html
```

Run the local validation command:

```bash
npm test
```

The `5 + 4 = 9` calculation test is intentionally **not run locally through JavaScript** in this lab. It is defined directly in the GitHub Actions CI workflow.

Start the application locally:

```bash
npm run dev
```

Wrangler should provide a local address similar to:

```text
http://localhost:8787
```

Open it in a browser.

Expected page:

```text
Hello World!

My first CI/CD deployment.
```

---

# 13. Initialise Git

Run:

```bash
git init
```

Set the default branch:

```bash
git branch -M main
```

Add all files:

```bash
git add .
```

Check:

```bash
git status
```

Commit:

```bash
git commit -m "Initial Hello World CI/CD project"
```

---

# 14. Create the GitHub Repository

Create a GitHub repository called:

```text
hello-cicd
```

Connect your local project:

```bash
git remote add origin https://github.com/YOUR_USERNAME/hello-cicd.git
```

Push:

```bash
git push -u origin main
```

Current flow:

```text
Developer
   │
   │ git push
   ▼
GitHub
```

---

# 15. Create the GitHub Actions Pipeline

Create:

```text
.github/workflows/cicd.yml
```

Add:

```yaml
name: Hello World CI/CD

on:
  pull_request:
    branches:
      - main

  push:
    branches:
      - main

jobs:

  test:
    name: CI - Test
    runs-on: ubuntu-latest

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v7

      - name: Setup Node.js
        uses: actions/setup-node@v7
        with:
          node-version: 24
          cache: npm

      - name: Install Dependencies
        run: npm ci

      - name: Check Source Files
        run: |
          echo "Checking public/index.html..."

          if [ ! -f "public/index.html" ]; then
            echo "❌ Required source file public/index.html does not exist"
            exit 1
          fi

          echo "✅ Required source file exists"

      - name: Test 5 + 4
        run: |
          RESULT=$((5 + 4))

          echo "Testing: 5 + 4 = $RESULT"

          if [ "$RESULT" -ne 9 ]; then
            echo "❌ Calculation test failed"
            exit 1
          fi

          echo "✅ Calculation test passed"

      - name: Validate Source Code
        run: npm run test:html

      - name: CI Completed
        run: |
          echo "================================"
          echo "✅ ALL CI TESTS PASSED"
          echo "================================"


  deploy:
    name: CD - Deploy to Cloudflare
    runs-on: ubuntu-latest

    needs:
      - test

    if: github.event_name == 'push' && github.ref == 'refs/heads/main'

    steps:

      - name: Checkout Source Code
        uses: actions/checkout@v7

      - name: Deploy to Cloudflare
        id: deploy
        uses: cloudflare/wrangler-action@v4
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: deploy

      - name: Deployment Completed
        run: |
          echo "================================"
          echo "🚀 DEPLOYMENT COMPLETED"
          echo "================================"
```

---

# 16. Understand the CI Stage

The CI job performs:

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

The three tests demonstrate different concepts:

```text
SOURCE FILE CHECK
Does the required application source file exist?

        +

AUTOMATED TEST
Does 5 + 4 return 9?

        +

STATIC ANALYSIS
Does the source code meet validation rules?

        ↓

QUALITY GATE
```

---

# 17. Understand the CD Stage

The deployment job contains:

```yaml
needs:
  - test
```

This means:

```text
CI Test
   ↓
PASS?
   │
   ├── NO → Stop
   │
   └── YES
          ↓
       Deploy
```

If CI fails:

```text
CI FAILED
    ↓
Deployment BLOCKED
```

---

# 18. Why Pull Requests Do Not Deploy

The deployment job contains:

```yaml
if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

Therefore:

```text
Pull Request
    ↓
Run CI
    ↓
Tests Pass
    ↓
STOP
```

No production deployment occurs from the Pull Request.

After the Pull Request is merged:

```text
Merge to main
    ↓
GitHub Push Event
    ↓
CI Tests
    ↓
PASS
    ↓
CD Deploy
    ↓
Production
```

---

# 19. Commit the CI/CD Pipeline

Run:

```bash
git add .
```

Commit:

```bash
git commit -m "Add GitHub Actions CI/CD pipeline"
```

Push:

```bash
git push origin main
```

Go to:

```text
GitHub
  → hello-cicd
  → Actions
```

You should see the pipeline execute.

---

# 20. Configure Cloudflare Credentials

You need:

```text
CLOUDFLARE_ACCOUNT_ID
CLOUDFLARE_API_TOKEN
```

In Cloudflare, locate your Account ID.

Create an API token with the appropriate Workers deployment permissions.

Treat the API token as a secret.

Never place the token directly inside:

```text
cicd.yml
```

---

# 21. Add GitHub Secrets

Open:

```text
GitHub Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
Repository secrets
```

Create:

```text
CLOUDFLARE_API_TOKEN
```

Create:

```text
CLOUDFLARE_ACCOUNT_ID
```

GitHub Actions accesses them through:

```yaml
${{ secrets.CLOUDFLARE_API_TOKEN }}
```

and:

```yaml
${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
```

---

# 22. Trigger a Production Deployment

Change:

```html
<h1>Hello World!</h1>
```

to:

```html
<h1>Hello CI/CD World!</h1>
```

Commit:

```bash
git add .
git commit -m "Update application message"
```

Push:

```bash
git push origin main
```

The pipeline should now execute:

```text
Push Source Code
      ↓
GitHub
      ↓
GitHub Actions
      ↓
CI
 ├── ✅ Source file exists
 ├── ✅ 5 + 4 = 9
 └── ✅ Source validation
      ↓
CI PASSED
      ↓
CD
      ↓
Cloudflare
      ↓
Production
```

---

# 23. Expected Production URL

Cloudflare should provide a URL similar to:

```text
https://hello-cicd.<your-subdomain>.workers.dev
```

Open it.

Expected:

```text
Hello CI/CD World!

My first CI/CD deployment.
```

---

# 24. Demonstrate a Failed Test

This is highly recommended for a live CI/CD demonstration.

Open:

```text
.github/workflows/cicd.yml
```

Find the calculation test:

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

Temporarily change:

```bash
if [ "$RESULT" -ne 9 ]; then
```

to:

```bash
if [ "$RESULT" -ne 10 ]; then
```

The pipeline still calculates:

```text
5 + 4 = 9
```

but CI now expects:

```text
10
```

Commit:

```bash
git add .
git commit -m "Demo failing CI test"
git push origin main
```

GitHub Actions should show:

```text
Testing: 5 + 4 = 9
❌ Calculation test failed
```

Pipeline result:

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

This demonstrates the value of CI clearly:

> A failed automated test prevents deployment to production.

---

# 25. Fix the Failed Test

Change:

```bash
if [ "$RESULT" -ne 10 ]; then
```

back to:

```bash
if [ "$RESULT" -ne 9 ]; then
```

Commit and push:

```bash
git add .
git commit -m "Fix calculation test"
git push origin main
```

Now:

```text
✅ Source File Check
        ↓
✅ 5 + 4 = 9
        ↓
✅ Source Validation
        ↓
✅ CI PASSED
        ↓
🚀 CD
        ↓
🌐 Production
```

---

# 26. Recommended Feature-Branch Workflow

Create a feature branch:

```bash
git checkout -b feature/change-title
```

Modify the source code:

```html
<h1>Hello DevOps!</h1>
```

Commit:

```bash
git add .
git commit -m "Change homepage title"
```

Push:

```bash
git push -u origin feature/change-title
```

Create a Pull Request:

```text
feature/change-title
        ↓
       main
```

GitHub Actions runs:

```text
Pull Request
     ↓
CI
     ↓
Source File Check
     ↓
Automated Test
     ↓
Source Validation
     ↓
PASS
```

No deployment occurs yet.

Merge the Pull Request:

```text
Merge
 ↓
main
 ↓
CI
 ↓
PASS
 ↓
CD
 ↓
Production
```

---

# 27. Final Repository

```text
hello-cicd/
│
├── .github/
│   └── workflows/
│       └── cicd.yml
│
├── public/
│   └── index.html
│
├── .gitignore
├── .htmlhintrc
├── package.json
├── package-lock.json
└── wrangler.jsonc
```

---

# 28. Complete CI/CD Flow

```text
                    DEVELOPER
                        │
                        │ git push
                        ▼
                 ┌──────────────┐
                 │    GitHub    │
                 └──────┬───────┘
                        │
                        ▼
              ┌─────────────────────┐
              │   GitHub Actions    │
              └──────────┬──────────┘
                         │
                 ┌───────▼───────┐
                 │      CI       │
                 └───────┬───────┘
                         │
              ┌──────────▼──────────┐
              │ Check Source Files │
              │        ✅           │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │ Test 5 + 4 = 9     │
              │        ✅           │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │ Validate Source    │
              │        ✅           │
              └──────────┬──────────┘
                         │
                         ▼
                    CI PASSED
                         │
                         ▼
                 ┌───────────────┐
                 │      CD       │
                 └───────┬───────┘
                         │
                         ▼
                Cloudflare Deploy
                         │
                         ▼
                  LIVE WEBSITE
```

---

# 29. DevOps Concepts Demonstrated

| Component | DevOps Concept |
|---|---|
| Source Code | Application |
| Git | Version Control |
| GitHub | Central Source Repository |
| Feature Branch | Isolated Development |
| Pull Request | Change Review |
| GitHub Actions | Automation |
| Source File Check | Basic Validation |
| `5 + 4 = 9` in GitHub Actions | CI Automated Test |
| HTMLHint | Static Analysis |
| `needs: test` | Quality Gate |
| GitHub Secrets | Secrets Management |
| Wrangler | Deployment Tooling |
| Cloudflare | Production Environment |

---


# 30. Key Message

Without CI/CD:

```text
Developer
    ↓
Manual Testing
    ↓
Manual Deployment
    ↓
Higher Risk of Human Error
```

With CI/CD:

```text
Developer
    ↓
Git Push
    ↓
Automated Tests
    ↓
Quality Gate
    ↓
Automated Deployment
    ↓
Production
```

The most important demonstration is:

```text
Bad Source Code / Failed Test
            ↓
         CI FAILS
            ↓
     Deployment Blocked
            ↓
 Production Protected
```

---

# Summary

The complete workflow is:

```text
Source Code
    ↓
Git
    ↓
GitHub
    ↓
GitHub Actions
    ↓
CI Tests
    ├── Check Source Files
    ├── Test 5 + 4 = 9
    └── Validate Source Code
    ↓
Quality Gate
    ↓
CD
    ↓
Cloudflare
    ↓
Production
```

This simple lab introduces the core concepts of an end-to-end DevOps CI/CD pipeline without requiring a complex application.
