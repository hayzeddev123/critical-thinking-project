# Task 2: Implement Git Workflows for a Team Project

## 1. Scenario

Our team is developing a new web application. Multiple developers need to work on different features at the same time (for example: user authentication, dashboard UI, payment integration, and API endpoints) while keeping the `main` branch stable and always deployable.

This requires a clear, consistent Git workflow that supports parallel development, code review, and quality checks before any code reaches the main branch.

## 2. Recommended Git Workflow: Feature Branch + Pull Request Workflow

We will use a **Feature Branch Workflow** (also known as GitHub Flow). This is simple, effective for distributed teams, and works very well with GitHub.

### Core Principles

- `main` is always stable and production-ready.
- All new work happens on short-lived feature branches.
- Code only enters `main` through a Pull Request (PR) after review and testing.
- Continuous Integration (CI) runs automated tests on every PR.

---

## 3. Branching Strategy

### Main Branch
- Name: `main`
- Purpose: Contains only tested, reviewed, and stable code.
- Protection: Direct pushes to `main` are disabled. All changes must come through Pull Requests.

### Feature Branches
- Naming convention: `feature/<short-description>` or `feature/<ticket-id>-short-description`
  - Examples:
    - `feature/user-login`
    - `feature/JIRA-123-payment-integration`
    - `feature/dashboard-charts`
- Created from the latest `main`.
- Deleted after the feature is successfully merged.

### Optional Supporting Branches (when needed)
- `bugfix/<description>` – for fixing bugs found in production or main.
- `hotfix/<description>` – for urgent production fixes (still goes through PR).
- `chore/<description>` or `docs/<description>` – for non-feature work (documentation, configuration, etc.).

### Branch Lifecycle
1. Create feature branch from `main`.
2. Develop and commit regularly.
3. Push branch and open a Pull Request.
4. Address review feedback and pass CI checks.
5. Merge into `main`.
6. Delete the feature branch.

---

## 4. How to Handle Pull Requests (Code Reviews)

### Creating a Pull Request
1. Push your feature branch to the remote repository.
2. Open a Pull Request on GitHub targeting `main`.
3. Write a clear title and description:
   - What problem does this solve?
   - What changes were made?
   - How to test it?
   - Any screenshots or related tickets?
4. Assign at least one reviewer.
5. Link related issues if applicable.

### Code Review Process
- At least **one approval** is required before merging.
- Reviewers check for:
  - Correctness and logic
  - Code quality and readability
  - Test coverage
  - Security concerns
  - Consistency with project standards
- Use constructive comments. Prefer suggestions over vague criticism.
- Authors should respond to all comments and push additional commits if needed.

### Integration Testing Before Merge
- Every Pull Request triggers Continuous Integration (CI) pipelines (GitHub Actions, for example).
- CI should run:
  - Unit tests
  - Integration tests
  - Linting / static analysis
  - Build verification
- The PR **cannot be merged** until all required status checks pass (branch protection rules enforce this).

### Merging Options
Preferred order:
1. **Squash and merge** (recommended for most feature branches) – keeps `main` history clean.
2. **Rebase and merge** – useful when you want a linear history.
3. **Merge commit** – only when preserving the full branch history is important.

After merging:
- Delete the feature branch (both locally and on the remote).
- Pull the latest `main` on your machine.

---

## 5. Best Practices for Merging and Rebasing

### Merging
- Always merge via Pull Request (never push directly to `main`).
- Prefer **squash and merge** for feature work so that each feature becomes a single clean commit on `main`.
- Keep Pull Requests reasonably small (ideally < 400 lines of meaningful change) so reviews are thorough and fast.

### Rebasing
- Rebase your feature branch onto the latest `main` **before** opening or updating a PR when your branch is behind:
  ```bash
  git checkout feature/my-feature
  git fetch origin
  git rebase origin/main
  ```
- Resolve any conflicts during rebase, then force-push (safely) to update the PR:
  ```bash
  git push --force-with-lease
  ```
- **Never rebase a branch that other people are actively using** unless the whole team agrees.
- Do not rebase commits that have already been merged into `main`.

### Conflict Resolution
- Resolve conflicts as soon as they appear.
- Prefer rebasing early and often on long-running feature branches to keep conflicts small.
- After resolving conflicts, re-run tests locally before pushing.

---

## 6. How This Workflow Facilitates Collaboration in a Distributed Team

| Challenge in Distributed Teams              | How This Workflow Helps                                      |
|---------------------------------------------|--------------------------------------------------------------|
| Developers work in different time zones     | Asynchronous Pull Requests allow review whenever convenient  |
| Risk of breaking the main branch            | Branch protection + required reviews + CI prevent bad merges |
| Difficulty tracking who changed what        | Clear feature branches + PR history provide full audit trail |
| Parallel feature development                | Independent feature branches avoid blocking each other       |
| Code quality and knowledge sharing          | Mandatory code reviews spread knowledge across the team      |
| Server or network issues                    | Full local Git history + offline commits still work          |
| Onboarding new team members                 | Documented process + visible PRs make the system transparent |

### Additional Collaboration Benefits
- **Visibility**: Everyone can see active work by looking at open Pull Requests.
- **Accountability**: Reviewers and authors are clearly recorded.
- **Safety net**: Even if one developer’s machine fails, the remote branches and PRs preserve the work.
- **Continuous delivery readiness**: Because `main` is always stable, the team can deploy frequently with confidence.

---

## 7. Quick Reference – Daily Developer Workflow

```bash
# 1. Start from latest main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/my-new-feature

# 3. Work, commit often with clear messages
git add .
git commit -m "Add login form validation"

# 4. Keep branch up to date (recommended)
git fetch origin
git rebase origin/main

# 5. Push and open Pull Request
git push -u origin feature/my-new-feature
# → Open PR on GitHub, request review, wait for CI

# 6. After approval and green CI → Squash and merge
# 7. Clean up
git checkout main
git pull origin main
git branch -d feature/my-new-feature
```

---

## 8. Summary

This Feature Branch + Pull Request workflow gives the team:

- Safe parallel development
- Mandatory code review and testing
- A always-stable `main` branch
- Clear history and accountability
- Excellent support for distributed collaboration

By following this process consistently, the team can move fast without sacrificing quality or stability.
