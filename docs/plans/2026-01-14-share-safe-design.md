# Share-Safe Design Document

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** A slash command that helps vibe coders find and fix secrets/personal info before sharing code anywhere.

**Architecture:** Claude Code skill that scans files, explains risks in plain English, and auto-fixes by moving secrets to .env files.

**Tech Stack:** Claude Code slash command (markdown skill file)

---

## Core Problem

Vibe coders don't know what they don't know. They might accidentally share:
- API keys hardcoded in files
- Database passwords in connection strings
- Personal paths like `/Users/theirname/`
- OAuth tokens in URLs

They need a tool that finds these AND explains why they're dangerous.

---

## Risk Categories

### 🔴 CRITICAL - Immediate danger if shared
- API keys (OpenAI, Anthropic, Stripe, AWS, Google, etc.)
- Database connection strings with embedded credentials
- Private keys and certificates (SSH, SSL, PGP)
- Passwords and auth tokens in code
- Webhook URLs with secret tokens

### 🟠 HIGH - Could enable account compromise
- OAuth client secrets
- JWT signing keys
- Session secrets
- Firebase/Supabase keys with write access

### 🟡 MEDIUM - Reveals personal identity
- Personal email addresses (gmail, yahoo, etc.)
- Home directory paths (`/Users/name/`, `C:\Users\name\`)
- Full names in comments or configs
- Phone numbers
- IP addresses (non-localhost)

### 🟢 LOW - Minor privacy leak
- Localhost URLs (reveals dev setup)
- Local file paths revealing OS/username
- AI conversation artifacts (prompts, "TODO: ask Claude")
- Debug flags left enabled

### ⚪ INFO - Not a problem, just informational
- `.env` exists and is properly gitignored ✓
- Secrets loaded from environment variables correctly ✓
- `.env.example` exists ✓

---

## Command Modes

### Default: Full Repo Scan
```
/share-safe
```
Scans entire project directory. Best before pushing to GitHub or sharing a zip.

### Quick Mode (experienced users)
```
/share-safe --quick
```
Same scan, no explanations. Just findings and fix options.

### Clipboard Check
```
/share-safe clipboard
```
Scans clipboard contents. Use before pasting into Discord, Twitter, blogs, Stack Overflow.

### Single File
```
/share-safe path/to/file.py
```
Quick check on one file you're about to share.

### Pre-Deploy Mode
```
/share-safe --deploy
```
Extra paranoid checks:
- Secrets in frontend code (would leak to browser)
- Debug flags left on
- Localhost URLs that won't work in production
- Console.log/print with sensitive data

---

## Auto-Fix Flow

When an issue is found, present:

```
🔴 CRITICAL: Database connection string with password
   File: src/config.py, line 12
   Found: mongodb://admin:secretpass123@cluster.mongodb.net/mydb

   Why dangerous: Anyone who sees this can access your entire
   database - read, modify, or delete all your data.

   Fix options:
   [1] Move to .env (recommended)
   [2] Skip this one
   [3] Not actually sensitive, ignore forever
```

### If user picks [1], the tool:

1. **Creates/updates `.env`:**
   ```
   DATABASE_URL=mongodb://admin:secretpass123@cluster.mongodb.net/mydb
   ```

2. **Creates/updates `.env.example`:**
   ```
   DATABASE_URL=mongodb://username:password@your-cluster.mongodb.net/dbname
   ```

3. **Updates original file:**
   ```python
   # Before
   db_url = "mongodb://admin:secretpass123@cluster.mongodb.net/mydb"

   # After
   import os
   db_url = os.environ.get("DATABASE_URL")
   ```

4. **Adds import if missing** (`import os` for Python, etc.)

5. **Ensures `.env` is in `.gitignore`**

### Language-specific replacements:
- **Python:** `os.environ.get("VAR_NAME")`
- **JavaScript/Node:** `process.env.VAR_NAME`
- **Ruby:** `ENV['VAR_NAME']`
- **Go:** `os.Getenv("VAR_NAME")`
- **Other/unknown:** Flag for manual fix

---

## Output Format

### During Scan (live progress)
```
🔍 Scanning for secrets and personal info...
  ✓ API keys & tokens
  ✓ Database credentials
  ⚠ Found 2 issues
  ✓ Personal paths
  ⚠ Found 1 issue
  ✓ Email addresses
  ✓ Private keys
  ✓ Environment setup
```

### Summary Report
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARE-SAFE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 1 critical issue (must fix before sharing)
🟡 2 medium issues (recommend fixing)
🟢 3 items properly protected

Fixed this session: 2 secrets moved to .env
```

### Learning Tips (verbose mode only)
```
💡 TIP: You had a MongoDB connection string in your code.
   In the future, always put database URLs in .env from
   the start - never paste them directly into code files.
```

---

## Detection Patterns

### API Keys
```
# OpenAI
sk-[a-zA-Z0-9]{48}

# Anthropic
sk-ant-[a-zA-Z0-9-]{40,}

# AWS Access Key
AKIA[0-9A-Z]{16}

# Stripe
sk_live_[a-zA-Z0-9]{24,}
sk_test_[a-zA-Z0-9]{24,}

# Google
AIza[0-9A-Za-z-_]{35}

# GitHub
gh[ps]_[a-zA-Z0-9]{36}
github_pat_[a-zA-Z0-9]{22}_[a-zA-Z0-9]{59}
```

### Database Connection Strings
```
# MongoDB
mongodb(\+srv)?://[^:]+:[^@]+@

# PostgreSQL
postgres(ql)?://[^:]+:[^@]+@

# MySQL
mysql://[^:]+:[^@]+@

# Redis
redis://:[^@]+@
```

### Personal Paths
```
/Users/[a-zA-Z0-9_-]+/
/home/[a-zA-Z0-9_-]+/
C:\\Users\\[a-zA-Z0-9_-]+\\
```

### Personal Emails
```
[a-zA-Z0-9._%+-]+@(gmail|yahoo|hotmail|outlook|icloud|protonmail|aol)\.[a-zA-Z]{2,}
```

### Private Keys
```
-----BEGIN (RSA |DSA |EC |OPENSSH |PGP )?PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
```

---

## File Types to Scan

**Code files:**
- `.py`, `.js`, `.ts`, `.jsx`, `.tsx`
- `.rb`, `.go`, `.java`, `.kt`
- `.php`, `.rs`, `.swift`

**Config files:**
- `.json`, `.yaml`, `.yml`, `.toml`
- `.ini`, `.cfg`, `.conf`
- `.env*` (flag if not gitignored)

**Documentation:**
- `.md`, `.txt`, `.rst`

**Skip:**
- `.git/` directory
- `node_modules/`
- `venv/`, `.venv/`, `env/`
- `__pycache__/`
- `.next/`, `.nuxt/`, `dist/`, `build/`
- Binary files

---

## Edge Cases

1. **Secrets in comments** - Still flag them. "Even in comments, secrets can leak."

2. **Test/mock credentials** - If clearly fake (`password123`, `test@test.com`), flag as 🟢 LOW with note: "Looks like test data, but double-check."

3. **Already using env vars correctly** - Show as ⚪ INFO: "Good job! This secret is properly loaded from environment."

4. **Mixed content** - Some secrets fixed, some skipped. Track and report separately.

5. **Non-git directories** - Still scan, just skip the .gitignore checks.

---

## Success Criteria

- Finds all common secret patterns (API keys, DB strings, private keys)
- Explains risks in plain English a non-technical person understands
- Auto-fix works for Python, JavaScript, and config files
- Creates proper .env and .env.example
- Doesn't overwhelm with false positives
- Quick mode exists for experienced users
