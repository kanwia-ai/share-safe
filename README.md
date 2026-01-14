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
