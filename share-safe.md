# Share-Safe: Find & Fix Secrets Before Sharing

Scan your code for secrets, personal info, and sensitive data before sharing anywhere - GitHub, Discord, Twitter, blogs, or deploying live.

## Arguments

- No arguments: Full scan with explanations (default)
- `--quick`: Just the findings, no explanations
- `clipboard`: Scan clipboard contents before pasting somewhere
- `--deploy`: Extra paranoid mode for pre-deployment
- `--sweep`: Full GitHub account sweep — scan ALL repos for leaked secrets, missing .gitignore, and git history exposure
- `--sweep --fix`: Sweep + auto-fix (add .gitignore, patch gaps, add LICENSE)
- A file path: Scan single file

## How to Run This

### Step 1: Determine the mode

Check if arguments were provided:
- `--quick` → Quick mode (findings only, no explanations)
- `clipboard` → Read clipboard and scan that content
- `--deploy` → Add extra deployment checks
- `--sweep` → Full GitHub account audit (scan ALL repos). If `--fix` also present, auto-fix issues.
- File path → Scan only that file
- Nothing → Full verbose scan of current directory

If `--sweep` is provided, skip Steps 2-5 and jump directly to the "Sweep mode" section below.

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

### Sweep mode: Full GitHub account audit (--sweep)

If `--sweep` flag, run a full security audit across ALL of the user's GitHub repos. This checks every repo (public and private) for leaked secrets, missing protections, and git history exposure.

#### Sweep Step 1: Get GitHub account info

```bash
# Get username
GITHUB_USER=$(gh api user --jq '.login')
echo "Sweeping all repos for: $GITHUB_USER"

# Get all repos with metadata
gh repo list "$GITHUB_USER" --limit 100 --json name,isPrivate,createdAt,primaryLanguage,pushedAt --jq '.[] | "\(.name)|\(.isPrivate)|\(.createdAt)|\(.primaryLanguage.name // "none")|\(.pushedAt)"'
```

#### Sweep Step 2: GitHub Code Search for secrets (public repos only)

Run these searches via the GitHub API. Each returns files containing potential secrets. Sleep 2 seconds between each to avoid rate limiting.

```bash
# Search patterns - run each one separately
PATTERNS=(
  "filename:.env"
  "filename:credentials"
  "filename:secret"
  "filename:.pem"
  "filename:token"
  "api_key"
  "apikey"
  "API_SECRET"
  "client_secret"
  "password"
  "PRIVATE_KEY"
  "sk-"
  "bearer"
  "AWS_ACCESS"
  "ghp_"
  "gho_"
)

for pattern in "${PATTERNS[@]}"; do
  echo "=== Searching for: $pattern ==="
  gh api "search/code?q=user:$GITHUB_USER+$pattern" 2>/dev/null | \
    python3 -c "import sys,json; d=json.load(sys.stdin); [print(f'  {i[\"repository\"][\"full_name\"]}: {i[\"path\"]}') for i in d.get('items',[])]" 2>/dev/null
  sleep 2
done
```

#### Sweep Step 3: Clone and deep-scan all repos

Clone every repo (public AND private) into a temp directory and scan locally:

```bash
SCAN_DIR=$(mktemp -d)
cd "$SCAN_DIR"

gh repo list "$GITHUB_USER" --limit 100 --json name --jq '.[].name' | while read -r repo; do
  git clone --quiet "https://github.com/$GITHUB_USER/$repo.git" 2>/dev/null
done
```

**3a. Check for secret files in current tree:**
```bash
find "$SCAN_DIR" -type f \( \
  -name ".env" -o -name ".env.*" -o -name "*.pem" -o -name "*.key" \
  -o -name "credentials.json" -o -name "creds.json" -o -name "service-account*.json" \
  -o -name "*.pfx" -o -name "*.p12" -o -name "id_rsa" -o -name "id_ed25519" \
  -o -name ".netrc" -o -name ".npmrc" -o -name "token.json" -o -name "token.pickle" \
  -o -name "secrets.json" -o -name "secrets.yaml" -o -name "secrets.yml" \
\) -not -path "*/.git/*" 2>/dev/null
```
Exclude `.env.template` and `.env.example` from findings — those are safe (verify they contain only placeholder values).

**3b. Search for hardcoded secret patterns in code:**
```bash
for dir in "$SCAN_DIR"/*/; do
  repo=$(basename "$dir")
  results=$(grep -rn --include="*.py" --include="*.js" --include="*.ts" --include="*.json" --include="*.yaml" --include="*.yml" --include="*.sh" --include="*.html" \
    -E '(sk-[a-zA-Z0-9]{20,}|ghp_[a-zA-Z0-9]{36}|gho_[a-zA-Z0-9]{36}|AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}|xox[bpsar]-[0-9A-Za-z-]{10,}|-----BEGIN (RSA |EC )?PRIVATE KEY|api[_-]?key\s*[:=]\s*["'"'"'][a-zA-Z0-9]{16,}|password\s*[:=]\s*["'"'"'][^"'"'"']{8,})' \
    "$dir" --exclude-dir=.git --exclude-dir=node_modules 2>/dev/null)
  if [ -n "$results" ]; then
    echo "🔴 POTENTIAL SECRETS in $repo:"
    echo "$results"
  fi
done
```

**3c. Check git history for deleted secret files (still recoverable!):**
```bash
for dir in "$SCAN_DIR"/*/; do
  repo=$(basename "$dir")
  cd "$dir"
  added=$(git log --all --diff-filter=A --name-only --pretty=format: 2>/dev/null | \
    grep -iE '\.env$|\.env\.|credential|secret|token\.json|creds|\.pem|\.key|password' | sort -u)
  deleted=$(git log --all --diff-filter=D --name-only --pretty=format: 2>/dev/null | \
    grep -iE '\.env$|\.env\.|credential|secret|token\.json|creds|\.pem|\.key|password' | sort -u)
  if [ -n "$deleted" ]; then
    echo "⚠️  DELETED BUT RECOVERABLE in $repo:"
    echo "$deleted"
  fi
  if [ -n "$added" ]; then
    echo "📋 Secret-pattern files ever committed to $repo:"
    echo "$added"
  fi
  cd "$SCAN_DIR"
done
```

**3d. Check .gitignore coverage:**
```bash
for dir in "$SCAN_DIR"/*/; do
  repo=$(basename "$dir")
  if [ ! -f "$dir/.gitignore" ]; then
    echo "❌ MISSING .gitignore: $repo"
  else
    has_env=$(grep -c '\.env' "$dir/.gitignore" 2>/dev/null || echo 0)
    has_creds=$(grep -ci 'credential\|creds\|secret\|token' "$dir/.gitignore" 2>/dev/null || echo 0)
    has_keys=$(grep -ci '\.pem\|\.key' "$dir/.gitignore" 2>/dev/null || echo 0)
    missing=""
    [ "$has_env" -eq 0 ] && missing="$missing .env"
    [ "$has_creds" -eq 0 ] && missing="$missing credentials"
    [ "$has_keys" -eq 0 ] && missing="$missing keys/pem"
    if [ -n "$missing" ]; then
      echo "⚠️  INCOMPLETE .gitignore ($repo): missing$missing"
    else
      echo "✅ $repo"
    fi
  fi
done
```

**3e. Check for committed node_modules or other bloat:**
```bash
for dir in "$SCAN_DIR"/*/; do
  repo=$(basename "$dir")
  [ -d "$dir/node_modules" ] && echo "🗑️  $repo has node_modules committed"
  [ -d "$dir/__pycache__" ] && echo "🗑️  $repo has __pycache__ committed"
  [ -d "$dir/venv" ] && echo "🗑️  $repo has venv committed"
  [ -d "$dir/.venv" ] && echo "🗑️  $repo has .venv committed"
done
```

#### Sweep Step 4: Check traffic for bot activity (public repos)

```bash
gh repo list "$GITHUB_USER" --limit 100 --json name,isPrivate --jq '.[] | select(.isPrivate==false) | .name' | while read -r repo; do
  views=$(gh api "repos/$GITHUB_USER/$repo/traffic/views" 2>/dev/null | \
    python3 -c "import sys,json; d=json.load(sys.stdin); print(f'{d[\"count\"]}:{d[\"uniques\"]}')" 2>/dev/null || echo "0:0")
  clones=$(gh api "repos/$GITHUB_USER/$repo/traffic/clones" 2>/dev/null | \
    python3 -c "import sys,json; d=json.load(sys.stdin); print(f'{d[\"count\"]}:{d[\"uniques\"]}')" 2>/dev/null || echo "0:0")
  echo "$repo | views=$views | clones=$clones"
done
```

Flag repos with high clones but zero/low views as potential bot scanning targets.

#### Sweep Step 5: Present findings

Organize results by severity:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SHARE-SAFE GITHUB SWEEP REPORT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Account: [username]
Repos scanned: [total] ([public] public, [private] private)

🔴 CRITICAL: Actual secrets found in code
   [list each with repo, file, line]

🟠 HIGH: Secrets in git history (deleted but recoverable)
   [list each with repo, file]

🟡 MEDIUM: .gitignore issues
   Missing .gitignore: [count] repos
   [list repos]
   Incomplete .gitignore: [count] repos
   [list repos with what's missing]

🟢 LOW: Bloat (committed dependencies)
   [list repos with node_modules, etc.]

ℹ️  INFO: Bot activity detected
   [repos with suspicious clone patterns]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Sweep Step 6: Auto-fix (if --fix flag included)

If `--sweep --fix`, automatically fix all non-critical issues:

**6a. Add .gitignore to repos that are missing one:**
Use this comprehensive template:
```
# Environment variables
.env
.env.*
!.env.example
!.env.template

# Credentials and secrets
credentials.json
creds.json
service-account*.json
token.json
token.pickle
secrets.json
secrets.yaml
secrets.yml
*.pem
*.key
*.pfx
*.p12
id_rsa
id_ed25519
.netrc

# Dependencies
node_modules/
venv/
.venv/

# Python
__pycache__/
*.py[cod]
*.egg-info/
dist/
build/

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
*.swo
```

**6b. Patch existing .gitignore files with missing entries:**
Append only the lines that don't already exist in each repo's .gitignore.

**6c. Remove committed bloat:**
```bash
# For repos with committed node_modules
git rm -r --quiet node_modules 2>/dev/null

# For repos with committed __pycache__
git rm -r --quiet __pycache__ 2>/dev/null
```

**6d. Add LICENSE to repos missing one:**
- Public repos: MIT License
- Private repos: All Rights Reserved
- Use the repo creation year for the copyright date
- Ask the user for their name on the copyright line (or use their GitHub display name)

**6e. Commit and push all fixes:**
```bash
git add -A
git commit -m "Add LICENSE and fix .gitignore"
git push
```

**6f. For CRITICAL findings (actual leaked secrets):**
Do NOT auto-fix. Instead, present each one and tell the user:
```
🔴 CRITICAL: Found [secret type] in [repo]/[file]

This secret is exposed in a public repo. You should:
1. Rotate this credential immediately (generate a new one)
2. Remove it from the repo (moving to .env won't erase git history)
3. If it was ever public, consider the old credential compromised

The old secret will remain in git history forever unless you
rewrite history with `git filter-branch` or BFG Repo-Cleaner.
```

#### Sweep cleanup

```bash
rm -rf "$SCAN_DIR"
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
