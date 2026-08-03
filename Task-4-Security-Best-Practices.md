# Task 4: Security Best Practices for the Source Code Repository

## 1. Scenario

With distributed teams, securing the source code repository is critical. Unauthorized access or untracked changes can lead to data leaks, malicious code injection, or loss of intellectual property. This document describes the security measures implemented (and recommended) for the `critical-thinking-project` repository.

---

## 2. Security Measures Implemented / Configured

### 2.1 User Access Controls

**Current Status**
- The repository currently has only one collaborator: the repository owner (`hayzeddev123`) with **Admin** permissions.
- No external collaborators have been added yet.

**Recommended Access Control Policy**

| Role              | Permission Level | Who should have it                          | What they can do                                      |
|-------------------|------------------|---------------------------------------------|-------------------------------------------------------|
| Admin             | Admin            | Project lead / repository owner only        | Full control (settings, collaborators, deletion)      |
| Maintainer        | Maintain         | Senior developers / tech leads              | Manage issues, PRs, some settings                     |
| Developer         | Write            | Regular team developers                     | Push to non-protected branches, open PRs              |
| Reviewer / QA     | Triage or Read   | QA engineers, external reviewers            | Comment on PRs, create issues (limited write)         |
| Viewer            | Read             | Stakeholders who only need to view code     | View code and issues only                             |

**How to configure (GitHub UI)**
1. Go to the repository → **Settings** → **Collaborators and teams**
2. Click **Add people**
3. Assign the appropriate role (Read / Triage / Write / Maintain / Admin)
4. Prefer the principle of **least privilege** — give only the minimum permissions needed.

**Best Practice**
- Never give Admin access to regular developers.
- Review collaborator list regularly and remove people who leave the project.
- For larger teams, use GitHub Teams (if in an Organization) instead of individual collaborators.

---

### 2.2 Managing SSH Keys

SSH keys are the preferred and more secure way to authenticate with GitHub (compared to HTTPS + personal access tokens for daily use).

**Recommended Setup for Every Developer**

1. Generate a strong SSH key (ed25519 is preferred):
   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. Add the public key to GitHub:
   - GitHub → **Settings** → **SSH and GPG keys** → **New SSH key**

3. Test the connection:
   ```bash
   ssh -T git@github.com
   ```

4. Use SSH URLs for all clones and remotes:
   ```bash
   git remote set-url origin git@github.com:hayzeddev123/critical-thinking-project.git
   ```

**Additional Security Practices**
- Use a passphrase on the SSH key.
- Prefer hardware security keys (YubiKey, etc.) or SSH certificates for higher security environments.
- Regularly audit and revoke unused SSH keys.
- Never share private keys.

---

### 2.3 Enforcing Branch Protection

Branch protection is one of the most important security controls. It prevents unauthorized or low-quality changes from reaching the `main` branch.

**Recommended Branch Protection Rules for `main`**

Go to: **Settings → Branches → Add rule** (or Edit rule for `main`)

Enable the following:

| Setting                                      | Recommended Value                          | Why it matters                                      |
|----------------------------------------------|--------------------------------------------|-----------------------------------------------------|
| Branch name pattern                          | `main`                                     | Protects the production-ready branch                |
| Require a pull request before merging        | Enabled                                    | No direct pushes to main                            |
| Require approvals                            | At least 1 (preferably 2 for critical code)| Code review is mandatory                            |
| Dismiss stale pull request approvals         | Enabled                                    | Forces re-review after new commits                  |
| Require status checks to pass                | Enabled                                    | CI must be green (links to Task 3 pipeline)         |
| Require branches to be up to date            | Enabled                                    | Prevents merging outdated code                      |
| Require conversation resolution              | Enabled                                    | All review comments must be addressed               |
| Require signed commits                       | Enabled (see section below)                | Ensures commit authenticity                         |
| Require linear history                       | Optional (recommended)                     | Cleaner history, no merge commits if preferred      |
| Do not allow bypassing the above settings    | Enabled                                    | Even admins cannot bypass (or limit who can)        |
| Restrict who can push                        | Only specific people/teams                 | Extra layer of control                              |
| Allow force pushes                           | Disabled                                   | Prevents history rewriting on main                  |
| Allow deletions                              | Disabled                                   | Prevents accidental deletion of main                |

**Result**: No one (including the owner, if “Do not allow bypassing” is on) can push directly to `main`. All changes must go through a reviewed and tested Pull Request.

---

### 2.4 Using Signed Commits

Signed commits prove that a commit was really made by the claimed author and that the content has not been altered.

**How to enable commit signing (GPG or SSH)**

#### Option A: GPG Signing (classic method)

1. Generate a GPG key:
   ```bash
   gpg --full-generate-key
   ```

2. Add the public key to GitHub (Settings → SSH and GPG keys → New GPG key).

3. Configure Git:
   ```bash
   git config --global user.signingkey YOUR_KEY_ID
   git config --global commit.gpgsign true
   ```

#### Option B: SSH Signing (simpler, recommended in 2024+)

1. Use an existing SSH key or create one.
2. Add it as a signing key in GitHub (Settings → SSH and GPG keys).
3. Configure Git:
   ```bash
   git config --global gpg.format ssh
   git config --global user.signingkey ~/.ssh/id_ed25519.pub
   git config --global commit.gpgsign true
   ```

**Enforcing Signed Commits**
- In Branch Protection rules, enable **“Require signed commits”**.
- Once enabled, unsigned commits will be rejected when trying to merge into `main`.

**When to require signed commits**
- Always recommended for the `main` branch.
- Especially important for critical code changes, releases, and security-sensitive repositories.

---

## 3. Auditing Changes (Change Tracking & Verification)

Git + GitHub provide strong built-in auditing:

| Audit Capability                    | How it works                                                                 | Benefit                                      |
|-------------------------------------|------------------------------------------------------------------------------|----------------------------------------------|
| Full commit history                 | Every change is recorded with author, timestamp, and message                 | Complete timeline of all modifications       |
| Pull Request history                | All discussions, reviews, approvals, and commits are preserved               | Traceable decision-making                    |
| Commit signatures                   | Cryptographic proof of who made the commit                                   | Detects impersonation or tampering           |
| GitHub Audit Log (for Organizations)| Records who added/removed collaborators, changed settings, force-pushed, etc.| Administrative action tracking               |
| Branch protection logs              | Shows when rules blocked a push or merge                                     | Visibility into security enforcement         |
| CI/CD run history                   | Every pipeline run is stored with results                                    | Proof that tests passed before merge         |

**Additional Recommendations**
- Enable GitHub’s **Security** features (Dependabot alerts, secret scanning, code scanning) if available.
- Regularly review the **Insights → Network / Contributors / Commits** pages.
- For high-security environments, consider enabling required status checks + signed commits + CODEOWNERS file.

---

## 4. How These Measures Ensure Security and Integrity

| Threat                              | Protection Mechanism                                      | Result                                              |
|-------------------------------------|-----------------------------------------------------------|-----------------------------------------------------|
| Unauthorized person pushes code     | Access controls + Branch protection                       | They cannot push to protected branches              |
| Developer accidentally pushes broken code | Required PR + Required status checks (CI)              | Broken code is blocked before reaching main         |
| Someone pretends to be another developer | Signed commits + “Require signed commits”              | Impersonation is cryptographically detectable       |
| History is rewritten (force push)   | Branch protection (force pushes disabled)                 | Main history remains immutable                      |
| Sensitive data is leaked            | Least-privilege access + regular collaborator reviews     | Fewer people have access; easier to revoke          |
| Changes cannot be traced            | Full Git history + PR history + signatures                | Every change is attributable and verifiable         |

Together, these controls create **defense in depth**:
1. Only authorized people can access the repository.
2. Even authorized people cannot push directly to the critical branch.
3. Every change is reviewed, tested, and cryptographically signed.
4. A complete, tamper-evident audit trail exists for every modification.

---

## 5. Summary of Actions Taken / Recommended

| Security Control              | Status in this repository                          |
|-------------------------------|----------------------------------------------------|
| User Access Controls          | Currently only owner has Admin access (good baseline). Documented role recommendations. |
| SSH Key Management            | Documented best practices for all team members.    |
| Branch Protection on `main`   | Fully documented with recommended settings (to be enabled in repository Settings). |
| Signed Commits                | Documented setup (GPG or SSH) + enforcement via branch protection. |
| Auditing & Change Tracking    | Native Git + GitHub history + signatures provide full auditability. |

By following the configurations described above, the repository is protected against unauthorized changes while still allowing efficient collaboration for a distributed team.
