# security-checks-skill

A comprehensive security audit skill for AI coding agents. Performs 5 progressive security checks on any codebase:

1. **Secret Leak Prevention** — Finds and moves hardcoded secrets to environment variables
2. **Personal Data Flow Audit** — Maps PII flow, cleans logs, audits third-party integrations
3. **Pre-Deploy Production Audit** — Validates env vars, removes debug code, checks security headers, rate limiting, CORS, DB config
4. **Deep Security Audit** — Auth/IDOR, payment logic, SQL injection, XSS, file uploads
5. **Attacker's Perspective Review** — ID manipulation, login bypass, privilege escalation, feature abuse, injection, internal exposure, business logic flaws

## Installation

### Via skills.sh (Vercel)

```bash
npx skills add raviycoder/security-checks-skill
```

### Via skill.sh (zero-dep)

```bash
curl -sL skil.sh | sh -s -- raviycoder/security-checks-skill
```

### Manual

Clone this repo to your agent's skill directory:

```bash
# OpenCode
git clone https://github.com/raviycoder/security-checks-skill.git .opencode/skill/security-checks

# Claude Code
git clone https://github.com/raviycoder/security-checks-skill.git .claude/skills/security-checks
```

## Usage

Once installed, the agent will automatically use this skill when you ask for:
- "Run a security audit"
- "Check for secrets"
- "Security review my code"
- "Find vulnerabilities"
- "Harden my app for production"

## Structure

```
security-checks-skill/
├── SKILL.md          # Main skill instructions (required)
├── README.md         # This file
└── .gitignore        # Git ignore rules
```

## Skill Format

This skill follows the [Agent Skills specification](https://agentskills.io) with:
- `name`: security-checks
- `description`: Comprehensive security audit covering secrets, PII, production readiness, auth/payments, and attack surface review

## Repository

GitHub: https://github.com/raviycoder/security-checks-skill

Skills.sh: https://skills.sh/raviycoder/security-checks-skill