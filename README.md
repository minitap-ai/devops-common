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

Runs only the pytest tests a PR's changed lines impact, via [`pytest-impacted`](https://github.com/promptromp/pytest-impacted) (git diff + AST import-graph). Skips tests the change can't affect, so the PR check stays fast on large suites.

Behaviour:
- **`pull_request`:** runs only affected tests (diff vs `base-ref`); passes if none are affected.
- **Other events (e.g. `push` to `main`):** runs the **full** suite as a safety net.
- **Force-all (native to `pytest-impacted`):** a dependency file (`uv.lock`, `pyproject.toml`, …) change runs everything; a `conftest.py` change runs all tests under its directory.

Requires `pytest-impacted` as a dev dependency in the consuming repo (`uv add --dev pytest-impacted`).

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

- `impacted-module` (required): `--impacted-module` value. Src-layout: full path with `src/` prefix (e.g. `src/testing_service`). Flat-layout: package name (e.g. `minitap`).
- `tests-dir` (optional, default `tests`): tests directory (`--impacted-tests-dir`); empty to omit.
- `base-ref` (optional, default `main`): branch the PR is diffed against.
- `pytest-args` (optional, default empty): args appended to every pytest run, e.g. `-m "not ios_simulator"`.
- `pre-test` (optional, default empty): shell command run before pytest, e.g. `cp .env.example .env`.
