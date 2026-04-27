# GitLab → GitHub Migration (GitHub Actions) — Setup Guide


## Executive Summary – Objective
  This document provides detailed procedures to migrate source code repositories from **GitLab Server** to **GitHub Enterprise cloud/EMU/Data Residency**.

## 1. Pre-requisites

### 1.1 GitHub Runner Host Requirements
- **OS:** Ubuntu
- **Docker:** latest stable
- **Node.js:** v20+
- **npm:** v10+
- **Docker image:** `gl-exporter`

### 1.2 GitHub Requirements
- **Object Storge Feature Flag:** GitHub Object Storage feature flag should be enabled for the target GitHub Org and GH Handle.
- **GitHub Personal Access Token (PAT):** - Create a GitHub PAT toke with the scopes **repo, admin:org, workflow, user**
- **Adding GitHub Runner:** GitHub Runner should be added to execute migration.
-  **To add new runner:**  Repo Settings --> Code and automation(section) --> Actions --> Runners --> new self-hosted runner --> Runner image (Linux) --> Execute the commands to configure

### 1.3 Intermediate Storage for Archive Files
- GitHub Storage (up to 30 GB)
- Azure / AWS Storage (up to 40 GB)
- **GitHub Enterprise Cloud with Data Residency requires Azure or AWS storage**

### 1.4 Install GH CLI
- Install GitHub CLI by following the link below
```
https://github.com/cli/cli#installation
```

### 1.5 Install GH CLI extensions
- Install gh-gitlab-stats CLI extension by running below command. This extension is required for generating inventory reports.
```
gh extension install https://github.com/mona-actions/gh-gitlab-stats
```
- Install gh-migration-monitor CLI extension by running below command. This extension is required to monitor the status of migrations.
```
gh extension install https://github.com/mona-actions/gh-migration-monitor
```
- Install the gh-ado2gh CLI extension. This extension is required to generate mannequins (user identity mapping) CSV files, reclaim mannequins, and wait for/check the migration status.
```
gh extension install https://github.com/github/gh-ado2gh
```
## 2. Repository Contents
    .
    ├── .github/workflows
    |      |-- gl-to-gh-migration.yaml
    ├── README.md
    ├── config.sh
    ├── runner.sh
    ├── gl-migration-readiness-check.sh
    ├── generate-gl-migration-archive.sh
    ├── upload-gl-migration-archive.sh
    ├── start-gl2gh-repo-migration.sh
    ├── gl-post-migration-validation.sh
    ├── gitlab-stats-sample.csv
    └── migration_scripts/
        ├── batch.js
        ├── create-env-vars.js
        ├── create-migration-source.js
        ├── gh-api.js
        ├── index.js
        ├── issue.js
        ├── migration.js
        ├── package.json
        ├── repository.js
        ├── start-repo-migration.js
        ├── state.js
        ├── team.js
        ├── upload-to-github-blob.sh
        ├── upload-to-azure-blob.sh
        ├── upload-to-aws-blob.sh
        ├── user.js
        └── workflow.js

## 3. Scripts and Purpose

### 3.1 Shell scripts
| Script | Purpose |
|------|---------|
| `config.sh` | Contains shared / generic variables used by multiple scripts. |
| `runner.sh` | Runner helper / wrapper script (used to execute the workflow in the runner environment). |
| `gl-migration-readiness-check.sh` | Check for active merge requests and running pipelines. |
| `generate-gl-migration-archive.sh` | Generates GitLab migration archives (exports) for repositories defined in the inventory. |
| `upload-gl-migration-archive.sh` | Uploads the generated archives to GitHub storage (used later by migration jobs). |
| `start-gl2gh-repo-migration.sh` | Triggers repository migrations in GitHub. |
| `gl-post-migration-validation.sh` | Compares branch and commit counts between GitLab and GitHub to validate migration. This script is not part of the CI/CD pipeline and must be run manually after migration completes. |

### 3.2 Scripts in `migration_scripts/` directory
This directory contains JavaScript modules used to orchestrate GitHub migration operations.

| List of JS scripts |
|------|
| `batch.js` |
| `create-env-vars.js` |
| `create-migration-source.js` |
| `gh-api.js` |
| `index.js` |
| `issue.js` |
| `migration.js` |
| `repository.js` |
| `start-repo-migration.js` |
| `state.js` |
| `team.js` |
| `user.js` |
| `workflow.js` |
| `upload-to-github-blob.sh` |
| `upload-to-azure-blob.sh` |
| `upload-to-aws-blob.sh` |
 

 









