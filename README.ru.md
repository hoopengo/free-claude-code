<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg">
    <img alt="banner" src="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg" style="max-width: 100%;">
  </picture>
</p>

## Бесплатный Claude Code

> Получите $225 бесплатных кредитов на AgentRouter без банковской карты — достаточно для примерно 450 запросов к Claude Sonnet 4.5.

### Что это такое?

Это руководство поможет вам настроить [Claude Code](https://docs.claude.com/en/docs/claude-code) с промо-кредитами AgentRouter, позволяя использовать модели Claude AI от Anthropic через интерфейс командной строки без немедленной оплаты.

### Требования

- Аккаунт GitHub (возрастом не менее 2 месяцев)
- macOS, Linux или Windows с WSL
- Доступ к терминалу с bash, zsh или fish shell

### Шаги настройки

#### 1. Создайте аккаунт AgentRouter

Зарегистрируйтесь через GitHub по ссылке: https://agentrouter.org/register?aff=WrjY

**Примечание:** Использование этой реферальной ссылки дает вам **$225** вместо $200 в качестве бонуса при регистрации. Ваш аккаунт GitHub должен быть возрастом не менее 2 месяцев, чтобы пройти квалификацию.

#### 2. Сгенерируйте API токен

Перейдите в консоль AgentRouter: https://agentrouter.org/console/token

Создайте новый токен и установите квоту на **Unlimited** (Неограниченно).

#### 3. Установите Claude Code

Следуйте официальной инструкции по установке: https://docs.claude.com/en/docs/claude-code/setup

#### 4. Настройте переменные окружения

Вы можете либо запустить отдельный скрипт, либо скопировать/вставить скрипт ниже в ваш терминал. Скрипт настройки включает:

**Возможности:**

- Безопасный ввод токена (ввод будет скрыт)
- Автоматическое определение shell (bash, zsh или fish) с совместимостью macOS/Linux
- Идемпотентная конфигурация (предотвращает дубликаты, предлагает обновить существующую конфигурацию)
- Валидация токена и всесторонняя обработка ошибок
- Автоматическое резервное копирование существующей конфигурации
- Цветной вывод для лучшей читаемости
- Проверка конфигурации с подробной диагностикой
- Проверка установки Claude CLI

**Для запуска отдельного скрипта:**

```bash
curl -fsSL https://raw.githubusercontent.com/yourusername/free-claude-code/master/setup_claude_agentrouter.sh | bash
```

**Или вставьте скрипт ниже напрямую в ваш терминал:**

```bash
#!/usr/bin/env bash
set -euo pipefail

# Color codes for better UX (fallback to empty if not supported)
if [ -t 1 ]; then
  GREEN='\033[0;32m'
  YELLOW='\033[1;33m'
  RED='\033[0;31m'
  BLUE='\033[0;34m'
  NC='\033[0m' # No Color
else
  GREEN=''
  YELLOW=''
  RED=''
  BLUE=''
  NC=''
fi

echo
echo -e "${BLUE}=== Claude / AgentRouter setup script ===${NC}"
echo

# Prompt for token (no echo)
read -rsp "Enter your AgentRouter token (will not be shown): " TOKEN
echo
if [ -z "$TOKEN" ]; then
  echo -e "${RED}Error: token is empty. Exiting.${NC}"
  exit 1
fi

# Basic token validation
TOKEN_LENGTH=${#TOKEN}
if [ "$TOKEN_LENGTH" -lt 20 ]; then
  echo -e "${YELLOW}Warning: Token seems unusually short (${TOKEN_LENGTH} characters).${NC}"
  read -rp "Continue anyway? [y/N]: " CONTINUE
  if [[ ! "$CONTINUE" =~ ^[Yy]$ ]]; then
    echo "Exiting."
    exit 1
  fi
fi

# Choose base URL (AgentRouter)
BASE_URL="https://agentrouter.org/"

# Detect shell and rc file to edit - improved for cross-platform compatibility
# Fixed: getent doesn't exist on macOS
if [ -n "${SHELL:-}" ]; then
  SHELL_NAME=$(basename "$SHELL")
else
  # Fallback if SHELL is not set
  SHELL_NAME="bash"
fi

RC_FILE=""
case "$SHELL_NAME" in
  bash)
    # Check for .bashrc first, fallback to .bash_profile on macOS
    if [ -f "${HOME}/.bashrc" ] || [ "$(uname)" != "Darwin" ]; then
      RC_FILE="${HOME}/.bashrc"
    else
      RC_FILE="${HOME}/.bash_profile"
    fi
    ;;
  zsh)
    RC_FILE="${HOME}/.zshrc"
    ;;
  fish) # fish uses a different syntax; we will write to config.fish
    RC_FILE="${HOME}/.config/fish/config.fish"
    ;;
  *)
    # fallback to .profile if unknown
    RC_FILE="${HOME}/.profile"
    ;;
esac

echo -e "${GREEN}Detected shell: $SHELL_NAME${NC}"
echo "Will add environment variables to: $RC_FILE"
echo

# Check if configuration already exists (idempotency)
ALREADY_CONFIGURED=false
if [ -f "$RC_FILE" ]; then
  if grep -q "setup_claude_agentrouter.sh" "$RC_FILE" 2>/dev/null; then
    ALREADY_CONFIGURED=true
    echo -e "${YELLOW}⚠ Found existing Claude/AgentRouter configuration in $RC_FILE${NC}"
    echo
    read -rp "Do you want to UPDATE the existing configuration? [y/N]: " UPDATE_CONFIG
    if [[ ! "$UPDATE_CONFIG" =~ ^[Yy]$ ]]; then
      echo "Keeping existing configuration. Exiting."
      exit 0
    fi
    echo -e "${BLUE}Removing old configuration...${NC}"
    # Create a temp file without the old configuration
    TEMP_FILE=$(mktemp)
    # Remove lines between the marker and the next empty line or EOF
    awk '
      /# Added by setup_claude_agentrouter.sh/ { skip=1 }
      skip && /^$/ { skip=0; next }
      !skip
    ' "$RC_FILE" > "$TEMP_FILE"
    mv "$TEMP_FILE" "$RC_FILE"
    echo -e "${GREEN}✓ Old configuration removed${NC}"
  fi
fi

# Show what will be changed
echo
echo -e "${BLUE}=== Configuration summary ===${NC}"
echo "Base URL: $BASE_URL"
echo "Token length: ${TOKEN_LENGTH} characters"
echo "Target file: $RC_FILE"
echo

# Confirmation before modifying files
read -rp "Proceed with configuration? [Y/n]: " CONFIRM
if [[ "$CONFIRM" =~ ^[Nn]$ ]]; then
  echo "Configuration cancelled. Exiting."
  exit 0
fi
echo

# Backup RC file
if [ -f "$RC_FILE" ]; then
  BACKUP_FILE="${RC_FILE}.claude_backup_$(date +%Y%m%d_%H%M%S)"
  cp "$RC_FILE" "$BACKUP_FILE"
  echo -e "${GREEN}✓ Backup created: $BACKUP_FILE${NC}"
else
  mkdir -p "$(dirname "$RC_FILE")"
  touch "$RC_FILE"
  echo -e "${GREEN}✓ Created new rc file: $RC_FILE${NC}"
fi

# Write exports (handle fish specially)
echo -e "${BLUE}Writing configuration...${NC}"
if [ "$SHELL_NAME" = "fish" ]; then
  {
    echo ""
    echo "# Added by setup_claude_agentrouter.sh — AgentRouter / Claude"
    echo "set -x ANTHROPIC_BASE_URL $BASE_URL"
    echo "set -x ANTHROPIC_AUTH_TOKEN $TOKEN"
    echo "set -x ANTHROPIC_API_KEY $TOKEN"
  } >> "$RC_FILE"
else
  {
    echo ""
    echo "# Added by setup_claude_agentrouter.sh — AgentRouter / Claude"
    echo "export ANTHROPIC_BASE_URL=\"$BASE_URL\""
    echo "export ANTHROPIC_AUTH_TOKEN=\"$TOKEN\""
    echo "export ANTHROPIC_API_KEY=\"$TOKEN\""
  } >> "$RC_FILE"
fi

echo -e "${GREEN}✓ Environment variables appended to $RC_FILE${NC}"

# Source the rc file for current session if possible
echo
echo -e "${BLUE}Loading configuration...${NC}"

if [ "$SHELL_NAME" = "fish" ]; then
  # fish: source config.fish (only works if fish is installed)
  if command -v fish >/dev/null 2>&1; then
    fish -c "source $RC_FILE" 2>/dev/null || true
    echo -e "${GREEN}✓ Sourced $RC_FILE in fish${NC}"
  else
    echo -e "${YELLOW}⚠ fish shell not available to source${NC}"
    echo "  Restart your terminal to apply changes."
  fi
else
  # Try to source RC file in current shell
  # shellcheck disable=SC1090
  if source "$RC_FILE" 2>/dev/null; then
    echo -e "${GREEN}✓ Configuration loaded in current session${NC}"
  else
    echo -e "${YELLOW}⚠ Could not source $RC_FILE automatically${NC}"
    echo "  Restart your terminal or run: source $RC_FILE"
  fi
fi

# Verify the configuration
echo
echo -e "${BLUE}=== Verification ===${NC}"

# Export variables for verification (in case source didn't work)
export ANTHROPIC_BASE_URL="$BASE_URL"
export ANTHROPIC_AUTH_TOKEN="$TOKEN"
export ANTHROPIC_API_KEY="$TOKEN"

VERIFY_SUCCESS=true

if [ "${ANTHROPIC_BASE_URL:-}" = "$BASE_URL" ]; then
  echo -e "${GREEN}✓${NC} ANTHROPIC_BASE_URL: $ANTHROPIC_BASE_URL"
else
  echo -e "${RED}✗${NC} ANTHROPIC_BASE_URL: not set correctly"
  VERIFY_SUCCESS=false
fi

if [ -n "${ANTHROPIC_AUTH_TOKEN:-}" ]; then
  echo -e "${GREEN}✓${NC} ANTHROPIC_AUTH_TOKEN: set (${#ANTHROPIC_AUTH_TOKEN} characters)"
else
  echo -e "${RED}✗${NC} ANTHROPIC_AUTH_TOKEN: not set"
  VERIFY_SUCCESS=false
fi

if [ -n "${ANTHROPIC_API_KEY:-}" ]; then
  echo -e "${GREEN}✓${NC} ANTHROPIC_API_KEY: set (${#ANTHROPIC_API_KEY} characters)"
else
  echo -e "${RED}✗${NC} ANTHROPIC_API_KEY: not set"
  VERIFY_SUCCESS=false
fi

# Check Claude CLI installation
echo
echo -e "${BLUE}=== Claude CLI check ===${NC}"

CLAUDE_CMD="claude" # change to the actual CLI name if different (e.g., 'claude-code')

if command -v "$CLAUDE_CMD" >/dev/null 2>&1; then
  echo -e "${GREEN}✓ Claude CLI found: $(command -v $CLAUDE_CMD)${NC}"
  echo
  echo "Testing Claude CLI help command..."
  if "$CLAUDE_CMD" --help >/dev/null 2>&1; then
    echo -e "${GREEN}✓ Claude CLI is working${NC}"
  else
    echo -e "${YELLOW}⚠ Claude CLI responded but may have issues${NC}"
  fi
else
  echo -e "${YELLOW}⚠ '$CLAUDE_CMD' not found in PATH${NC}"
  echo
  echo "To install Claude Code, visit:"
  echo "  ${BLUE}https://docs.claude.com/en/docs/claude-code/setup${NC}"
  echo
  echo "After installation, restart your terminal and run: $CLAUDE_CMD"
fi

# Final summary
echo
echo -e "${BLUE}=== Setup complete! ===${NC}"
echo

if [ "$VERIFY_SUCCESS" = true ]; then
  echo -e "${GREEN}✓ All checks passed${NC}"
  echo
  echo "Next steps:"
  echo "  1. Restart your terminal or run: ${BLUE}source $RC_FILE${NC}"
  echo "  2. Start Claude: ${BLUE}$CLAUDE_CMD${NC}"
  echo "  3. Begin coding with AI assistance!"
else
  echo -e "${YELLOW}⚠ Some checks failed${NC}"
  echo
  echo "Troubleshooting:"
  echo "  1. Restart your terminal"
  echo "  2. Check that the token was copied correctly"
  echo "  3. Run: echo \$ANTHROPIC_API_KEY"
  echo "  4. Re-run this script if needed"
fi

echo
echo -e "${YELLOW}Security reminder:${NC}"
echo "  • Keep your token private and never share it"
echo "  • Your token is stored in: $RC_FILE"
echo "  • To revoke access, delete the token from AgentRouter console"
echo

if [ "$ALREADY_CONFIGURED" = true ]; then
  echo -e "${GREEN}Configuration updated successfully!${NC}"
else
  echo -e "${GREEN}Configuration added successfully!${NC}"
  echo "If you used the special AgentRouter signup link, you should have promotional credit applied."
fi
```

#### 5. Запустите Claude Code

После успешного завершения скрипта перезапустите терминал или выполните:

```bash
# Для bash/zsh
source ~/.bashrc  # или ~/.zshrc

# Для fish
source ~/.config/fish/config.fish
```

Затем запустите Claude Code:

```bash
claude
```

Вы можете безопасно игнорировать любые предупреждения от CLI и продолжать его использование.

**Важно:** Используйте только токены, которые вы сгенерировали в своем собственном аккаунте AgentRouter. Храните токены в секрете и никогда не делитесь ими.

### Устранение неполадок

#### Скрипт сообщает о существующей конфигурации

Улучшенный скрипт определяет, запускали ли вы его ранее. Если найдено:

- Выберите "y" для обновления вашего токена
- Выберите "n" для сохранения текущей конфигурации

Ваша старая конфигурация будет безопасно удалена перед добавлением новой.

#### Токен не работает

- Проверьте, что токен был скопирован правильно (без лишних пробелов)
- Убедитесь, что переменные окружения установлены: `echo $ANTHROPIC_API_KEY`
- Перезапустите терминал после выполнения скрипта настройки
- Скрипт теперь показывает длину токена для проверки

#### Проблемы с определением shell

Скрипт автоматически определяет ваш shell. Если определение неверное:

- Проверьте вашу переменную окружения SHELL: `echo $SHELL`
- На macOS скрипт автоматически использует .bash_profile для bash вместо .bashrc
- При необходимости отредактируйте определенный RC файл вручную

#### Конфигурация не загружается

Скрипт теперь автоматически проверяет конфигурацию. Если вы видите красные ✗ метки:

- Полностью перезапустите терминал (не просто откройте новую вкладку)
- Вручную загрузите ваш RC файл: `source ~/.zshrc` (или ~/.bashrc и т.д.)
- Проверьте резервный файл, созданный скриптом, чтобы убедиться, что ничего не было повреждено

#### Аккаунт GitHub слишком новый

- AgentRouter требует, чтобы аккаунтам GitHub было не менее 2 месяцев
- Подождите, пока ваш аккаунт не будет соответствовать требованию по возрасту

#### Ошибки скрипта

- Скрипт включает комплексную проверку ошибок с цветным выводом
- Зеленые ✓ метки указывают на успех
- Желтые ⚠ метки указывают на предупреждения (обычно безопасно продолжать)
- Красные ✗ метки указывают на ошибки, требующие внимания
- Все резервные копии имеют временные метки, поэтому вы всегда можете восстановить предыдущие конфигурации

### Цены

| Модель (Релиз)                 | Model Ratio | Completion Ratio | Group Ratio | Запрос $ / 1М токенов | Завершение $ / 1М токенов |
| ------------------------------ | ----------- | ---------------- | ----------- | --------------------- | ------------------------- |
| claude-3-5-haiku (2024-10-22)  | 0.5         | 5                | 1           | $1.000                | $5.000                    |
| claude-haiku-4-5 (2025-10-01)  | 0.5         | 0.5              | 1           | $1.000                | $0.500                    |
| claude-sonnet-4 (2025-05-14)   | 1.5         | 5                | 1           | $3.000                | $15.000                   |
| claude-sonnet-4-5 (2025-09-29) | 2           | 5                | 1           | $4.000                | $20.000                   |

**Приблизительное использование:** С $225 кредита вы можете сделать примерно:

- **450 запросов** к Claude Sonnet 4.5 (исходя из среднего размера запроса)
- **1,125 запросов** к Claude Sonnet 4
- **4,500 запросов** к Claude Haiku 4.5

### Дополнительные ресурсы

- [Документация Claude Code](https://docs.claude.com/en/docs/claude-code)
- [Руководство по настройке Claude Code](https://docs.claude.com/en/docs/claude-code/setup)
- [Сайт AgentRouter](https://agentrouter.org)
- [Консоль AgentRouter](https://agentrouter.org/console)

### Часто задаваемые вопросы

**В: Это полностью бесплатно?**

О: Да, AgentRouter предоставляет $225 промо-кредитов при регистрации по реферальной ссылке. Кредитная карта не требуется.

**В: Безопасны ли мои данные?**

О: Это китайский сервис, так что..

**В: Могу ли я использовать несколько аккаунтов?**

О: Да, если у вас есть другой аккаунт GitHub.

### Вклад в проект

Нашли проблему или хотите улучшить это руководство? Вклад приветствуется!

---

**Отказ от ответственности:** Это неофициальное руководство. AgentRouter и Anthropic — это отдельные сервисы. Всегда просматривайте условия использования и политику конфиденциальности перед использованием любого сервиса.
