# Git Quick Reference Guide

## Essential Git Commands

### Setup & Configuration
```bash
# Initial setup
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
git config --global init.defaultBranch main

# View configuration
git config --list
git config user.name

# Edit config
git config --global --edit
```

### Creating Repositories
```bash
# Initialize new repository
git init

# Clone existing repository
git clone <url>
git clone <url> <directory>
git clone -b <branch> <url>
```

### Basic Commands
```bash
# Check status
git status
git status -s  # Short format

# Stage changes
git add <file>
git add .  # Add all files
git add -p  # Interactive staging

# Commit changes
git commit -m "message"
git commit -am "message"  # Stage and commit tracked files
git commit --amend  # Amend last commit

# View history
git log
git log --oneline
git log --graph --all --decorate

# View changes
git diff  # Unstaged changes
git diff --staged  # Staged changes
git diff <commit1> <commit2>  # Between commits
```

### Branching
```bash
# List branches
git branch
git branch -a  # All branches (including remote)
git branch -v  # Verbose

# Create branch
git branch <branch-name>
git checkout -b <branch-name>  # Create and switch
git switch -c <branch-name>  # Git 2.23+

# Switch branches
git checkout <branch-name>
git switch <branch-name>  # Git 2.23+

# Delete branch
git branch -d <branch-name>  # Safe delete
git branch -D <branch-name>  # Force delete
```

### Merging
```bash
# Merge branch
git merge <branch-name>
git merge --no-ff <branch-name>  # No fast-forward

# Abort merge
git merge --abort

# View merged branches
git branch --merged
git branch --no-merged
```

### Remote Operations
```bash
# View remotes
git remote
git remote -v
git remote show origin

# Add/remove remote
git remote add origin <url>
git remote remove origin

# Fetch changes
git fetch origin
git fetch --all --prune

# Pull changes
git pull origin <branch>
git pull --rebase origin <branch>

# Push changes
git push origin <branch>
git push -u origin <branch>  # Set upstream
git push --all  # All branches
git push --tags  # All tags
git push --force-with-lease  # Safer force push
```

### Undoing Changes
```bash
# Discard working directory changes
git restore <file>
git checkout -- <file>  # Old syntax

# Unstage changes
git restore --staged <file>
git reset HEAD <file>  # Old syntax

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert commit (creates new commit)
git revert <commit-hash>
```

### Stashing
```bash
# Stash changes
git stash
git stash save "message"
git stash -u  # Include untracked files

# List stashes
git stash list

# Apply stash
git stash apply
git stash apply stash@{n}
git stash pop  # Apply and remove

# Drop stash
git stash drop stash@{n}
git stash clear  # Clear all stashes
```

### Tags
```bash
# List tags
git tag
git tag -l "v1.*"

# Create tag
git tag v1.0.0
git tag -a v1.0.0 -m "Version 1.0.0"

# Push tags
git push origin v1.0.0
git push origin --tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0

# Checkout tag
git checkout v1.0.0
```

### Rebasing
```bash
# Rebase onto branch
git rebase <branch>
git rebase main

# Interactive rebase
git rebase -i HEAD~3

# Continue/abort rebase
git rebase --continue
git rebase --abort
```

### Advanced Commands
```bash
# Cherry-pick commit
git cherry-pick <commit-hash>

# Bisect (find bug)
git bisect start
git bisect bad
git bisect good <commit>

# Reflog (recover lost commits)
git reflog

# Clean untracked files
git clean -fd  # Force remove files and directories
git clean -n  # Dry run

# Show file from commit
git show <commit>:<file>

# Blame (who changed what)
git blame <file>
git blame -L 10,20 <file>  # Specific lines
```

---

## Common Workflows

### Feature Development Workflow
```bash
# 1. Update main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/new-feature

# 3. Make changes
# Edit files...
git add .
git commit -m "Implement new feature"

# 4. Push to remote
git push -u origin feature/new-feature

# 5. Create pull request (on GitHub/GitLab)

# 6. After approval, merge and delete branch
git checkout main
git pull origin main
git branch -d feature/new-feature
```

### Fix Bug Workflow
```bash
# 1. Create bugfix branch
git checkout -b bugfix/fix-login

# 2. Fix bug
# Edit files...
git add .
git commit -m "fix: resolve login issue

Fixes #123"

# 3. Push and create PR
git push -u origin bugfix/fix-login
```

### Update Feature Branch with Main
```bash
# Option 1: Merge
git checkout feature-branch
git merge main

# Option 2: Rebase (cleaner history)
git checkout feature-branch
git rebase main

# If conflicts, resolve and:
git add <resolved-files>
git rebase --continue
```

### Syncing Fork
```bash
# 1. Add upstream (once)
git remote add upstream <original-repo-url>

# 2. Fetch upstream changes
git fetch upstream

# 3. Merge into your branch
git checkout main
git merge upstream/main

# 4. Push to your fork
git push origin main
```

---

## Git Aliases

Add to `~/.gitconfig`:

```ini
[alias]
    # Shortcuts
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = reset HEAD --
    
    # Logs
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    last = log -1 HEAD
    ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
    ll = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --numstat
    
    # Diffs
    d = diff
    ds = diff --staged
    dc = diff --cached
    
    # Branch management
    bv = branch -vv
    ba = branch -a
    bd = branch -d
    
    # Stash shortcuts
    sl = stash list
    sa = stash apply
    ss = stash save
    
    # Undo
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
    
    # Other
    aliases = config --get-regexp alias
    contributors = shortlog --summary --numbered
```

---

## .gitignore Templates

### Node.js
```
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment
.env
.env.local
.env.*.local

# Build
dist/
build/
*.log

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db
```

### Python
```
# Byte-compiled
__pycache__/
*.py[cod]
*$py.class

# Virtual environments
venv/
env/
ENV/

# IDE
.vscode/
.idea/
*.swp

# Testing
.pytest_cache/
.coverage
htmlcov/

# Distribution
dist/
build/
*.egg-info/
```

### Java
```
# Compiled
*.class
*.jar
*.war
*.ear

# Build
target/
build/
out/

# IDE
.idea/
*.iml
.classpath
.project
.settings/

# Logs
*.log
```

---

## Troubleshooting Quick Fixes

### Undo last commit (keep changes)
```bash
git reset --soft HEAD~1
```

### Undo last commit (discard changes)
```bash
git reset --hard HEAD~1
```

### Amend last commit
```bash
# Add more changes
git add <files>
git commit --amend
```

### Remove file from staging
```bash
git restore --staged <file>
```

### Discard local changes
```bash
git restore <file>
# Or discard all changes
git restore .
```

### Recover deleted branch
```bash
git reflog
git checkout -b <branch-name> <commit-hash>
```

### Remove large file from history
```bash
git filter-branch --tree-filter 'rm -f <large-file>' HEAD
# Or use BFG Repo-Cleaner (faster)
```

### Fix wrong commit author
```bash
git commit --amend --author="Name <email>"
```

### Sync with remote (force)
```bash
# ⚠️ CAUTION: Overwrites local changes
git fetch origin
git reset --hard origin/main
```

---

## Git Configuration Tips

### Useful Global Settings
```bash
# Auto-correct typos
git config --global help.autocorrect 20

# Default branch name
git config --global init.defaultBranch main

# Pull behavior
git config --global pull.rebase false  # merge (default)
git config --global pull.rebase true   # rebase

# Push behavior
git config --global push.default current

# Reuse recorded conflict resolutions
git config --global rerere.enabled true

# Show original in conflicts
git config --global merge.conflictstyle diff3

# Prune on fetch
git config --global fetch.prune true

# Colors
git config --global color.ui auto
```

### Line Ending Settings
```bash
# Windows
git config --global core.autocrlf true

# Mac/Linux
git config --global core.autocrlf input
```

---

## Commit Message Format

### Conventional Commits
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting, missing semicolons, etc.
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance

**Example:**
```
feat(auth): add JWT authentication

Implement JWT-based authentication with refresh tokens.
Includes login, logout, and token refresh endpoints.

Closes #42
```

---

## Performance Tips

### Speed up status
```bash
git config --global core.fsmonitor true
git config --global core.untrackedCache true
```

### Shallow clone (faster)
```bash
git clone --depth 1 <url>
```

### Partial clone
```bash
git clone --filter=blob:none <url>
```

### Maintenance
```bash
# Optimize repository
git gc

# Aggressive optimization
git gc --aggressive

# Verify integrity
git fsck
```

---

## Security Best Practices

### Never Commit
- Passwords
- API keys
- Private keys
- .env files with secrets
- Database credentials

### Use .gitignore
```bash
# Add file to .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .env to .gitignore"
```

### Remove committed secrets
```bash
# Remove from history (use BFG)
java -jar bfg.jar --delete-files secret.key
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

### Sign commits
```bash
# Configure GPG
git config --global user.signingkey <key-id>
git config --global commit.gpgsign true

# Sign commit
git commit -S -m "Signed commit"
```

---

## Git Cheat Sheet

| Command | Description |
|---------|-------------|
| `git init` | Initialize repository |
| `git clone <url>` | Clone repository |
| `git add <file>` | Stage file |
| `git commit -m "msg"` | Commit changes |
| `git status` | Check status |
| `git log` | View history |
| `git diff` | View changes |
| `git branch` | List branches |
| `git checkout -b <branch>` | Create and switch branch |
| `git merge <branch>` | Merge branch |
| `git pull` | Fetch and merge |
| `git push` | Push changes |
| `git stash` | Stash changes |
| `git tag <name>` | Create tag |
| `git rebase <branch>` | Rebase |
| `git reset --hard HEAD~1` | Undo last commit |
| `git reflog` | View all actions |

---

**Quick tip**: Use `git help <command>` for detailed help on any Git command!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

