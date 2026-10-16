# Credential Safety Checklist

## Environment Variables

- [ ] `.env` files listed in `.gitignore`
- [ ] `.env.example` provided with placeholder values (no real credentials)
- [ ] AI tool configuration blocks `.env` file reads
- [ ] Environment variables injected at runtime, not build time
- [ ] No credentials in `docker-compose.yml` or `Dockerfile`

## API Key Management

- [ ] API keys rotated on a regular schedule
- [ ] Each environment (dev/staging/prod) uses separate keys
- [ ] Key scopes restricted to minimum required permissions
- [ ] Unused keys revoked promptly
- [ ] Key usage monitored for anomalies

For detailed guides on managing specific API keys:
- [OpenAI API Key Management](https://agentpwn.com/tools/openai-api-key-management) -- rotation, scoping, and monitoring
- [AI Agent Environment File Security](https://agentpwn.com/tools/ai-agent-env-file-security) -- protecting .env files from agent access

## Git Safety

- [ ] Pre-commit hooks scan for secrets (e.g., `detect-secrets`, `gitleaks`)
- [ ] Git history scanned for previously committed credentials
- [ ] `.gitignore` covers all credential file patterns
- [ ] CI/CD pipeline includes secret scanning step
- [ ] Branch protection prevents force-push that could rewrite secret removal

## AI Agent Configuration

- [ ] Agent cannot read credential files directly
- [ ] Credentials passed via environment variables, not file reads
- [ ] Agent conversation history does not persist secrets
- [ ] Tool outputs filtered for credential patterns
- [ ] Agent sandboxed from credential storage locations
