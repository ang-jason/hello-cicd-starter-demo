# Hello CI/CD

A minimal end-to-end CI/CD demo using:

- Source Code
- Git / GitHub
- GitHub Actions
- Automated CI checks
- Cloudflare Workers Static Assets

## Table of Contents

- [Project Structure](#project-structure)
- [CI Checks](#ci-checks)
- [First-time Setup](#first-time-setup)
- [Git Setup](#git-setup)
- [GitHub Repository Secrets](#github-repository-secrets)
- [CI Failure Demo](#ci-failure-demo)
- [Optional Hardening](#optional-hardening-after-package-lockjson-exists)

---

## Project structure

```text
hello-cicd/
├── .github/
│   └── workflows/
│       └── cicd.yml
├── public/
│   └── index.html
├── .env.example
├── .gitignore
├── .htmlhintrc
├── package.json
└── wrangler.jsonc
```

## CI checks

The GitHub Actions pipeline runs:

1. Check that `public/index.html` exists.
2. Run the pipeline calculation test: `5 + 4 = 9`.
3. Validate the source code with HTMLHint.
4. Deploy only when CI passes and the change is pushed or merged to `main`.

## First-time setup

Install dependencies:

```bash
npm install
```

This will generate the real `package-lock.json`. Commit it after it is created:

```bash
git add package-lock.json
git commit -m "Add dependency lockfile"
```

Run local validation:

```bash
npm test
```

Run locally:

```bash
npm run dev
```

## Git setup

```bash
git init
git branch -M main
git add .
git commit -m "Initial CI/CD project"
```

Create a GitHub repository, then:

```bash
git remote add origin https://github.com/YOUR_USERNAME/hello-cicd.git
git push -u origin main
```

## GitHub repository secrets

Configure these under:

`Settings > Secrets and variables > Actions`

Required secrets:

```text
CLOUDFLARE_ACCOUNT_ID
CLOUDFLARE_API_TOKEN
```

Do not commit the real values into the repository.

## CI failure demo

In `.github/workflows/cicd.yml`, temporarily change:

```bash
if [ "$RESULT" -ne 9 ]; then
```

to:

```bash
if [ "$RESULT" -ne 10 ]; then
```

The CI job will fail and the deployment job will be blocked.

Change it back to `9` to restore the successful pipeline.


## Optional hardening after package-lock.json exists

Once `package-lock.json` has been generated and committed, you can change the CI dependency step from:

```yaml
- name: Install Dependencies
  run: npm install
```

to:

```yaml
- name: Install Dependencies
  run: npm ci
```

`npm ci` is preferred for reproducible CI builds when a valid lockfile is committed.
