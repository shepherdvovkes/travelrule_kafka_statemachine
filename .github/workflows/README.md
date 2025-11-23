# GitHub Actions Workflows

This directory contains all GitHub Actions workflows for the Remedy project.

## Workflows

### 1. `ci-cd.yml` - Main CI/CD Pipeline

**Triggers:**
- Push to `main` or `develop`
- Pull Request to `main` or `develop`
- Manual trigger (workflow_dispatch)

**Jobs:**
1. **security-scan** - Security scanning
   - Gitleaks (secret scanning)
   - npm audit (dependency vulnerabilities)
   - Semgrep (SAST)
   - CodeQL analysis

2. **lint** - Code quality check
   - ESLint
   - Prettier
   - TypeScript type check

3. **test** - Testing
   - Unit tests
   - Integration tests
   - Coverage reporting (minimum 80%)

4. **build** - Application build
   - TypeScript compilation
   - Build artifacts

5. **compliance-check** - Compliance checks
   - Secret detection
   - PII logging checks
   - Audit logging verification
   - Travel Rule compliance

6. **docker-build** - Docker image build
   - Docker image build
   - Trivy security scan

7. **deploy-staging** - Deploy to staging
   - Only for `develop` branch
   - Requires all checks to pass

8. **deploy-production** - Deploy to production
   - Only for `main` branch
   - Requires manual approval (2 approvals)
   - Requires all checks to pass

### 2. `dependency-review.yml` - Dependency Review

**Triggers:**
- Pull Request to `main` or `develop`

**Functionality:**
- Automatic dependency review
- License checking
- Block unsafe dependencies

### 3. `codeql.yml` - CodeQL Security Analysis

**Triggers:**
- Push to `main` or `develop`
- Pull Request to `main` or `develop`
- Weekly (Sunday)

**Functionality:**
- Static security code analysis
- Vulnerability detection
- Integration with GitHub Security

## Requirements

### Secrets

The following secrets must be configured in GitHub:

- `GITHUB_TOKEN` - automatically provided by GitHub
- `CODECOV_TOKEN` - for uploading coverage reports
- `SLACK_WEBHOOK` - for deployment notifications
- AWS credentials (for deployment):
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_REGION`

### Environments

The following environments must be configured:

- `staging` - for staging deployments
- `production` - for production deployments (requires manual approval)

## Setup

### 1. Configure Secrets

Go to **Settings → Secrets and variables → Actions** and add necessary secrets.

### 2. Configure Environments

Go to **Settings → Environments** and create:
- `staging`
- `production` (with required reviewers)

### 3. Configure Branch Protection

See [BRANCH_PROTECTION.md](../BRANCH_PROTECTION.md) for detailed instructions.

## Monitoring

### Check Status

- Go to **Actions** tab on GitHub
- Select workflow run
- View details of each job

### Notifications

- Slack notifications sent on deployment
- Email notifications for failed workflows (configured in GitHub)

## Troubleshooting

### Workflow not starting

1. Check that workflow file is in `.github/workflows/`
2. Check YAML syntax
3. Check triggers (on:)

### Job fails

1. Check job logs
2. Check that all secrets are configured
3. Check that all dependencies are installed

### Security scan fails

1. Check that there are no real secrets in code
2. Check `.gitleaks.toml` for allowlist
3. Fix found issues

### Tests fail

1. Check that all tests pass locally
2. Check coverage threshold (80%)
3. Check that database is configured correctly

## Best Practices

1. **Do not commit secrets** - use GitHub Secrets
2. **Test locally** - before creating PR
3. **Follow CODE_GUIDE.md** - for all changes
4. **Check coverage** - minimum 80% for all modules
5. **Review before merge** - minimum 1 approval for develop, 2 for main

---

**Important**: All workflows are critical for ensuring security and code quality. Do not disable them without extreme necessity.
