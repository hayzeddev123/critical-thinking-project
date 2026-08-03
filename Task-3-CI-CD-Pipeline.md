# Task 3: CI/CD Integration with GitHub Actions

## 1. Overview

To ensure code quality and enable rapid, safe iteration, the team integrates Git with a Continuous Integration / Continuous Deployment (CI/CD) pipeline using **GitHub Actions**.

The pipeline automatically:

- Runs **code quality checks** (linting and formatting)
- Executes **unit tests**
- Deploys successful changes from the `main` branch to a **staging environment**

This happens every time a developer pushes a feature branch or opens a Pull Request.

---

## 2. Pipeline Location

The complete workflow is defined in:

```
.github/workflows/ci-cd.yml
```

This file is automatically detected and executed by GitHub Actions.

---

## 3. How the Pipeline Works

### Trigger Events

The pipeline runs automatically when:

| Event                        | Branches                          | What happens                                      |
|-----------------------------|-----------------------------------|---------------------------------------------------|
| Push                        | `feature/**`, `bugfix/**`, `hotfix/**`, `main` | Runs quality checks + unit tests                  |
| Pull Request                | Targeting `main`                  | Runs quality checks + unit tests                  |
| Push to `main` (after merge)| `main` only                       | Runs all checks **and** deploys to staging        |

### Pipeline Jobs (in order)

#### Job 1: Code Quality Checks
- Checks out the code
- Installs dependencies
- Runs ESLint (static code analysis)
- Runs Prettier format check

**Purpose:** Catch style issues, potential bugs, and enforce team coding standards early.

#### Job 2: Unit Tests
- Runs only if Code Quality passes (`needs: code-quality`)
- Executes the test suite with coverage reporting
- Uploads the coverage report as an artifact

**Purpose:** Verify that the new code does not break existing functionality.

#### Job 3: Deploy to Staging
- Runs only when:
  - All previous jobs succeed, **and**
  - The event is a push to the `main` branch
- Builds the application
- Deploys the build to the staging environment

**Purpose:** Automatically make verified code available for further testing in a staging environment that mirrors production.

---

## 4. How to Trigger the Tests

### Automatic Triggers (Recommended)

1. Create or switch to a feature branch:
   ```bash
   git checkout -b feature/my-new-feature
   ```

2. Make changes, commit, and push:
   ```bash
   git add .
   git commit -m "Add new feature"
   git push -u origin feature/my-new-feature
   ```

3. GitHub Actions will automatically start the pipeline.
   - Go to the repository → **Actions** tab to watch the live run.

4. Open a Pull Request to `main`. The same checks will run again on the PR.

### Manual Trigger (Optional)
You can also re-run a failed or successful workflow from the Actions tab by clicking **Re-run all jobs**.

---

## 5. How Deployment to Staging Works

Deployment happens **only after a successful merge into `main`**.

Typical flow:

1. Developer finishes work on a feature branch.
2. Opens a Pull Request → Code review + CI checks pass.
3. Pull Request is merged into `main` (preferably using “Squash and merge”).
4. The push to `main` triggers the full pipeline.
5. After tests pass, the **Deploy to Staging** job runs.
6. The application is built and deployed to the staging environment.

### Customizing the Deployment Step

In the workflow file, the deployment step currently contains placeholder commands:

```yaml
- name: Deploy to Staging
  run: |
    echo "Deploying to staging environment..."
    # Replace this section with your real deployment method
```

Common real-world options you can plug in:

- **Vercel / Netlify**: Use their official GitHub Actions
- **AWS**: `aws s3 sync` or Elastic Beanstalk / ECS
- **Docker + Kubernetes**: Build image → push to registry → deploy
- **SSH / rsync**: Copy files to a staging server
- **Custom script**: `npm run deploy:staging`

You should also configure secrets (API keys, SSH keys, etc.) in:
**Repository Settings → Secrets and variables → Actions**

---

## 6. Branch Protection (Recommended Setup)

To force the use of this pipeline, enable Branch Protection Rules on `main`:

1. Go to **Settings → Branches → Add rule**
2. Branch name pattern: `main`
3. Enable:
   - Require a pull request before merging
   - Require status checks to pass before merging
   - Select the status checks: `Code Quality Checks` and `Unit Tests`
   - Do not allow bypassing the above settings

This guarantees that no code reaches `main` (and therefore staging) without passing the automated checks.

---

## 7. Required Project Scripts

For the pipeline to work, the project’s `package.json` should contain at least these scripts:

```json
{
  "scripts": {
    "lint": "eslint .",
    "format:check": "prettier --check .",
    "test": "jest",
    "build": "your-build-command",
    "deploy:staging": "your-deploy-command"
  }
}
```

(Adjust the exact commands according to the technology stack you choose.)

---

## 8. Benefits for the Distributed Team

| Benefit                        | How the pipeline delivers it                              |
|--------------------------------|-----------------------------------------------------------|
| Fast feedback                  | Developers see test results within minutes of pushing     |
| Consistent quality             | Every change is checked the same way, regardless of who wrote it |
| Reduced risk                   | Broken code is stopped before it reaches staging/production |
| Asynchronous collaboration     | Team members in different time zones get automatic verification |
| Confidence to merge            | Green checks + code review = safe to merge                |
| Faster releases                | Staging is always up-to-date with verified main branch    |

---

## 9. Summary

The GitHub Actions CI/CD pipeline:

- Automatically runs on every feature branch push and every Pull Request
- Enforces code quality and unit tests before any merge
- Deploys only verified code from `main` to the staging environment
- Integrates seamlessly with the Feature Branch + Pull Request workflow defined in Task 2

This setup gives the team rapid iteration while maintaining high code quality and a stable staging environment.
