# Termux Pro Workflow: Coding Agent + Chat Agent

এই গাইডে Termux-এ advanced level workflow implement করার জন্য ready-to-use process দেওয়া হলো।
All command examples assume UTF-8 terminal encoding.

## Requested Quick Setup (10-Step, Copy-Paste Friendly)

1) **Termux update + base tools install**

```bash
pkg update -y && pkg upgrade -y
pkg install -y git curl wget openssh jq ripgrep fd nodejs python clang make
termux-setup-storage
```

2) **Optional Python tooling**

```bash
pip install --upgrade pip
pip install virtualenv pipx
```

3) **Workspace create**

```bash
# Replace <YOUR_REPO_URL> and <YOUR_REPO_NAME> with your actual repository URL and repo name
# Example: git clone https://github.com/username/repo.git && cd repo
mkdir -p ~/workspaces ~/bin ~/logs ~/backups
cd ~/workspaces
git clone <YOUR_REPO_URL>
cd <YOUR_REPO_NAME>
```

4) **Project local structure**

```bash
mkdir -p .agent/{prompts,reports,logs,tmp}
cp .env.example .env 2>/dev/null || touch .env
chmod 600 .env
```

5) **(If Python project) virtual env**

```bash
python -m venv .venv
source .venv/bin/activate
```

6) **Task execution workflow (every task)**
- Explore code
- Plan ছোট করে লিখো
- Small change করো
- Test চালাও
- Review করো
- Security check করো
- Finalize/commit

7) **Prompt template (প্রতি task এ)**
- Goal
- Constraints
- Files/Scope
- Expected Output
- Done Criteria
- Validation Commands

8) **Batch কাজ (অনেক request হলে)**
- প্রতি batch 10–20 task
- প্রতিটায় retry/backoff rule রাখো
- error summary আলাদা log-এ লিখো

9) **Daily maintenance**

```bash
pkg update -y && pkg upgrade -y
git fetch --all --prune
# interactive cleanup by design (asks confirmation for each file)
find . -type f -name "*.log" -mtime +7 -ok rm {} \;
```

10) **Final map আলাদা রাখা**
- Setup commands
- Task workflow
- Validation commands
- Rollback steps
- Common errors + fixes

## 1) Base Setup Strong

```bash
pkg update -y && pkg upgrade -y
pkg install -y git curl wget openssh jq ripgrep fd nodejs python clang make
termux-setup-storage
```

Optional (Python tools):

```bash
pip install --upgrade pip
pip install virtualenv pipx
```

Optional (Node tools):

```bash
npm i -g npm@latest
```

## 2) Project Workspace Standard

```bash
mkdir -p ~/workspaces ~/bin ~/logs ~/backups
cd ~/workspaces
git clone <YOUR_REPO_URL>
cd <YOUR_REPO_NAME>
```

Create standard local structure:

```bash
mkdir -p .agent/{prompts,reports,logs,tmp}
cp .env.example .env 2>/dev/null || touch .env
chmod 600 .env
```

Python project হলে:

```bash
python -m venv .venv
source .venv/bin/activate
```

## 3) Agent Roles (Clear Separation)

- **Chat Agent**: plan, task breakdown, debugging strategy, prompt refinement
- **Coding Agent**: file changes, refactor, tests, fixes
- **Review Agent**: PR review, risk check, security check

Rule: এক agent = এক focus.

## 4) Reusable Prompting Framework

প্রতি task-এ নিচের template follow করুন:

```text
Goal:
Constraints:
Files/Scope:
Expected Output:
Done Criteria:
Validation Commands:
```

## 5) Safe Development Flow

1. Explore
2. Plan
3. Small Change
4. Test
5. Review
6. Security Scan
7. Finalize

## 6) Batch Strategy (100-request type কাজ)

Batch config (JSON):

```json
{
  "total_requests": 100,
  "batch_size": 10,
  "max_retries": 3,
  "retry_backoff_sec": 5,
  "rate_limit_per_min": 30,
  "log_file": "./.agent/logs/batch.log"
}
```

Best practices:

- ছোট batch
- failure retry with backoff
- প্রতি batch শেষে summary log
- শেষের report: success, failed, retry_used

## 7) Token/Rate/Cost Control

- prompt short রাখুন
- context chunk করে দিন
- repeated instructions template থেকে দিন
- summary cache ব্যবহার করুন

## 8) Automation Layer

`~/.bashrc` এ useful aliases:

```bash
alias gs='git status -sb'
alias ga='git add -p'
alias gc='git commit -m'
alias gp='git pull --rebase'
alias ll='ls -lah'
alias lg='git --no-pager log --oneline -n 15'
alias tfindlogs='find . -type f -name "*.log" -size +20M -print'
```

## 9) Quality Gate (Mandatory)

প্রতি change এ:

- lint
- test
- build
- manual smoke check
- regression checklist

## 10) Security & Privacy Discipline

- `.env` ছাড়া key রাখবেন না
- secrets hardcode করবেন না
- logs/public output-এ token redact করুন
- secret scan habit রাখুন

## 11) Failure Recovery Playbook

Agent stuck হলে:

1. context reset
2. task split ছোট করুন
3. known-good checkpoint থেকে restart
4. reproducible note লিখুন (input + expected + actual)

## 12) Pro Daily Routine

- **Morning**: sync + plan
- **Work block**: implement + test
- **End**: review + security + summary + next queue

---

## Copy-Paste Command Pack

### A) Initial setup pack

```bash
pkg update -y && pkg upgrade -y
pkg install -y git curl wget openssh jq ripgrep fd nodejs python clang make
termux-setup-storage
mkdir -p ~/workspaces ~/bin ~/logs ~/backups
```

### B) Project bootstrap pack

```bash
cd ~/workspaces
# Replace <YOUR_REPO_URL> and <YOUR_REPO_NAME> with your actual repository URL and repo name
# Example: git clone https://github.com/username/repo.git && cd repo
git clone <YOUR_REPO_URL> && cd <YOUR_REPO_NAME>
mkdir -p .agent/{prompts,reports,logs,tmp}
cp .env.example .env 2>/dev/null || touch .env
chmod 600 .env
if command -v python >/dev/null 2>&1; then
  python -m venv .venv
else
  echo "Python not found, skipping .venv creation"
fi
```

### C) Run flow pack (per task)

```bash
# 1) sync
git pull --rebase

# 2) branch
git checkout -b feat/<short-task-name>

# 3) do small change, then:
git add -p
git commit -m "feat: <short-message>"

# 4) local quality gate (use your repo commands)
# npm run lint && npm test && npm run build
# or: pytest
```

### D) 100-request execution pack (pseudo-runbook)

```bash
mkdir -p .agent/logs
echo "Start: $(date -Iseconds)" >> .agent/logs/batch.log
echo "Plan: total=100 batch=10 retry=3" >> .agent/logs/batch.log
echo "Run batches 1..10 and append status per batch" >> .agent/logs/batch.log
echo "End: $(date -Iseconds)" >> .agent/logs/batch.log
```

### E) Daily maintenance pack

```bash
git fetch --all --prune
# interactive cleanup by design (asks confirmation for each file)
find . -type f -name "*.log" -mtime +7 -ok rm {} \;
find . -type d -name "__pycache__" -prune -exec rm -rf {} +
```

---

## Day-1 to Day-7 Execution Map (Exact Copy-Paste)

### Day-1 (Base install + workspace)
```bash
pkg update -y && pkg upgrade -y
pkg install -y git curl wget openssh jq ripgrep fd nodejs python clang make
termux-setup-storage
mkdir -p ~/workspaces ~/bin ~/logs ~/backups
cd ~/workspaces
# Replace <YOUR_REPO_URL> and <YOUR_REPO_NAME> with your actual repository URL and repo name
git clone <YOUR_REPO_URL>
cd <YOUR_REPO_NAME>
mkdir -p .agent/{prompts,reports,logs,tmp}
cp .env.example .env 2>/dev/null || touch .env
chmod 600 .env
```

### Day-2 (Python optional setup + baseline check)
```bash
pip install --upgrade pip
pip install virtualenv pipx
python -m venv .venv
source .venv/bin/activate
git status -sb
```

### Day-3 (Start first task safely)
```bash
git pull --rebase
git checkout -b feat/first-small-task
# make small change
git add -p
git commit -m "feat: first small task"
```

### Day-4 (Validation day)
```bash
# replace with your repo commands
# npm run lint && npm test && npm run build
# or: pytest
echo "Run repo validation commands here"
```

### Day-5 (Batch run discipline)
```bash
mkdir -p .agent/logs
echo "Start: $(date -Iseconds)" >> .agent/logs/batch.log
echo "Batch rule: size=10 retry=3 backoff=5s" >> .agent/logs/batch.log
echo "Process batches and append per-batch summary" >> .agent/logs/batch.log
echo "End: $(date -Iseconds)" >> .agent/logs/batch.log
```

### Day-6 (Review + security habit)
```bash
git --no-pager log --oneline -n 10
git --no-pager diff --stat HEAD~1..HEAD
echo "Run code review and security scan before finalize"
```

### Day-7 (Maintenance + next-week prep)
```bash
pkg update -y && pkg upgrade -y
git fetch --all --prune
# interactive cleanup by design (asks confirmation for each file)
find . -type f -name "*.log" -mtime +7 -ok rm {} \;
find . -type d -name "__pycache__" -prune -exec rm -rf {} +
echo "Prepare next week: tasks, risks, rollback notes"
```
