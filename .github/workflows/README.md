# GitHub Actions Workflows

Этот каталог содержит все GitHub Actions workflows для Remedy проекта.

## Workflows

### 1. `ci-cd.yml` - Основной CI/CD Pipeline

**Триггеры:**
- Push в `main` или `develop`
- Pull Request в `main` или `develop`
- Manual trigger (workflow_dispatch)

**Jobs:**
1. **security-scan** - Сканирование безопасности
   - Gitleaks (secret scanning)
   - npm audit (dependency vulnerabilities)
   - Semgrep (SAST)
   - CodeQL analysis

2. **lint** - Проверка качества кода
   - ESLint
   - Prettier
   - TypeScript type check

3. **test** - Тестирование
   - Unit tests
   - Integration tests
   - Coverage reporting (минимум 80%)

4. **build** - Сборка приложения
   - TypeScript compilation
   - Build artifacts

5. **compliance-check** - Compliance проверки
   - Secret detection
   - PII logging checks
   - Audit logging verification
   - Travel Rule compliance

6. **docker-build** - Сборка Docker образа
   - Docker image build
   - Trivy security scan

7. **deploy-staging** - Деплой в staging
   - Только для ветки `develop`
   - Требует прохождения всех checks

8. **deploy-production** - Деплой в production
   - Только для ветки `main`
   - Требует manual approval (2 approvals)
   - Требует прохождения всех checks

### 2. `dependency-review.yml` - Dependency Review

**Триггеры:**
- Pull Request в `main` или `develop`

**Функциональность:**
- Автоматический review зависимостей
- Проверка лицензий
- Блокировка небезопасных зависимостей

### 3. `codeql.yml` - CodeQL Security Analysis

**Триггеры:**
- Push в `main` или `develop`
- Pull Request в `main` или `develop`
- Еженедельно (воскресенье)

**Функциональность:**
- Статический анализ безопасности кода
- Поиск уязвимостей
- Интеграция с GitHub Security

## Требования

### Secrets

Следующие secrets должны быть настроены в GitHub:

- `GITHUB_TOKEN` - автоматически предоставляется GitHub
- `CODECOV_TOKEN` - для загрузки coverage reports
- `SLACK_WEBHOOK` - для уведомлений о деплое
- AWS credentials (для деплоя):
  - `AWS_ACCESS_KEY_ID`
  - `AWS_SECRET_ACCESS_KEY`
  - `AWS_REGION`

### Environments

Следующие environments должны быть настроены:

- `staging` - для staging деплоев
- `production` - для production деплоев (требует manual approval)

## Настройка

### 1. Настройка Secrets

Перейдите в **Settings → Secrets and variables → Actions** и добавьте необходимые secrets.

### 2. Настройка Environments

Перейдите в **Settings → Environments** и создайте:
- `staging`
- `production` (с required reviewers)

### 3. Настройка Branch Protection

См. [BRANCH_PROTECTION.md](../BRANCH_PROTECTION.md) для детальной инструкции.

## Мониторинг

### Проверка статуса

- Перейдите в **Actions** tab на GitHub
- Выберите workflow run
- Просмотрите детали каждого job

### Уведомления

- Slack уведомления отправляются при деплое
- Email уведомления для failed workflows (настраивается в GitHub)

## Troubleshooting

### Workflow не запускается

1. Проверьте, что workflow файл находится в `.github/workflows/`
2. Проверьте синтаксис YAML
3. Проверьте триггеры (on:)

### Job fails

1. Проверьте логи job
2. Проверьте, что все secrets настроены
3. Проверьте, что все зависимости установлены

### Security scan fails

1. Проверьте, нет ли реальных секретов в коде
2. Проверьте `.gitleaks.toml` для allowlist
3. Исправьте найденные проблемы

### Tests fail

1. Проверьте, что все тесты проходят локально
2. Проверьте coverage threshold (80%)
3. Проверьте, что база данных настроена правильно

## Best Practices

1. **Не коммитьте секреты** - используйте GitHub Secrets
2. **Тестируйте локально** - перед созданием PR
3. **Следуйте CODE_GUIDE.md** - для всех изменений
4. **Проверяйте coverage** - минимум 80% для всех модулей
5. **Review перед merge** - минимум 1 approval для develop, 2 для main

---

**Важно**: Все workflows критически важны для обеспечения безопасности и качества кода. Не отключайте их без крайней необходимости.

