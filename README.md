# MyBOQ Frontend Deployment Pipeline Implementation Plan

## 1. Overview

This document describes the implementation plan for the **MyBOQ Frontend Deployment Pipeline** using **GitHub Actions**.
The objective is to create a **reliable, automated, and traceable deployment system** for the React frontend application hosted on **Hostinger**.

Currently, deployments require manual steps such as building the React project locally and uploading files to the server. This process can lead to:

* Deploying outdated code
* Missing commits during build
* Difficulty tracking which version is deployed
* Lack of rollback capability

To solve these problems, we will implement a **CI/CD pipeline using GitHub Actions** that will automate the deployment process while maintaining **backup, deployment history, and rollback capabilities**.

---

# 2. Goals of This Pipeline

The deployment pipeline will achieve the following goals:

### 1. Automated Deployment

The React frontend will be automatically built and deployed to the production server through a workflow.

### 2. Deployment Consistency

Production deployments will always use the **latest verified code from the main branch**.

### 3. Version Tracking

Each deployment will record:

* commit hash
* deployment datetime
* version name
* release notes

### 4. Safe Rollback

If a deployment introduces a problem, we can quickly revert to the **previous working version**.

### 5. Deployment History

A dedicated branch will maintain a complete log of all production deployments.

---

# 3. Technology Stack Used

The deployment pipeline will use the following tools:

* **GitHub Actions** – CI/CD workflow automation
* **GitHub Repository Branching Strategy**
* **React Build System (react-scripts build)**
* **FTP Deployment to Hostinger**
* **Git-based deployment history tracking**

---

# 4. Branch Strategy

The repository will use a structured branching model to manage deployments.

| Branch         | Purpose                                           |
| -------------- | ------------------------------------------------- |
| main           | Primary development branch                        |
| deploy         | Represents the currently deployed production code |
| backup         | Stores the previous deployment for rollback       |
| deploy-history | Stores deployment records and metadata            |

### Explanation

**main**

* Developers push feature updates here.
* This branch always contains the latest development version.

**deploy**

* Contains the exact code currently deployed to production.
* The production build is generated from this branch.

**backup**

* Stores the previous production version before a new deployment.
* Used for rollback in case of deployment failure.

**deploy-history**

* Stores deployment records.
* Contains a structured log of deployments including version details.

---

# 5. Deployment Flow

The deployment pipeline will follow the process below.

```
Developer triggers deployment workflow
            ↓
Pipeline reads deployment inputs
(version name + release notes)
            ↓
Current deploy branch is backed up
(deploy → backup)
            ↓
Latest code from main is merged into deploy
            ↓
React production build is generated
            ↓
Build files are uploaded to Hostinger
(public_html)
            ↓
Deployment details are recorded in history
```

This ensures that every production deployment is:

* reproducible
* traceable
* recoverable

---

# 6. Workflow 1 – Production Deployment

The primary workflow will be:

**deploy-to-production**

This workflow will be triggered manually from GitHub.

### Trigger Type

Manual trigger using:

```
workflow_dispatch
```

This allows developers to deploy only when the code is verified.

---

# 7. Workflow Inputs

The deployment workflow will accept the following inputs.

| Input        | Description                          |
| ------------ | ------------------------------------ |
| version_name | version label for the deployment     |
| notes        | description of new features or fixes |

Example:

```
version_name: v1.4.2
notes: Steel calculation fixes and UI improvements
```

---

# 8. Deployment Steps

## Step 1 – Checkout Repository

The workflow first checks out the repository.

This ensures the pipeline has access to the latest source code.

---

## Step 2 – Install Dependencies

The React project dependencies will be installed.

```
npm install
```

---

## Step 3 – Backup Current Deployment

Before deploying new code, the current deployment will be backed up.

Process:

```
backup branch ← deploy branch
```

This ensures the previously deployed version is preserved.

---

## Step 4 – Update Deploy Branch

The deploy branch will be updated with the latest code from the main branch.

```
deploy branch ← main branch
```

After this step, the deploy branch represents the **new production candidate**.

---

## Step 5 – Build React Application

The production build will be created using:

```
npm run build
```

This generates optimized static files in the **build/** directory.

The build process includes:

* production React compilation
* JavaScript obfuscation
* asset optimization

---

# 9. Deploying to Hostinger

The build output will be deployed to the Hostinger server.

Target directory:

```
public_html/
```

The deployment process will:

* upload build files
* replace existing production files

However, one important file must be preserved.

### `.htaccess`

The `.htaccess` file contains important Apache rules such as:

* React Router refresh handling
* rewrite rules
* caching headers

Since React builds do not generate `.htaccess`, the deployment process will **exclude this file**.

Therefore:

```
.htaccess will NOT be replaced
```

All other files and folders (such as `static/`, `index.html`, images, etc.) will be replaced.

---

# 10. Deployment History Tracking

A separate branch called **deploy-history** will maintain deployment records.

This branch will contain a structured log of deployments.

Example structure:

```
deploy-history/
    deployments.json
```

Example deployment record:

```json
[
  {
    "version": "v1.4.2",
    "commit": "c91ae82",
    "date": "2026-03-05 14:22",
    "notes": "Steel calculation fixes and UI improvements",
    "deployed_by": "dhanu"
  }
]
```

This allows the team to easily track:

* which commit was deployed
* when deployment occurred
* what features were released

---

# 11. Workflow 2 – Revert Deployment

The second workflow will allow quick rollback.

Workflow name:

```
revert-last-deployment
```

### Purpose

If the current deployment causes problems, the system can restore the previous version.

### Process

```
deploy branch ← backup branch
```

After this step, the previous version becomes the production candidate again.

The pipeline will then redeploy that version to the server.

This provides a **fast recovery mechanism**.

---

# 12. GitHub Secrets Configuration

Sensitive deployment credentials must be stored securely.

GitHub repository secrets will include:

| Secret       | Purpose              |
| ------------ | -------------------- |
| FTP_HOST     | Hostinger FTP server |
| FTP_USERNAME | FTP username         |
| FTP_PASSWORD | FTP password         |

These secrets will be used by the deployment workflow to upload files securely.

---

# 13. Advantages of This Pipeline

Implementing this pipeline provides several important benefits.

### 1. Fully Automated Deployments

No need to manually upload files to the server.

---

### 2. Reduced Human Errors

The pipeline ensures that deployments always use the correct branch and latest commits.

---

### 3. Deployment History

The team can always check:

* when deployment happened
* which commit was deployed
* what features were included

---

### 4. Safe Rollbacks

If a deployment fails, the system can revert quickly.

---

### 5. Better Team Collaboration

All developers can clearly see the deployment process and version history.

---

### 6. Production Stability

Structured deployment reduces risk and improves production reliability.

---

# 14. Conclusion

This deployment pipeline introduces a **structured CI/CD system** for the MyBOQ frontend application.

By combining **GitHub Actions automation**, **branch-based deployment control**, **backup strategy**, and **deployment history tracking**, the team will gain:

* safer deployments
* better version visibility
* faster recovery from errors
* reduced manual workload

Once implemented, this pipeline will ensure that production deployments are **consistent, traceable, and reliable**.

---
