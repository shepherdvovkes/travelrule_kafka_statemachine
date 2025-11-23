# Настройка Remedy Development Environment

Этот документ описывает процесс настройки локального окружения для разработки Remedy с учетом всех требований финтех систем.

## Предварительные требования

### Обязательные инструменты

1. **Node.js** (версия 20.x)
   ```bash
   # Использование nvm (рекомендуется)
   nvm install 20
   nvm use 20
   ```

2. **PostgreSQL** (версия 15+)
   ```bash
   # macOS
   brew install postgresql@15
   
   # Linux
   sudo apt-get install postgresql-15
   ```

3. **Git** (версия 2.30+)
   ```bash
   git --version
   ```

4. **Docker** (опционально, для локального тестирования)
   ```bash
   docker --version
   ```

### Инструменты безопасности

1. **Gitleaks** (для проверки секретов)
   ```bash
   # macOS
   brew install gitleaks
   
   # Linux
   wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks-linux-amd64
   chmod +x gitleaks-linux-amd64
   sudo mv gitleaks-linux-amd64 /usr/local/bin/gitleaks
   ```

2. **Husky** (для Git hooks)
   ```bash
   npm install -g husky
   ```

## Настройка проекта

### 1. Клонирование репозитория

```bash
git clone https://github.com/remedy/remedy-backend.git
cd remedy-backend
```

### 2. Установка зависимостей

```bash
npm install
```

### 3. Настройка переменных окружения

```bash
# Скопируйте пример файла
cp .env.example .env

# Отредактируйте .env и добавьте необходимые переменные
# НИКОГДА не коммитьте реальные секреты в .env
```

**Важно**: Используйте только тестовые значения для локальной разработки. Реальные секреты должны храниться в AWS Secrets Manager.

### 4. Настройка базы данных

```bash
# Создайте базу данных
createdb remedy_dev

# Запустите миграции
npm run migration:run
```

### 5. Настройка Git Hooks

```bash
# Установите Husky hooks
npm run prepare

# Проверьте, что hooks установлены
ls -la .husky/
```

### 6. Проверка настройки

```bash
# Проверка линтинга
npm run lint

# Проверка форматирования
npm run format:check

# Проверка типов
npm run type-check

# Запуск тестов
npm run test
```

## Настройка IDE

### VS Code

Рекомендуемые расширения:

1. **ESLint** - для линтинга
2. **Prettier** - для форматирования
3. **TypeScript and JavaScript Language Features** - для TypeScript
4. **GitLens** - для работы с Git
5. **SonarLint** - для анализа качества кода

Настройки VS Code (`.vscode/settings.json`):

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

## Рабочий процесс разработки

### 1. Создание новой ветки

```bash
# Создайте feature ветку
git checkout -b feature/travel-rule-validation

# Или fix ветку
git checkout -b fix/payment-timeout
```

### 2. Разработка

- Следуйте CODE_GUIDE.md
- Пишите тесты параллельно с кодом
- Коммитьте часто с понятными сообщениями

### 3. Перед коммитом

Pre-commit hooks автоматически проверят:
- Линтинг
- Форматирование
- Типы TypeScript
- Тесты
- Секреты в коде

Если проверки не пройдут, коммит будет заблокирован.

### 4. Создание Pull Request

1. Запушьте ветку:
   ```bash
   git push origin feature/travel-rule-validation
   ```

2. Создайте PR через GitHub UI
3. Заполните PR template
4. Дождитесь прохождения CI/CD checks
5. Получите approval от ревьюеров

### 5. После merge

- Удалите локальную ветку:
  ```bash
  git checkout main
  git pull
  git branch -d feature/travel-rule-validation
  ```

## Troubleshooting

### Проблема: Pre-commit hooks не работают

```bash
# Переустановите Husky
rm -rf .husky
npm run prepare
chmod +x .husky/pre-commit
chmod +x .husky/commit-msg
```

### Проблема: ESLint ошибки

```bash
# Автоматически исправить большинство ошибок
npm run lint
```

### Проблема: TypeScript ошибки

```bash
# Проверьте типы
npm run type-check
```

### Проблема: Тесты не проходят

```bash
# Запустите тесты с подробным выводом
npm run test -- --verbose

# Запустите конкретный тест
npm run test -- travel-rule.spec.ts
```

### Проблема: База данных не подключается

1. Проверьте, что PostgreSQL запущен:
   ```bash
   pg_isready
   ```

2. Проверьте переменные окружения в `.env`:
   ```bash
   DATABASE_URL=postgresql://user:password@localhost:5432/remedy_dev
   ```

3. Проверьте права доступа к базе данных

## Полезные команды

```bash
# Разработка
npm run start:dev          # Запуск в режиме разработки
npm run start:debug        # Запуск с отладкой

# Тестирование
npm run test               # Все тесты
npm run test:unit          # Только unit тесты
npm run test:integration   # Только integration тесты
npm run test:e2e           # E2E тесты
npm run test:cov           # С покрытием

# Качество кода
npm run lint               # Линтинг с автофиксом
npm run lint:check         # Линтинг без автофикса
npm run format             # Форматирование
npm run format:check       # Проверка форматирования
npm run type-check         # Проверка типов

# База данных
npm run migration:generate # Генерация миграции
npm run migration:run      # Применение миграций
npm run migration:revert   # Откат миграции

# Билд
npm run build              # Сборка проекта
```

## Дополнительные ресурсы

- [CODE_GUIDE.md](../CODE_GUIDE.md) - Полное руководство по коду
- [BRANCH_PROTECTION.md](./BRANCH_PROTECTION.md) - Правила защиты веток
- [PULL_REQUEST_TEMPLATE.md](./PULL_REQUEST_TEMPLATE.md) - Шаблон PR

## Поддержка

Если у вас возникли проблемы с настройкой:

1. Проверьте [CODE_GUIDE.md](../CODE_GUIDE.md)
2. Проверьте issues на GitHub
3. Обратитесь в #dev-support в Slack
4. Создайте issue с описанием проблемы

---

**Важно**: Все разработчики должны пройти security training перед началом работы с кодом.

