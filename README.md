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
