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

## Available Workflows

### Gradle Workflows

| Workflow | Description |
|----------|-------------|
| [`gradle-build.yml`](.github/workflows/gradle-build.yml) | Fast-fail compilation check |
| [`gradle-unit-tests.yml`](.github/workflows/gradle-unit-tests.yml) | Unit tests with Kover coverage |
| [`gradle-context-tests.yml`](.github/workflows/gradle-context-tests.yml) | Integration/context tests |
| [`gradle-matrix-tests.yml`](.github/workflows/gradle-matrix-tests.yml) | Parallel tests across modules |
| [`gradle-detect-modules.yml`](.github/workflows/gradle-detect-modules.yml) | Detect changed modules for matrix builds |
| [`gradle-detekt.yml`](.github/workflows/gradle-detekt.yml) | Detekt static analysis |
| [`gradle-task-commit.yml`](.github/workflows/gradle-task-commit.yml) | Run task and commit changes to PR |

### Frontend Workflows

| Workflow | Description |
|----------|-------------|
| [`frontend-unit-tests-npm.yml`](.github/workflows/frontend-unit-tests-npm.yml) | Node.js/npm unit tests |
| [`frontend-unit-tests-bun.yml`](.github/workflows/frontend-unit-tests-bun.yml) | Bun unit tests |
| [`frontend-e2e-tests-npm.yml`](.github/workflows/frontend-e2e-tests-npm.yml) | Playwright E2E tests (npm) |
| [`frontend-e2e-tests-bun.yml`](.github/workflows/frontend-e2e-tests-bun.yml) | Playwright E2E tests (Bun) |

### Infrastructure Workflows

| Workflow | Description |
|----------|-------------|
| [`check-changes.yml`](.github/workflows/check-changes.yml) | Detect file changes with patterns |
| [`docker-build-push.yml`](.github/workflows/docker-build-push.yml) | Build and push Docker images |
| [`deploy-digitalocean-ssh.yml`](.github/workflows/deploy-digitalocean-ssh.yml) | SSH deployment to DigitalOcean |
| [`version-bump.yml`](.github/workflows/version-bump.yml) | Bump version and commit |
| [`notify-discord.yml`](.github/workflows/notify-discord.yml) | Discord notifications |

### Composite Actions

| Action | Description |
|--------|-------------|
| [`setup-java-gradle`](.github/actions/setup-java-gradle/action.yml) | Setup Java, Gradle, and caching |
| [`setup-node`](.github/actions/setup-node/action.yml) | Setup Node.js with npm caching |
| [`check-path-changes`](.github/actions/check-path-changes/action.yml) | Check if specific paths changed |

## Documentation

- **[Gradle Workflows](docs/gradle-workflows.md)** - Build, test, and code quality workflows
- **[Frontend Workflows](docs/frontend-workflows.md)** - Node.js, Bun, and Playwright testing
- **[Infrastructure Workflows](docs/infrastructure-workflows.md)** - Docker, deployment, and utilities

## Examples

See the [`examples/`](examples/) directory for complete workflow examples:

| Example | Description |
|---------|-------------|
| [`simple-gradle-ci.yml`](examples/simple-gradle-ci.yml) | Minimal Gradle CI setup |
| [`pr-tests-complete.yml`](examples/pr-tests-complete.yml) | Full PR testing pipeline |
| [`gradle-task-commit.yml`](examples/gradle-task-commit.yml) | Auto-format and commit changes |
| [`monorepo-selective.yml`](examples/monorepo-selective.yml) | Selective testing for monorepos |
| [`deploy-complete.yml`](examples/deploy-complete.yml) | Full deployment pipeline |
| [`frontend-only.yml`](examples/frontend-only.yml) | Frontend-only project |

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
