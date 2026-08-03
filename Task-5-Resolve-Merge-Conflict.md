# Task 5: Handle a Real-World Git Challenge – Resolving Merge Conflicts

## 1. Scenario

During development, two developers worked on separate feature branches and both modified the same parts of the codebase. After a failed merge attempt, conflicting changes were introduced. The `main` branch is now in an inconsistent state (or a developer’s local branch cannot be cleanly merged).

**Goal:**
1. Resolve the merge conflict correctly.
2. Restore code integrity.
3. Update the team’s Git workflow so this kind of problem is much less likely to happen again.

---

## 2. Demonstrating How to Resolve a Merge Conflict

Below is a complete, practical walkthrough of the resolution process.

### Step 1: Pull the Latest Code from the Remote

Always start by making sure you have the most recent state of the repository.

```bash
# Switch to the branch you want to update (usually main or your feature branch)
git checkout main

# Fetch and merge the latest changes from the remote
git pull origin main
```

If you are resolving a conflict while merging a feature branch into `main`:

```bash
git checkout main
git pull origin main
git merge feature/conflicting-feature
```

Or, if you prefer a rebase approach:

```bash
git checkout feature/conflicting-feature
git fetch origin
git rebase origin/main
```

### Step 2: Identify the Conflicting Files

Git will stop and tell you there are conflicts. You can see exactly which files are affected with:

```bash
git status
```

Example output:

```
You have unmerged paths.
  (fix conflicts and run "git commit")

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   src/components/UserProfile.js
	both modified:   src/utils/api.js
```

You can also search for conflict markers inside files:

```bash
git diff --name-only --diff-filter=U
```

or open the files and look for the special markers Git inserts:

```
<<<<<<< HEAD
// Code from the current branch (e.g. main)
=======
// Code from the branch being merged
>>>>>>> feature/conflicting-feature
```

### Step 3: Resolve the Conflicts Manually

Open each conflicting file in your editor and decide what the final correct code should be.

**Guidelines for resolving:**
- Keep the logic that is correct and complete.
- Combine useful changes from both sides when possible.
- Remove the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) completely.
- Do **not** leave any conflict markers in the code.
- Test the code after resolving (run unit tests, check the feature still works).

Example of a resolved section:

```javascript
// Final clean version after resolving
function getUserProfile(userId) {
  // Combined and corrected logic from both developers
  return api.fetch(`/users/${userId}`).then(data => ({
    ...data,
    lastUpdated: new Date().toISOString()
  }));
}
```

### Step 4: Mark the Conflicts as Resolved and Complete the Merge

After editing the files:

```bash
# Stage the resolved files
git add src/components/UserProfile.js
git add src/utils/api.js

# Or stage everything that was resolved
git add .

# Complete the merge (or rebase)
git commit          # For a merge
# or
git rebase --continue   # If you were rebasing
```

Git will open an editor with a default merge commit message. You can keep it or improve it.

### Step 5: Verify Code Integrity

Before pushing, make sure everything still works:

```bash
# Run the test suite
npm test

# Run linting / code quality checks
npm run lint

# (Optional) Build the project
npm run build
```

If all checks pass, push the resolved branch:

```bash
git push origin main
# or
git push origin feature/conflicting-feature
```

If you rebased a feature branch that was already pushed, you will need:

```bash
git push --force-with-lease
```

(`--force-with-lease` is safer than `--force` because it prevents overwriting someone else’s new work.)

---

## 3. Updating the Team’s Git Workflow to Prevent Future Conflicts

After resolving the immediate problem, we strengthen the workflow so large, painful conflicts become rare.

### 3.1 Prevention Measures Added to the Workflow

| Prevention Measure                        | How it helps                                              | How the team will apply it                              |
|-------------------------------------------|-----------------------------------------------------------|---------------------------------------------------------|
| **Smaller, more frequent Pull Requests**  | Less code overlap → fewer and smaller conflicts           | Aim for PRs that can be reviewed in < 30–60 minutes     |
| **Rebase / update feature branches daily**| Keeps your branch close to `main`                         | Developers run `git fetch` + `git rebase origin/main` at least once a day |
| **Better communication**                  | Team knows who is touching which files                    | Announce in chat/standup when starting work on shared modules |
| **Code ownership / CODEOWNERS file**      | Clear responsibility for critical files                   | Create a `CODEOWNERS` file so the right people are auto-requested for review |
| **Feature flags / modular design**        | Reduces the need for long-lived branches that touch the same files | Prefer small, independently deployable changes          |
| **Required status checks + reviews**      | Already in place (Task 3 & 4)                             | Conflicts are caught and discussed in the PR, not after a failed merge to main |
| **Avoid long-lived feature branches**     | The longer a branch lives, the higher the chance of conflicts | Merge or close branches within a few days whenever possible |

### 3.2 Updated Daily Developer Habits

1. Start the day by updating your branch:
   ```bash
   git checkout feature/my-feature
   git fetch origin
   git rebase origin/main
   ```

2. Keep commits small and focused.

3. Open a Draft Pull Request early so others can see what files you are touching.

4. Communicate in the team channel when you plan to change shared files (e.g. “I’m updating the authentication service today”).

5. Prefer **squash and merge** so the history on `main` stays clean and easy to understand.

### 3.3 Team Agreement (to be added to the Git Workflow Guide)

> **Conflict Prevention Policy**
>
> - Feature branches should be short-lived (ideally < 3–5 days).
> - Developers must rebase onto the latest `main` before opening or updating a Pull Request.
> - Pull Requests should be small and focused on one logical change.
> - Any developer who starts work on a heavily shared file must notify the team.
> - All conflicts must be resolved and tested **before** requesting final review.

---

## 4. Summary of Actions Taken

| Action                                      | Status |
|---------------------------------------------|--------|
| Demonstrated full conflict resolution process (pull → identify → resolve → test → push) | Completed |
| Showed exact Git commands for both merge and rebase approaches | Completed |
| Updated team workflow with concrete prevention measures | Completed |
| Added daily habits and a clear team policy to reduce future conflicts | Completed |

By following the resolution steps above and adopting the prevention measures, the team can recover quickly from conflicts and significantly reduce how often serious merge problems occur.
