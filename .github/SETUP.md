# Remedy Development Environment Setup

This document describes the process of setting up a local environment for Remedy development, taking into account all FinTech system requirements.

## Prerequisites

### Required Tools

1. **Node.js** (version 20.x)
   ```bash
   # Using nvm (recommended)
   nvm install 20
   nvm use 20
   ```

2. **PostgreSQL** (version 15+)
   ```bash
   # macOS
   brew install postgresql@15
   
   # Linux
   sudo apt-get install postgresql-15
   ```

3. **Git** (version 2.30+)
   ```bash
   git --version
   ```

4. **Docker** (optional, for local testing)
   ```bash
   docker --version
   ```

### Security Tools

1. **Gitleaks** (for secret checking)
   ```bash
   # macOS
   brew install gitleaks
   
   # Linux
   wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks-linux-amd64
   chmod +x gitleaks-linux-amd64
   sudo mv gitleaks-linux-amd64 /usr/local/bin/gitleaks
   ```

2. **Husky** (for Git hooks)
   ```bash
   npm install -g husky
   ```

## Project Setup

### 1. Clone Repository

```bash
git clone https://github.com/remedy/remedy-backend.git
cd remedy-backend
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Configure Environment Variables

```bash
# Copy example file
cp .env.example .env

# Edit .env and add necessary variables
# NEVER commit real secrets in .env
```

**Important**: Use only test values for local development. Real secrets must be stored in AWS Secrets Manager.

### 4. Database Setup

```bash
# Create database
createdb remedy_dev

# Run migrations
npm run migration:run
```

### 5. Setup Git Hooks

```bash
# Install Husky hooks
npm run prepare

# Verify hooks are installed
ls -la .husky/
```

### 6. Verify Setup

```bash
# Linting check
npm run lint

# Formatting check
npm run format:check

# Type check
npm run type-check

# Run tests
npm run test
```

## IDE Setup

### VS Code

Recommended extensions:

1. **ESLint** - for linting
2. **Prettier** - for formatting
3. **TypeScript and JavaScript Language Features** - for TypeScript
4. **GitLens** - for Git work
5. **SonarLint** - for code quality analysis

VS Code settings (`.vscode/settings.json`):

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "files.exclude": {
    "**/.git": true,
    "**/.DS_Store": true,
    "**/node_modules": true,
    "**/dist": true,
    "**/coverage": true
  }
}
```

## Development Workflow

### 1. Create New Branch

```bash
# Create feature branch
git checkout -b feature/travel-rule-validation

# Or fix branch
git checkout -b fix/payment-timeout
```

### 2. Development

- Follow CODE_GUIDE.md
- Write tests in parallel with code
- Commit frequently with clear messages

### 3. Before Commit

Pre-commit hooks will automatically check:
- Linting
- Formatting
- TypeScript types
- Tests
- Secrets in code

If checks fail, commit will be blocked.

### 4. Create Pull Request

1. Push branch:
   ```bash
   git push origin feature/travel-rule-validation
   ```

2. Create PR via GitHub UI
3. Fill PR template
4. Wait for CI/CD checks to pass
5. Get approval from reviewers

### 5. After Merge

- Delete local branch:
  ```bash
  git checkout main
  git pull
  git branch -d feature/travel-rule-validation
  ```

## Troubleshooting

### Issue: Pre-commit hooks not working

```bash
# Reinstall Husky
rm -rf .husky
npm run prepare
chmod +x .husky/pre-commit
chmod +x .husky/commit-msg
```

### Issue: ESLint errors

```bash
# Automatically fix most errors
npm run lint
```

### Issue: TypeScript errors

```bash
# Check types
npm run type-check
```

### Issue: Tests failing

```bash
# Run tests with verbose output
npm run test -- --verbose

# Run specific test
npm run test -- travel-rule.spec.ts
```

### Issue: Database not connecting

1. Check that PostgreSQL is running:
   ```bash
   pg_isready
   ```

2. Check environment variables in `.env`:
   ```bash
   DATABASE_URL=postgresql://user:password@localhost:5432/remedy_dev
   ```

3. Check database access rights

## Useful Commands

```bash
# Development
npm run start:dev          # Run in development mode
npm run start:debug        # Run with debugging

# Testing
npm run test               # All tests
npm run test:unit          # Unit tests only
npm run test:integration  # Integration tests only
npm run test:e2e           # E2E tests
npm run test:cov           # With coverage

# Code Quality
npm run lint               # Linting with autofix
npm run lint:check         # Linting without autofix
npm run format             # Formatting
npm run format:check       # Formatting check
npm run type-check         # Type check

# Database
npm run migration:generate # Generate migration
npm run migration:run      # Apply migrations
npm run migration:revert   # Revert migration

# Build
npm run build              # Build project
```

## Additional Resources

- [CODE_GUIDE.md](../CODE_GUIDE.md) - Complete code guide
- [BRANCH_PROTECTION.md](./BRANCH_PROTECTION.md) - Branch protection rules
- [PULL_REQUEST_TEMPLATE.md](./PULL_REQUEST_TEMPLATE.md) - PR template

## Support

If you encounter setup issues:

1. Check [CODE_GUIDE.md](../CODE_GUIDE.md)
2. Check issues on GitHub
3. Contact #dev-support in Slack
4. Create an issue with problem description

---

**Important**: All developers must complete security training before starting work with code.
