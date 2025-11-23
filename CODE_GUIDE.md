# Remedy Code Guide
## Standards for FinTech Infrastructure Systems

> **Remedy** — инфраструктурный финтех-продукт, создающий "SWIFT для стейблкоинов".  
> Этот guide устанавливает повышенные требования к коду, безопасности и compliance для всех разработчиков.

---

## 📋 Содержание

1. [Общие принципы](#общие-принципы)
2. [Требования безопасности](#требования-безопасности)
3. [Стандарты кодирования](#стандарты-кодирования)
4. [Compliance и регуляторные требования](#compliance-и-регуляторные-требования)
5. [Тестирование](#тестирование)
6. [Документация](#документация)
7. [Code Review](#code-review)
8. [Git Workflow](#git-workflow)
9. [Мониторинг и логирование](#мониторинг-и-логирование)
10. [Инцидент-менеджмент](#инцидент-менеджмент)

---

## 🎯 Общие принципы

### Критичность системы
Remedy обрабатывает финансовые транзакции между регулируемыми компаниями. Каждая строка кода должна соответствовать стандартам:
- **Безопасность превыше всего** — zero-tolerance к уязвимостям
- **Аудит-трейл** — все действия должны быть логируемы и отслеживаемы
- **Compliance-first** — соответствие Travel Rule, AML/KYC, IVMS-101
- **Надежность** — 99.9%+ uptime для production систем

### Принципы разработки
1. **Defense in Depth** — множественные уровни защиты
2. **Principle of Least Privilege** — минимальные необходимые права доступа
3. **Fail Secure** — система должна безопасно обрабатывать ошибки
4. **Secure by Default** — безопасные настройки по умолчанию
5. **Never Trust, Always Verify** — валидация всех входных данных

---

## 🔒 Требования безопасности

### 1. Управление секретами

#### ❌ ЗАПРЕЩЕНО
- Хранить секреты в коде или в git-репозитории
- Коммитить `.env` файлы с реальными ключами
- Использовать hardcoded пароли, API ключи, приватные ключи
- Передавать секреты через URL параметры или логи

#### ✅ ОБЯЗАТЕЛЬНО
- Использовать AWS Secrets Manager / Parameter Store для production
- Использовать переменные окружения для локальной разработки (`.env.example` без реальных значений)
- Ротация секретов каждые 90 дней
- Использовать разные секреты для dev/staging/production
- Шифрование секретов в покое и при передаче (TLS 1.3+)

```typescript
// ✅ Правильно
const apiKey = process.env.API_KEY;
if (!apiKey) {
  throw new Error('API_KEY environment variable is required');
}

// ❌ Неправильно
const apiKey = 'sk_live_1234567890abcdef';
```

### 2. Аутентификация и авторизация

#### Требования
- Все API endpoints должны быть защищены аутентификацией
- Использовать JWT с коротким временем жизни (15-30 минут)
- Refresh tokens должны быть httpOnly, secure, sameSite
- Реализовать rate limiting на все публичные endpoints
- Использовать RBAC (Role-Based Access Control) для авторизации
- Все критические операции требуют MFA (Multi-Factor Authentication)

```typescript
// ✅ Правильно: проверка прав доступа
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('admin', 'compliance_officer')
@Post('/travel-rule/submit')
async submitTravelRule(@Body() data: TravelRuleDTO) {
  // ...
}
```

### 3. Валидация входных данных

#### Обязательные проверки
- Валидация всех входных данных (DTO validation)
- Sanitization строковых данных
- Проверка типов и форматов
- Ограничение размера payload
- Защита от SQL injection, XSS, CSRF

```typescript
// ✅ Правильно: использование class-validator
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

### 4. Шифрование данных

#### Требования
- TLS 1.3+ для всех внешних соединений
- Шифрование PII (Personally Identifiable Information) в базе данных
- Использование AES-256 для данных в покое
- Хеширование паролей с использованием bcrypt (cost factor ≥ 12)
- Использование криптографически стойких генераторов случайных чисел

### 5. Защита от уязвимостей

#### Обязательные проверки
- Регулярное сканирование зависимостей (npm audit, Snyk, Dependabot)
- SAST (Static Application Security Testing) в CI/CD
- DAST (Dynamic Application Security Testing) для production
- Проверка на OWASP Top 10 уязвимости
- Регулярные security audits кода

---

## 💻 Стандарты кодирования

### 1. TypeScript / NestJS

#### Стиль кода
- Использовать строгий режим TypeScript (`strict: true`)
- Всегда указывать типы, избегать `any`
- Использовать ESLint + Prettier для форматирования
- Максимальная длина строки: 100 символов
- Использовать async/await вместо Promises.then()

```typescript
// ✅ Правильно
async function processTransaction(
  transactionId: string
): Promise<TransactionResult> {
  const transaction = await this.transactionRepository.findById(transactionId);
  if (!transaction) {
    throw new NotFoundException(`Transaction ${transactionId} not found`);
  }
  return this.process(transaction);
}

// ❌ Неправильно
function processTransaction(transactionId: any) {
  return this.transactionRepository.findById(transactionId).then(t => {
    return this.process(t);
  });
}
```

### 2. Обработка ошибок

#### Требования
- Всегда использовать типизированные исключения (NestJS HttpException)
- Логировать все ошибки с контекстом
- Не раскрывать внутренние детали ошибок клиенту
- Использовать error codes для категоризации

```typescript
// ✅ Правильно
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

### 3. Логирование

#### Требования
- Структурированное логирование (JSON format)
- Уровни: ERROR, WARN, INFO, DEBUG
- Всегда включать correlation ID для трейсинга
- Не логировать PII или секреты
- Использовать Winston или Pino для production

```typescript
// ✅ Правильно
this.logger.info('Transaction initiated', {
  transactionId: transaction.id,
  amount: transaction.amount,
  currency: transaction.currency,
  correlationId: request.headers['x-correlation-id'],
  // НЕ логируем: accountNumber, privateKey, etc.
});
```

### 4. База данных

#### Требования
- Всегда использовать параметризованные запросы (защита от SQL injection)
- Использовать транзакции для критических операций
- Индексы на все foreign keys и часто используемые поля
- Миграции базы данных через TypeORM migrations
- Регулярные бэкапы (минимум ежедневно)

```typescript
// ✅ Правильно: использование транзакций
async transferFunds(from: string, to: string, amount: number) {
  return await this.dataSource.transaction(async (manager) => {
    await manager.decrement(Account, { id: from }, 'balance', amount);
    await manager.increment(Account, { id: to }, 'balance', amount);
    return manager.save(Transaction, { from, to, amount });
  });
}
```

---

## 📜 Compliance и регуляторные требования

### 1. Travel Rule (IVMS-101)

#### Обязательные поля
- Originator information (имя, адрес, дата рождения, ID)
- Beneficiary information
- Transaction details (amount, currency, timestamp)
- Compliance status

#### Требования
- Все данные должны быть зашифрованы при передаче
- Хранение данных согласно требованиям GDPR
- Audit trail всех операций с Travel Rule данными
- Валидация данных перед отправкой

### 2. AML/KYC

#### Требования
- Проверка всех участников транзакций
- Sanctions screening (OFAC, EU, UN sanctions lists)
- PEP (Politically Exposed Persons) проверка
- Risk scoring для каждой транзакции
- Suspicious Activity Reporting (SAR)

### 3. Data Privacy (GDPR)

#### Требования
- Минимизация сбора данных (data minimization)
- Право на удаление данных (right to erasure)
- Шифрование PII данных
- Логирование доступа к персональным данным
- Data Retention Policy (хранение данных только необходимое время)

### 4. Audit Trail

#### Обязательные логи
- Все финансовые транзакции
- Изменения в конфигурации системы
- Доступ к чувствительным данным
- Изменения прав доступа пользователей
- Все API запросы и ответы

```typescript
// ✅ Правильно: audit logging
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

## 🧪 Тестирование

### 1. Типы тестов

#### Unit Tests
- Покрытие минимум 80% для критических модулей
- Все бизнес-логика должна быть покрыта
- Моки для внешних зависимостей

#### Integration Tests
- Тестирование взаимодействия с базой данных
- Тестирование API endpoints
- Тестирование интеграций с внешними сервисами (Defens.com, Circle CCTP)

#### E2E Tests
- Критические пользовательские сценарии
- Полный цикл транзакций
- Travel Rule flow

#### Security Tests
- Penetration testing
- Vulnerability scanning
- Fuzzing для входных данных

### 2. Требования к тестам

```typescript
// ✅ Правильно: comprehensive test
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

- Минимум 80% покрытия для всех модулей
- 100% покрытие для критических модулей (payments, compliance, security)
- CI/CD должен блокировать merge при снижении покрытия

---

## 📚 Документация

### 1. Код документация

#### Требования
- JSDoc комментарии для всех публичных методов
- Описание параметров и возвращаемых значений
- Примеры использования для сложных функций
- Описание бизнес-логики для compliance модулей

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

### 2. API документация

- OpenAPI/Swagger спецификация для всех endpoints
- Описание всех возможных ошибок
- Примеры запросов и ответов
- Authentication requirements

### 3. Architecture документация

- Диаграммы архитектуры системы
- Описание интеграций с внешними сервисами
- Data flow диаграммы
- Security architecture

---

## 👥 Code Review

### 1. Обязательные проверки

#### Перед созданием PR
- [ ] Все тесты проходят локально
- [ ] Линтер не выдает ошибок
- [ ] Нет hardcoded секретов
- [ ] Добавлены тесты для новой функциональности
- [ ] Обновлена документация (если необходимо)
- [ ] Проверка на security vulnerabilities

#### Требования к ревьюерам
- Минимум 2 approval для критических изменений
- Security review для изменений в security/compliance модулях
- Senior engineer review для изменений в payment/transaction логике
- Compliance officer review для изменений в Travel Rule/AML логике

### 2. PR Checklist Template

```markdown
## Description
[Описание изменений]

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

## 🌿 Git Workflow

### 1. Branch Strategy

- `main` — production-ready код, защищен от прямых коммитов
- `develop` — development ветка для интеграции
- `feature/*` — новые фичи
- `fix/*` — багфиксы
- `security/*` — security fixes
- `compliance/*` — compliance updates

### 2. Commit Messages

#### Формат
```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Типы
- `feat`: новая функциональность
- `fix`: исправление бага
- `security`: security fix
- `compliance`: compliance update
- `refactor`: рефакторинг
- `test`: добавление тестов
- `docs`: документация

#### Примеры
```
feat(travel-rule): add IVMS-101 validation

Implement comprehensive validation for Travel Rule data
according to IVMS-101 standard. Includes originator and
beneficiary data validation, encryption, and audit logging.

Closes #123
```

### 3. Branch Protection Rules

- Запрет на force push в `main` и `develop`
- Требование PR для всех изменений
- Требование прохождения всех CI checks
- Требование approval от минимум 1 ревьюера
- Требование прохождения security scans

---

## 📊 Мониторинг и логирование

### 1. Мониторинг

#### Метрики
- API response times (p50, p95, p99)
- Error rates по endpoint
- Transaction success/failure rates
- Database connection pool usage
- Memory и CPU usage

#### Алерты
- Error rate > 1%
- Response time > 1s (p95)
- Failed transactions
- Security events (failed auth, suspicious activity)

### 2. Логирование

#### Уровни
- **ERROR**: критические ошибки, требующие немедленного внимания
- **WARN**: предупреждения, потенциальные проблемы
- **INFO**: важные бизнес-события (транзакции, compliance events)
- **DEBUG**: детальная информация для отладки

#### Структура логов
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

## 🚨 Инцидент-менеджмент

### 1. Классификация инцидентов

#### Critical (P0)
- Система недоступна
- Потеря данных
- Security breach
- Compliance violation

#### High (P1)
- Деградация производительности
- Частичная недоступность сервиса
- Ошибки в критических операциях

#### Medium (P2)
- Некритические ошибки
- Проблемы с мониторингом

#### Low (P3)
- Мелкие баги
- Улучшения

### 2. Процесс реагирования

1. **Detection** — автоматическое обнаружение через мониторинг
2. **Alert** — уведомление команды (PagerDuty, Slack)
3. **Response** — оценка критичности и назначение ответственного
4. **Mitigation** — временное решение для восстановления сервиса
5. **Resolution** — полное исправление проблемы
6. **Post-mortem** — анализ инцидента и улучшение процессов

### 3. Security Incident Response

- Немедленное изолирование затронутых систем
- Сохранение логов и артефактов для расследования
- Уведомление compliance команды
- Документирование всех действий
- Post-incident review

---

## ✅ Чеклист для разработчиков

### Перед началом работы
- [ ] Прочитан и понят CODE_GUIDE
- [ ] Настроено локальное окружение
- [ ] Установлены pre-commit hooks

### Перед коммитом
- [ ] Код соответствует стандартам (ESLint, Prettier)
- [ ] Все тесты проходят
- [ ] Нет hardcoded секретов
- [ ] Добавлены необходимые тесты
- [ ] Обновлена документация

### Перед созданием PR
- [ ] PR соответствует template
- [ ] Все CI checks проходят
- [ ] Код отревьюен самостоятельно
- [ ] Security проверки пройдены

### После merge в main
- [ ] Мониторинг настроен для новых изменений
- [ ] Документация обновлена
- [ ] Команда уведомлена о breaking changes (если есть)

---

## 📞 Контакты

- **Security Issues**: security@remedy.finance
- **Compliance Questions**: compliance@remedy.finance
- **Technical Questions**: tech@remedy.finance

---

**Последнее обновление**: Ноябрь 2025  
**Версия**: 1.0

