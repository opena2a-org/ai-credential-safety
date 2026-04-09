# AI Credential Safety

A practical guide to protecting API keys, tokens, and credentials from exposure through AI agents, coding assistants, and automated tools.

AI agents routinely interact with files, environment variables, and configuration that may contain secrets. Without proper isolation, credentials can leak through prompt context, tool outputs, commit history, or agent memory.

## The Problem

AI coding assistants and autonomous agents can expose credentials in several ways:

1. **Reading .env files** -- agents read environment files to understand project configuration and inadvertently include secrets in their context window
2. **Committing secrets** -- agents stage and commit files containing credentials
3. **Logging credentials** -- tool outputs containing secrets are stored in conversation history
4. **Prompt injection exfiltration** -- malicious content in repositories or web pages tricks agents into sending credentials to attacker-controlled endpoints
5. **Memory persistence** -- some agents persist conversation context that may contain secrets

## Quick Start

```bash
# Install Secretless AI for credential isolation
npm install -g secretless-ai

# Initialize in your project
npx secretless-ai init

# Verify your project is protected
npx secretless-ai verify
```

## Credential Safety Checklist

### Environment Variables

- [ ] `.env` files listed in `.gitignore`
- [ ] `.env.example` provided with placeholder values (no real credentials)
- [ ] AI tool configuration blocks `.env` file reads
- [ ] Environment variables injected at runtime, not build time
- [ ] No credentials in `docker-compose.yml` or `Dockerfile`

### API Key Management

- [ ] API keys rotated on a regular schedule
- [ ] Each environment (dev/staging/prod) uses separate keys
- [ ] Key scopes restricted to minimum required permissions
- [ ] Unused keys revoked promptly
- [ ] Key usage monitored for anomalies

For detailed guides on managing specific API keys:
- [OpenAI API Key Management](https://agentpwn.com/tools/openai-api-key-management) -- rotation, scoping, and monitoring
- [AI Agent Environment File Security](https://agentpwn.com/tools/ai-agent-env-file-security) -- protecting .env files from agent access

### Git Safety

- [ ] Pre-commit hooks scan for secrets (e.g., `detect-secrets`, `gitleaks`)
- [ ] Git history scanned for previously committed credentials
- [ ] `.gitignore` covers all credential file patterns
- [ ] CI/CD pipeline includes secret scanning step
- [ ] Branch protection prevents force-push that could rewrite secret removal

### AI Agent Configuration

- [ ] Agent cannot read credential files directly
- [ ] Credentials passed via environment variables, not file reads
- [ ] Agent conversation history does not persist secrets
- [ ] Tool outputs filtered for credential patterns
- [ ] Agent sandboxed from credential storage locations

## .env.example Templates

### Node.js / TypeScript Project

```bash
# .env.example
# Copy to .env and fill in your values
# NEVER commit .env to version control

# AI Provider Keys
OPENAI_API_KEY=sk-your-openai-key-here
ANTHROPIC_API_KEY=your-anthropic-key-here
GOOGLE_API_KEY=your-google-api-key-here

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=generate-a-random-string-here
SESSION_SECRET=generate-another-random-string-here

# External Services
STRIPE_SECRET_KEY=sk_test_your-stripe-key
SENDGRID_API_KEY=SG.your-sendgrid-key
AWS_ACCESS_KEY_ID=your-aws-key-id
AWS_SECRET_ACCESS_KEY=your-aws-secret-key

# MCP Server
MCP_AUTH_TOKEN=your-mcp-auth-token
MCP_SERVER_URL=http://localhost:3001
```

### Python Project

```bash
# .env.example
# Copy to .env and fill in your values

# AI Providers
OPENAI_API_KEY=sk-your-key-here
ANTHROPIC_API_KEY=your-key-here
HUGGINGFACE_TOKEN=hf_your-token-here

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
MONGO_URI=mongodb://localhost:27017/mydb

# Cloud
AWS_ACCESS_KEY_ID=your-key-id
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
GCP_PROJECT_ID=your-project-id
GOOGLE_APPLICATION_CREDENTIALS=./service-account.json

# Agent Configuration
AGENT_API_KEY=your-agent-api-key
LANGCHAIN_API_KEY=your-langchain-key
LANGCHAIN_TRACING_V2=true
```

## Secretless AI Setup

[Secretless AI](https://github.com/opena2a-org/secretless-ai) isolates credentials from AI agent context windows. It works with Claude Code, Cursor, GitHub Copilot, and other AI coding tools.

### Installation

```bash
npm install -g secretless-ai
```

### Initialize in Your Project

```bash
cd your-project
npx secretless-ai init
```

This creates a `.secretless.json` configuration:

```json
{
  "version": "1.0",
  "blockedFiles": [
    ".env",
    ".env.*",
    "*.key",
    "*.pem",
    "*.p12",
    "*.pfx",
    ".aws/credentials",
    ".ssh/*",
    "*.tfstate",
    "*.tfvars",
    "secrets/**",
    "credentials/**"
  ],
  "blockedPatterns": [
    "(?i)(sk-[a-zA-Z0-9]{20,})",
    "(?i)(ghp_[a-zA-Z0-9]{36})",
    "(?i)(AKIA[A-Z0-9]{16})",
    "(?i)(xox[bpras]-[a-zA-Z0-9-]+)",
    "(?i)(ya29\\.[a-zA-Z0-9_-]+)",
    "(?i)(AIza[a-zA-Z0-9_-]{35})"
  ],
  "allowedReferences": [
    "$OPENAI_API_KEY",
    "$ANTHROPIC_API_KEY",
    "$DATABASE_URL",
    "process.env.OPENAI_API_KEY",
    "os.environ.get('OPENAI_API_KEY')"
  ]
}
```

### Verify Protection

```bash
npx secretless-ai verify
```

Output:

```
Secretless AI - Credential Safety Verification

[PASS] .env is in .gitignore
[PASS] No credentials found in tracked files
[PASS] AI tool config blocks .env reads
[PASS] Pre-commit hook installed
[PASS] .env.example uses placeholder values

5/5 checks passed. Your project is protected.
```

## Detecting Credential Exposure

### Secret Scanning Patterns

Common credential patterns to scan for in your codebase and agent logs:

| Provider | Pattern | Example |
|---|---|---|
| OpenAI | `sk-[a-zA-Z0-9]{20,}` | `sk-abc123...` |
| Anthropic | `sk-ant-[a-zA-Z0-9]{20,}` | `sk-ant-abc123...` |
| GitHub | `ghp_[a-zA-Z0-9]{36}` | `ghp_abc123...` |
| AWS | `AKIA[A-Z0-9]{16}` | `AKIAIOSFODNN7...` |
| Slack | `xox[bpras]-[a-zA-Z0-9-]+` | `xoxb-123-456-abc` |
| Google | `AIza[a-zA-Z0-9_-]{35}` | `AIzaSyAbc123...` |
| Stripe | `sk_(live\|test)_[a-zA-Z0-9]{24,}` | `sk_live_abc123...` |

### Pre-commit Hook

Install a pre-commit hook to catch secrets before they're committed:

```bash
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']

  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

## Testing Credential Safety

Validate that your credential protection works by testing against real attack patterns:

| Test | Description | Resource |
|---|---|---|
| .env exfiltration | Attempts to read .env through various tool paths | [agentpwn.com/tools/ai-agent-env-file-security](https://agentpwn.com/tools/ai-agent-env-file-security) |
| API key extraction | Prompt injection to extract keys from context | [agentpwn.com/attacks/credential-theft](https://agentpwn.com/attacks/credential-theft) |
| Key management audit | Verify key rotation and scoping | [agentpwn.com/tools/openai-api-key-management](https://agentpwn.com/tools/openai-api-key-management) |
| Agent behavior testing | Test how agents handle credential files | [agentpwn.com/tools](https://agentpwn.com/tools) |

### Automated Testing with HackMyAgent

```bash
# Test your project for credential exposure risks
npx hackmyagent scan --checks credentials,env-files,git-secrets
```

Learn more about AI agent security fundamentals at [agentpwn.com/learn](https://agentpwn.com/learn).

## Related Projects

- [Secretless AI](https://github.com/opena2a-org/secretless-ai) -- credential isolation for AI development
- [HackMyAgent](https://github.com/opena2a-org/hackmyagent) -- automated agent security scanner
- [MCP Security Checklist](https://github.com/opena2a-org/mcp-security-checklist) -- MCP deployment security
- [Agent Hardening Guide](https://github.com/opena2a-org/agent-hardening-guide) -- production agent hardening

## License

MIT
