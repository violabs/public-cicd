# Public CI/CD Workflows

A collection of reusable GitHub Actions workflows and composite actions for CI/CD pipelines.

## Quick Start

Reference these workflows from your repository:

```yaml
jobs:
  build:
    uses: violabs/public-cicd/.github/workflows/gradle-build.yml@main
    with:
      java-version: '21'
```

## Available Components

### Reusable Workflows

| Workflow                                                                     | Description                                           |
|------------------------------------------------------------------------------|-------------------------------------------------------|
| [`check-changes.yml`](.github/workflows/check-changes.yml)                   | Detect file changes based on include/exclude patterns |
| [`gradle-build.yml`](.github/workflows/gradle-build.yml)                     | Fast-fail Gradle build check                          |
| [`gradle-unit-tests.yml`](.github/workflows/gradle-unit-tests.yml)           | Run unit tests with Kover coverage                    |
| [`gradle-context-tests.yml`](.github/workflows/gradle-context-tests.yml)     | Run context/integration tests                         |
| [`gradle-matrix-tests.yml`](.github/workflows/gradle-matrix-tests.yml)       | Run tests across multiple modules in parallel         |
| [`gradle-detekt.yml`](.github/workflows/gradle-detekt.yml)                   | Detekt static analysis for single module              |
| [`gradle-detekt-matrix.yml`](.github/workflows/gradle-detekt-matrix.yml)     | Detekt analysis across multiple modules               |
| [`frontend-tests.yml`](.github/workflows/frontend-tests-npm.yml)                 | Node.js/Playwright frontend tests                     |
| [`notify-discord.yml`](.github/workflows/notify-discord.yml)                 | Discord notifications for build results               |

### Composite Actions

| Action                                                                | Description                     |
|-----------------------------------------------------------------------|---------------------------------|
| [`setup-java-gradle`](.github/actions/setup-java-gradle/action.yml)   | Setup Java, Gradle, and caching |
| [`setup-node`](.github/actions/setup-node/action.yml)                 | Setup Node.js with npm caching  |
| [`check-path-changes`](.github/actions/check-path-changes/action.yml) | Check if specific paths changed |

## Workflow Reference

### check-changes.yml

Detects file changes between branches.

```yaml
jobs:
  check-backend:
    uses: violabs/public-cicd/.github/workflows/check-changes.yml@main
    with:
      mode: exclude                              # 'include' or 'exclude'
      exclude-patterns: '^(frontend/|docs/)'     # Regex pattern
```

**Outputs:**

- `changed`: `true` or `false`
- `changed-files`: Comma-separated list of changed files

---

### gradle-build.yml

Fast compilation check without running tests.

```yaml
jobs:
  build:
    uses: violabs/public-cicd/.github/workflows/gradle-build.yml@main
    with:
      java-version: '21'                         # Default: '21'
      java-distribution: 'temurin'               # Default: 'temurin'
      build-tasks: 'classes testClasses'         # Default: 'classes testClasses'
      gradle-args: '--no-daemon'                 # Default: '--no-daemon'
```

---

### gradle-unit-tests.yml

Run unit tests with optional Kover coverage.

```yaml
jobs:
  test:
    uses: violabs/public-cicd/.github/workflows/gradle-unit-tests.yml@main
    with:
      java-version: '21'
      module: ':app'                             # Optional: specific module
      module-path: 'app'                         # Path for artifacts
      artifact-suffix: 'app'                     # Unique artifact name suffix
      coverage-enabled: true                     # Enable Kover coverage
      exclude-tasks: 'npmInstall'                # Tasks to skip
      add-pr-coverage-comment: true              # Add coverage to PR
      min-coverage-overall: 40                   # Minimum coverage %
```

---

### gradle-matrix-tests.yml

Run tests across multiple modules in parallel using a matrix strategy.

```yaml
jobs:
  determine-modules:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.detect.outputs.matrix }}
    steps:
      - id: detect
        run: |
          # Output JSON array: [{"module": ":app", "path": "app", "filename": "app"}]
          echo 'matrix=[{"module":":app","path":"app","filename":"app"}]' >> "$GITHUB_OUTPUT"

  test:
    needs: determine-modules
    uses: violabs/public-cicd/.github/workflows/gradle-matrix-tests.yml@main
    with:
      modules-json: ${{ needs.determine-modules.outputs.matrix }}
      coverage-enabled: true
      exclude-tasks: 'npmInstall testComponents'
```

---

### gradle-context-tests.yml

Run context/integration tests.

```yaml
jobs:
  context-tests:
    uses: violabs/public-cicd/.github/workflows/gradle-context-tests.yml@main
    with:
      test-name: 'Context Tests'                 # Display name
      test-task: 'testComponents'                # Default: 'testComponents'
      exclude-tasks: 'npmInstall'
      artifact-name: 'context-test-reports'
```

---

### gradle-detekt.yml

Run Detekt static analysis.

```yaml
jobs:
  detekt:
    uses: violabs/public-cicd/.github/workflows/gradle-detekt.yml@main
    with:
      module: ':app'                             # Optional: specific module
      fallback-to-root: true                     # Run root detekt if module fails
```

---

### frontend-tests.yml

Run frontend tests with Node.js and optional Playwright.

```yaml
jobs:
  frontend:
    uses: violabs/public-cicd/.github/workflows/frontend-tests-npm.yml@main
    with:
      node-version: '22'
      working-directory: 'frontend'
      cache-dependency-path: 'frontend/package-lock.json'
      test-command: 'npm run test:coverage'
      install-playwright: true
      playwright-browsers: 'chromium'
```

---

### notify-discord.yml

Send Discord notifications about workflow results.

```yaml
jobs:
  notify:
    uses: violabs/public-cicd/.github/workflows/notify-discord.yml@main
    if: always()
    with:
      workflow_name: ${{ github.workflow }}
      build_result: ${{ needs.build.result }}
      test_result: ${{ needs.test.result }}
      pr_title: ${{ github.event.pull_request.title }}
      pr_number: ${{ github.event.pull_request.number }}
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}
```

## Examples

See the [`examples/`](examples/) directory for complete workflow examples:

- [`pr-tests-complete.yml`](examples/pr-tests-complete.yml) - Full PR testing pipeline
- [`simple-gradle-ci.yml`](examples/simple-gradle-ci.yml) - Minimal Gradle CI
- [`frontend-only.yml`](examples/frontend-only.yml) - Frontend-only project
- [`monorepo-selective.yml`](examples/monorepo-selective.yml) - Selective testing for monorepos

## Using Composite Actions

Composite actions can be used directly in your workflow steps:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: violabs/public-cicd/.github/actions/setup-java-gradle@main
        with:
          java-version: '21'

      - run: ./gradlew build
```

## License

MIT License - see [LICENSE](LICENSE) for details.
