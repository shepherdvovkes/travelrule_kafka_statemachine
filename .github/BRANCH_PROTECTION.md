# Branch Protection Rules

This document describes branch protection settings to ensure code quality and security in Remedy.

## Setup via GitHub UI

Go to **Settings → Branches** and configure the following rules:

### 1. Main Branch Protection

**Branch name pattern**: `main`

#### Required Settings:
- **Require a pull request before merging**
  - Require approvals: **2**
  - Dismiss stale pull request approvals when new commits are pushed
  - Require review from Code Owners
  
- **Require status checks to pass before merging**
  - Required status checks:
    - `security-scan`
    - `lint`
    - `test`
    - `build`
    - `compliance-check`
    - `CodeQL / Analyze (javascript)`
    - `CodeQL / Analyze (typescript)`
  - Require branches to be up to date before merging
  
- **Require conversation resolution before merging**
  
- **Require signed commits** (recommended)
  
- **Require linear history** (recommended)
  
- **Do not allow bypassing the above settings**
  - Even administrators cannot bypass
  
- **Restrict who can push to matching branches**
  - Only specific teams/individuals

#### Additional Settings:
- **Lock branch** (after release, temporarily)
- **Allow force pushes**: NO
- **Allow deletions**: NO

### 2. Develop Branch Protection

**Branch name pattern**: `develop`

#### Required Settings:
- **Require a pull request before merging**
  - Require approvals: **1**
  - Dismiss stale pull request approvals when new commits are pushed
  
- **Require status checks to pass before merging**
  - Required status checks:
    - `security-scan`
    - `lint`
    - `test`
    - `build`
    - `compliance-check`
  - Require branches to be up to date before merging
  
- **Require conversation resolution before merging**
  
- **Do not allow bypassing the above settings**
  - Even administrators cannot bypass

#### Additional Settings:
- **Allow force pushes**: NO
- **Allow deletions**: NO

### 3. Feature/Fix Branch Protection (optional)

**Branch name pattern**: `feature/*`, `fix/*`, `security/*`, `compliance/*`

#### Recommended Settings:
- **Require a pull request before merging**
  - Require approvals: **1**
  
- **Require status checks to pass before merging**
  - Required status checks:
    - `lint`
    - `test`
  - Require branches to be up to date before merging

## CODEOWNERS File

Create `.github/CODEOWNERS` file for automatic reviewer assignment:

```
# Global owners
* @remedy-security-team @remedy-compliance-team

# Security-critical files
/.github/workflows/ @remedy-security-team
/src/**/security/ @remedy-security-team
/src/**/auth/ @remedy-security-team

# Compliance-critical files
/src/**/travel-rule/ @remedy-compliance-team
/src/**/aml/ @remedy-compliance-team
/src/**/kyc/ @remedy-compliance-team

# Payment/Transaction files
/src/**/payment/ @remedy-senior-engineers
/src/**/transaction/ @remedy-senior-engineers

# Infrastructure
/docker/ @remedy-devops
/.github/ @remedy-devops
/aws/ @remedy-devops

# Database
/src/**/database/ @remedy-backend-team
/migrations/ @remedy-backend-team
```

## Setup via GitHub API (alternative)

If you want to configure via API or Infrastructure as Code:

```bash
# Example setup via GitHub CLI
gh api repos/:owner/:repo/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["security-scan","lint","test","build","compliance-check"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":2,"dismiss_stale_reviews":true}' \
  --field restrictions=null
```

## Terraform Configuration (optional)

If using Terraform for infrastructure management:

```hcl
resource "github_branch_protection" "main" {
  repository_id = github_repository.remedy.id
  pattern       = "main"

  required_status_checks {
    strict   = true
    contexts = [
      "security-scan",
      "lint",
      "test",
      "build",
      "compliance-check",
      "CodeQL / Analyze (javascript)",
      "CodeQL / Analyze (typescript)"
    ]
  }

  required_pull_request_reviews {
    required_approving_review_count = 2
    dismiss_stale_reviews           = true
    require_code_owner_reviews       = true
  }

  enforce_admins                  = true
  require_conversation_resolution = true
  require_signed_commits          = true
  require_linear_history          = true

  restrictions {
    users = []
    teams = ["senior-engineers", "security-team"]
  }
}
```

## Settings Verification

After setup, verify:

1. Try to create PR to `main` without passing all checks — should be blocked
2. Try to merge without approval — should be blocked
3. Try to force push to `main` — should be blocked
4. Verify that CODEOWNERS automatically assigns reviewers

## Exceptions

In emergency cases (security incidents, critical production bugs) it may be necessary to bypass branch protection. For this:

1. Create an issue describing the emergency situation
2. Get approval from Security Lead and CTO
3. Temporarily disable protection (only for the specific incident)
4. Immediately restore protection after fix
5. Conduct post-mortem review

---

**Important**: These settings are critical for ensuring security and compliance of the FinTech system. Do not disable them without extreme necessity and appropriate approval.
