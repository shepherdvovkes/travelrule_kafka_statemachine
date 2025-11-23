## Description
<!-- Опишите изменения, которые вы внесли -->

## Type of Change
<!-- Отметьте соответствующий тип изменения -->
- [ ] 🐛 Bug fix (non-breaking change which fixes an issue)
- [ ] ✨ New feature (non-breaking change which adds functionality)
- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] 🔒 Security fix
- [ ] 📜 Compliance update
- [ ] ♻️ Refactor (code change that neither fixes a bug nor adds a feature)
- [ ] 📝 Documentation update
- [ ] 🧪 Test update

## Related Issues
<!-- Ссылки на связанные issues -->
Closes #
Related to #

## Testing
<!-- Опишите, как вы тестировали изменения -->
- [ ] ✅ Unit tests added/updated
- [ ] ✅ Integration tests added/updated
- [ ] ✅ E2E tests added/updated (if applicable)
- [ ] ✅ Manual testing performed
- [ ] ✅ All existing tests pass

### Test Coverage
<!-- Укажите покрытие тестами для новых/измененных модулей -->
- Coverage: ___%

## Security Checklist
<!-- Критически важно для финтех систем -->
- [ ] ✅ No secrets or credentials in code
- [ ] ✅ Input validation implemented
- [ ] ✅ Error handling implemented
- [ ] ✅ SQL injection protection verified
- [ ] ✅ XSS protection verified
- [ ] ✅ CSRF protection verified (if applicable)
- [ ] ✅ Authentication/Authorization checks added
- [ ] ✅ Rate limiting considered (if applicable)
- [ ] ✅ Security scan passed (npm audit, Snyk, etc.)

## Compliance Checklist
<!-- Обязательно для compliance-критичных изменений -->
- [ ] ✅ Travel Rule compliance maintained (IVMS-101)
- [ ] ✅ AML/KYC requirements met
- [ ] ✅ GDPR requirements met (data privacy)
- [ ] ✅ Audit trail/logging added for critical operations
- [ ] ✅ Data encryption verified
- [ ] ✅ PII handling verified
- [ ] ✅ Data retention policy followed

## Code Quality
- [ ] ✅ Code follows project style guidelines (ESLint, Prettier)
- [ ] ✅ TypeScript types are properly defined (no `any` types)
- [ ] ✅ Code is self-documenting with clear variable/function names
- [ ] ✅ Complex logic is commented
- [ ] ✅ No console.log statements (use logger instead)
- [ ] ✅ No commented-out code

## Documentation
- [ ] ✅ Code comments added for complex logic
- [ ] ✅ JSDoc comments added for public methods
- [ ] ✅ API documentation updated (OpenAPI/Swagger)
- [ ] ✅ README updated (if applicable)
- [ ] ✅ Architecture documentation updated (if applicable)
- [ ] ✅ Migration guide added (for breaking changes)

## Performance
- [ ] ✅ Performance impact considered
- [ ] ✅ Database queries optimized
- [ ] ✅ No N+1 queries introduced
- [ ] ✅ Caching considered (if applicable)

## Dependencies
- [ ] ✅ New dependencies are necessary and justified
- [ ] ✅ Dependencies are up-to-date
- [ ] ✅ No known vulnerabilities in dependencies
- [ ] ✅ License compatibility verified

## Deployment Notes
<!-- Любые специальные инструкции для деплоя -->
- [ ] ✅ Database migrations added (if applicable)
- [ ] ✅ Environment variables documented
- [ ] ✅ Configuration changes documented
- [ ] ✅ Breaking changes documented
- [ ] ✅ Rollback plan considered

## Screenshots/Examples
<!-- Если применимо, добавьте скриншоты или примеры -->

## Additional Notes
<!-- Любая дополнительная информация для ревьюеров -->

---

## Reviewer Checklist
<!-- Для ревьюеров -->
- [ ] Code review completed
- [ ] Security review completed (for security/compliance changes)
- [ ] Compliance review completed (for compliance changes)
- [ ] Performance review completed (if applicable)
- [ ] Documentation review completed

---

**By submitting this PR, I confirm that:**
- [ ] I have read and followed the CODE_GUIDE.md
- [ ] All checks pass locally
- [ ] I have tested the changes thoroughly
- [ ] I understand the security and compliance implications of these changes

