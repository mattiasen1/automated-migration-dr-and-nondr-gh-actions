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

### 1.4 Actions Secrets Setup (GitHub)
Configure Github Actions needed Secrets at:
**GitHub Repo → Settings → Security and quality(section) → Secrets and Variables → Actions →New repository secret**

### For GitHub Enterprise Cloud

| Key | Example value | Description |
|-----|---------------|-------------|
| `SOURCE_GL_SERVER_URL` | `https://gitlab.company.com` | GitLab Server URL |
| `GITLAB_USERNAME` | `gitlab-user` | GitLab username |
| `GITLAB_API_PRIVATE_TOKEN` | `glpat-xxxxxxx` | GitLab API private token (masked) |
| `GH_PAT` | `ghp_xxxxx` | GitHub Personal Access Token (masked) |
| `GH_ORG` | `myghorg` | GitHub Organization name |

### For GitHub Enterprise Cloud with Data Residency

| Key | Example value | Description |
|-----|---------------|-------------|
| `SOURCE_GL_SERVER_URL` | `https://gitlab.company.com` | GitLab Server URL |
| `GITLAB_USERNAME` | `gitlab-user` | GitLab username |
| `GITLAB_API_PRIVATE_TOKEN` | `glpat-xxxxxxx` | GitLab API private token  |
| `GH_PAT` | `ghp_xxxxx` | GitHub Personal Access Token  |
| `GH_ORG` | `myghorg` | GitHub Organization name |
| `GH_SERVER_URL` | `https://SUBDOMAIN.ghe.com` | GitHub URL |
| `GH_API_URL` | `https://api.SUBDOMAIN.ghe.com` | GitHub API URL |
| `STORAGE_TYPE` | `AZURE` or `AWS` | Intermediate storage type |

- If storage type is Azure, export the following env

| Key | Example value | Description |
|-----|---------------|-------------|
| `AZ_CONTAINER` | `container-name` | Azure container name |
| `AZURE_STORAGE_CONNECTION_STRING` | `connection-string` | Azure storage connection string  |

- If storage type is AWS, export the following env

| Key | Example value | Description |
|-----|---------------|-------------|
| `AWS_BUCKET_NAME` | `my-aws-bucket` | AWS S3 bucket name |
| `AWS_REGION` | `us-west-2` | AWS region |
| `AWS_ACCESS_KEY_ID` | `ABCDEFNN7EXAMPLE` | AWS access key ID  |
| `AWS_SECRET_ACCESS_KEY` | `ZYHXWVUtnFEXAMPLE/...` | AWS secret access key  |

### 1.5 Install GH CLI
- Install GitHub CLI by following the link below
```
https://github.com/cli/cli#installation
```

### 1.6 Install GH CLI extensions
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
 

 ## 4. Generate & Update inventory CSV
Before triggering the pipeline, generate an inventory file using the GitHub CLI extension `gitlab-stats`:

```bash
gh gitlab-stats --hostname "$SOURCE_GL_SERVER_URL" --token "$GITLAB_API_PRIVATE_TOKEN" --namespace <gitlab-group>
```

This produces a CSV inventory of repositories.

After generation, edit the CSV and add two columns:
- `github_org`
- `github_repo`

Fill in the target GitHub organization and repository name for each row.

#### Example Inventory CSV

| Namespace | Project | Commit_Count | Branch_Count | Full_URL | github_org | github_repo |
| -------- | -------- | -------- | -------- | -------- | -------- | -------- |
| demo-group/sub-group | demo-project | 20 | 1 | http://gitlab-server/demo-group/sub-group/demo-project | my-enterprise-org | demoproject |
| demo-group-1/sub-group-1 | demo-project-1 | 20 | 1 | http://gitlab-server/demo-group/sub-group/demo-project-1 | my-enterprise-org | demoproject1 |

**Notes**
- The example shows only the minimum required columns.
- The actual inventory CSV may contain additional metadata columns generated by `gh gitlab-stats`.
- Columns `github_org` and `github_repo` must be populated before running the pipeline.
- Upload the CSV to the GitLab project.
- This file name will be passed as the `INVENTORY_FILE` user input when running the pipeline.










