# Git Commit Skill

Smart commit workflow for applications and non-package changes following RTrading project standards.

## When to Use This Skill

- Committing **application changes** (RPreTrade, RReporting, RJournal, ROrder)
- Committing **box modules** in Tuser/
- Committing **documentation updates**
- **Mid-development commits** before building packages
- Any non-package commit requiring intelligent commit message generation

**Note**: For **package builds**, use `/build` command which handles commits automatically.

## Pre-Commit Checks

Before committing, verify:

### 1. Git Status and Changes

Run in parallel to understand what's being committed:

```bash
# View untracked and modified files
git status

# View staged and unstaged changes
git diff
git diff --cached

# View recent commit history for message style reference
git log --oneline -10
```

### 2. Security Check (CRITICAL)

**NEVER commit**:
- ❌ `config.yml` - Contains credentials
- ❌ `.Renviron` or `Renviron.site` - Contains secrets
- ❌ API keys or authentication tokens
- ❌ Database passwords or connection strings
- ❌ IBKR credentials or account information
- ❌ Files ending in `.secret`, `.key`, `.pem`, `.p12`

**Verify before staging**:
```bash
# Check what's being staged
git diff --cached

# Look for sensitive patterns
git diff --cached | grep -i "password\|api_key\|secret\|token"
```

If credentials are accidentally staged:
```bash
# Unstage immediately
git reset HEAD <file>
```

### 3. Test Before Commit

For applications:
```bash
# Run automated tests
Rscript test_automated.R
```

For packages:
```bash
cd <package_dir> && devtools::test()
```

## Commit Message Guidelines

### Structure

```
<Type>: <Short summary> (max 72 chars)

<Detailed explanation of WHY, not WHAT>
- Focus on motivation and context
- Reference related issues or features
- Note any breaking changes

🤖 Generated with Claude Code
```

### Types

- **feat**: New feature
- **fix**: Bug fix
- **refactor**: Code restructure without behavior change
- **docs**: Documentation updates
- **test**: Adding or updating tests
- **chore**: Maintenance tasks (dependencies, config)
- **perf**: Performance improvements
- **style**: Code style changes (formatting, no logic change)

### Examples

**Good commit message**:
```
feat: Add projection table to Position Analysis tab

Enables users to see P/L projections at different underlying prices.
Uses optional parameter enable_projection=TRUE for backward compatibility.
Position 2 projection remains disabled until Position 2 implementation complete.

- Added projection_range reactive for price calculation
- Extended an_lineUI module with projection support
- Updated test suite with projection scenarios

🤖 Generated with Claude Code
```

**Bad commit message**:
```
Update files
```

### Generating Intelligent Commit Messages

Analyze `git diff` output to understand:
1. **What changed**: Files and code modifications
2. **Why it changed**: Infer purpose from code context
3. **Impact**: What this enables or fixes

## Commit Workflow

### 1. Stage Changes

```bash
# Stage specific files
git add <file1> <file2>

# Stage all changes (be careful!)
git add .

# Interactive staging (review each change)
git add -p
```

### 2. Create Commit

```bash
# Commit with message
git commit -m "$(cat <<'EOF'
feat: Short summary

Detailed explanation here.

🤖 Generated with Claude Code
EOF
)"
```

### 3. Verify Commit

```bash
# View the commit you just made
git log -1 --stat

# View the diff
git show HEAD
```

### 4. Push to Remote

```bash
# Push to remote repository
git push

# Or push and set upstream for new branch
git push -u origin <branch-name>
```

## Handling Pre-Commit Hooks

If commit fails due to pre-commit hook modifications:

### 1. Check What Changed

```bash
# See what the hook modified
git diff
```

### 2. Review and Stage Hook Changes

```bash
# If changes are safe (e.g., auto-formatting)
git add .

# Amend the commit ONLY IF safe to do so
git log -1 --format='%an %ae'  # Check it's your commit
git status  # Verify not pushed yet

# If both true, amend
git commit --amend --no-edit
```

### 3. When NOT to Amend

❌ Don't amend if:
- Commit is from another developer (check author with `git log -1 --format='%an %ae'`)
- Commit has been pushed to remote (check with `git status`)
- Multiple people are working on the branch

In these cases, create a new commit instead:
```bash
git add .
git commit -m "chore: Apply pre-commit hook formatting

🤖 Generated with Claude Code"
```

## Multi-File / Multi-Project Changes

When changes span multiple projects:

### 1. Commit Each Project Separately

```bash
# Commit Tuser changes
cd Tuser && git add . && git commit -m "..."

# Commit RPreTrade changes
cd ../RPreTrade && git add . && git commit -m "..."
```

### 2. Update System change.log

Add summary to `RApplication/change.log` (not in git):

```markdown
## 2025-MM-DD - Brief description of change

**Projects affected**: Tuser, RPreTrade

**Summary**: One paragraph explaining the overall change across projects.

**Files Modified (Tuser)**:
- analysis/view/an_lineUI.R

**Files Modified (RPreTrade)**:
- server.R
- ui.R
- test_automated.R

**Git Commits**:
- Tuser: <commit_hash>
- RPreTrade: <commit_hash>

**Testing**: X/X tests passing
```

## Common Scenarios

### Scenario 1: Bug Fix

```bash
git status
git diff
git add <files>
git commit -m "$(cat <<'EOF'
fix: Handle missing IV column for Gonet portfolios

Greeks calculation was failing when IV column didn't exist.
Now checks for column presence and uses position-based delta
calculation for stock-only portfolios.

Fixes #42

🤖 Generated with Claude Code
EOF
)"
git push
```

### Scenario 2: New Feature

```bash
git status
git diff
git add <files>
git commit -m "$(cat <<'EOF'
feat: Add margin display to account metrics

Shows FullInitMarginReq and FullMaintMarginReq in account
overview tab. Handles multi-currency conversion correctly.

- Added margin fields to account.var list
- Updated getAccountGonet() to populate margin data
- Modified chart display logic in server.R

🤖 Generated with Claude Code
EOF
)"
git push
```

### Scenario 3: Refactoring

```bash
git status
git diff
git add <files>
git commit -m "$(cat <<'EOF'
refactor: Extract position analysis logic to module

Moved 200+ lines from server.R to position_analysis module.
No functional changes, purely structural improvement for
maintainability. All tests still passing.

🤖 Generated with Claude Code
EOF
)"
git push
```

### Scenario 4: Documentation

```bash
git add docs/
git commit -m "$(cat <<'EOF'
docs: Add troubleshooting guide for startup errors

Documents common environment variable issues and how to diagnose
module loading failures. Based on recurring support questions.

🤖 Generated with Claude Code
EOF
)"
git push
```

## Troubleshooting

### "fatal: not a git repository"

```bash
# Verify you're in the right directory
pwd

# Navigate to repository root
cd <repo_dir>

# Verify it's a git repo
git status
```

**Remember**: Chain commands with `&&`:
```bash
cd Tuser && git status  # ✅ Correct
```

### Merge Conflicts

```bash
# Pull latest changes first
git pull

# If conflicts, resolve them in files
# Look for <<<<<<, ======, >>>>>> markers

# After resolving
git add <resolved_files>
git commit -m "Merge: resolve conflicts in <files>"
```

### Accidental Commit

```bash
# Undo last commit, keep changes
git reset --soft HEAD~1

# Undo last commit, discard changes (DANGER!)
git reset --hard HEAD~1

# Amend last commit (if not pushed)
git add <forgotten_file>
git commit --amend --no-edit
```

## Emergency: Credentials Committed

If credentials are accidentally committed:

1. **Immediately rotate/revoke** the exposed credentials
2. Remove from git history (DANGER - rewrites history):
   ```bash
   git filter-branch --force --index-filter \
     "git rm --cached --ignore-unmatch <file>" \
     --prune-empty --tag-name-filter cat -- --all
   ```
3. Force push (coordinate with team first): `git push --force`
4. Contact security team if applicable

## Best Practices

✅ **Do**:
- Commit frequently (small, logical units)
- Write descriptive commit messages
- Review `git diff --cached` before committing
- Run tests before committing
- Pull before push to avoid conflicts

❌ **Don't**:
- Commit broken code
- Commit secrets or credentials
- Use generic messages ("Update files", "Fix stuff")
- Commit large binary files (unless necessary)
- Force push to shared branches (main/master)

## See Also

- `/build` command - Automated build workflow with intelligent commits (for packages)
- Security section in CLAUDE.md - Credential management
- Project CHANGELOG.md files - Version history and change documentation
