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
curl -fsSL https://raw.githubusercontent.com/hoopengo/free-claude-code/master/setup_claude_agentrouter.sh | bash
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

### Contributing

Found an issue or want to improve this guide? Contributions are welcome!

---

**Disclaimer:** This is an unofficial guide. AgentRouter and Anthropic are separate services. Always review terms of service and privacy policies before using any service.
