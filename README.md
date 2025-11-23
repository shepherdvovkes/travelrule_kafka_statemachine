# Remedy Backend

> **Remedy** is an infrastructure FinTech product creating "SWIFT for stablecoins"

## Description

Remedy is an API network for banks and crypto-financial companies, allowing seamless connection to each other and sending stablecoins while complying with all regulatory requirements (Travel Rule, AML/KYC), regardless of blockchain or token.

## Architecture

- **Backend**: NestJS + TypeScript
- **Database**: PostgreSQL 15+
- **Infrastructure**: AWS
- **MPC Wallets**: Defens.com
- **Cross-chain**: Circle CCTP (planned)

## Quick Start

### Prerequisites

- Node.js 20.x
- PostgreSQL 15+
- Git 2.30+

### Installation

```bash
# Clone repository
git clone https://github.com/remedy/remedy-backend.git
cd remedy-backend

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
# Edit .env file

# Setup database
createdb remedy_dev
npm run migration:run

# Setup Git hooks
npm run prepare

# Run in development mode
npm run start:dev
```

Detailed setup instructions: [.github/SETUP.md](.github/SETUP.md)

## Documentation

- **[CODE_GUIDE.md](CODE_GUIDE.md)** - Complete code standards guide for FinTech systems
- **[.github/SETUP.md](.github/SETUP.md)** - Development environment setup
- **[.github/BRANCH_PROTECTION.md](.github/BRANCH_PROTECTION.md)** - Branch protection rules
- **[.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md)** - Pull Request template

## Testing

```bash
# All tests
npm run test

# Unit tests
npm run test:unit

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# With coverage
npm run test:cov
```

**Requirement**: Minimum 80% test coverage for all modules.

## Security

Remedy follows strict security standards for FinTech systems:

- SAST (Static Application Security Testing) in CI/CD
- Dependency scanning (npm audit, Snyk, Dependabot)
- Secret scanning (Gitleaks)
- CodeQL analysis
- Security reviews for all changes

**Report security issue**: security@remedy.finance

## Compliance

Remedy complies with the following regulatory requirements:

- **Travel Rule (IVMS-101)**: Exchange of AML/KYC data between companies
- **AML/KYC**: Verification of all transaction participants
- **GDPR**: Personal data protection
- **Audit Trail**: Complete logging of all operations

## Development

### Code Style

The project uses:
- **ESLint** for linting
- **Prettier** for formatting
- **TypeScript strict mode** for typing

```bash
# Linting
npm run lint

# Formatting
npm run format

# Type checking
npm run type-check
```

### Git Workflow

1. Create feature/fix branch from `develop`
2. Develop and test changes
3. Create Pull Request
4. Wait for CI/CD checks to pass
5. Get approval from reviewers
6. Merge to `develop` or `main`

**Important**: All commits must follow [Conventional Commits](https://www.conventionalcommits.org/) format.

### Pre-commit Hooks

Automatically checked before each commit:
- Linting (ESLint)
- Formatting (Prettier)
- Types (TypeScript)
- Tests (Unit)
- Secrets (Gitleaks)

## Project Structure

```
remedy-backend/
├── src/
│   ├── travel-rule/      # Travel Rule (IVMS-101) module
│   ├── aml/              # AML/KYC module
│   ├── payment/          # Payment processing
│   ├── transaction/      # Transaction management
│   ├── address-book/     # Address Book / RemiTags
│   ├── policy-engine/    # Policy Engine
│   ├── security/         # Security utilities
│   └── common/           # Common utilities
├── test/                 # Tests
├── migrations/           # Database migrations
├── .github/              # GitHub workflows and templates
└── docs/                 # Documentation
```

## CI/CD

GitHub Actions automatically performs:

1. **Security Scanning**
   - Gitleaks (secret scanning)
   - npm audit (dependency vulnerabilities)
   - Semgrep (SAST)
   - CodeQL analysis

2. **Code Quality**
   - ESLint
   - Prettier
   - TypeScript type checking

3. **Testing**
   - Unit tests
   - Integration tests
   - Coverage reporting (minimum 80%)

4. **Compliance Checks**
   - Secret detection
   - PII logging checks
   - Audit logging verification
   - Travel Rule compliance

5. **Build & Deploy**
   - Docker image build
   - Security scan (Trivy)
   - Deployment to staging/production

## Contacts

- **Security Issues**: security@remedy.finance
- **Compliance Questions**: compliance@remedy.finance
- **Technical Questions**: tech@remedy.finance

## License

Proprietary - All rights reserved

---

**Remedy Team** | November 2025
