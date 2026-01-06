# Git Interview Questions
## From Basic to Advanced

Complete interview preparation covering fundamentals to expert-level Git knowledge.

---

## Table of Contents
1. [Basic Questions](#basic-questions)
2. [Branching & Merging](#branching--merging)
3. [Remote Repositories](#remote-repositories)
4. [Advanced Operations](#advanced-operations)
5. [Git Internals](#git-internals)
6. [Workflows & Best Practices](#workflows--best-practices)
7. [Troubleshooting Scenarios](#troubleshooting-scenarios)
8. [Behavioral Questions](#behavioral-questions)

---

## Basic Questions

### Q1: What is Git and why use it?
**Expected Answer:**

Git is a distributed version control system (DVCS) that tracks changes in source code during software development.

**Key Benefits:**
- **Distributed**: Every developer has full repository history
- **Branching**: Lightweight branches for parallel development
- **Speed**: Most operations are local and fast
- **Data Integrity**: Everything is checksummed (SHA-1)
- **Collaboration**: Easy merging and conflict resolution
- **History**: Complete project history with ability to rollback
- **Open Source**: Free and widely adopted

**Follow-up**: What's the difference between Git and GitHub?
**Answer**: Git is version control software. GitHub is a hosting service for Git repositories (like GitLab, Bitbucket).

---

### Q2: Explain Git's three-state architecture
**Expected Answer:**

Git has three main states for files:

**1. Working Directory (Modified)**
- Your actual project files
- Where you make changes

**2. Staging Area/Index (Staged)**
- Snapshot of changes to be committed
- Files ready for next commit

**3. Repository (Committed)**
- Git database (.git directory)
- Permanent snapshot of project

**Workflow:**
```
Working Directory → (git add) → Staging Area → (git commit) → Repository
```

**Example:**
```bash
# Modify file (working directory)
echo "New content" >> file.txt

# Stage file (staging area)
git add file.txt

# Commit file (repository)
git commit -m "Add new content"
```

---

### Q3: What is the difference between git pull and git fetch?
**Expected Answer:**

**git fetch:**
- Downloads commits, files, refs from remote
- Updates remote-tracking branches
- Does NOT merge into local branches
- Safe operation (no conflicts)

```bash
git fetch origin
# Downloads updates but doesn't modify working directory
```

**git pull:**
- `git fetch` + `git merge`
- Downloads AND merges into current branch
- Can cause merge conflicts
- Less safe than fetch

```bash
git pull origin main
# Equivalent to:
git fetch origin
git merge origin/main
```

**When to use:**
- **fetch**: Review changes before merging, safer workflow
- **pull**: Quick update when confident no conflicts

**Follow-up**: What is `git pull --rebase`?
**Answer**: Replays local commits on top of fetched commits instead of creating merge commit. Creates linear history.

---

### Q4: Explain the difference between git merge and git rebase
**Expected Answer:**

**git merge:**
- Creates new merge commit
- Preserves complete history
- Non-destructive operation
- Creates branching history

```
main:    A---B---C---D
              \       \
feature:       E---F---M (merge commit)
```

**git rebase:**
- Rewrites commit history
- Replays commits on new base
- Creates linear history
- Destructive operation (changes commit hashes)

```
main:    A---B---C---D
                      \
feature:               E'---F' (rebased commits)
```

**When to use:**
- **Merge**: Public branches, preserving history, team collaboration
- **Rebase**: Personal branches, clean history, before pull request

**Golden Rule**: Never rebase public branches!

---

### Q5: What is a Git commit hash and how is it generated?
**Expected Answer:**

**Commit Hash** is a 40-character SHA-1 checksum that uniquely identifies each commit.

**Generated from:**
- Commit metadata (author, date, message)
- File contents (tree object)
- Parent commit hash(es)

**Example:**
```bash
git log --oneline
# abc1234 Add feature
# def5678 Fix bug

# Full hash:
git log
# commit abc1234567890abcdef1234567890abcdef123456
```

**Properties:**
- **Unique**: Virtually impossible to have collisions
- **Content-addressable**: Same content = same hash
- **Immutable**: Changing commit changes hash
- **Distributed**: Same commit has same hash everywhere

**Why important:**
- Ensures data integrity
- Detects corruption
- Uniquely identifies commits
- Enables distributed nature

---

### Q6: What is HEAD in Git?
**Expected Answer:**

**HEAD** is a pointer to the current branch reference, which points to the last commit on that branch.

**States:**

**1. Normal (Attached HEAD)**
```bash
git branch
# * main
# HEAD points to 'main' branch
```

**2. Detached HEAD**
```bash
git checkout abc1234
# HEAD points directly to commit, not branch
```

**References:**
- `HEAD`: Current commit
- `HEAD~1`: Parent of current commit (1 commit back)
- `HEAD~2`: Grandparent (2 commits back)
- `HEAD^`: Parent commit (same as HEAD~1)
- `HEAD^^`: Grandparent (same as HEAD~2)

**Example:**
```bash
# Show what HEAD points to
cat .git/HEAD
# ref: refs/heads/main

# Show HEAD commit
git show HEAD

# Show previous commit
git show HEAD~1
```

---

### Q7: What is the difference between git reset, git revert, and git checkout?
**Expected Answer:**

**git reset** - Move branch pointer
- Changes commit history
- Three modes: `--soft`, `--mixed`, `--hard`
- Dangerous for shared branches

```bash
git reset --soft HEAD~1   # Keep changes staged
git reset --mixed HEAD~1  # Keep changes unstaged (default)
git reset --hard HEAD~1   # Discard changes
```

**git revert** - Create new commit that undoes changes
- Doesn't change history
- Safe for shared branches
- Creates reverse commit

```bash
git revert abc1234
# Creates new commit that undoes abc1234
```

**git checkout** - Switch branches or restore files
- Move HEAD to different branch/commit
- Can restore files from commits
- Doesn't change history

```bash
git checkout main          # Switch to main branch
git checkout abc1234       # Detached HEAD state
git checkout -- file.txt   # Restore file from index
```

**Summary:**
- **reset**: Move branch backward (rewrite history)
- **revert**: Undo commit forward (safe for shared)
- **checkout**: Move HEAD or restore files

---

### Q8: What is .gitignore and why is it important?
**Expected Answer:**

**.gitignore** specifies intentionally untracked files that Git should ignore.

**Common patterns:**
```gitignore
# Dependencies
node_modules/
vendor/

# Environment files
.env
.env.local

# Build output
dist/
build/
*.min.js

# IDE
.vscode/
.idea/
*.swp

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Sensitive data
secrets/
*.key
*.pem
```

**Why important:**
- Prevent committing sensitive data (passwords, keys)
- Keep repository clean
- Reduce repository size
- Avoid committing generated files
- Platform-specific files

**Best practices:**
- Add .gitignore before first commit
- Use templates for language/framework
- Add to repository (commit .gitignore)
- Global gitignore for personal preferences

```bash
# Create global gitignore
git config --global core.excludesfile ~/.gitignore_global
```

---

### Q9: What are Git hooks?
**Expected Answer:**

**Git hooks** are scripts that run automatically at certain points in Git workflow.

**Common hooks:**

**Client-side:**
- `pre-commit`: Before commit (lint, test)
- `prepare-commit-msg`: Before commit message editor
- `commit-msg`: After commit message (validate format)
- `post-commit`: After commit (notifications)
- `pre-push`: Before push (integration tests)
- `post-checkout`: After checkout
- `post-merge`: After merge

**Server-side:**
- `pre-receive`: Before push accepted
- `update`: Before branch updated
- `post-receive`: After push completed

**Example pre-commit hook:**
```bash
#!/bin/sh
# .git/hooks/pre-commit

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed! Fix errors and try again."
    exit 1
fi

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed! Fix tests and try again."
    exit 1
fi

exit 0
```

**Use cases:**
- Code quality enforcement
- Testing before commit/push
- Message format validation
- Automated deployments
- Notifications

---

### Q10: What is the difference between git stash and git commit?
**Expected Answer:**

**git stash:**
- Temporarily saves uncommitted changes
- Doesn't create permanent commit
- Can apply multiple times
- Stack-based (LIFO)
- Use for: Switching context quickly

```bash
git stash                # Save changes
git stash list           # View stashes
git stash apply          # Apply stash
git stash pop            # Apply and remove
git stash drop           # Delete stash
```

**git commit:**
- Permanently saves changes
- Creates commit in history
- Part of project timeline
- Has author, message, timestamp
- Use for: Actual project progress

```bash
git add .
git commit -m "Feature implementation"
```

**When to use stash:**
- Need to switch branches urgently
- Not ready to commit yet
- Want to try different approach
- Pull latest changes first

**When to commit:**
- Completed logical unit of work
- Ready to share changes
- Want permanent history
- Follow commit message conventions

---

## Branching & Merging

### Q11: Explain Git branching strategies
**Expected Answer:**

**1. Git Flow** (Complex projects)
```
main (production)
├── develop (integration)
│   ├── feature/login
│   ├── feature/signup
│   └── feature/profile
├── release/v1.0
└── hotfix/critical-bug
```

**Branches:**
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `release/*`: Release preparation
- `hotfix/*`: Emergency fixes

**2. GitHub Flow** (Continuous deployment)
```
main
├── feature-1
├── fix-2
└── feature-3
```

**Process:**
1. Create branch from main
2. Make changes
3. Create pull request
4. Review and test
5. Merge to main
6. Deploy immediately

**3. Trunk-Based Development** (Fast-paced)
```
main (trunk)
├── short-fix-1 (< 1 day)
└── short-feature-2 (< 1 day)
```

**Key points:**
- Short-lived branches
- Frequent integration
- Feature flags for incomplete features
- High automation required

**4. Feature Branch Workflow** (Simple)
```
main
├── feature/user-auth
├── feature/payment
└── bugfix/login-error
```

**Choose based on:**
- Team size
- Release frequency
- Deployment process
- Team experience

---

### Q12: How do you resolve merge conflicts?
**Expected Answer:**

**Step-by-step process:**

**1. Identify conflict:**
```bash
git merge feature-branch
# CONFLICT (content): Merge conflict in file.txt
```

**2. Check conflicted files:**
```bash
git status
# Unmerged paths:
#   both modified:   file.txt
```

**3. Open conflicted file:**
```
<<<<<<< HEAD
Current branch content
=======
Merging branch content
>>>>>>> feature-branch
```

**4. Resolve conflict:**
Options:
- Keep current (HEAD)
- Keep incoming (feature-branch)
- Keep both
- Write new solution

**5. Remove conflict markers:**
```
Final resolved content
```

**6. Stage resolved file:**
```bash
git add file.txt
```

**7. Complete merge:**
```bash
git commit
# OR if all resolved:
git merge --continue
```

**Tools:**
```bash
# Use merge tool
git mergetool

# Configure diff3 style (shows original)
git config merge.conflictstyle diff3
```

**Best practices:**
- Pull frequently to minimize conflicts
- Communicate with team
- Test after resolving
- Use merge tools for complex conflicts
- Understand the code before resolving

---

### Q13: What is fast-forward merge?
**Expected Answer:**

**Fast-forward merge** occurs when target branch has no new commits since feature branch diverged.

**Example:**
```
Before merge:
main:    A---B---C
              \
feature:       D---E

After fast-forward:
main:    A---B---C---D---E (main moves forward)
```

**Characteristics:**
- No merge commit created
- Linear history
- Simple and clean
- Only possible with linear history

**Force merge commit (no fast-forward):**
```bash
git merge --no-ff feature-branch

Result:
main:    A---B---C-------M
              \         /
feature:       D---E---
```

**When to use:**
- `--ff` (default): Clean linear history for simple features
- `--no-ff`: Always create merge commit for tracking feature integration

**Configuration:**
```bash
# Disable fast-forward by default
git config --global merge.ff false
```

---

### Q14: What is cherry-pick and when to use it?
**Expected Answer:**

**Cherry-pick** applies specific commits from one branch to another.

**Basic usage:**
```bash
# Pick single commit
git cherry-pick abc1234

# Pick multiple commits
git cherry-pick abc1234 def5678

# Pick range (exclusive start)
git cherry-pick abc1234..def5678
```

**Use cases:**

**1. Bug fix to multiple versions:**
```bash
# Fix in develop
git checkout develop
git commit -m "Fix critical bug"

# Apply to main
git checkout main
git cherry-pick abc1234
```

**2. Specific feature from branch:**
```bash
# Take only one commit from feature branch
git cherry-pick def5678
```

**3. Undo accidental commit on wrong branch:**
```bash
git checkout correct-branch
git cherry-pick abc1234
git checkout wrong-branch
git reset --hard HEAD~1
```

**Handling conflicts:**
```bash
git cherry-pick abc1234
# CONFLICT!

# Resolve conflict
git add resolved-file.txt
git cherry-pick --continue

# Or abort
git cherry-pick --abort
```

**Caution:**
- Creates new commit (different hash)
- Can cause confusion if overused
- Prefer merging for related commits

---

### Q15: Explain rebase and when to use it
**Expected Answer:**

**Rebase** replays commits from current branch onto another branch.

**Basic rebase:**
```bash
git checkout feature-branch
git rebase main

# Or in one command:
git rebase main feature-branch
```

**Result:**
```
Before:
main:    A---B---C---D
              \
feature:       E---F---G

After:
main:    A---B---C---D
                      \
feature:               E'---F'---G' (new commits)
```

**Interactive rebase:**
```bash
git rebase -i HEAD~3
```

**Options:**
- `pick`: Use commit
- `reword`: Change commit message
- `edit`: Amend commit
- `squash`: Combine with previous
- `fixup`: Combine, discard message
- `drop`: Remove commit

**When to use rebase:**
✅ Clean up local commits before push
✅ Update feature branch with main
✅ Create linear history
✅ Squash fixup commits

**When NOT to rebase:**
❌ Public/shared branches
❌ After others have pulled your branch
❌ Commits pushed to remote

**Golden rule:** Never rebase commits that have been pushed to public branches!

**Safe rebase:**
```bash
# Update local feature branch
git checkout feature
git pull origin main --rebase
git push origin feature --force-with-lease
```

---

## Remote Repositories

### Q16: What is the difference between origin and upstream?
**Expected Answer:**

**origin:**
- Default name for cloned repository
- Usually your fork or direct remote
- Where you push changes

```bash
git remote add origin https://github.com/YOUR-USERNAME/repo.git
```

**upstream:**
- Original repository (when forked)
- Where you pull updates from
- Read-only for contributors

```bash
git remote add upstream https://github.com/ORIGINAL-OWNER/repo.git
```

**Fork workflow:**
```bash
# 1. Fork on GitHub

# 2. Clone your fork
git clone https://github.com/YOUR-USERNAME/repo.git

# 3. Add upstream
git remote add upstream https://github.com/ORIGINAL-OWNER/repo.git

# 4. Fetch from upstream
git fetch upstream

# 5. Merge upstream changes
git checkout main
git merge upstream/main

# 6. Push to your fork
git push origin main

# 7. Create feature branch
git checkout -b feature-branch

# 8. Push feature to your fork
git push origin feature-branch

# 9. Create Pull Request to upstream
```

---

### Q17: What is git reflog and when is it useful?
**Expected Answer:**

**reflog** records all references updates (HEAD movements).

**View reflog:**
```bash
git reflog

# Output:
abc1234 HEAD@{0}: commit: Add feature
def5678 HEAD@{1}: checkout: moving from main to feature
ghi9012 HEAD@{2}: reset: moving to HEAD~1
```

**Use cases:**

**1. Recover deleted commits:**
```bash
# Accidentally reset
git reset --hard HEAD~3

# Find commits
git reflog

# Recover
git checkout abc1234
git checkout -b recovered
```

**2. Recover deleted branch:**
```bash
# Deleted branch
git branch -D feature

# Find last commit
git reflog

# Recreate branch
git checkout -b feature abc1234
```

**3. Undo rebase:**
```bash
# Bad rebase
git rebase main

# Find pre-rebase state
git reflog

# Reset to before rebase
git reset --hard HEAD@{1}
```

**4. See all actions:**
```bash
# View detailed reflog
git reflog show --all

# Reflog for specific branch
git reflog show feature-branch
```

**Important:**
- Reflog is local (not pushed)
- Expires after 90 days (default)
- Last resort for recovery

---

### Q18: Explain git fetch --prune
**Expected Answer:**

**--prune** removes remote-tracking references that no longer exist on remote.

**Problem:**
```bash
# Branch deleted on remote
# But still shows locally:
git branch -r
# origin/deleted-branch  # Shouldn't exist!
```

**Solution:**
```bash
# Fetch and prune
git fetch --prune

# Or prune separately
git remote prune origin

# Verify
git branch -r
# origin/deleted-branch is gone
```

**Auto-prune configuration:**
```bash
# Always prune on fetch
git config --global fetch.prune true

# Now every fetch prunes automatically
git fetch
```

**Why important:**
- Keeps local remote refs clean
- Prevents confusion about branch status
- Reduces clutter in branch listings
- Good housekeeping practice

---

## Advanced Operations

### Q19: What is git bisect and how to use it?
**Expected Answer:**

**git bisect** uses binary search to find commit that introduced a bug.

**Process:**
```bash
# 1. Start bisect
git bisect start

# 2. Mark current as bad
git bisect bad

# 3. Mark known good commit
git bisect good v1.0

# 4. Git checks out middle commit
# Test for bug...

# 5. Mark result
git bisect bad  # Bug exists
# OR
git bisect good  # No bug

# 6. Repeat until Git finds first bad commit
# Git will say: "abc1234 is the first bad commit"

# 7. End bisect
git bisect reset
```

**Automated bisect:**
```bash
# Start bisect
git bisect start HEAD v1.0

# Run automated test
git bisect run npm test

# Git automatically tests each commit
# Shows first failing commit

# End bisect
git bisect reset
```

**Real example:**
```bash
# Bug somewhere in last 100 commits
git bisect start HEAD HEAD~100

# Git checks ~7 commits (log2(100))
# Much faster than checking all 100!
```

**Benefits:**
- Efficient (logarithmic time)
- Automate with tests
- Works with any criteria
- Finds exact commit

---

### Q20: Explain git submodules
**Expected Answer:**

**Submodules** allow you to keep a Git repository as a subdirectory of another repository.

**Add submodule:**
```bash
git submodule add https://github.com/user/library.git libs/library
git commit -m "Add library submodule"
```

**Clone with submodules:**
```bash
# Option 1: After clone
git clone https://github.com/user/main-repo.git
cd main-repo
git submodule init
git submodule update

# Option 2: Clone with submodules
git clone --recursive https://github.com/user/main-repo.git
```

**Update submodule:**
```bash
# Update to latest
cd libs/library
git pull origin main
cd ../..
git add libs/library
git commit -m "Update library submodule"
```

**Submodule commands:**
```bash
# Update all submodules
git submodule update --remote

# Initialize submodules
git submodule init

# Status of submodules
git submodule status

# Remove submodule
git submodule deinit libs/library
git rm libs/library
rm -rf .git/modules/libs/library
```

**Use cases:**
- External libraries
- Shared components
- Vendor dependencies

**Alternatives:**
- Subtrees (simpler)
- Package managers (npm, pip)
- Monorepos

---

## Git Internals

### Q21: Explain Git's internal object model
**Expected Answer:**

Git stores data as four types of objects:

**1. Blob (Binary Large Object)**
- File contents
- No filename, just content
- SHA-1 hash of content

**2. Tree**
- Directory structure
- Points to blobs and other trees
- Contains filenames and permissions

**3. Commit**
- Snapshot of tree
- Parent commit(s)
- Author, committer, message
- Timestamp

**4. Tag**
- Named reference to commit
- Annotated tags are objects
- Contains tagger info

**Relationships:**
```
Commit
  ├── tree (root)
  │   ├── blob (file1.txt)
  │   ├── blob (file2.txt)
  │   └── tree (subdirectory)
  │       └── blob (file3.txt)
  └── parent commit(s)
```

**Exploring objects:**
```bash
# Show object type
git cat-file -t abc1234

# Show object content
git cat-file -p abc1234

# Show commit
git cat-file -p HEAD

# Show tree
git ls-tree HEAD

# Show blob
git cat-file -p <blob-hash>
```

**Content-addressable:**
- Same content = same hash
- Changes propagate up tree
- Ensures data integrity

---

### Q22: What is the difference between .git directory and working directory?
**Expected Answer:**

**.git directory** (Repository)
- Git's database
- All commits, branches, history
- Objects, refs, config
- Hidden directory

**Contents:**
```
.git/
├── HEAD              # Current branch pointer
├── config            # Repository config
├── description       # Repository description
├── hooks/            # Git hooks
├── index             # Staging area
├── objects/          # All Git objects
│   ├── pack/         # Packed objects
│   └── info/         # Object info
├── refs/             # Branch and tag references
│   ├── heads/        # Local branches
│   ├── remotes/      # Remote branches
│   └── tags/         # Tags
└── logs/             # Reference logs (reflog)
```

**Working Directory**
- Your actual files
- What you see and edit
- Current checkout of specific commit
- Can be modified

**Relationship:**
```
Working Directory ← checkout ← .git/objects
Working Directory → add → .git/index → commit → .git/objects
```

**Example:**
```bash
# Working directory
ls
# file1.txt, file2.txt, subdirectory/

# Git directory
ls .git
# HEAD, config, objects/, refs/, ...

# Staging area (part of .git)
git ls-files --stage
```

---

## Workflows & Best Practices

### Q23: Describe a good Git workflow for a team
**Expected Answer:**

**Recommended Workflow (GitHub Flow variant):**

**1. Main branch protection:**
```bash
# main always deployable
# Direct pushes disabled
# Require pull request reviews
```

**2. Feature development:**
```bash
# Update main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/user-auth

# Work and commit frequently
git add .
git commit -m "feat: add login endpoint"

# Keep updated with main
git fetch origin
git rebase origin/main

# Push feature branch
git push -u origin feature/user-auth
```

**3. Pull Request process:**
- Create PR on GitHub
- Add description and screenshots
- Request reviews
- Address feedback
- Ensure CI passes
- Squash commits if messy

**4. Merge and deploy:**
```bash
# After approval
# Merge via GitHub (squash or merge commit)

# Delete branch
git branch -d feature/user-auth
git push origin --delete feature/user-auth

# Update local main
git checkout main
git pull origin main
```

**Best Practices:**
✅ Small, focused pull requests
✅ Clear commit messages
✅ Regular pulls from main
✅ Review your own PR first
✅ Run tests before pushing
✅ Use conventional commits
✅ Tag releases
✅ Document workflow

**Commit Message Convention:**
```
type(scope): subject

body

footer

Types: feat, fix, docs, style, refactor, test, chore
```

---

### Q24: How do you handle hotfixes in production?
**Expected Answer:**

**Hotfix process:**

**1. Create hotfix branch from production:**
```bash
# From main (production)
git checkout main
git pull origin main

# Create hotfix branch
git checkout -b hotfix/critical-security-fix
```

**2. Fix the issue:**
```bash
# Make fix
git add .
git commit -m "fix: patch security vulnerability CVE-2024-1234"

# Test thoroughly
npm test
```

**3. Deploy directly:**
```bash
# Push hotfix
git push origin hotfix/critical-security-fix

# Fast-track review and merge to main
# Deploy main immediately
```

**4. Merge back to develop:**
```bash
# Ensure fix is in develop too
git checkout develop
git pull origin develop
git merge hotfix/critical-security-fix
git push origin develop
```

**5. Tag release:**
```bash
git checkout main
git tag -a v1.2.3 -m "Hotfix: Security patch"
git push origin v1.2.3
```

**6. Cleanup:**
```bash
git branch -d hotfix/critical-security-fix
git push origin --delete hotfix/critical-security-fix
```

**Git Flow hotfix:**
```bash
git flow hotfix start critical-fix
# Make changes
git flow hotfix finish critical-fix
```

**Key points:**
- Branch from production
- Fast-track review
- Test thoroughly
- Deploy immediately
- Merge to all active branches
- Tag release
- Document incident

---

## Troubleshooting Scenarios

### Q25: How do you recover from a bad rebase?
**Expected Answer:**

**Problem**: Rebase went wrong, need to undo it.

**Solution using reflog:**

```bash
# 1. View reflog
git reflog

# Output:
abc1234 HEAD@{0}: rebase finished: ...
def5678 HEAD@{1}: rebase: ...
ghi9012 HEAD@{2}: checkout: moving to feature  # BEFORE REBASE

# 2. Reset to before rebase
git reset --hard HEAD@{2}
# OR
git reset --hard ghi9012

# 3. Verify
git log --oneline
# Should be back to pre-rebase state
```

**Prevention:**
```bash
# Create backup branch before risky operations
git branch backup-feature
git rebase main

# If rebase goes wrong:
git reset --hard backup-feature
```

**If pushed:**
```bash
# After recovering locally
git push origin feature --force-with-lease

# Warn team if they pulled broken version
```

---

### Q26: Committed sensitive data (passwords, keys). What to do?
**Expected Answer:**

**CRITICAL: Act immediately!**

**Step 1: Change all exposed credentials**
- Rotate passwords
- Revoke API keys
- Generate new secrets
- **DO THIS FIRST!**

**Step 2: Remove from history**

**Option A: BFG Repo Cleaner (recommended)**
```bash
# Download BFG
# https://rtyley.github.io/bfg-repo-cleaner/

# Clone fresh copy
git clone --mirror https://github.com/user/repo.git

# Remove sensitive file
java -jar bfg.jar --delete-files secrets.env repo.git

# Or replace sensitive strings
java -jar bfg.jar --replace-text passwords.txt repo.git

# Clean up
cd repo.git
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push --force
```

**Option B: git filter-branch (built-in)**
```bash
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret.key" \
  --prune-empty --tag-name-filter cat -- --all

# Clean up
rm -rf .git/refs/original/
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push origin --force --all
git push origin --force --tags
```

**Step 3: Notify**
- Team must re-clone
- GitHub: Invalidate caches
- Notify security team

**Step 4: Prevention**
```bash
# Add to .gitignore
echo "secrets/" >> .gitignore
echo ".env" >> .gitignore

# Pre-commit hook to detect secrets
# Use tools like:
# - git-secrets
# - detect-secrets
# - gitleaks
```

---

## Behavioral Questions

### Q27: Describe a time you had to resolve a complex merge conflict
**Expected Answer Structure:**

**Situation:**
"On a project with 5 developers, we had feature branches that weren't synced with main for 2 weeks. When attempting to merge my authentication feature, I encountered conflicts in 15 files."

**Task:**
"I needed to safely merge my changes without breaking the application or losing others' work."

**Action:**
1. "First, I pulled latest main and created backup branch"
2. "Used `git mergetool` with VS Code for complex conflicts"
3. "For each conflict, I reviewed both versions and context"
4. "Consulted with teammates whose code conflicted"
5. "Ran full test suite after each resolution"
6. "Documented decisions in merge commit message"

**Result:**
"Successfully merged all changes, tests passed, and deployed without issues. Learned to sync branches more frequently and established team practice of daily rebases."

**Key points:**
- Show problem-solving approach
- Mention communication
- Highlight testing
- Include lesson learned

---

### Q28: How do you ensure code quality with Git?
**Expected Answer:**

**Pre-commit controls:**
```bash
# 1. Pre-commit hooks
- Linting (ESLint, Pylint)
- Formatting (Prettier, Black)
- Unit tests
- Security scanning
```

**Branching strategy:**
```bash
# 2. Protected main branch
- No direct pushes
- Require pull requests
- Require reviews (2+ approvers)
- Require CI pass
```

**Code review process:**
```bash
# 3. Pull request standards
- Clear description
- Linked issues
- Screenshots/demos
- Test coverage report
- Small, focused changes
```

**Automation:**
```bash
# 4. CI/CD pipeline
- Automated tests
- Code coverage checks
- Security scans
- Build verification
- Deploy previews
```

**Commit practices:**
```bash
# 5. Meaningful commits
- Conventional commits format
- Atomic commits
- Clear messages
- Reference issues
```

**Tools:**
- Husky (Git hooks)
- GitHub Actions/GitLab CI
- SonarQube
- Dependabot
- CodeClimate

---

## Summary

### Key Concepts to Remember

**Basic:**
- Three states (working, staging, repository)
- Common commands (add, commit, push, pull)
- .gitignore importance

**Intermediate:**
- Branching and merging
- Rebase vs merge
- Conflict resolution

**Advanced:**
- Git internals (objects, refs)
- Reflog for recovery
- Hooks and automation

**Best Practices:**
- Clear commit messages
- Frequent commits and pulls
- Never rebase public branches
- Protect sensitive data
- Code review culture

---

### Interview Tips

**Preparation:**
1. Practice commands daily
2. Understand concepts, not just commands
3. Have real examples ready
4. Know your project's workflow
5. Review common issues you've faced

**During Interview:**
1. Explain your thinking process
2. Mention trade-offs
3. Discuss team collaboration
4. Show problem-solving approach
5. Ask clarifying questions

**Follow-up Resources:**
- [Learning Roadmap](git-learning-roadmap.md)
- [Quick Reference](git-quick-reference.md)
- [Hands-On Exercises](git-hands-on-exercises.md)
- [Troubleshooting Guide](git-troubleshooting-guide.md)

**Good luck with your interview! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

