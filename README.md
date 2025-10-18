<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg">
    <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg">
    <img alt="banner" src="https://raw.githubusercontent.com/hoopengo/hoopengo/refs/heads/master/images/banner-FCC.svg" style="max-width: 100%;">
  </picture>
</p>

## Free Claude Code

> Get $225 in free credits on AgentRouter without a bank card — enough for approximately 450 requests to Claude Sonnet 4.5.

### What is this?

This guide helps you set up [Claude Code](https://docs.claude.com/en/docs/claude-code) with AgentRouter's promotional credits, allowing you to use Anthropic's Claude AI models through the command line interface without immediate payment.

### Prerequisites

- GitHub account (at least 2 months old)
- macOS, Linux, or Windows with WSL
- Terminal access with bash, zsh, or fish shell

### Setup Steps

#### 1. Create AgentRouter Account

Register via GitHub at: https://agentrouter.org/register?aff=WrjY

**Note:** Using this referral link gives you **$225** instead of $200 as a signup bonus. Your GitHub account must be at least 2 months old to qualify.

#### 2. Generate API Token

Go to the AgentRouter console: https://agentrouter.org/console/token

Create a new token and set the quota to **Unlimited**.

#### 3. Install Claude Code

Follow the official installation instructions at: https://docs.claude.com/en/docs/claude-code/setup

#### 4. Configure Environment Variables

You can either run the standalone script or copy/paste the script below into your terminal. The setup script includes:

**Features:**

- Secure token input (input will be hidden)
- Automatic shell detection (bash, zsh, or fish) with macOS/Linux compatibility
- Idempotent configuration (prevents duplicates, offers to update existing config)
- Token validation and comprehensive error handling
- Automatic backup of your existing configuration
- Color-coded output for better readability
- Configuration verification with detailed diagnostics
- Claude CLI installation check

**To run the standalone script:**

```bash
curl -fsSL https://raw.githubusercontent.com/yourusername/free-claude-code/master/setup_claude_agentrouter.sh | bash
```

**Or paste the script below directly into your terminal:**

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

#### 5. Start Claude Code

After the script completes successfully, restart your terminal or run:

```bash
# For bash/zsh
source ~/.bashrc  # or ~/.zshrc

# For fish
source ~/.config/fish/config.fish
```

Then start Claude Code:

```bash
claude
```

You can safely ignore any warnings from the CLI and proceed to use it.

**Important:** Only use tokens you generated in your own AgentRouter account. Keep tokens secret and never share them.

### Troubleshooting

#### Script Reports Existing Configuration

The improved script detects if you've already run it before. If found:

- Choose "y" to update your token
- Choose "n" to keep your current configuration

Your old configuration will be safely removed before adding the new one.

#### Token Not Working

- Verify the token was copied correctly (no extra spaces)
- Check that environment variables are set: `echo $ANTHROPIC_API_KEY`
- Restart your terminal after running the setup script
- The script now shows token length for verification

#### Shell Detection Issues

The script automatically detects your shell. If it detects incorrectly:

- Check your SHELL environment variable: `echo $SHELL`
- On macOS, the script automatically uses .bash_profile for bash instead of .bashrc
- Manually edit the detected RC file if needed

#### Configuration Not Loading

The script now verifies configuration automatically. If you see red ✗ marks:

- Restart your terminal completely (don't just open a new tab)
- Manually source your RC file: `source ~/.zshrc` (or ~/.bashrc, etc.)
- Check the backup file created by the script to ensure nothing was corrupted

#### GitHub Account Too New

- AgentRouter requires GitHub accounts to be at least 2 months old
- Wait until your account meets the age requirement

#### Script Errors

- The script includes comprehensive error checking with color-coded output
- Green ✓ marks indicate success
- Yellow ⚠ marks indicate warnings (usually safe to proceed)
- Red ✗ marks indicate errors that need attention
- All backups are timestamped, so you can always restore previous configurations

### Pricing

| Model (Release)                | Model Ratio | Completion Ratio | Group Ratio | Prompt $ / 1M tokens | Completion $ / 1M tokens |
| ------------------------------ | ----------- | ---------------- | ----------- | -------------------- | ------------------------ |
| claude-3-5-haiku (2024-10-22)  | 0.5         | 5                | 1           | $1.000               | $5.000                   |
| claude-haiku-4-5 (2025-10-01)  | 0.5         | 0.5              | 1           | $1.000               | $0.500                   |
| claude-sonnet-4 (2025-05-14)   | 1.5         | 5                | 1           | $3.000               | $15.000                  |
| claude-sonnet-4-5 (2025-09-29) | 2           | 5                | 1           | $4.000               | $20.000                  |

**Estimated Usage:** With $225 credit, you can make approximately:

- **450 requests** to Claude Sonnet 4.5 (based on average request size)
- **1,125 requests** to Claude Sonnet 4
- **4,500 requests** to Claude Haiku 4.5

### Additional Resources

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Claude Code Setup Guide](https://docs.claude.com/en/docs/claude-code/setup)
- [AgentRouter Website](https://agentrouter.org)
- [AgentRouter Console](https://agentrouter.org/console)

### FAQ

**Q: Is this completely free?**

A: Yes, AgentRouter provides $225 in promotional credits when you sign up using the referral link. No credit card required.

**Q: Is my data secure?**

A: It's Chinese service so..

**Q: Can I use multiple accounts?**

A: Yeah, if you have another github account.

### Contributing

Found an issue or want to improve this guide? Contributions are welcome!

---

**Disclaimer:** This is an unofficial guide. AgentRouter and Anthropic are separate services. Always review terms of service and privacy policies before using any service.
