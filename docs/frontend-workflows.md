# Frontend Workflows

Reusable workflows for Node.js and Bun-based frontend projects with support for unit tests and Playwright E2E testing.

## frontend-unit-tests-npm.yml

Run unit tests with npm and optional coverage reporting.

```yaml
jobs:
  test:
    uses: violabs/public-cicd/.github/workflows/frontend-unit-tests-npm.yml@main
    with:
      node-version: '22'
      working-directory: 'frontend'
      test-command: 'npm run test:coverage'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `node-version` | `'22'` | Node.js version |
| `working-directory` | `'frontend'` | Working directory |
| `cache-dependency-path` | `'frontend/package-lock.json'` | Path to package-lock.json |
| `test-command` | `'npm run test:coverage'` | Command to run tests |
| `upload-coverage` | `true` | Upload coverage reports |
| `coverage-path` | `'frontend/coverage/'` | Path to coverage reports |
| `artifact-name` | `'frontend-coverage-report'` | Coverage artifact name |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## frontend-unit-tests-bun.yml

Run unit tests with Bun package manager.

```yaml
jobs:
  test:
    uses: violabs/public-cicd/.github/workflows/frontend-unit-tests-bun.yml@main
    with:
      bun-version: 'latest'
      working-directory: 'frontend'
      test-command: 'bun test'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `bun-version` | `'latest'` | Bun version |
| `working-directory` | `'frontend'` | Working directory |
| `test-command` | `'bun test'` | Command to run tests |
| `upload-coverage` | `true` | Upload coverage reports |
| `coverage-path` | `'frontend/coverage/'` | Path to coverage reports |
| `artifact-name` | `'frontend-coverage-report'` | Coverage artifact name |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## frontend-e2e-tests-npm.yml

Run Playwright E2E tests with npm.

```yaml
jobs:
  e2e:
    uses: violabs/public-cicd/.github/workflows/frontend-e2e-tests-npm.yml@main
    with:
      node-version: '22'
      working-directory: 'frontend'
      test-command: 'npm run test:e2e'
      playwright-browsers: 'chromium'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `node-version` | `'22'` | Node.js version |
| `working-directory` | `'frontend'` | Working directory |
| `cache-dependency-path` | `'frontend/package-lock.json'` | Path to package-lock.json |
| `test-command` | `'npm run test:e2e'` | Command to run E2E tests |
| `playwright-browsers` | `'chromium'` | Playwright browsers to install |
| `upload-results` | `true` | Upload test results |
| `results-path` | `'frontend/playwright-report/'` | Path to test results |
| `artifact-name` | `'frontend-e2e-report'` | Results artifact name |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## frontend-e2e-tests-bun.yml

Run Playwright E2E tests with Bun.

```yaml
jobs:
  e2e:
    uses: violabs/public-cicd/.github/workflows/frontend-e2e-tests-bun.yml@main
    with:
      bun-version: 'latest'
      working-directory: 'frontend'
      test-command: 'bun run test:e2e'
      playwright-browsers: 'chromium'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `bun-version` | `'latest'` | Bun version |
| `working-directory` | `'frontend'` | Working directory |
| `test-command` | `'bun run test:e2e'` | Command to run E2E tests |
| `playwright-browsers` | `'chromium'` | Playwright browsers to install |
| `upload-results` | `true` | Upload test results |
| `results-path` | `'frontend/playwright-report/'` | Path to test results |
| `artifact-name` | `'frontend-e2e-report'` | Results artifact name |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## Complete Frontend CI Example

```yaml
name: Frontend CI

on:
  pull_request:
    paths:
      - 'frontend/**'

jobs:
  unit-tests:
    uses: violabs/public-cicd/.github/workflows/frontend-unit-tests-npm.yml@main
    with:
      node-version: '22'
      working-directory: 'frontend'
      test-command: 'npm run test:coverage'

  e2e-tests:
    uses: violabs/public-cicd/.github/workflows/frontend-e2e-tests-npm.yml@main
    with:
      node-version: '22'
      working-directory: 'frontend'
      playwright-browsers: 'chromium webkit'
```
