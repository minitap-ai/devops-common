# DevOps Common

This repository contains common DevOps resources shared across Minitap projects - for instance, to store reusable GitHub Actions.

## Available GitHub Actions

### Setup Go with Private Repositories

This action configures a Go environment and sets up Git to allow fetching private Go modules from the `minitap-ai` GitHub organization.

#### Usage

```yaml
- uses: minitap-ai/devops-common/.github/actions/setup-go-private@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

#### Inputs

- `github-token` (required): A GitHub Personal Access Token (PAT) with access to the private repositories.

---

### GCP Docker Build and Push

This action builds a Docker image and pushes it to the Google Cloud Platform (GCP) Artifact Registry. It automatically tags the image based on the Git reference:
- **Tags (e.g., `v1.2.3`):** The image is tagged with the version number.
- **`development` branch:** The image is tagged as `latest`.

#### Usage

```yaml
- uses: minitap-ai/devops-common/.github/actions/gcp-docker-build-push@v1
  with:
    gcp-credentials: ${{ secrets.GCP_SA_KEY }}
    gcp-project-id: ${{ vars.GCP_PROJECT_ID }}
    gcp-region: ${{ vars.GCP_REGION }}
    gcp-artifact-repo: ${{ vars.GCP_ARTIFACT_REPO }}
    github-token: ${{ secrets.GH_PAT }}
    image-name: ${{ vars.IMAGE_NAME }}
```

#### Inputs

- `gcp-credentials` (required): The JSON key for a GCP Service Account with permissions to push to the Artifact Registry. Store this as a secret (e.g., `GCP_SA_KEY`).
- `gcp-project-id` (required): The ID of the GCP project. It is recommended to store this as a repository variable (e.g., `GCP_PROJECT_ID`).
- `gcp-region` (required): The GCP region where the Artifact Registry is located (e.g., `us-central1`). It is recommended to store this as a repository variable (e.g., `GCP_REGION`).
- `gcp-artifact-repo` (required): The name of the GCP Artifact Registry repository. It is recommended to store this as a repository variable (e.g., `GCP_ARTIFACT_REPO`).
- `github-token` (required): A GitHub Personal Access Token (PAT) used to access private Go modules during the Docker build. Store this as a secret (e.g., `GH_PAT`).
- `image-name` (required): The name of the Docker image to be built and pushed. It is recommended to store this as a repository variable (e.g., `IMAGE_NAME`).

---

### Affected Pytest

Runs only the pytest tests impacted by a pull request's changed lines, using [`pytest-impacted`](https://github.com/promptromp/pytest-impacted) (git diff + AST import-graph analysis). This makes the PR test check dramatically faster on large suites by skipping tests that the change cannot affect.

Behaviour:
- **On `pull_request` events:** selects and runs only the affected tests (diff against `base-ref`). If nothing is affected (pytest exit code 5), the step passes.
- **On any other event (e.g. `push` to `main`):** runs the **full** test suite as a safety net, so anything the per-PR selection might miss is always caught before it lands.
- **Force-run-all triggers (native to `pytest-impacted`):** when dependency files (`uv.lock`, `pyproject.toml`, …) change, the whole suite runs; when a `conftest.py` changes, all tests in its directory and subdirectories run.

Requires `pytest-impacted` to be installed as a dev dependency in the consuming repo (`uv add --dev pytest-impacted`).

#### Usage

```yaml
- name: Run affected tests
  uses: minitap-ai/devops-common/.github/actions/affected-pytest@v5
  with:
    # src-layout repo: pass the full path including the src/ prefix
    impacted-module: src/testing_service
    pre-test: cp .env.example .env

# Flat-layout repo with a marker filter:
- name: Run affected tests
  uses: minitap-ai/devops-common/.github/actions/affected-pytest@v5
  with:
    impacted-module: minitap
    pytest-args: -m "not ios_simulator"
```

#### Inputs

- `impacted-module` (required): Module path passed to `pytest-impacted`'s `--impacted-module`. For a **src-layout** repo use the full path including the `src/` prefix (e.g. `src/testing_service`); for a **flat-layout** repo use the package name (e.g. `minitap`). The plugin resolves `src/<pkg>` to the importable module name automatically.
- `tests-dir` (optional, default `tests`): Directory holding the test files (`--impacted-tests-dir`). Required when tests live outside the package so the dependency graph can include them. Set empty to omit.
- `base-ref` (optional, default `main`): Branch to diff the pull request against when selecting affected tests.
- `pytest-args` (optional, default empty): Extra arguments appended verbatim to every pytest invocation (both the affected and full-suite paths), e.g. `-m "not ios_simulator"`.
- `pre-test` (optional, default empty): Shell command run once before pytest, e.g. `cp .env.example .env`.
