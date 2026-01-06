# Git Troubleshooting Guide

Common Git problems and their solutions.

---

## Table of Contents
1. [Commit Issues](#commit-issues)
2. [Branch Problems](#branch-problems)
3. [Merge Conflicts](#merge-conflicts)
4. [Remote Issues](#remote-issues)
5. [Repository Problems](#repository-problems)
6. [History Issues](#history-issues)
7. [Performance Problems](#performance-problems)
8. [Security & Recovery](#security--recovery)

---

## Commit Issues

### Issue 1: Undo Last Commit (Keep Changes)
**Problem**: Made a commit but want to undo it and keep the changes

**Solution:**
```bash
# Undo commit, keep changes staged
git reset --soft HEAD~1

# Undo commit, keep changes unstaged
git reset HEAD~1
git reset --mixed HEAD~1  # Same as above
```

**When to use:**
- Want to modify commit message
- Forgot to add files
- Want to split commit into multiple commits

---

### Issue 2: Undo Last Commit (Discard Changes)
**Problem**: Made a commit with wrong changes, want to completely remove it

**Solution:**
```bash
# ⚠️ WARNING: This discards all changes
git reset --hard HEAD~1

# If already pushed
git push origin main --force-with-lease
```

**Prevention:**
- Always review changes before committing
- Use `git diff --staged` before commit
- Consider using `git stash` instead of discarding

---

### Issue 3: Change Last Commit Message
**Problem**: Typo in last commit message

**Solution:**
```bash
# Not yet pushed
git commit --amend -m "New commit message"

# Already pushed
git commit --amend -m "New commit message"
git push origin main --force-with-lease
```

**Warning**: Don't amend public commits that others may have pulled!

---

### Issue 4: Add Files to Last Commit
**Problem**: Forgot to include files in last commit

**Solution:**
```bash
# Add forgotten files
git add forgotten-file.txt

# Amend commit
git commit --amend --no-edit

# If already pushed
git push origin main --force-with-lease
```

---

### Issue 5: Commit to Wrong Branch
**Problem**: Made commits on main instead of feature branch

**Solution:**
```bash
# Create new branch from current position
git branch feature-branch

# Reset main to before commits
git reset --hard origin/main  # If main was synced
# OR
git reset --hard HEAD~3  # Go back 3 commits

# Switch to feature branch
git checkout feature-branch
# Commits are now on feature branch!
```

---

### Issue 6: Fix Wrong Commit Author
**Problem**: Committed with wrong author name/email

**Solution:**
```bash
# Fix last commit only
git commit --amend --author="Correct Name <correct@email.com>"

# Fix multiple commits
git rebase -i HEAD~3
# Mark commits to edit with 'edit'
# For each commit:
git commit --amend --author="Correct Name <correct@email.com>"
git rebase --continue
```

---

## Branch Problems

### Issue 7: Accidentally Deleted Branch
**Problem**: Deleted branch with `git branch -D` but it had unmerged work

**Solution:**
```bash
# Find last commit of deleted branch
git reflog

# Look for entries like:
# abc123 HEAD@{2}: checkout: moving from deleted-branch to main

# Recreate branch
git checkout -b recovered-branch abc123

# Verify it has your work
git log --oneline
```

**Prevention**: Use `git branch -d` instead of `-D` (safe delete)

---

### Issue 8: Can't Delete Branch
**Problem**: Getting error "branch is not fully merged"

**Symptoms:**
```bash
git branch -d feature-branch
# error: The branch 'feature-branch' is not fully merged.
```

**Solutions:**
```bash
# Option 1: Merge first
git checkout main
git merge feature-branch
git branch -d feature-branch

# Option 2: Force delete (if sure)
git branch -D feature-branch

# Option 3: Check if already merged
git branch --merged  # Shows merged branches
git branch --no-merged  # Shows unmerged branches
```

---

### Issue 9: Switched to Wrong Branch
**Problem**: Started working on wrong branch

**Solution:**
```bash
# If changes not committed
git stash
git checkout correct-branch
git stash pop

# If changes committed
# Get commit hash
git log -1

# Switch to correct branch
git checkout correct-branch

# Cherry-pick the commit
git cherry-pick <commit-hash>

# Go back and remove from wrong branch
git checkout wrong-branch
git reset --hard HEAD~1
```

---

### Issue 10: Branch Name Typo
**Problem**: Created branch with typo in name

**Solution:**
```bash
# Rename local branch
git branch -m old-naem new-name

# If already pushed
git push origin :old-naem  # Delete old
git push origin new-name  # Push new
git push origin -u new-name  # Set upstream
```

---

## Merge Conflicts

### Issue 11: Merge Conflict
**Problem**: Conflicts during merge

**Symptoms:**
```bash
git merge feature-branch
# CONFLICT (content): Merge conflict in file.txt
```

**Solution:**
```bash
# 1. Check conflicted files
git status

# 2. Open conflicted file
# You'll see:
<<<<<<< HEAD
Current branch content
=======
Merging branch content
>>>>>>> feature-branch

# 3. Edit file to resolve conflict
# Remove conflict markers and keep desired content

# 4. Stage resolved file
git add file.txt

# 5. Complete merge
git commit

# OR abort merge
git merge --abort
```

**Tips:**
- Use merge tool: `git mergetool`
- Configure diff3: `git config merge.conflictstyle diff3`
- Take your time, review carefully

---

### Issue 12: Too Many Conflicts
**Problem**: Merge has dozens of conflicts

**Solutions:**
```bash
# Option 1: Abort and rebase first
git merge --abort
git rebase feature-branch
# Resolve conflicts incrementally

# Option 2: Use merge strategy
git merge -X ours feature-branch  # Favor current branch
git merge -X theirs feature-branch  # Favor merging branch

# Option 3: Start fresh
git merge --abort
# Recreate changes manually or use different approach
```

---

### Issue 13: Accidentally Committed Conflict Markers
**Problem**: Committed files with `<<<<<<<` markers still in them

**Solution:**
```bash
# Find conflict markers
grep -r "<<<<<<< HEAD" .
grep -r "=======" .
grep -r ">>>>>>>" .

# Fix files
# Remove markers manually

# Amend last commit
git add fixed-file.txt
git commit --amend
```

**Prevention**: Use pre-commit hook to detect markers

---

### Issue 14: Want to Undo Merge
**Problem**: Merged but want to undo it

**Solution:**
```bash
# Before push - reset
git reset --hard HEAD~1

# After push - revert
git revert -m 1 <merge-commit-hash>
# -m 1 means keep first parent (usually main branch)
```

---

## Remote Issues

### Issue 15: Push Rejected
**Problem**: `git push` rejected with "non-fast-forward"

**Symptoms:**
```bash
git push origin main
# error: failed to push some refs
# hint: Updates were rejected because the remote contains work
```

**Solution:**
```bash
# Option 1: Pull first (recommended)
git pull origin main
# Resolve any conflicts
git push origin main

# Option 2: Pull with rebase
git pull --rebase origin main
git push origin main

# Option 3: Force push (DANGEROUS!)
git push origin main --force
# Better: force with lease (safer)
git push origin main --force-with-lease
```

**When to force push:**
- ✅ On your personal feature branches
- ❌ Never on shared branches (main, develop)

---

### Issue 16: Pushed Wrong Branch
**Problem**: Accidentally pushed to wrong remote branch

**Solution:**
```bash
# Delete remote branch
git push origin --delete wrong-branch

# Push correct branch
git push origin correct-branch
```

---

### Issue 17: Can't Pull - Uncommitted Changes
**Problem**: `git pull` fails due to local changes

**Symptoms:**
```bash
git pull
# error: Your local changes to the following files would be overwritten
```

**Solutions:**
```bash
# Option 1: Stash changes
git stash
git pull
git stash pop

# Option 2: Commit changes
git add .
git commit -m "WIP"
git pull

# Option 3: Discard changes (if not needed)
git reset --hard
git pull
```

---

### Issue 18: Wrong Remote URL
**Problem**: Remote URL is incorrect

**Solution:**
```bash
# Check current remote
git remote -v

# Change URL
git remote set-url origin https://correct-url.git

# Verify
git remote -v
```

---

### Issue 19: Lost Connection to Remote
**Problem**: Can't connect to remote repository

**Symptoms:**
```bash
git push
# fatal: Could not read from remote repository
```

**Solutions:**
```bash
# Check remote exists
git remote -v

# Test connection
ping github.com

# Check SSH key (if using SSH)
ssh -T git@github.com

# Check credentials (HTTPS)
git credential-cache exit
git push  # Will ask for credentials

# Re-add remote if missing
git remote add origin <url>
```

---

## Repository Problems

### Issue 20: Repository Corrupted
**Problem**: Git database corrupted

**Symptoms:**
```bash
git status
# error: object file .git/objects/... is empty
```

**Solutions:**
```bash
# Option 1: Try to recover
git fsck --full

# Option 2: Clean corrupted objects
find .git/objects/ -type f -empty | xargs rm
git fetch origin

# Option 3: Re-clone repository
cd ..
mv repo repo-backup
git clone <url> repo
cd repo
# Copy any uncommitted work from backup
```

---

### Issue 21: Huge Repository Size
**Problem**: .git folder is very large

**Solution:**
```bash
# Check repository size
du -sh .git

# Find large files in history
git rev-list --objects --all | \
  git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | \
  sed -n 's/^blob //p' | \
  sort -k2nr | \
  head -20

# Remove large file from history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/large-file' \
  --prune-empty --tag-name-filter cat -- --all

# Or use BFG Repo Cleaner (recommended)
java -jar bfg.jar --strip-blobs-bigger-than 10M

# Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Force push
git push origin --force --all
```

---

### Issue 22: Accidentally Committed Sensitive Data
**Problem**: Committed passwords, API keys, etc.

**Solution:**
```bash
# ⚠️ URGENT: Act immediately!

# 1. Change all exposed credentials first!

# 2. Remove from history
git filter-branch --force --index-filter \
  'git rm --cached --ignore-unmatch path/to/secret-file' \
  --prune-empty --tag-name-filter cat -- --all

# Or use BFG (easier)
java -jar bfg.jar --delete-files secret-file

# 3. Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 4. Force push
git push origin --force --all
git push origin --force --tags

# 5. Rotate all credentials!
```

**Prevention:**
- Use .gitignore before first commit
- Use environment variables
- Use secret management tools
- Enable pre-commit hooks

---

### Issue 23: Can't Initialize Repository
**Problem**: `git init` fails

**Solutions:**
```bash
# Check permissions
ls -la .git

# Remove and reinitialize
rm -rf .git
git init

# Check disk space
df -h
```

---

## History Issues

### Issue 24: Lost Commits
**Problem**: Commits seem to have disappeared

**Solution:**
```bash
# Check reflog
git reflog

# Find lost commit
git log --all --full-history --oneline | grep "search-term"

# Recover commit
git checkout <commit-hash>
git checkout -b recovered-branch

# Or cherry-pick
git cherry-pick <commit-hash>
```

**Common causes:**
- Hard reset
- Rebase gone wrong
- Branch deletion

---

### Issue 25: Detached HEAD State
**Problem**: In detached HEAD state after checkout

**Symptoms:**
```bash
git branch
# * (HEAD detached at abc123)
```

**Solutions:**
```bash
# Option 1: Create branch from here
git checkout -b new-branch

# Option 2: Go back to branch
git checkout main

# Option 3: Save work and return
git stash
git checkout main
```

---

### Issue 26: Need to Modify Old Commit
**Problem**: Made mistake in commit from 5 commits ago

**Solution:**
```bash
# Interactive rebase
git rebase -i HEAD~6

# In editor, change 'pick' to 'edit' for the commit
# Save and close

# Make changes
# Edit files...
git add fixed-file.txt

# Amend commit
git commit --amend

# Continue rebase
git rebase --continue

# If pushed, force push
git push origin branch --force-with-lease
```

---

### Issue 27: Want to Split Commit
**Problem**: Commit contains too many unrelated changes

**Solution:**
```bash
# Reset to before commit
git reset HEAD~1

# Stage and commit changes separately
git add file1.txt
git commit -m "Change 1"

git add file2.txt
git commit -m "Change 2"

# Or use interactive staging
git add -p file.txt  # Stage hunks separately
```

---

## Performance Problems

### Issue 28: Git Commands Are Slow
**Problem**: All Git operations taking long time

**Solutions:**
```bash
# Check repository size
git count-objects -vH

# Optimize repository
git gc --aggressive

# Enable filesystem monitor
git config core.fsmonitor true
git config core.untrackedcache true

# Shallow clone for faster cloning
git clone --depth 1 <url>

# Sparse checkout for large repos
git sparse-checkout init
git sparse-checkout set dir1 dir2
```

---

### Issue 29: git status Is Slow
**Problem**: `git status` takes long time

**Solutions:**
```bash
# Enable caching
git config core.untrackedCache true
git config core.fsmonitor true

# Limit depth of untracked directory search
git config status.showUntrackedFiles normal

# Or only show tracked files
git config status.showUntrackedFiles no
```

---

## Security & Recovery

### Issue 30: Forgot to Add .gitignore
**Problem**: Already committed files that should be ignored

**Solution:**
```bash
# Create .gitignore
echo "*.log" >> .gitignore
echo "node_modules/" >> .gitignore

# Remove from index but keep files
git rm -r --cached .
git add .
git commit -m "Start using .gitignore"

# Remove from history (if sensitive)
git filter-branch --index-filter \
  'git rm -r --cached --ignore-unmatch node_modules/' HEAD
```

---

## Quick Reference

### Emergency Commands
```bash
# Undo anything (if local)
git reflog  # Find where you want to go
git reset --hard <commit>

# Abort operations
git merge --abort
git rebase --abort
git cherry-pick --abort

# Save work immediately
git stash
git commit -am "WIP"

# Start fresh (DANGER!)
git reset --hard origin/main
```

### Safety Checks
```bash
# Before force push
git push --dry-run --force

# Before reset
git reflog  # Note current position

# Before cleaning
git clean -n  # Dry run
git clean -fd  # Actually clean
```

---

## Prevention Tips

### Best Practices
✅ Commit often, push daily
✅ Pull before starting work
✅ Use .gitignore from start
✅ Review changes before committing
✅ Write clear commit messages
✅ Use branches for features
✅ Test before merging
✅ Keep repository clean

### Safety Measures
✅ Enable branch protection
✅ Require pull request reviews
✅ Use pre-commit hooks
✅ Regular backups
✅ Never force push to main
✅ Use `--force-with-lease`
✅ Document your workflow

---

## Getting More Help

```bash
# Command help
git help <command>
git <command> --help

# Search git help
git help -a  # All commands
git help -g  # Guides

# Check git version
git --version
```

### Resources
- Git Documentation: https://git-scm.com/doc
- Stack Overflow: [git] tag
- GitHub Community: https://github.community/

---

**Remember**: Almost everything in Git can be undone. Don't panic, use reflog! 🚀


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

