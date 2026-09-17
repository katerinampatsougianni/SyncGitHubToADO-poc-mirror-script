# SyncGitHubToADO-poc-mirror-script

POC - Mirror GitHub repository to Azure DevOps via GitHub Actions

## Description

This repo demonstrates how to sync a GitHub repository to Azure DevOps (ADO). **GitHub is the source of truth** — developers push code to GitHub and all changes are automatically mirrored to ADO.

## Overall Flow

```
Developer push → GitHub
       │
       ├── mirror-to-ado.yml ──────→ ADO repo (sync all branches)
       │
       └── build-and-publish.yml ──→ Azure Artifacts (my-app-katerina)
                                            │
                                            └── Azure_pipeline_push_ACR.yaml ──→ ACR
```

## Files

### `.github/workflows/mirror-to-ado.yml`

Syncs GitHub → ADO automatically.

**Triggers:**
- Push to any branch
- Branch deletion
- Manual (workflow_dispatch)

**Jobs:**
- `mirror-push`: Checks out code from GitHub and force-pushes to ADO repo
- `mirror-delete`: When a branch is deleted in GitHub, deletes it from ADO as well

**Secrets required:** `POC_MIRROR_SCRIPT_PAT`

**Supported sync operations:**
- Add, update, delete, rename files (via commits)
- Create new branches
- Delete branches
- Rename branches (GitHub fires delete + push events)

---

### `.github/workflows/build-and-publish.yml`

Builds a Docker image and publishes it to Azure Artifacts.

**Triggers:** Push to `main`, or manual (workflow_dispatch)

**Steps:**
1. Build Docker image from `docker/Dockerfile`
2. Save image as `image.tar.gz`
3. Publish to Azure Artifacts feed `images` as package `my-app-katerina` with version `1.0.{run_number}`
4. Trigger ADO pipeline `103` (ener-gh-test) with the version as parameter

**Secrets required:** `ADO_PUBLISH_ARTIFACT`

---

### `pipelines/Azure_pipeline_push_ACR.yaml`

Downloads the Docker image from Azure Artifacts and pushes it to Azure Container Registry (ACR).

**Trigger:** Manual only (triggered automatically by `build-and-publish.yml`)

**Parameter:** `version` (e.g. `1.0.5`)

**Steps:**
1. Download package `my-app-katerina` with the specified version from Azure Artifacts
2. Load the image into Docker daemon
3. Login to ACR (`enerwaveacrpoc001.azurecr.io`) and push the image

---

### `docker/Dockerfile`

Defines the Docker image. Currently a minimal POC image based on Alpine Linux.

---

### `sync-test-files/`

Sample files for testing the GitHub → ADO sync:

| File | Purpose |
|------|---------|
| `file1-add.txt` | Test adding a new file |
| `file2-update.txt` | Test updating an existing file |
| `file3-delete.txt` | Test deleting a file |
| `file4-rename.txt` | Test renaming a file |

## Secrets

| Secret | Used by | Purpose |
|--------|---------|---------|
| `POC_MIRROR_SCRIPT_PAT` | `mirror-to-ado.yml` | ADO PAT for pushing to ADO repo |
| `ADO_PUBLISH_ARTIFACT` | `build-and-publish.yml` | ADO PAT for publishing to Azure Artifacts and triggering pipelines |
