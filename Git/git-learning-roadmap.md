# Git: Beginner to Expert
## Complete Learning Roadmap

> Master Git version control from basics to advanced workflows

---

## Table of Contents
1. [Introduction](#introduction)
2. [Level 1: Beginner (Weeks 1-2)](#level-1-beginner-weeks-1-2)
3. [Level 2: Intermediate (Weeks 3-4)](#level-2-intermediate-weeks-3-4)
4. [Level 3: Advanced (Weeks 5-8)](#level-3-advanced-weeks-5-8)
5. [Level 4: Expert (Weeks 9-12)](#level-4-expert-weeks-9-12)
6. [Git Workflows](#git-workflows)
7. [Best Practices](#best-practices)
8. [Resources](#resources)

---

## Introduction

### What is Git?
**Git** is a distributed version control system that tracks changes in source code during software development.

### Key Benefits:
- **Version Control**: Track every change
- **Collaboration**: Work with teams seamlessly
- **Branching**: Experiment without fear
- **Distributed**: Every clone is a full backup
- **Speed**: Fast operations
- **Open Source**: Free and widely adopted

### Prerequisites:
- Basic command line knowledge
- Text editor installed
- Understanding of files and directories

---

## Level 1: Beginner (Weeks 1-2)

### Week 1: Git Fundamentals

#### 1.1 Installation & Setup

**Install Git:**
```bash
# Windows (Chocolatey)
choco install git

# Mac
brew install git

# Linux (Ubuntu/Debian)
sudo apt-get install git

# Verify installation
git --version
```

**Initial Configuration:**
```bash
# Set your identity (REQUIRED)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"  # VS Code
# OR
git config --global core.editor "vim"  # Vim

# Enable colored output
git config --global color.ui auto

# View all config
git config --list
git config --global --list

# View specific config
git config user.name
git config user.email
```

**Config Levels:**
- `--system`: All users on system
- `--global`: All repos for current user
- `--local`: Specific repository (default)

**✅ Week 1 Checkpoint**: Git installed and configured

---

#### 1.2 Creating Your First Repository

**Initialize a Repository:**
```bash
# Create directory
mkdir my-project
cd my-project

# Initialize Git
git init

# Check status
git status
```

**Understanding .git Directory:**
```bash
# View .git contents
ls -la .git/

# Key files/directories:
# - HEAD: Points to current branch
# - config: Repository configuration
# - objects/: All data (commits, trees, blobs)
# - refs/: Pointers to commits (branches, tags)
```

**Create Your First Commit:**
```bash
# Create a file
echo "# My Project" > README.md

# Check status
git status  # Shows untracked file

# Stage the file
git add README.md

# Check status again
git status  # Shows staged file

# Commit
git commit -m "Initial commit"

# View commit history
git log
```

---

#### 1.3 Understanding Git Workflow

**The Three States:**

```
Working Directory  →  Staging Area  →  Repository
   (modified)         (git add)        (git commit)
```

**Example Workflow:**
```bash
# 1. Modify file
echo "Hello Git" > hello.txt

# 2. Check what changed
git status
git diff  # Show unstaged changes

# 3. Stage changes
git add hello.txt

# 4. Check staged changes
git diff --staged

# 5. Commit
git commit -m "Add hello.txt"

# 6. View history
git log
git log --oneline
```

**Common Commands:**
```bash
# Check status (use frequently!)
git status

# Add specific file
git add filename.txt

# Add all files
git add .
git add -A  # Same as above

# Add by pattern
git add *.txt
git add src/

# Remove from staging
git restore --staged filename.txt  # Git 2.23+
git reset HEAD filename.txt         # Older versions

# Discard changes in working directory
git restore filename.txt
git checkout -- filename.txt  # Older versions
```

---

#### 1.4 Writing Good Commit Messages

**Commit Message Format:**
```
Short summary (50 characters or less)

More detailed explanation (wrap at 72 characters).
Explain the "why" not the "what".

- Bullet points are okay
- Use present tense: "Add feature" not "Added feature"
- Reference issue numbers if applicable: Fixes #123
```

**Examples:**

**Good:**
```bash
git commit -m "Add user authentication feature

Implement JWT-based authentication with refresh tokens.
Includes login, logout, and token refresh endpoints.

Fixes #42"
```

**Bad:**
```bash
git commit -m "changes"
git commit -m "fixed stuff"
git commit -m "asdfasdf"
```

**Conventional Commits:**
```bash
# Format: <type>(<scope>): <subject>

git commit -m "feat: add user login"
git commit -m "fix: resolve memory leak in cache"
git commit -m "docs: update API documentation"
git commit -m "style: format code with prettier"
git commit -m "refactor: simplify database queries"
git commit -m "test: add unit tests for auth module"
git commit -m "chore: update dependencies"
```

---

#### 1.5 Viewing History

**Basic Log:**
```bash
# Full log
git log

# One line per commit
git log --oneline

# Last N commits
git log -n 5
git log -5  # Same as above

# Show changes in each commit
git log -p
git log --patch

# Show stats
git log --stat

# Graph view
git log --graph --oneline --all

# Pretty format
git log --pretty=format:"%h - %an, %ar : %s"
# %h = short hash
# %an = author name
# %ar = author date, relative
# %s = subject
```

**Filtering History:**
```bash
# By author
git log --author="John"

# By date
git log --since="2 weeks ago"
git log --after="2024-01-01"
git log --before="2024-12-31"

# By message
git log --grep="bug fix"

# By file
git log README.md
git log -- src/

# Combining filters
git log --author="John" --since="1 month ago" --oneline
```

**Viewing Specific Commit:**
```bash
# Show commit details
git show <commit-hash>
git show HEAD
git show HEAD~1  # Previous commit

# Show specific file in commit
git show <commit-hash>:path/to/file.txt
```

**✅ Week 1 Checkpoint**: Comfortable with init, add, commit, status, log

---

### Week 2: Branching Basics

#### 2.1 Understanding Branches

**What is a Branch?**
A branch is simply a pointer to a commit. The default branch is usually `main` or `master`.

**Branch Basics:**
```bash
# List branches
git branch
git branch -v  # Show last commit
git branch -vv  # Show tracking branches

# Create new branch
git branch feature-login

# Switch to branch
git checkout feature-login
git switch feature-login  # Git 2.23+

# Create and switch in one command
git checkout -b feature-signup
git switch -c feature-signup  # Git 2.23+

# View all branches (including remote)
git branch -a
```

**Understanding HEAD:**
```
HEAD → main → [commit]
       ↓
    feature-login → [commit]
```

**Making Changes on Branch:**
```bash
# Create and switch to new branch
git switch -c add-header

# Make changes
echo "# Header" > header.txt
git add header.txt
git commit -m "Add header component"

# Switch back to main
git switch main

# header.txt doesn't exist on main!
ls  # No header.txt

# Switch back to branch
git switch add-header
ls  # header.txt is back!
```

---

#### 2.2 Merging Branches

**Fast-Forward Merge:**
```bash
# On main branch
git switch main

# Merge feature branch
git merge add-header

# If no conflicts and linear history, Git does fast-forward:
# main pointer just moves forward
```

**Three-Way Merge:**
```bash
# Create two branches with different changes
git switch -c feature-a
echo "Feature A" > a.txt
git add a.txt
git commit -m "Add feature A"

git switch main
git switch -c feature-b
echo "Feature B" > b.txt
git add b.txt
git commit -m "Add feature B"

# Merge feature-b into main
git switch main
git merge feature-b  # Fast-forward

# Merge feature-a into main
git merge feature-a  # Three-way merge (creates merge commit)
```

**Merge Commit Message:**
```bash
# Merge with custom message
git merge feature-login -m "Merge feature: user login"

# Merge without fast-forward (always create merge commit)
git merge --no-ff feature-branch
```

---

#### 2.3 Handling Merge Conflicts

**What Causes Conflicts?**
When same lines in same file are changed differently in two branches.

**Creating a Conflict (Practice):**
```bash
# On main branch
echo "Hello from main" > conflict.txt
git add conflict.txt
git commit -m "Add conflict.txt on main"

# Create and switch to branch
git switch -c conflict-branch main~1  # Branch from parent

# Modify same file
echo "Hello from branch" > conflict.txt
git add conflict.txt
git commit -m "Add conflict.txt on branch"

# Try to merge
git switch main
git merge conflict-branch  # CONFLICT!
```

**Resolving Conflicts:**
```bash
# Check status
git status  # Shows conflicted files

# Open conflicted file
# You'll see conflict markers:
<<<<<<< HEAD
Hello from main
=======
Hello from branch
>>>>>>> conflict-branch

# Edit file to resolve:
# Remove markers and keep desired content
Hello from main and branch

# Stage resolved file
git add conflict.txt

# Complete merge
git commit  # Git provides default merge message

# Or abort merge
git merge --abort
```

**Merge Tools:**
```bash
# Configure merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Use merge tool
git mergetool
```

---

#### 2.4 Deleting Branches

```bash
# Delete merged branch
git branch -d feature-login

# Force delete unmerged branch
git branch -D feature-experimental

# Delete remote branch
git push origin --delete feature-old

# Prune deleted remote branches
git fetch --prune
git remote prune origin
```

**✅ Week 2 Checkpoint**: Comfortable with branching, merging, and conflict resolution

---

## Level 2: Intermediate (Weeks 3-4)

### Week 3: Working with Remotes

#### 3.1 Understanding Remote Repositories

**What is a Remote?**
A remote is a version of your repository hosted elsewhere (GitHub, GitLab, Bitbucket, etc.)

**Common Remotes:**
- **origin**: Default remote name when you clone
- **upstream**: Often used for original repository in forks

**Remote Commands:**
```bash
# View remotes
git remote
git remote -v  # Show URLs

# Add remote
git remote add origin https://github.com/user/repo.git

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Rename remote
git remote rename origin upstream

# Remove remote
git remote remove origin

# Show remote details
git remote show origin
```

---

#### 3.2 Cloning Repositories

```bash
# Clone repository
git clone https://github.com/user/repo.git

# Clone to specific directory
git clone https://github.com/user/repo.git my-repo

# Clone specific branch
git clone -b develop https://github.com/user/repo.git

# Clone with depth (shallow clone)
git clone --depth 1 https://github.com/user/repo.git

# Clone with submodules
git clone --recursive https://github.com/user/repo.git
```

**After Cloning:**
```bash
# Check remote
git remote -v

# Check branches
git branch -a

# Check current branch
git branch --show-current
```

---

#### 3.3 Pushing Changes

```bash
# Push to remote
git push origin main

# Push new branch
git push -u origin feature-login
# -u sets upstream tracking

# Push all branches
git push --all

# Push tags
git push --tags

# Force push (DANGEROUS!)
git push --force origin main
# Better: force with lease (safer)
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete old-feature
```

**Push Workflow:**
```bash
# Make changes
echo "Update" >> README.md
git add README.md
git commit -m "Update README"

# Push to remote
git push origin main

# First time pushing a new branch
git push -u origin feature-new
# After setting upstream, just:
git push
```

---

#### 3.4 Fetching and Pulling

**Fetch vs Pull:**
- **fetch**: Download changes but don't merge
- **pull**: fetch + merge

**Fetch:**
```bash
# Fetch from origin
git fetch origin

# Fetch specific branch
git fetch origin main

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune

# View fetched changes
git log origin/main  # Remote main branch
git diff origin/main  # Compare with local
```

**Pull:**
```bash
# Pull from origin
git pull origin main

# Pull with rebase instead of merge
git pull --rebase origin main

# Pull all branches
git pull --all

# Configure pull behavior
git config pull.rebase false  # merge (default)
git config pull.rebase true   # rebase
git config pull.ff only       # fast-forward only
```

**Pull Workflow:**
```bash
# Before starting work
git pull origin main

# After others push changes
git fetch origin
git merge origin/main
# OR
git pull origin main
```

---

#### 3.5 Working with Forks

**Fork Workflow:**
```bash
# 1. Fork on GitHub (use web interface)

# 2. Clone your fork
git clone https://github.com/YOUR-USERNAME/repo.git
cd repo

# 3. Add upstream remote
git remote add upstream https://github.com/ORIGINAL-OWNER/repo.git

# 4. Verify remotes
git remote -v
# origin    https://github.com/YOUR-USERNAME/repo.git
# upstream  https://github.com/ORIGINAL-OWNER/repo.git

# 5. Keep fork updated
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 6. Create feature branch
git checkout -b fix-bug

# 7. Make changes and commit
git add .
git commit -m "Fix bug in function"

# 8. Push to your fork
git push origin fix-bug

# 9. Create Pull Request on GitHub
```

**✅ Week 3 Checkpoint**: Comfortable with remotes, push, pull, fetch

---

### Week 4: Advanced Operations

#### 4.1 Git Stash

**What is Stash?**
Temporarily save changes without committing.

**Basic Stash:**
```bash
# Stash changes
git stash
git stash save "Work in progress"

# List stashes
git stash list

# Apply latest stash
git stash apply

# Apply and remove stash
git stash pop

# Apply specific stash
git stash apply stash@{1}

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

**Advanced Stash:**
```bash
# Stash including untracked files
git stash -u
git stash --include-untracked

# Stash with patch mode
git stash -p

# Create branch from stash
git stash branch feature-from-stash

# View stash contents
git stash show
git stash show -p  # Show diff
```

**Stash Workflow:**
```bash
# Working on feature
echo "WIP" >> feature.txt
git add feature.txt

# Urgent bug fix needed!
git stash

# Fix bug
git checkout main
# ... make fixes ...
git commit -m "Fix critical bug"

# Return to feature
git checkout feature-branch
git stash pop
# Continue working
```

---

#### 4.2 Git Tags

**Lightweight Tags:**
```bash
# Create lightweight tag
git tag v1.0.0

# Tag specific commit
git tag v0.9.0 <commit-hash>

# List tags
git tag
git tag -l "v1.*"  # Pattern matching
```

**Annotated Tags (Recommended):**
```bash
# Create annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag with detailed message
git tag -a v1.0.0

# Show tag details
git show v1.0.0

# List with messages
git tag -n
```

**Working with Tags:**
```bash
# Push tags to remote
git push origin v1.0.0
git push origin --tags  # Push all tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Checkout tag
git checkout v1.0.0  # Detached HEAD state
git checkout -b version1.0 v1.0.0  # Create branch from tag
```

---

#### 4.3 Git Cherry-Pick

**What is Cherry-Pick?**
Apply specific commits from one branch to another.

**Basic Cherry-Pick:**
```bash
# Pick single commit
git cherry-pick <commit-hash>

# Pick multiple commits
git cherry-pick <commit1> <commit2>

# Pick range of commits
git cherry-pick <commit1>..<commit2>
```

**Cherry-Pick with Options:**
```bash
# Cherry-pick without committing
git cherry-pick -n <commit-hash>
git cherry-pick --no-commit <commit-hash>

# Cherry-pick with new message
git cherry-pick -e <commit-hash>

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

**Example Workflow:**
```bash
# On main, found good commit from feature branch
git log feature-branch  # Find commit hash

# Cherry-pick it to main
git cherry-pick abc123

# If conflicts occur
# 1. Resolve conflicts
# 2. git add <resolved-files>
# 3. git cherry-pick --continue
```

---

#### 4.4 Git Revert

**What is Revert?**
Create new commit that undoes changes from previous commit(s).

**Basic Revert:**
```bash
# Revert last commit
git revert HEAD

# Revert specific commit
git revert <commit-hash>

# Revert without auto-commit
git revert -n <commit-hash>

# Revert merge commit
git revert -m 1 <merge-commit-hash>
```

**Revert vs Reset:**
- **Revert**: Safe for public branches (creates new commit)
- **Reset**: Rewrites history (dangerous for public branches)

```bash
# Revert (safe for shared branches)
git revert abc123
# Creates new commit undoing abc123

# Reset (dangerous for shared branches)
git reset --hard abc123
# Deletes all commits after abc123
```

**✅ Week 4 Checkpoint**: Comfortable with stash, tags, cherry-pick, revert

---

## Level 3: Advanced (Weeks 5-8)

### Week 5-6: Git Rebase

#### 5.1 Understanding Rebase

**Rebase vs Merge:**

**Merge:**
```
main:    A---B---C---D
                  \   \
feature:           E---F---M (merge commit)
```

**Rebase:**
```
main:    A---B---C---D
                      \
feature:               E'---F' (rebased commits)
```

**Basic Rebase:**
```bash
# On feature branch
git switch feature-branch

# Rebase onto main
git rebase main

# Or in one step
git rebase main feature-branch
```

**Interactive Rebase:**
```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# Rebase from specific commit
git rebase -i <commit-hash>
```

**Interactive Options:**
```
pick   = use commit
reword = use commit, but edit message
edit   = use commit, but stop for amending
squash = use commit, but meld into previous commit
fixup  = like squash, but discard commit message
drop   = remove commit
```

**Example Interactive Rebase:**
```bash
git rebase -i HEAD~3

# Editor opens:
pick abc123 Add feature A
pick def456 Fix typo
pick ghi789 Add feature B

# Change to:
pick abc123 Add feature A
fixup def456 Fix typo
pick ghi789 Add feature B

# Save and close
# Result: def456 is squashed into abc123
```

---

#### 5.2 Rebase Workflows

**Update Feature Branch:**
```bash
# Main has new commits
git checkout feature-branch
git rebase main

# Resolve conflicts if any
# For each conflict:
# 1. Fix conflicts
# 2. git add <files>
# 3. git rebase --continue

# Or abort
git rebase --abort
```

**Clean Up Commits Before Merge:**
```bash
# Interactive rebase to squash commits
git rebase -i main

# Squash multiple commits into one
pick abc123 Add login feature
squash def456 Fix login bug
squash ghi789 Update login tests

# Result: One clean commit
```

**Rebase onto Different Base:**
```bash
# Move branch from old-base to new-base
git rebase --onto new-base old-base feature-branch
```

---

#### 5.3 Rebase Best Practices

**Golden Rule: Never Rebase Public Branches!**
```bash
# ❌ DON'T do this if others have the branch
git push --force origin main

# ✅ DO this for your local feature branches
git push --force-with-lease origin feature-branch
```

**When to Use Rebase:**
- ✅ Clean up local commits before pushing
- ✅ Update feature branch with main
- ✅ Squash fixup commits
- ✅ Edit commit messages

**When NOT to Use Rebase:**
- ❌ On public/shared branches
- ❌ After force-pushing (unless coordinated)
- ❌ If team prefers merge commits

**✅ Weeks 5-6 Checkpoint**: Understand and use rebase confidently

---

### Week 7-8: Advanced Topics

#### 7.1 Git Reset (Advanced)

**Three Types of Reset:**

**1. Soft Reset** (Keep changes staged)
```bash
git reset --soft HEAD~1
# Moves HEAD back one commit
# Changes remain staged
# Use for: Amending last commit
```

**2. Mixed Reset** (Default, unstage changes)
```bash
git reset HEAD~1
git reset --mixed HEAD~1
# Moves HEAD back
# Changes remain in working directory but unstaged
# Use for: Redo commits differently
```

**3. Hard Reset** (Discard changes)
```bash
git reset --hard HEAD~1
# Moves HEAD back
# Discards all changes
# Use for: Completely undo commits (DANGEROUS!)
```

**Reset Examples:**
```bash
# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last 3 commits, keep changes unstaged
git reset HEAD~3

# Discard all changes and commits
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard abc123

# Reset specific file
git reset HEAD filename.txt
```

---

#### 7.2 Git Reflog (Recovery)

**What is Reflog?**
Reference log - tracks all changes to HEAD (even deleted commits).

**Using Reflog:**
```bash
# View reflog
git reflog

# Output:
# abc123 HEAD@{0}: commit: Add feature
# def456 HEAD@{1}: commit: Fix bug
# ghi789 HEAD@{2}: reset: moving to HEAD~1

# Recover "lost" commit
git reset --hard HEAD@{1}

# Or use commit hash
git reset --hard def456
```

**Recovery Scenarios:**
```bash
# Scenario 1: Accidental hard reset
git reset --hard HEAD~5  # Oops! Lost 5 commits
git reflog  # Find commits
git reset --hard HEAD@{1}  # Recover

# Scenario 2: Deleted branch with unmerged commits
git branch -D feature  # Oops! Had important changes
git reflog  # Find last commit of branch
git checkout -b feature abc123  # Recreate branch

# Scenario 3: Bad rebase
git rebase -i HEAD~10  # Made mistakes
git reflog
git reset --hard HEAD@{1}  # Before rebase
```

---

#### 7.3 Git Bisect (Finding Bugs)

**What is Bisect?**
Binary search through commit history to find bug introduction.

**Basic Bisect:**
```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark known good commit
git bisect good v1.0

# Git checks out middle commit
# Test it...

# If bug exists
git bisect bad

# If no bug
git bisect good

# Repeat until Git finds the commit
# Git will say: "abc123 is the first bad commit"

# End bisect
git bisect reset
```

**Automated Bisect:**
```bash
# Start bisect
git bisect start HEAD v1.0

# Run automated test
git bisect run npm test
# Git automatically tests each commit

# Result shows first failing commit
git bisect reset
```

---

#### 7.4 Git Hooks

**What are Hooks?**
Scripts that run automatically at certain Git events.

**Common Hooks:**
- `pre-commit`: Before commit is created
- `prepare-commit-msg`: Before commit message editor
- `commit-msg`: After commit message is saved
- `post-commit`: After commit is created
- `pre-push`: Before push
- `pre-rebase`: Before rebase

**Hook Location:**
```bash
# Hooks are in .git/hooks/
ls .git/hooks/

# Create pre-commit hook
nano .git/hooks/pre-commit
```

**Example pre-commit Hook:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit aborted."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi
```

**Make Hook Executable:**
```bash
chmod +x .git/hooks/pre-commit
```

**Example commit-msg Hook (enforce format):**
```bash
#!/bin/sh
# .git/hooks/commit-msg

commit_msg=$(cat $1)

# Enforce conventional commits
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore):"; then
    echo "Error: Commit message must start with type (feat, fix, docs, etc.)"
    exit 1
fi
```

**✅ Weeks 7-8 Checkpoint**: Master advanced Git features

---

## Level 4: Expert (Weeks 9-12)

### Week 9-10: Git Internals

#### 9.1 Understanding Git Objects

**Four Object Types:**
1. **Blob**: File contents
2. **Tree**: Directory structure
3. **Commit**: Snapshot with metadata
4. **Tag**: Named reference to commit

**Exploring Objects:**
```bash
# View object type
git cat-file -t <hash>

# View object content
git cat-file -p <hash>

# View commit
git cat-file -p HEAD

# View tree
git ls-tree HEAD

# View blob (file content)
git cat-file -p <blob-hash>
```

**How Git Stores Data:**
```
Commit → Tree → Blob
         ├── Blob
         └── Tree → Blob
```

---

#### 9.2 Git Plumbing Commands

**Plumbing vs Porcelain:**
- **Porcelain**: User-friendly (commit, push, pull)
- **Plumbing**: Low-level (hash-object, update-ref)

**Useful Plumbing Commands:**
```bash
# Hash an object
echo "test" | git hash-object --stdin
git hash-object filename.txt

# Create blob
echo "content" | git hash-object -w --stdin

# Update reference
git update-ref refs/heads/test <commit-hash>

# Show reference
git show-ref

# Symbolic references
git symbolic-ref HEAD
```

---

#### 9.3 Advanced Configuration

**Useful Configurations:**
```bash
# Aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

# Diff tools
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Merge tools
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Auto-correct typos
git config --global help.autocorrect 20

# Default push behavior
git config --global push.default current

# Rebase on pull
git config --global pull.rebase true

# Prune on fetch
git config --global fetch.prune true

# Show original in conflicts
git config --global merge.conflictstyle diff3
```

**✅ Weeks 9-10 Checkpoint**: Understand Git internals

---

### Week 11-12: Team Workflows & Best Practices

#### 11.1 Git Workflows

**1. Feature Branch Workflow:**
```bash
# Create feature branch
git checkout -b feature/user-auth

# Work on feature
git add .
git commit -m "Add user authentication"

# Push feature branch
git push -u origin feature/user-auth

# Create pull request
# After review and approval, merge
```

**2. Git Flow:**
```
main (production)
├── develop (integration)
│   ├── feature/login
│   ├── feature/signup
│   └── feature/profile
├── release/v1.0
└── hotfix/critical-bug
```

```bash
# Initialize Git Flow
git flow init

# Start feature
git flow feature start login

# Finish feature
git flow feature finish login

# Start release
git flow release start 1.0.0

# Finish release
git flow release finish 1.0.0

# Start hotfix
git flow hotfix start critical-bug

# Finish hotfix
git flow hotfix finish critical-bug
```

**3. GitHub Flow (Simplified):**
```bash
# 1. Create branch
git checkout -b fix-bug

# 2. Make changes and commit
git commit -m "Fix bug"

# 3. Push and create PR
git push origin fix-bug

# 4. Discuss and review
# 5. Merge to main
# 6. Deploy immediately
```

**4. Trunk-Based Development:**
```bash
# Short-lived branches (< 1 day)
git checkout -b short-fix

# Commit frequently
git commit -m "Part 1"
git commit -m "Part 2"

# Merge quickly to main
git checkout main
git merge short-fix
git push origin main
```

---

#### 11.2 Code Review Best Practices

**Creating Good Pull Requests:**
```bash
# 1. Update from main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/add-comments

# 3. Make focused changes
# Keep PR small and focused

# 4. Write good commit messages
git commit -m "feat: add comment system

Implements user comments with:
- Comment creation
- Comment editing
- Comment deletion
- Nested replies

Closes #123"

# 5. Push and create PR
git push -u origin feature/add-comments
```

**Reviewing Pull Requests:**
```bash
# Fetch PR
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Review changes
git diff main...pr-123

# Test locally
npm install
npm test

# Leave comments and approve/request changes
```

---

#### 11.3 Release Management

**Semantic Versioning:**
```
MAJOR.MINOR.PATCH
1.0.0

MAJOR: Breaking changes
MINOR: New features (backward compatible)
PATCH: Bug fixes
```

**Creating Releases:**
```bash
# 1. Update version
# Update package.json, version files

# 2. Create release branch
git checkout -b release/v2.0.0

# 3. Final testing and fixes
git commit -m "chore: bump version to 2.0.0"

# 4. Merge to main
git checkout main
git merge release/v2.0.0

# 5. Tag release
git tag -a v2.0.0 -m "Release version 2.0.0

- New feature X
- Bug fix Y
- Performance improvements"

# 6. Push with tags
git push origin main --tags

# 7. Merge back to develop
git checkout develop
git merge main
git push origin develop
```

**✅ Weeks 11-12 Checkpoint**: Master team workflows and best practices

---

## Git Workflows

### Choosing a Workflow

| Workflow | Team Size | Release Frequency | Complexity |
|----------|-----------|-------------------|------------|
| **Feature Branch** | Small | Continuous | Low |
| **Git Flow** | Medium-Large | Scheduled | High |
| **GitHub Flow** | Small-Medium | Continuous | Low |
| **Trunk-Based** | Any | Continuous | Low |

---

## Best Practices

### Commit Practices
✅ Commit often, push daily
✅ Write clear commit messages
✅ Keep commits focused and small
✅ Test before committing
✅ Don't commit sensitive data
✅ Use conventional commits

### Branch Practices
✅ Use descriptive branch names
✅ Keep branches short-lived
✅ Delete merged branches
✅ Protect main/production branches
✅ Use branch naming conventions

### Collaboration Practices
✅ Pull before starting work
✅ Create focused pull requests
✅ Review code thoroughly
✅ Discuss major changes
✅ Document decisions
✅ Keep history clean

---

## Resources

### Official Documentation
- **Git Documentation**: https://git-scm.com/doc
- **Pro Git Book**: https://git-scm.com/book (Free!)

### Interactive Learning
- **Learn Git Branching**: https://learngitbranching.js.org/
- **GitHub Skills**: https://skills.github.com/

### Practice
- **Git Exercises**: https://gitexercises.fracz.com/
- **Oh My Git!**: https://ohmygit.org/ (Game)

### Community
- **Stack Overflow**: [git] tag
- **GitHub Community**: https://github.community/

---

**Congratulations on completing the Git Learning Roadmap! 🎉**

Continue practicing, and remember: mastery comes with experience!


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

