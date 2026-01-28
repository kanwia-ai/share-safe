# Share-Safe

A Claude Code slash command that finds and fixes secrets, personal info, and sensitive data before you share code anywhere.

Built for vibe coders who don't know what they don't know.

## The Problem

You're about to push to GitHub, paste into Discord, or share a snippet on Twitter. But your code might contain:

- API keys you forgot to remove
- Database passwords in connection strings
- Your home path (`/Users/yourname/`) revealing your identity
- Personal email addresses in comments
- Webhook URLs that give others access to your Slack

**Share-safe finds these and offers to fix them automatically.**

## Installation

Copy `share-safe.md` to your Claude Code commands directory:

```bash
# Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy the command
cp share-safe.md ~/.claude/commands/
```

## Usage

In any Claude Code session:

```
/share-safe              # Full scan with explanations
/share-safe --quick      # Just findings, no explanations
/share-safe clipboard    # Check clipboard before pasting
/share-safe --deploy     # Extra paranoid mode for deployment
/share-safe --sweep      # Audit ALL your GitHub repos for leaked secrets
/share-safe --sweep --fix # Audit + auto-fix (.gitignore, LICENSE, bloat removal)
/share-safe path/to/file # Scan a single file
```

## What It Catches

### 🔴 Critical (must fix)
- API keys (OpenAI, Anthropic, Stripe, AWS, Google, GitHub)
- Database connection strings with passwords
- Private keys and certificates
- Hardcoded passwords and tokens

### 🟠 High (should fix)
- Webhook URLs with secret tokens
- OAuth client secrets
- JWT signing keys

### 🟡 Medium (personal info)
- Personal email addresses (gmail, yahoo, etc.)
- Home directory paths (`/Users/yourname/`)
- Phone numbers

### 🟢 Low (minor leaks)
- Localhost URLs
- AI conversation artifacts ("TODO: ask Claude...")
- Debug flags left enabled

## GitHub Sweep Mode

The `--sweep` flag goes beyond your current project — it audits **every repo on your GitHub account** for security issues.

**What it checks across all your repos:**

- Leaked secrets in code (API keys, tokens, passwords, private keys)
- Secret files in git history (deleted but still recoverable)
- Missing or incomplete `.gitignore` files
- Committed bloat (`node_modules/`, `__pycache__/`, `venv/`)
- Bot/scraper activity on public repos (high clones, zero views)

**With `--fix`, it auto-repairs:**

- Adds comprehensive `.gitignore` to repos missing one
- Patches incomplete `.gitignore` files with missing exclusions
- Removes committed `node_modules` and other bloat
- Adds a LICENSE to every repo (MIT for public, All Rights Reserved for private)
- Commits and pushes all fixes

**It will NOT auto-fix actual leaked secrets** — those require credential rotation. It tells you exactly what to do instead.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARE-SAFE GITHUB SWEEP REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Account: your-username
Repos scanned: 31 (22 public, 9 private)

🔴 0 critical (no leaked secrets)
🟡 10 repos missing .gitignore    → Fixed ✓
🟡 12 repos with incomplete .gitignore → Patched ✓
🟢 1 repo with committed node_modules → Removed ✓
ℹ️  3 repos with suspicious bot clone activity

✅ All repos secured!
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Auto-Fix

When share-safe finds something, it offers to fix it:

```
🔴 CRITICAL: Database connection string with password
   File: config.py, line 12
   Found: mongodb://admin:****@cluster.mongodb.net/mydb

   Why dangerous: Anyone who sees this can access your
   entire database.

   [1] Move to .env (recommended)
   [2] Skip
   [3] Not sensitive
```

If you choose [1], share-safe will:

1. Create `.env` with your secret
2. Create `.env.example` with a placeholder (so others know what vars to set)
3. Update your code to use `os.environ.get("VARIABLE")`
4. Ensure `.env` is in `.gitignore`

## Example Output

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARE-SAFE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 1 critical issue
🟡 2 medium issues
🟢 0 low issues

Fixed: 2 secrets moved to .env
Skipped: 1 item

✅ Safe to share!
```

## Why "Share-Safe"?

Because "PII checker" is jargon. You just want to know: *is this safe to share?*

---

Built with Claude Code.
