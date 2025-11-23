# Branch Protection Rules

Этот документ описывает настройки защиты веток для обеспечения качества и безопасности кода в Remedy.

## Настройка через GitHub UI

Перейдите в **Settings → Branches** и настройте следующие правила:

### 1. Main Branch Protection

**Branch name pattern**: `main`

#### Обязательные настройки:
- ✅ **Require a pull request before merging**
  - Require approvals: **2**
  - Dismiss stale pull request approvals when new commits are pushed
  - Require review from Code Owners
  
- ✅ **Require status checks to pass before merging**
  - Required status checks:
    - `security-scan`
    - `lint`
    - `test`
    - `build`
    - `compliance-check`
    - `CodeQL / Analyze (javascript)`
    - `CodeQL / Analyze (typescript)`
  - Require branches to be up to date before merging
  
- ✅ **Require conversation resolution before merging**
  
- ✅ **Require signed commits** (рекомендуется)
  
- ✅ **Require linear history** (рекомендуется)
  
- ✅ **Do not allow bypassing the above settings**
  - Even administrators cannot bypass
  
- ✅ **Restrict who can push to matching branches**
  - Only specific teams/individuals

#### Дополнительные настройки:
- ✅ **Lock branch** (после релиза, временно)
- ✅ **Allow force pushes**: ❌ НЕТ
- ✅ **Allow deletions**: ❌ НЕТ

### 2. Develop Branch Protection

**Branch name pattern**: `develop`

#### Обязательные настройки:
- ✅ **Require a pull request before merging**
  - Require approvals: **1**
  - Dismiss stale pull request approvals when new commits are pushed
  
- ✅ **Require status checks to pass before merging**
  - Required status checks:
    - `security-scan`
    - `lint`
    - `test`
    - `build`
    - `compliance-check`
  - Require branches to be up to date before merging
  
- ✅ **Require conversation resolution before merging**
  
- ✅ **Do not allow bypassing the above settings**
  - Even administrators cannot bypass

#### Дополнительные настройки:
- ✅ **Allow force pushes**: ❌ НЕТ
- ✅ **Allow deletions**: ❌ НЕТ

### 3. Feature/Fix Branch Protection (опционально)

**Branch name pattern**: `feature/*`, `fix/*`, `security/*`, `compliance/*`

#### Рекомендуемые настройки:
- ✅ **Require a pull request before merging**
  - Require approvals: **1**
  
- ✅ **Require status checks to pass before merging**
  - Required status checks:
    - `lint`
    - `test`
  - Require branches to be up to date before merging

## CODEOWNERS File

Создайте файл `.github/CODEOWNERS` для автоматического назначения ревьюеров:

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

## Настройка через GitHub API (альтернатива)

Если вы хотите настроить через API или Infrastructure as Code:

```bash
# Пример настройки через GitHub CLI
gh api repos/:owner/:repo/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["security-scan","lint","test","build","compliance-check"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":2,"dismiss_stale_reviews":true}' \
  --field restrictions=null
```

## Terraform Configuration (опционально)

Если используете Terraform для управления инфраструктурой:

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

## Проверка настроек

После настройки проверьте:

1. Попробуйте создать PR в `main` без прохождения всех checks — должен быть заблокирован
2. Попробуйте merge без approval — должен быть заблокирован
3. Попробуйте force push в `main` — должен быть заблокирован
4. Проверьте, что CODEOWNERS автоматически назначает ревьюеров

## Исключения

В экстренных случаях (security incidents, critical production bugs) может потребоваться обход защиты веток. Для этого:

1. Создайте issue с описанием экстренной ситуации
2. Получите approval от Security Lead и CTO
3. Временно отключите защиту (только для конкретного инцидента)
4. После исправления немедленно восстановите защиту
5. Проведите post-mortem review

---

**Важно**: Эти настройки критически важны для обеспечения безопасности и compliance финтех системы. Не отключайте их без крайней необходимости и соответствующего approval.

