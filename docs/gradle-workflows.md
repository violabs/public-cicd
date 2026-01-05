# Gradle Workflows

Reusable workflows for Gradle-based Java/Kotlin projects.

## gradle-build.yml

Fast compilation check without running tests.

```yaml
jobs:
  build:
    uses: violabs/public-cicd/.github/workflows/gradle-build.yml@main
    with:
      java-version: '21'
      build-tasks: 'classes testClasses'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `build-tasks` | `'classes testClasses'` | Gradle tasks to run |
| `gradle-args` | `'--no-daemon'` | Additional Gradle arguments |
| `runs-on` | `'ubuntu-latest'` | Runner (string or JSON array) |

### Outputs

| Output | Description |
|--------|-------------|
| `result` | Build result (success/failure) |

---

## gradle-unit-tests.yml

Run unit tests with optional Kover coverage reporting.

```yaml
jobs:
  test:
    uses: violabs/public-cicd/.github/workflows/gradle-unit-tests.yml@main
    with:
      java-version: '21'
      coverage-enabled: true
      min-coverage-overall: 60
      add-pr-coverage-comment: true
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `module` | `''` | Gradle module (e.g., `:app`). Empty for root |
| `module-path` | `'.'` | Path to module for artifacts |
| `artifact-suffix` | `'root'` | Suffix for artifact names |
| `test-task` | `'test'` | Test task name |
| `coverage-enabled` | `true` | Enable Kover coverage |
| `coverage-report-task` | `'koverXmlReport'` | Kover report task |
| `exclude-tasks` | `''` | Tasks to exclude (space-separated) |
| `gradle-args` | `''` | Additional Gradle arguments |
| `upload-test-reports` | `true` | Upload test reports |
| `upload-coverage-reports` | `true` | Upload coverage reports |
| `add-pr-coverage-comment` | `true` | Add coverage comment to PR |
| `min-coverage-overall` | `40` | Minimum overall coverage % |
| `min-coverage-changed-files` | `40` | Minimum coverage for changed files % |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## gradle-context-tests.yml

Run integration/context tests (Spring Boot, database, etc.).

```yaml
jobs:
  context-tests:
    uses: violabs/public-cicd/.github/workflows/gradle-context-tests.yml@main
    with:
      test-name: 'Component Tests'
      test-task: 'testComponents'
      exclude-tasks: 'npmInstall'
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `test-name` | `'Context Tests'` | Display name |
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `test-task` | `'testComponents'` | Gradle test task |
| `exclude-tasks` | `''` | Tasks to exclude |
| `gradle-args` | `''` | Additional Gradle arguments |
| `upload-test-reports` | `true` | Upload test reports |
| `artifact-name` | `''` | Artifact name (auto-generated from test-name) |
| `reports-path` | `'build/reports/tests/'` | Path to reports |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## gradle-matrix-tests.yml

Run tests across multiple modules in parallel.

```yaml
jobs:
  detect:
    uses: violabs/public-cicd/.github/workflows/gradle-detect-modules.yml@main

  test:
    needs: detect
    if: needs.detect.outputs.has-modules == 'true'
    uses: violabs/public-cicd/.github/workflows/gradle-matrix-tests.yml@main
    with:
      modules-json: ${{ needs.detect.outputs.matrix }}
      coverage-enabled: true
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `modules-json` | (required) | JSON array: `[{"module":":app","path":"app","filename":"app"}]` |
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `test-task` | `'test'` | Test task name |
| `coverage-enabled` | `true` | Enable coverage |
| `exclude-tasks` | `''` | Tasks to exclude |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## gradle-detect-modules.yml

Detect changed modules for matrix testing. Requires a `detectChangedModules` Gradle task.

```yaml
jobs:
  detect:
    uses: violabs/public-cicd/.github/workflows/gradle-detect-modules.yml@main
    with:
      java-version: '21'
```

### Outputs

| Output | Description |
|--------|-------------|
| `matrix` | JSON array of modules |
| `has-modules` | `'true'` if modules were detected |

---

## gradle-detekt.yml

Run Detekt static analysis.

```yaml
jobs:
  detekt:
    uses: violabs/public-cicd/.github/workflows/gradle-detekt.yml@main
    with:
      module: ':app'
      fallback-to-root: true
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `module` | `''` | Specific module to analyze |
| `fallback-to-root` | `true` | Run root detekt if module fails |
| `gradle-args` | `''` | Additional Gradle arguments |
| `runs-on` | `'ubuntu-latest'` | Runner |

---

## gradle-task-commit.yml

Run a Gradle task and commit any resulting changes back to the PR branch. Useful for auto-formatting, code generation, or any task that modifies files.

```yaml
jobs:
  format:
    uses: violabs/public-cicd/.github/workflows/gradle-task-commit.yml@main
    with:
      gradle-task: spotlessApply
      commit-message: 'style: auto-format code [skip ci]'
    secrets:
      GIT_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `gradle-task` | (required) | Gradle task to run |
| `gradle-args` | `'--no-daemon'` | Additional Gradle arguments |
| `java-version` | `'21'` | Java version |
| `java-distribution` | `'temurin'` | Java distribution |
| `commit-message` | `'chore: apply {task} changes [skip ci]'` | Commit message (`{task}` replaced) |
| `git-user-name` | `'github-actions[bot]'` | Git user name |
| `git-user-email` | `'github-actions[bot]@users.noreply.github.com'` | Git user email |
| `paths-to-commit` | `'.'` | Paths to stage (space-separated) |
| `fail-on-changes` | `false` | Fail instead of committing (check mode) |
| `runs-on` | `'ubuntu-latest'` | Runner |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `GIT_TOKEN` | No | Token for pushing (defaults to `GITHUB_TOKEN`) |

### Outputs

| Output | Description |
|--------|-------------|
| `task-result` | Gradle task result |
| `changes-detected` | Whether changes were detected |
| `changes-committed` | Whether changes were committed |

### Use Cases

**Auto-format code:**
```yaml
format:
  uses: violabs/public-cicd/.github/workflows/gradle-task-commit.yml@main
  with:
    gradle-task: spotlessApply
```

**Check mode (fail if formatting needed):**
```yaml
format-check:
  uses: violabs/public-cicd/.github/workflows/gradle-task-commit.yml@main
  with:
    gradle-task: spotlessApply
    fail-on-changes: true
```

**Generate protobuf files:**
```yaml
generate:
  uses: violabs/public-cicd/.github/workflows/gradle-task-commit.yml@main
  with:
    gradle-task: generateProto
    paths-to-commit: 'src/main/generated'
```
