# Infrastructure Workflows

Reusable workflows for change detection, Docker builds, deployments, version management, and notifications.

## check-changes.yml

Detect file changes between branches using include/exclude patterns.

```yaml
jobs:
  check-backend:
    uses: violabs/public-cicd/.github/workflows/check-changes.yml@main
    with:
      mode: exclude
      exclude-patterns: '^(frontend/|docs/)'

  build:
    needs: check-backend
    if: needs.check-backend.outputs.changed == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "Backend changed!"
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `include-patterns` | `''` | Regex for paths to INCLUDE |
| `exclude-patterns` | `''` | Regex for paths to EXCLUDE |
| `mode` | `'include'` | Detection mode: `include` or `exclude` |
| `base-ref` | `''` | Base branch (auto-detected for PRs) |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Outputs

| Output | Description |
|--------|-------------|
| `changed` | `'true'` or `'false'` |
| `changed-files` | Comma-separated list of changed files |

### Examples

**Include only Kotlin/Java files:**
```yaml
with:
  mode: include
  include-patterns: '\.(kt|java)$'
```

**Exclude frontend and docs:**
```yaml
with:
  mode: exclude
  exclude-patterns: '^(frontend/|docs/|\.github/)'
```

---

## docker-build-push.yml

Build and push Docker images to a container registry.

```yaml
jobs:
  docker:
    uses: violabs/public-cicd/.github/workflows/docker-build-push.yml@main
    with:
      image-name: 'username/my-app'
      registry: 'docker.io'
      platforms: 'linux/amd64,linux/arm64'
    secrets:
      REGISTRY_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      REGISTRY_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `image-name` | (required) | Full image name (e.g., `user/repo`) |
| `registry` | `'docker.io'` | Container registry |
| `platforms` | `'linux/amd64'` | Target platforms (comma-separated) |
| `context` | `'.'` | Build context path |
| `dockerfile` | `'Dockerfile'` | Path to Dockerfile |
| `version-file` | `'VERSION'` | Path to VERSION file (empty to skip) |
| `build-args` | `''` | Build arguments (newline-separated) |
| `extra-tags` | `''` | Additional tags (newline-separated) |
| `push` | `true` | Push image to registry |
| `cache-enabled` | `true` | Enable GitHub Actions cache |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `REGISTRY_USERNAME` | Yes | Registry username |
| `REGISTRY_TOKEN` | Yes | Registry token/password |

### Outputs

| Output | Description |
|--------|-------------|
| `version` | Version from VERSION file |
| `image-digest` | Image digest |
| `image-tags` | Image tags |

---

## deploy-digitalocean-ssh.yml

Deploy to DigitalOcean droplets via SSH using Docker Compose.

```yaml
jobs:
  deploy:
    uses: violabs/public-cicd/.github/workflows/deploy-digitalocean-ssh.yml@main
    with:
      remote-directory: '~/app'
      compose-file: 'docker-compose.prod.yml'
    secrets:
      DO_SSH_KEY: ${{ secrets.DO_SSH_KEY }}
      DO_HOST: ${{ secrets.DO_HOST }}
      DO_USER: ${{ secrets.DO_USER }}
      DO_KNOWN_HOSTS: ${{ secrets.DO_KNOWN_HOSTS }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `remote-directory` | `'~/app'` | Remote directory for deployment |
| `compose-file` | `'docker-compose.yml'` | Docker compose file to copy |
| `compose-profile` | `''` | Docker compose profile to use |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `DO_SSH_KEY` | Yes | SSH private key |
| `DO_HOST` | Yes | DigitalOcean host IP/domain |
| `DO_USER` | Yes | SSH username |
| `DO_KNOWN_HOSTS` | No | Known hosts entry (auto-scanned if not provided) |
| `DOCKERHUB_USERNAME` | Yes | Docker Hub username |
| `DOCKERHUB_TOKEN` | Yes | Docker Hub token |

---

## version-bump.yml

Bump semantic version and commit the change.

```yaml
jobs:
  bump:
    uses: violabs/public-cicd/.github/workflows/version-bump.yml@main
    with:
      version-file: 'VERSION'
      bump-type: 'patch'
      commit-message: 'chore: bump version to {version} [skip ci]'
    secrets:
      GIT_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `version-file` | `'VERSION'` | Path to VERSION file |
| `bump-type` | `'patch'` | Version component: `major`, `minor`, `patch` |
| `commit-message` | `'chore: bump version to {version} [skip ci]'` | Commit message (`{version}` replaced) |
| `git-user-name` | `'github-actions[bot]'` | Git user name |
| `git-user-email` | `'github-actions[bot]@users.noreply.github.com'` | Git user email |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `GIT_TOKEN` | No | Token for pushing (defaults to `GITHUB_TOKEN`) |

### Outputs

| Output | Description |
|--------|-------------|
| `old-version` | Version before bump |
| `new-version` | Version after bump |

---

## notify-discord.yml

Send rich Discord notifications about workflow results.

```yaml
jobs:
  notify:
    needs: [build, test, deploy]
    if: always()
    uses: violabs/public-cicd/.github/workflows/notify-discord.yml@main
    with:
      workflow_name: ${{ github.workflow }}
      build_result: ${{ needs.build.result }}
      test_result: ${{ needs.test.result }}
      deploy_result: ${{ needs.deploy.result }}
      pr_title: ${{ github.event.pull_request.title }}
      pr_number: ${{ github.event.pull_request.number }}
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `workflow_name` | (required) | Name of the calling workflow |
| `build_result` | `'skipped'` | Result of build job |
| `test_result` | `'skipped'` | Result of test job |
| `context_tests_json` | `'{}'` | JSON object of context test results |
| `frontend_result` | `'skipped'` | Result of frontend test job |
| `e2e_result` | `'skipped'` | Result of E2E test job |
| `detekt_result` | `'skipped'` | Result of Detekt job |
| `deploy_result` | `'skipped'` | Result of deploy job |
| `pr_title` | `''` | Pull request title |
| `pr_number` | `''` | Pull request number |
| `version` | `''` | Current version |
| `next_version` | `''` | Next version |
| `custom_title` | `''` | Custom title override |
| `custom_description` | `''` | Custom description override |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `DISCORD_WEBHOOK_URL` | Yes | Discord webhook URL |

### Context Tests JSON Example

```yaml
with:
  context_tests_json: |
    {
      "Component Tests": "${{ needs.component-tests.result }}",
      "API Tests": "${{ needs.api-tests.result }}",
      "Flow Tests": "${{ needs.flow-tests.result }}"
    }
```

---

## Complete Deployment Pipeline Example

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  check-changes:
    uses: violabs/public-cicd/.github/workflows/check-changes.yml@main
    with:
      mode: exclude
      exclude-patterns: '^(docs/|\.github/)'

  build:
    needs: check-changes
    if: needs.check-changes.outputs.changed == 'true'
    uses: violabs/public-cicd/.github/workflows/docker-build-push.yml@main
    with:
      image-name: 'myuser/myapp'
    secrets:
      REGISTRY_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      REGISTRY_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  deploy:
    needs: build
    uses: violabs/public-cicd/.github/workflows/deploy-digitalocean-ssh.yml@main
    with:
      remote-directory: '~/myapp'
    secrets:
      DO_SSH_KEY: ${{ secrets.DO_SSH_KEY }}
      DO_HOST: ${{ secrets.DO_HOST }}
      DO_USER: ${{ secrets.DO_USER }}
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}

  version-bump:
    needs: deploy
    uses: violabs/public-cicd/.github/workflows/version-bump.yml@main
    with:
      bump-type: 'patch'

  notify:
    needs: [build, deploy, version-bump]
    if: always()
    uses: violabs/public-cicd/.github/workflows/notify-discord.yml@main
    with:
      workflow_name: ${{ github.workflow }}
      build_result: ${{ needs.build.result }}
      deploy_result: ${{ needs.deploy.result }}
      version: ${{ needs.version-bump.outputs.new-version }}
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}
```
