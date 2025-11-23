# Remedy Backend

> **Remedy** — инфраструктурный финтех-продукт, создающий "SWIFT для стейблкоинов"

##  Описание

Remedy — это API-сеть для банков и криптофинкомпаний, позволяющая безболезненно подключаться друг к другу и пересылать стейблкоины с соблюдением всех регуляторных требований (Travel Rule, AML/KYC), независимо от блокчейна или токена.

##  Архитектура

- **Backend**: NestJS + TypeScript
- **Database**: PostgreSQL 15+
- **Infrastructure**: AWS
- **MPC Wallets**: Defens.com
- **Cross-chain**: Circle CCTP (планируется)

##  Быстрый старт

### Предварительные требования

- Node.js 20.x
- PostgreSQL 15+
- Git 2.30+

### Установка

```bash
# Клонирование репозитория
git clone https://github.com/remedy/remedy-backend.git
cd remedy-backend

# Установка зависимостей
npm install

# Настройка переменных окружения
cp .env.example .env
# Отредактируйте .env файл

# Настройка базы данных
createdb remedy_dev
npm run migration:run

# Настройка Git hooks
npm run prepare

# Запуск в режиме разработки
npm run start:dev
```

Подробная инструкция по настройке: [.github/SETUP.md](.github/SETUP.md)

##  Документация

- **[CODE_GUIDE.md](CODE_GUIDE.md)** - Полное руководство по стандартам кода для финтех систем
- **[.github/SETUP.md](.github/SETUP.md)** - Настройка development окружения
- **[.github/BRANCH_PROTECTION.md](.github/BRANCH_PROTECTION.md)** - Правила защиты веток
- **[.github/PULL_REQUEST_TEMPLATE.md](.github/PULL_REQUEST_TEMPLATE.md)** - Шаблон Pull Request

##  Тестирование

```bash
# Все тесты
npm run test

# Unit тесты
npm run test:unit

# Integration тесты
npm run test:integration

# E2E тесты
npm run test:e2e

# С покрытием
npm run test:cov
```

**Требование**: Минимум 80% покрытия тестами для всех модулей.

##  Безопасность

Remedy следует строгим стандартам безопасности для финтех систем:

-  SAST (Static Application Security Testing) в CI/CD
-  Dependency scanning (npm audit, Snyk, Dependabot)
-  Secret scanning (Gitleaks)
-  CodeQL analysis
-  Security reviews для всех изменений

**Сообщить о проблеме безопасности**: security@remedy.finance

##  Compliance

Remedy соответствует следующим регуляторным требованиям:

- **Travel Rule (IVMS-101)**: Обмен AML/KYC данными между компаниями
- **AML/KYC**: Проверка всех участников транзакций
- **GDPR**: Защита персональных данных
- **Audit Trail**: Полное логирование всех операций

##  Разработка

### Code Style

Проект использует:
- **ESLint** для линтинга
- **Prettier** для форматирования
- **TypeScript strict mode** для типизации

```bash
# Линтинг
npm run lint

# Форматирование
npm run format

# Проверка типов
npm run type-check
```

### Git Workflow

1. Создайте feature/fix ветку от `develop`
2. Разработайте и протестируйте изменения
3. Создайте Pull Request
4. Дождитесь прохождения CI/CD checks
5. Получите approval от ревьюеров
6. Merge в `develop` или `main`

**Важно**: Все коммиты должны следовать [Conventional Commits](https://www.conventionalcommits.org/) формату.

### Pre-commit Hooks

Автоматически проверяются перед каждым коммитом:
-  Линтинг (ESLint)
-  Форматирование (Prettier)
-  Типы (TypeScript)
-  Тесты (Unit)
-  Секреты (Gitleaks)

##  Структура проекта

```
remedy-backend/
 src/
    travel-rule/      # Travel Rule (IVMS-101) модуль
    aml/              # AML/KYC модуль
    payment/          # Payment processing
    transaction/      # Transaction management
    address-book/     # Address Book / RemiTags
    policy-engine/    # Policy Engine
    security/         # Security utilities
    common/           # Общие утилиты
 test/                 # Тесты
 migrations/           # Database migrations
 .github/              # GitHub workflows и templates
 docs/                 # Документация
```

##  CI/CD

GitHub Actions автоматически выполняет:

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
   - Coverage reporting (минимум 80%)

4. **Compliance Checks**
   - Secret detection
   - PII logging checks
   - Audit logging verification
   - Travel Rule compliance

5. **Build & Deploy**
   - Docker image build
   - Security scan (Trivy)
   - Deployment to staging/production

##  Контакты

- **Security Issues**: security@remedy.finance
- **Compliance Questions**: compliance@remedy.finance
- **Technical Questions**: tech@remedy.finance

##  Лицензия

Proprietary - Все права защищены

---

**Remedy Team** | Ноябрь 2025

