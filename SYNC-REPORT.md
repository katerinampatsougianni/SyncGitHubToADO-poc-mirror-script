# GitHub to Azure DevOps Sync — POC Report

**Project:** SyncGitHubToADO-poc-mirror-script  
**Organization:** ACN-CCoE / Enerwave - Digital Services  
**Date:** 2026-09-18

---

## Overview

One-way sync from GitHub to Azure DevOps. GitHub is the **source of truth** — developers push code to GitHub and all changes are automatically mirrored to ADO.

---

## Flow

```
Developer push → GitHub
  ├── Mirror to ADO repo (all branches)
  └── Build & publish Docker image → Azure Artifacts → ACR
```

## Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| `mirror-to-ado.yml` | GitHub → `.github/workflows/` | Syncs GitHub repo to ADO on every push/delete |
| `build-and-publish.yml` | GitHub → `.github/workflows/` | Builds Docker image and publishes to Azure Artifacts |
| `Azure_pipeline_push_ACR.yaml` | GitHub → `pipelines/` / ADO pipeline 106 | Pushes image from Azure Artifacts to ACR |
| `github-ado-sync-template` | GitHub repo (template) | Template repo with pre-configured workflows |

---

## Test Results

| Scenario | Result |
|----------|--------|
| Add / Update / Delete / Rename file | ✅ |
| Create / Delete / Rename branch | ✅ |
| Build & publish image to Azure Artifacts | ✅ |
| Trigger ADO pipeline automatically | ✅ |
| Push image to ACR (`my-app-katerina:1.0.x`) | ✅ |
| Create new repo from template | ✅ |

---

## Notes

- Workflow must be **enabled** manually the first time in GitHub Actions
- `ADO_PUBLISH_ARTIFACT` PAT requires: Packaging + Build (Read & execute) scopes
- New repos from template require manual setup of secrets and ADO repository
