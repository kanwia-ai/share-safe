# Share-Safe: Find & Fix Secrets Before Sharing

Scan your code for secrets, personal info, and sensitive data before sharing anywhere - GitHub, Discord, Twitter, blogs, or deploying live.

## Arguments

- No arguments: Full scan with explanations (default)
- `--quick`: Just the findings, no explanations
- `clipboard`: Scan clipboard contents before pasting somewhere
- `--deploy`: Extra paranoid mode for pre-deployment
- A file path: Scan single file

## How to Run This

### Step 1: Determine the mode

Check if arguments were provided:
- `--quick` → Quick mode (findings only, no explanations)
- `clipboard` → Read clipboard and scan that content
- `--deploy` → Add extra deployment checks
- File path → Scan only that file
- Nothing → Full verbose scan of current directory

### Step 2: Run the scans

Run ALL of these scans on the target (current directory, clipboard, or specified file).

**🔴 CRITICAL: API Keys & Tokens**
```bash
grep -r -E "(sk-[a-zA-Z0-9]{20,}|sk-ant-[a-zA-Z0-9-]{20,}|AKIA[0-9A-Z]{16}|sk_live_[a-zA-Z0-9]{20,}|sk_test_[a-zA-Z0-9]{20,}|AIza[0-9A-Za-z-_]{35}|gh[ps]_[a-zA-Z0-9]{36}|github_pat_)" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.md" --include="*.env*" . 2>/dev/null | grep -v ".git" | grep -v "node_modules" | grep -v ".env.example"
```

**🔴 CRITICAL: Database Connection Strings with Passwords**
```bash
grep -r -E "(mongodb(\+srv)?|postgres(ql)?|mysql|redis)://[^:]+:[^@]+@" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.env*" . 2>/dev/null | grep -v ".git" | grep -v "node_modules" | grep -v ".env.example"
```

**🔴 CRITICAL: Private Keys**
```bash
grep -r -l "PRIVATE KEY-----" . 2>/dev/null | grep -v ".git" | grep -v "node_modules"
```

**🔴 CRITICAL: Hardcoded Passwords/Secrets**
```bash
grep -r -E "(password|passwd|secret|api_key|apikey|auth_token|access_token)\s*[=:]\s*['\"][^'\"]{8,}['\"]" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" . 2>/dev/null | grep -v ".git" | grep -v "node_modules" | grep -v "os.environ" | grep -v "process.env" | grep -v "getenv" | grep -v ".env.example" | grep -v "example" | grep -v "placeholder"
```

**🟠 HIGH: Webhook URLs with Tokens**
```bash
grep -r -E "https://[^/]+\.(slack|discord|webhook)\.com/[^\s\"']+" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" . 2>/dev/null | grep -v ".git" | grep -v "node_modules"
```

**🟡 MEDIUM: Personal Email Addresses**
```bash
grep -r -E "[a-zA-Z0-9._%+-]+@(gmail|yahoo|hotmail|outlook|icloud|protonmail|aol)\.(com|net|org)" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.md" . 2>/dev/null | grep -v ".git" | grep -v "node_modules" | grep -v "example"
```

**🟡 MEDIUM: Personal Home Directory Paths**
```bash
grep -r -E "(/Users/[a-zA-Z0-9_-]+/|/home/[a-zA-Z0-9_-]+/|C:\\\\Users\\\\[a-zA-Z0-9_-]+\\\\)" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.md" . 2>/dev/null | grep -v ".git" | grep -v "node_modules"
```

**🟡 MEDIUM: Phone Numbers**
```bash
grep -r -E "\+?[0-9]{1,3}[-.\s]?\(?[0-9]{3}\)?[-.\s]?[0-9]{3}[-.\s]?[0-9]{4}" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.md" . 2>/dev/null | grep -v ".git" | grep -v "node_modules" | grep -v "test" | grep -v "example"
```

**🟢 LOW: AI Conversation Artifacts**
```bash
grep -r -i -E "(TODO:?\s*(ask )?(claude|chatgpt|gpt|ai)|claude said|gpt said)" --include="*.py" --include="*.js" --include="*.ts" --include="*.md" . 2>/dev/null | grep -v ".git" | grep -v "node_modules"
```

**⚪ INFO: Check .env Setup**
```bash
# Check if .env exists
if [ -f ".env" ]; then
    # Check if .env is in .gitignore
    if grep -q "^\.env" .gitignore 2>/dev/null; then
        echo "✓ .env exists and is gitignored"
    else
        echo "⚠️ .env exists but NOT in .gitignore!"
    fi
fi

# Check if .env.example exists
if [ -f ".env.example" ]; then
    echo "✓ .env.example exists"
else
    echo "ℹ️ No .env.example (consider creating one for others)"
fi
```

### Step 3: Present findings

For EACH finding, present it in this format:

**Verbose mode (default):**
```
🔴 CRITICAL: [What was found]
   File: [filename], line [number]
   Found: [the actual content, partially redacted if very sensitive]

   Why dangerous: [Plain English explanation of what could happen]

   Fix options:
   [1] Move to .env (recommended)
   [2] Skip this one
   [3] Not actually sensitive, ignore
```

**Quick mode (--quick):**
```
🔴 [filename]:[line] - [brief description]
```

### Step 4: Auto-fix when requested

When user chooses [1] Move to .env:

1. **Generate a good variable name** from the context (e.g., `DATABASE_URL`, `OPENAI_API_KEY`)

2. **Add to .env file** (create if doesn't exist):
   ```
   VARIABLE_NAME=actual_secret_value
   ```

3. **Add to .env.example** (create if doesn't exist):
   ```
   VARIABLE_NAME=your_value_here
   ```

4. **Update the source file** based on language:
   - Python: `os.environ.get("VARIABLE_NAME")` (add `import os` if needed)
   - JavaScript/TypeScript: `process.env.VARIABLE_NAME`
   - JSON/YAML: Flag for manual fix (can't use env vars directly)

5. **Update .gitignore** if .env not already listed:
   ```
   # Secrets
   .env
   .env.local
   .env*.local
   ```

### Step 5: Final report

After all findings processed:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARE-SAFE REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🔴 [X] critical issues
🟠 [X] high issues
🟡 [X] medium issues
🟢 [X] low issues

Fixed: [X] secrets moved to .env
Skipped: [X] items
Ignored: [X] items marked as not sensitive

[If all critical/high fixed:]
✅ Safe to share!

[If critical/high remain:]
⚠️ Fix critical issues before sharing publicly
```

**Verbose mode only - add a tip:**
```
💡 TIP: [Contextual advice based on what was found]
```

Example tips:
- "Always create .env files at project start, never paste secrets directly into code"
- "Use .env.example to document what environment variables your project needs"
- "Personal paths often sneak into error messages and logs - watch for those too"

### Deploy mode extra checks (--deploy)

If `--deploy` flag, also check:

**Frontend secret leaks:**
```bash
grep -r -E "(NEXT_PUBLIC_|VITE_|REACT_APP_).*(SECRET|KEY|PASSWORD|TOKEN)" --include="*.js" --include="*.ts" --include="*.env*" . 2>/dev/null | grep -v "node_modules"
```

**Debug mode left on:**
```bash
grep -r -E "(DEBUG\s*=\s*True|DEBUG\s*=\s*true|\"debug\":\s*true)" --include="*.py" --include="*.js" --include="*.json" . 2>/dev/null | grep -v "node_modules" | grep -v ".git"
```

**Localhost URLs that won't work in production:**
```bash
grep -r -E "https?://(localhost|127\.0\.0\.1)" --include="*.py" --include="*.js" --include="*.ts" --include="*.json" . 2>/dev/null | grep -v "node_modules" | grep -v ".git" | grep -v "test" | grep -v ".env"
```

## Plain English Explanations Reference

Use these when explaining findings in verbose mode:

**API Keys:** "This is like a password to a paid service. Anyone with this key can use the service and you'll get the bill - or worse, they could access your data."

**Database connection string:** "This contains the username and password to your database. Anyone who sees this can read, modify, or delete all your data."

**Private key:** "This is like a master key to your server or encryption. If someone gets this, they can impersonate you or decrypt your secret messages."

**Personal email:** "This reveals your personal identity. Combined with other info, it could be used for targeted phishing or just unwanted contact."

**Home directory path:** "This reveals your computer username. It's not critical alone, but combined with other info helps identify you personally."

**Webhook URL:** "This URL can trigger actions in your Slack/Discord/etc. Anyone with it could spam your channels or trigger automations."

**AI artifacts:** "References to AI conversations might reveal your development process or contain sensitive context from your prompts."
