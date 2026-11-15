# Flask CI/CD Pipelines: Jenkins and GitHub Actions

A Flask + MongoDB student management application (add, update, delete and list students) with two automated CI/CD pipelines:

## A **Jenkins** pipeline: Build, Test and Deploy to Staging, with email notifications.

## A **GitHub Actions** workflow: Install, Test, Build, Deploy to Staging and Deploy to Production.

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [Prerequisites](#2-prerequisites)
3. [Verifying the Installation](#3-verifying-the-installation)
4. [Running the Application Locally](#4-running-the-application-locally)
5. [Part 1: Jenkins Pipeline](#5-part-1-jenkins-pipeline)
6. [Part 2: GitHub Actions Pipeline](#6-part-2-github-actions-pipeline)
7. [Troubleshooting](#7-troubleshooting)

> **Screenshot note:** Every place marked **[**ADD** **SCREENSHOT**]** needs an image saved in a `screenshots/` folder in the repository, using the exact file name shown. The image links will then display automatically on GitHub.

---

## 1. Project Structure

``` . ├── app.py                      # Flask application ├── test_app.py                 # pytest unit tests ├── requirements.txt            # Python dependencies ├── Jenkinsfile                 # Jenkins pipeline definition ├── .github/workflows/ci-cd.yml # GitHub Actions workflow ├── templates/                  # **HTML** templates └── screenshots/                # Evidence images used in this **README** ```

## 2. Prerequisites

| Tool | Purpose | Version used |
|------|---------|--------------|
| Windows 10 PC | Host machine | Windows 10 |
| Git | Source control | (add version) |
| Python | Runs the app and tests | 3.12 (installed for all users) |
| JDK | Required by Jenkins | Temurin 17 |
| MongoDB Community Server | Database used by the tests | (add version) |
| Jenkins LTS | CI server | (add version) |
| GitHub account | Hosts the repo and runs Actions | n/a |
| Gmail account with an App Password | Sends Jenkins email notifications | n/a |

Installation commands (PowerShell as Administrator):

![Folder structure](Screenshots/01-installation-check.png)

Python is installed with `--scope machine` so the Jenkins Windows service can find it.

## 3. Verifying the Installation

Run this in PowerShell (as Administrator) to confirm every tool is installed and the services are running:

```powershell git --version python --version pip --version java -version mongod --version mongosh --version Get-Service Jenkins, MongoDB | Format-Table Name, Status, DisplayName (Invoke-WebRequest [http://localhost:**8080**/login](http://localhost:**8080**/login) -UseBasicParsing).StatusCode ```

Expected result: version numbers for each tool, both services `Running`, and status code `**200**`.

**[**ADD** **SCREENSHOT** 1]** Save as `screenshots/01-installation-check.png`

![Installation check](screenshots/01-installation-check.png)

**[**ADD** **SCREENSHOT** 2]** Jenkins dashboard open at [http://localhost:**8080**.](http://localhost:**8080**.) Save as `screenshots/02-jenkins-dashboard.png`

![Jenkins dashboard](screenshots/02-jenkins-dashboard.png)

## 4. Running the Application Locally

```powershell python -m venv venv venv\Scripts\activate pip install -r requirements.txt $env:MONGO_URI = *mongodb://localhost:**27017**/test_student_db* $env:SECRET_KEY = *test* python -m pytest -v ```

All four tests (`test_home_page`, `test_add_student`, `test_update_student`, `test_delete_student`) should pass.

The app uses a **TLS** CA bundle only for remote MongoDB Atlas connections (`mongodb+srv://`). Local MongoDB connections do not use **TLS**.

**[**ADD** **SCREENSHOT** 3]** Local pytest run showing 4 passed. Save as `screenshots/03-local-tests.png`

![Local tests](screenshots/03-local-tests.png)

---

## 5. Part 1: Jenkins Pipeline

### 5.1 Jenkins setup

## Open [http://localhost:8080](http://localhost:8080) and unlock Jenkins using the initial admin password:

    ```powershell
    Get-Content "C:\ProgramData\Jenkins\.jenkins\secrets\initialAdminPassword"
    ```
## Choose **Install suggested plugins**.
## Create the first admin user and keep the Jenkins URL as `[http://localhost:8080/`.](http://localhost:8080/`.)
## Confirm these plugins are installed under *Manage Jenkins, Plugins*: **Git**, **Pipeline**, **GitHub**, **Email Extension**.
## Restart the service so Jenkins picks up the installed tools:
    ```powershell
    Restart-Service Jenkins
    ```

### 5.2 Pipeline job

1. *New Item*, name it `flask-pipeline`, type **Pipeline**.
## Under **Pipeline**, choose *Pipeline script from SCM*.
## SCM: **Git**. Repository URL: this repository's URL.
## Branch: `*/main`. Script Path: `Jenkinsfile`.
## Save, then click **Build Now**.

Jenkins clones the repository into its workspace (`C:\ProgramData\Jenkins\.jenkins\workspace\flask-pipeline`) on each build.

**[**ADD** **SCREENSHOT** 4]** Pipeline job configuration page. Save as `screenshots/04-jenkins-job-config.png`

![Jenkins job configuration](screenshots/04-jenkins-job-config.png)

### 5.3 Jenkinsfile stages

| Stage | What it does |
|-------|--------------|
| Build | Creates a virtual environment and installs dependencies with pip |
| Test | Runs the pytest suite against a local MongoDB |
| Deploy to Staging | If the tests pass, copies the application to `C:\staging\flask_app` |

The `post` section sends an email on success and on failure.

### 5.4 Pipeline execution

**[**ADD** **SCREENSHOT** 5]** Stage View showing Build, Test and Deploy to Staging all green. Save as `screenshots/05-jenkins-stage-view.png`

![Jenkins stage view](screenshots/05-jenkins-stage-view.png)

**[**ADD** **SCREENSHOT** 6]** Console Output of a successful build (ending with `Finished: **SUCCESS**`). Save as `screenshots/06-jenkins-console-success.png`

![Jenkins console output](screenshots/06-jenkins-console-success.png)

### 5.5 Trigger on push to `main`

The job uses **Poll **SCM**** with the schedule `H/2 * * * *`, so Jenkins checks the repository every two minutes and starts a build when it finds new commits on `main`.

To test: push any small change to `main` and wait up to two minutes. A new build appears automatically with the cause ***Started by an **SCM** change***.

**[**ADD** **SCREENSHOT** 7]** Poll **SCM** setting under Build Triggers. Save as `screenshots/07-jenkins-poll-scm.png`

![Poll **SCM** setting](screenshots/07-jenkins-poll-scm.png)

**[**ADD** **SCREENSHOT** 8]** A build showing *Started by an **SCM** change*. Save as `screenshots/08-jenkins-scm-trigger.png`

![Build started by **SCM** change](screenshots/08-jenkins-scm-trigger.png)

### 5.6 Email notifications

Configured under *Manage Jenkins, System, E-mail Notification*:

| Setting | Value |
|---------|-------|
| SMTP server | `smtp.gmail.com` |
| Use SMTP Authentication | Enabled, with the Gmail address and a Gmail App Password |
| Use SSL | Enabled |
| SMTP port | `465` |
| System Admin e-mail address (Jenkins Location) | The Gmail address used above |

An App Password is created in Google Account, Security, 2-Step Verification, App passwords. The App Password is never committed to the repository.

An email is sent when a build **succeeds** and when it **fails**.

**[**ADD** **SCREENSHOT** 9]** E-mail Notification settings (hide the password). Save as `screenshots/09-jenkins-email-config.png`

![Email configuration](screenshots/09-jenkins-email-config.png)

**[**ADD** **SCREENSHOT** 10]** Success email received in the inbox. Save as `screenshots/10-email-success.png`

![Success email](screenshots/10-email-success.png)

**[**ADD** **SCREENSHOT** 11]** Failure email received in the inbox (break a test on purpose, push, then fix it). Save as `screenshots/11-email-failure.png`

![Failure email](screenshots/11-email-failure.png)

---

## 6. Part 2: GitHub Actions Pipeline

### 6.1 Branches

The repository has two branches:

- `main`: production-ready code
- `staging`: pre-production code

### 6.2 Workflow file

The workflow lives at `.github/workflows/ci-cd.yml` and runs the following jobs:

| Job | Purpose |
|-----|---------|
| Install dependencies and run tests | Installs requirements with pip and runs pytest against a MongoDB service container |
| Build package | If tests pass, zips the application and uploads it as a build artifact |
| Deploy to Staging | Runs only on pushes to the `staging` branch |
| Deploy to Production | Runs only when a release tag starting with `v` is pushed |

### 6.3 When each job runs

| Event | Jobs that run |
|-------|---------------|
| Push to `main` | test, build |
| Push to `staging` | test, build, deploy-staging |
| Tag `v*` (for example `v1.0.0`) | test, build, deploy-production |

The deploy jobs download the built package, read the deploy key from GitHub Secrets, and simulate the deployment by unpacking the package and logging the release. The same structure works for a real host by replacing the deploy step with a real command.

### 6.4 Environment secrets

Sensitive values are stored in GitHub Secrets, never in the repository.

To add a secret: **Repository, Settings, Secrets and variables, Actions, New repository secret.**

| Secret name | Used by | Purpose |
|-------------|---------|---------|
| `STAGING_DEPLOY_KEY` | Deploy to Staging | Deployment key for the staging environment |
| `PRODUCTION_DEPLOY_KEY` | Deploy to Production | Deployment key for the production environment |

Inside the workflow, a secret is read as `${{ secrets.STAGING_DEPLOY_KEY }}`. GitHub masks secret values in logs.

**[**ADD** **SCREENSHOT** 12]** Secrets page showing both secret names (values stay hidden). Save as `screenshots/12-github-secrets.png`

![GitHub secrets](screenshots/12-github-secrets.png)

### 6.5 How to trigger each path

```powershell # Test and build (push to main) git push origin main

# Deploy to staging (push to staging)

git checkout staging git merge main git push origin staging

# Deploy to production (tag a release)

git checkout main git tag v1.0.0 git push origin v1.0.0 ```

### 6.6 Workflow execution

**[**ADD** **SCREENSHOT** 13]** Actions tab listing the runs for `main`, `staging` and the tag. Save as `screenshots/13-actions-runs-list.png`

![Actions runs](screenshots/13-actions-runs-list.png)

**[**ADD** **SCREENSHOT** 14]** Run on the `staging` branch showing test, build and deploy-staging all green. Save as `screenshots/14-actions-staging-run.png`

![Staging run](screenshots/14-actions-staging-run.png)

**[**ADD** **SCREENSHOT** 15]** Run on the `v1.0.0` tag showing test, build and deploy-production all green. Save as `screenshots/15-actions-production-run.png`

![Production run](screenshots/15-actions-production-run.png)

---

## 7. Troubleshooting

| Problem | Fix |
|---------|-----|
| Jenkins: `'python' is not recognized` | Install Python with `--scope machine`, then run `Restart-Service Jenkins` |
| Jenkins: `'venv\Scripts\py' is not recognized` | Use `venv\Scripts\python`, not `py`, after the virtual environment is created |
| Tests fail with `SSL handshake failed` on localhost | Use TLS options only for `mongodb+srv://` URIs (already handled in `app.py`) |
| Tests fail with a connection timeout | Start MongoDB: `Start-Service MongoDB` |
| Jenkins mail step: `Couldn't connect to host, port: localhost, 25` | SMTP is not configured; set it up under *Manage Jenkins, System, E-mail Notification* |
| Gmail authentication fails | Use a Gmail App Password, not the normal account password |
| Deploy job fails with `... is missing` | The matching GitHub Secret is not created or is misspelled |
