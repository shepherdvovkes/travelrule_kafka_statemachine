# Remedy Code Guide
## Standards for FinTech Infrastructure Systems

> **Remedy** is an infrastructure FinTech product creating "SWIFT for stablecoins".  
> This guide establishes enhanced requirements for code, security, and compliance for all developers.

---

## Table of Contents

1. [General Principles](#general-principles)
2. [Security Requirements](#security-requirements)
3. [Coding Standards](#coding-standards)
4. [Compliance and Regulatory Requirements](#compliance-and-regulatory-requirements)
5. [Testing](#testing)
6. [Documentation](#documentation)
7. [Code Review](#code-review)
8. [Git Workflow](#git-workflow)
9. [Monitoring and Logging](#monitoring-and-logging)
10. [Incident Management](#incident-management)

---

## General Principles

### System Criticality
Remedy processes financial transactions between regulated companies. Every line of code must meet the standards:
- **Security First** — zero-tolerance to vulnerabilities
- **Audit Trail** — all actions must be logged and traceable
- **Compliance-first** — compliance with Travel Rule, AML/KYC, IVMS-101
- **Reliability** — 99.9%+ uptime for production systems

### Development Principles
1. **Defense in Depth** — multiple layers of protection
2. **Principle of Least Privilege** — minimum necessary access rights
3. **Fail Secure** — system must handle errors securely
4. **Secure by Default** — secure settings by default
5. **Never Trust, Always Verify** — validate all input data

---

## Security Requirements

### 1. Secret Management

#### FORBIDDEN
- Store secrets in code or in git repository
- Commit `.env` files with real keys
- Use hardcoded passwords, API keys, private keys
- Pass secrets through URL parameters or logs

#### REQUIRED
- Use AWS Secrets Manager / Parameter Store for production
- Use environment variables for local development (`.env.example` without real values)
- Rotate secrets every 90 days
- Use different secrets for dev/staging/production
- Encrypt secrets at rest and in transit (TLS 1.3+)

```typescript
// Correct
const apiKey = process.env.API_KEY;
if (!apiKey) {
  throw new Error('API_KEY environment variable is required');
}

// Incorrect
const apiKey = 'sk_live_1234567890abcdef';
```

### 2. Authentication and Authorization

#### Requirements
- All API endpoints must be protected with authentication
- Use JWT with short lifetime (15-30 minutes)
- Refresh tokens must be httpOnly, secure, sameSite
- Implement rate limiting on all public endpoints
- Use RBAC (Role-Based Access Control) for authorization
- All critical operations require MFA (Multi-Factor Authentication)

```typescript
// Correct: access rights check
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin', 'compliance_officer')
@Post('/travel-rule/submit')
async submitTravelRule(@Body() data: TravelRuleDTO) {
  // ...
}
```

### 3. Input Validation

#### Required Checks
- Validate all input data (DTO validation)
- Sanitize string data
- Check types and formats
- Limit payload size
- Protect against SQL injection, XSS, CSRF

```typescript
// Correct: using class-validator
export class TravelRuleDTO {
  @IsString()
  @IsNotEmpty()
  @Length(1, 100)
  originatorName: string;

  @IsString()
  @IsUUID()
  originatorAccountId: string;

  @IsEnum(TransactionType)
  transactionType: TransactionType;

  @IsNumber()
  @Min(0)
  @Max(1000000)
  amount: number;
}
```

### 4. Data Encryption

#### Requirements
- TLS 1.3+ for all external connections
- Encrypt PII (Personally Identifiable Information) in database
- Use AES-256 for data at rest
- Hash passwords using bcrypt (cost factor ≥ 12)
- Use cryptographically secure random number generators

### 5. Vulnerability Protection

#### Required Checks
- Regular dependency scanning (npm audit, Snyk, Dependabot)
- SAST (Static Application Security Testing) in CI/CD
- DAST (Dynamic Application Security Testing) for production
- Check for OWASP Top 10 vulnerabilities
- Regular security audits of code

---

## Coding Standards

### 1. TypeScript / NestJS

#### Code Style
- Use strict TypeScript mode (`strict: true`)
- Always specify types, avoid `any`
- Use ESLint + Prettier for formatting
- Maximum line length: 100 characters
- Use async/await instead of Promises.then()

```typescript
// Correct
async function processTransaction(
  transactionId: string
): Promise<TransactionResult> {
  const transaction = await this.transactionRepository.findById(transactionId);
  if (!transaction) {
    throw new NotFoundException(`Transaction ${transactionId} not found`);
  }
  return this.process(transaction);
}

// Incorrect
function processTransaction(transactionId: any) {
  return this.transactionRepository.findById(transactionId).then(t => {
    return this.process(t);
  });
}
```

### 2. Error Handling

#### Requirements
- Always use typed exceptions (NestJS HttpException)
- Log all errors with context
- Do not expose internal error details to client
- Use error codes for categorization

```typescript
// Correct
try {
  await this.validateTravelRule(data);
} catch (error) {
  this.logger.error('Travel Rule validation failed', {
    error: error.message,
    data: this.sanitizeForLogging(data),
    userId: request.user.id,
  });
  throw new BadRequestException('Invalid travel rule data');
}
```

### 3. Logging

#### Requirements
- Structured logging (JSON format)
- Levels: ERROR, WARN, INFO, DEBUG
- Always include correlation ID for tracing
- Do not log PII or secrets
- Use Winston or Pino for production

```typescript
// Correct
this.logger.info('Transaction initiated', {
  transactionId: transaction.id,
  amount: transaction.amount,
  currency: transaction.currency,
  correlationId: request.headers['x-correlation-id'],
  // Do NOT log: accountNumber, privateKey, etc.
});
```

### 4. Database

#### Requirements
- Always use parameterized queries (protection against SQL injection)
- Use transactions for critical operations
- Indexes on all foreign keys and frequently used fields
- Database migrations through TypeORM migrations
- Regular backups (minimum daily)

```typescript
// Correct: using transactions
async transferFunds(from: string, to: string, amount: number) {
  return await this.dataSource.transaction(async (manager) => {
    await manager.decrement(Account, { id: from }, 'balance', amount);
    await manager.increment(Account, { id: to }, 'balance', amount);
    return manager.save(Transaction, { from, to, amount });
  });
}
```

---

## Compliance and Regulatory Requirements

### 1. Travel Rule (IVMS-101)

#### Required Fields
- Originator information (name, address, date of birth, ID)
- Beneficiary information
- Transaction details (amount, currency, timestamp)
- Compliance status

#### Requirements
- All data must be encrypted in transit
- Data storage according to GDPR requirements
- Audit trail of all operations with Travel Rule data
- Data validation before sending

### 2. AML/KYC

#### Requirements
- Check all transaction participants
- Sanctions screening (OFAC, EU, UN sanctions lists)
- PEP (Politically Exposed Persons) check
- Risk scoring for each transaction
- Suspicious Activity Reporting (SAR)

### 3. Data Privacy (GDPR)

#### Requirements
- Data minimization (collect only necessary data)
- Right to erasure (right to delete data)
- Encrypt PII data
- Log access to personal data
- Data Retention Policy (store data only for necessary time)

### 4. Audit Trail

#### Required Logs
- All financial transactions
- System configuration changes
- Access to sensitive data
- User access rights changes
- All API requests and responses

```typescript
// Correct: audit logging
async createTransaction(data: CreateTransactionDTO, user: User) {
  const transaction = await this.transactionService.create(data);
  
  await this.auditLogService.log({
    action: 'TRANSACTION_CREATED',
    userId: user.id,
    entityType: 'Transaction',
    entityId: transaction.id,
    metadata: {
      amount: transaction.amount,
      currency: transaction.currency,
    },
    timestamp: new Date(),
  });
  
  return transaction;
}
```

---

## Testing

### 1. Test Types

#### Unit Tests
- Minimum 80% coverage for critical modules
- All business logic must be covered
- Mocks for external dependencies

#### Integration Tests
- Test database interactions
- Test API endpoints
- Test integrations with external services (Defens.com, Circle CCTP)

#### E2E Tests
- Critical user scenarios
- Full transaction cycle
- Travel Rule flow

#### Security Tests
- Penetration testing
- Vulnerability scanning
- Fuzzing for input data

### 2. Test Requirements

```typescript
// Correct: comprehensive test
describe('TravelRuleService', () => {
  it('should validate and encrypt travel rule data', async () => {
    const data = createMockTravelRuleData();
    const result = await service.submitTravelRule(data);
    
    expect(result).toBeDefined();
    expect(result.encrypted).toBe(true);
    expect(result.ivms101Compliant).toBe(true);
  });

  it('should reject invalid travel rule data', async () => {
    const invalidData = { /* invalid */ };
    await expect(service.submitTravelRule(invalidData))
      .rejects.toThrow(BadRequestException);
  });
});
```

### 3. Test Coverage

- Minimum 80% coverage for all modules
- 100% coverage for critical modules (payments, compliance, security)
- CI/CD must block merge if coverage decreases

---

## Documentation

### 1. Code Documentation

#### Requirements
- JSDoc comments for all public methods
- Description of parameters and return values
- Usage examples for complex functions
- Business logic description for compliance modules

```typescript
/**
 * Submits Travel Rule data according to IVMS-101 standard
 * 
 * @param data - Travel rule data containing originator and beneficiary information
 * @param userId - ID of the user submitting the data
 * @returns Encrypted travel rule record with compliance status
 * 
 * @throws BadRequestException if data validation fails
 * @throws UnauthorizedException if user lacks compliance permissions
 * 
 * @example
 * const result = await travelRuleService.submit({
 *   originator: { name: 'John Doe', accountId: '...' },
 *   beneficiary: { name: 'Jane Smith', accountId: '...' },
 *   amount: 1000,
 *   currency: 'USDC'
 * }, userId);
 */
async submit(data: TravelRuleDTO, userId: string): Promise<TravelRuleRecord> {
  // ...
}
```

### 2. API Documentation

- OpenAPI/Swagger specification for all endpoints
- Description of all possible errors
- Request and response examples
- Authentication requirements

### 3. Architecture Documentation

- System architecture diagrams
- Description of integrations with external services
- Data flow diagrams
- Security architecture

---

## Code Review

### 1. Required Checks

#### Before Creating PR
- [ ] All tests pass locally
- [ ] Linter shows no errors
- [ ] No hardcoded secrets
- [ ] Tests added for new functionality
- [ ] Documentation updated (if necessary)
- [ ] Security vulnerability check

#### Reviewer Requirements
- Minimum 2 approvals for critical changes
- Security review for changes in security/compliance modules
- Senior engineer review for changes in payment/transaction logic
- Compliance officer review for changes in Travel Rule/AML logic

### 2. PR Checklist Template

```markdown
## Description
[Description of changes]

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Security fix
- [ ] Compliance update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed

## Security
- [ ] No secrets in code
- [ ] Input validation added
- [ ] Error handling implemented
- [ ] Audit logging added (if applicable)

## Compliance
- [ ] Travel Rule compliance maintained
- [ ] AML/KYC requirements met
- [ ] GDPR requirements met
- [ ] Audit trail updated

## Documentation
- [ ] Code comments added
- [ ] API documentation updated
- [ ] README updated (if applicable)
```

---

## Git Workflow

### 1. Branch Strategy

- `main` — production-ready code, protected from direct commits
- `develop` — development branch for integration
- `feature/*` — new features
- `fix/*` — bug fixes
- `security/*` — security fixes
- `compliance/*` — compliance updates

### 2. Commit Messages

#### Format
```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types
- `feat`: new feature
- `fix`: bug fix
- `security`: security fix
- `compliance`: compliance update
- `refactor`: refactoring
- `test`: adding tests
- `docs`: documentation

#### Examples
```
feat(travel-rule): add IVMS-101 validation

Implement comprehensive validation for Travel Rule data
according to IVMS-101 standard. Includes originator and
beneficiary data validation, encryption, and audit logging.

Closes #123
```

### 3. Branch Protection Rules

- Prohibit force push to `main` and `develop`
- Require PR for all changes
- Require all CI checks to pass
- Require approval from minimum 1 reviewer
- Require security scans to pass

---

## Monitoring and Logging

### 1. Monitoring

#### Metrics
- API response times (p50, p95, p99)
- Error rates per endpoint
- Transaction success/failure rates
- Database connection pool usage
- Memory and CPU usage

#### Alerts
- Error rate > 1%
- Response time > 1s (p95)
- Failed transactions
- Security events (failed auth, suspicious activity)

### 2. Logging

#### Levels
- **ERROR**: critical errors requiring immediate attention
- **WARN**: warnings, potential issues
- **INFO**: important business events (transactions, compliance events)
- **DEBUG**: detailed information for debugging

#### Log Structure
```json
{
  "timestamp": "2025-11-15T10:30:00Z",
  "level": "INFO",
  "service": "travel-rule-service",
  "correlationId": "abc-123-def",
  "userId": "user-456",
  "action": "TRAVEL_RULE_SUBMITTED",
  "metadata": {
    "transactionId": "tx-789",
    "amount": 1000,
    "currency": "USDC"
  }
}
```

---

## Incident Management

### 1. Incident Classification

#### Critical (P0)
- System unavailable
- Data loss
- Security breach
- Compliance violation

#### High (P1)
- Performance degradation
- Partial service unavailability
- Errors in critical operations

#### Medium (P2)
- Non-critical errors
- Monitoring issues

#### Low (P3)
- Minor bugs
- Improvements

### 2. Response Process

1. **Detection** — automatic detection through monitoring
2. **Alert** — team notification (PagerDuty, Slack)
3. **Response** — assess criticality and assign responsible person
4. **Mitigation** — temporary solution to restore service
5. **Resolution** — complete problem fix
6. **Post-mortem** — incident analysis and process improvement

### 3. Security Incident Response

- Immediately isolate affected systems
- Preserve logs and artifacts for investigation
- Notify compliance team
- Document all actions
- Post-incident review

---

## Developer Checklist

### Before Starting Work
- [ ] CODE_GUIDE read and understood
- [ ] Local environment configured
- [ ] Pre-commit hooks installed

### Before Commit
- [ ] Code meets standards (ESLint, Prettier)
- [ ] All tests pass
- [ ] No hardcoded secrets
- [ ] Necessary tests added
- [ ] Documentation updated

### Before Creating PR
- [ ] PR matches template
- [ ] All CI checks pass
- [ ] Code self-reviewed
- [ ] Security checks passed

### After Merge to main
- [ ] Monitoring configured for new changes
- [ ] Documentation updated
- [ ] Team notified of breaking changes (if any)

---

## Contacts

- **Security Issues**: security@remedy.finance
- **Compliance Questions**: compliance@remedy.finance
- **Technical Questions**: tech@remedy.finance

---

**Last Updated**: November 2025  
**Version**: 1.0
