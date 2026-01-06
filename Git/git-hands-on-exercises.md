# Git Hands-On Exercises

Complete practical exercises to master Git from beginner to advanced level.

---

## Beginner Level Exercises

### Exercise 1: First Repository
**Objective**: Create your first Git repository and make commits

1. Create a new directory called `my-first-repo`
2. Initialize Git in this directory
3. Create a README.md file
4. Stage and commit the file
5. View the commit history

<details>
<summary>Solution</summary>

```bash
# Create directory
mkdir my-first-repo
cd my-first-repo

# Initialize Git
git init

# Create README
echo "# My First Repository" > README.md

# Stage file
git add README.md

# Commit
git commit -m "Initial commit"

# View history
git log
git log --oneline
```
</details>

---

### Exercise 2: Multiple Commits
**Objective**: Practice making multiple commits

1. Create three files: `file1.txt`, `file2.txt`, `file3.txt`
2. Add content to each file
3. Make separate commits for each file
4. View the commit history

<details>
<summary>Solution</summary>

```bash
# Create and commit file1
echo "Content 1" > file1.txt
git add file1.txt
git commit -m "Add file1"

# Create and commit file2
echo "Content 2" > file2.txt
git add file2.txt
git commit -m "Add file2"

# Create and commit file3
echo "Content 3" > file3.txt
git add file3.txt
git commit -m "Add file3"

# View history
git log --oneline
```
</details>

---

### Exercise 3: Viewing Changes
**Objective**: Practice viewing changes with git diff

1. Modify an existing file
2. View unstaged changes
3. Stage the changes
4. View staged changes
5. Commit the changes

<details>
<summary>Solution</summary>

```bash
# Modify file
echo "New line" >> file1.txt

# View unstaged changes
git diff

# Stage changes
git add file1.txt

# View staged changes
git diff --staged

# Commit
git commit -m "Update file1"
```
</details>

---

### Exercise 4: Undoing Changes
**Objective**: Practice undoing changes

1. Modify a file
2. Discard the changes (before staging)
3. Modify the file again and stage it
4. Unstage the file
5. Stage and commit the change

<details>
<summary>Solution</summary>

```bash
# Modify file
echo "Wrong content" >> file1.txt

# Discard changes
git restore file1.txt
# OR: git checkout -- file1.txt

# Modify and stage
echo "Right content" >> file1.txt
git add file1.txt

# Unstage
git restore --staged file1.txt
# OR: git reset HEAD file1.txt

# Stage and commit
git add file1.txt
git commit -m "Add right content"
```
</details>

---

### Exercise 5: Creating Branches
**Objective**: Create and switch branches

1. Create a new branch called `feature-branch`
2. Switch to the new branch
3. Create a file called `feature.txt`
4. Commit the file
5. Switch back to main
6. Verify that `feature.txt` doesn't exist on main

<details>
<summary>Solution</summary>

```bash
# Create branch
git branch feature-branch

# Switch to branch
git checkout feature-branch
# OR: git switch feature-branch

# Create and commit file
echo "Feature content" > feature.txt
git add feature.txt
git commit -m "Add feature"

# Switch to main
git checkout main

# Verify file doesn't exist
ls  # feature.txt not present

# Switch back to see it
git checkout feature-branch
ls  # feature.txt is there
```
</details>

---

### Exercise 6: Merging Branches
**Objective**: Merge a feature branch into main

1. On main, note the files present
2. Merge `feature-branch` into main
3. Verify the feature file is now on main
4. Delete the feature branch

<details>
<summary>Solution</summary>

```bash
# Switch to main
git checkout main

# Merge feature branch
git merge feature-branch

# Verify file exists
ls  # feature.txt now present

# View history
git log --oneline --graph

# Delete feature branch
git branch -d feature-branch
```
</details>

---

### Exercise 7: Merge Conflicts
**Objective**: Create and resolve a merge conflict

1. Create a file on main with some content
2. Create a branch and modify the same line
3. Go back to main and modify the same line differently
4. Try to merge and resolve the conflict

<details>
<summary>Solution</summary>

```bash
# On main: create file
echo "Hello from main" > conflict.txt
git add conflict.txt
git commit -m "Add conflict.txt"

# Create branch from earlier commit
git branch conflict-branch HEAD~1

# Switch to branch
git checkout conflict-branch

# Modify same file
echo "Hello from branch" > conflict.txt
git add conflict.txt
git commit -m "Update conflict.txt"

# Switch to main
git checkout main

# Try to merge
git merge conflict-branch
# CONFLICT!

# Open conflict.txt, you'll see:
# <<<<<<< HEAD
# Hello from main
# =======
# Hello from branch
# >>>>>>> conflict-branch

# Edit file to resolve
echo "Hello from both!" > conflict.txt

# Stage resolved file
git add conflict.txt

# Complete merge
git commit -m "Merge conflict-branch"

# View history
git log --oneline --graph
```
</details>

---

### Exercise 8: Git Log Exploration
**Objective**: Practice various git log commands

1. View commit history in different formats
2. Filter commits by author
3. View commits for specific file
4. Use graphical log view

<details>
<summary>Solution</summary>

```bash
# Basic log
git log

# One line per commit
git log --oneline

# With graph
git log --graph --oneline --all

# With stats
git log --stat

# Last 3 commits
git log -3

# Pretty format
git log --pretty=format:"%h - %an, %ar : %s"

# By author (use your name)
git log --author="Your Name"

# For specific file
git log -- conflict.txt

# With patches
git log -p
```
</details>

---

### Exercise 9: .gitignore
**Objective**: Create and use .gitignore

1. Create files that should be ignored (logs, temp files)
2. Create .gitignore file
3. Verify files are ignored
4. Commit .gitignore

<details>
<summary>Solution</summary>

```bash
# Create files
echo "logs" > app.log
echo "temp" > temp.txt
mkdir build
echo "output" > build/output.txt

# Check status (files are untracked)
git status

# Create .gitignore
cat > .gitignore << EOF
*.log
temp.txt
build/
EOF

# Check status (files now ignored)
git status

# Only .gitignore is untracked
git add .gitignore
git commit -m "Add .gitignore"

# Verify ignored files aren't shown
git status
```
</details>

---

### Exercise 10: Viewing Specific Commits
**Objective**: View details of specific commits

1. Get commit hash of a previous commit
2. View details of that commit
3. View file contents from that commit
4. Compare two commits

<details>
<summary>Solution</summary>

```bash
# Get commit hashes
git log --oneline

# Show specific commit
git show <commit-hash>

# Show file from commit
git show <commit-hash>:file1.txt

# Compare commits
git diff <commit1> <commit2>

# Show changes in last commit
git show HEAD

# Show file from 3 commits ago
git show HEAD~3:README.md
```
</details>

---

## Intermediate Level Exercises

### Exercise 11: Stashing Changes
**Objective**: Use git stash to save work in progress

1. Make changes to a file
2. Stash the changes
3. Make other changes and commit
4. Apply the stashed changes
5. Resolve any conflicts

<details>
<summary>Solution</summary>

```bash
# Make changes
echo "Work in progress" >> file1.txt

# Stash changes
git stash
git stash save "WIP on feature"

# Check status (working directory clean)
git status

# Make other changes
echo "Urgent fix" >> file2.txt
git add file2.txt
git commit -m "Urgent fix"

# List stashes
git stash list

# Apply stash
git stash apply
# OR: git stash pop (applies and removes)

# If conflicts, resolve them
# Then commit
git add file1.txt
git commit -m "Complete WIP feature"
```
</details>

---

### Exercise 12: Interactive Staging
**Objective**: Stage changes interactively

1. Make multiple changes to a file
2. Use interactive staging to stage only some changes
3. Commit the staged changes
4. Commit remaining changes separately

<details>
<summary>Solution</summary>

```bash
# Make multiple changes
cat > file1.txt << EOF
Line 1 - Feature A
Line 2 - Feature B
Line 3 - Feature C
EOF

# Interactive staging
git add -p file1.txt

# For each hunk, choose:
# y = stage this hunk
# n = don't stage
# s = split into smaller hunks
# q = quit

# Commit staged changes
git commit -m "Add Feature A"

# Stage and commit remaining
git add file1.txt
git commit -m "Add Features B and C"
```
</details>

---

### Exercise 13: Rebasing
**Objective**: Rebase a feature branch onto main

1. Create a feature branch and make commits
2. Make commits on main
3. Rebase feature branch onto main
4. Resolve any conflicts during rebase

<details>
<summary>Solution</summary>

```bash
# Create feature branch
git checkout -b feature-rebase
echo "Feature line 1" >> feature.txt
git add feature.txt
git commit -m "Feature commit 1"
echo "Feature line 2" >> feature.txt
git add feature.txt
git commit -m "Feature commit 2"

# Switch to main and make commits
git checkout main
echo "Main line 1" >> main.txt
git add main.txt
git commit -m "Main commit 1"

# Rebase feature onto main
git checkout feature-rebase
git rebase main

# If conflicts:
# 1. Resolve conflicts in files
# 2. git add <resolved-files>
# 3. git rebase --continue

# View history
git log --oneline --graph --all
```
</details>

---

### Exercise 14: Interactive Rebase
**Objective**: Clean up commit history with interactive rebase

1. Make several small commits
2. Use interactive rebase to squash commits
3. Reword commit messages
4. Reorder commits

<details>
<summary>Solution</summary>

```bash
# Make several commits
echo "Part 1" > story.txt
git add story.txt
git commit -m "Add part 1"

echo "Part 2" >> story.txt
git add story.txt
git commit -m "Add part 2"

echo "Part 3" >> story.txt
git add story.txt
git commit -m "Add part 3"

echo "Typo fix" >> story.txt
git add story.txt
git commit -m "Fix typo"

# Interactive rebase last 4 commits
git rebase -i HEAD~4

# In editor, change:
pick abc123 Add part 1
squash def456 Add part 2
squash ghi789 Add part 3
fixup jkl012 Fix typo

# Save and close
# Edit combined commit message
# Save and close

# View clean history
git log --oneline
```
</details>

---

### Exercise 15: Cherry-Picking
**Objective**: Apply specific commits from one branch to another

1. Create two feature branches
2. Make commits on both branches
3. Cherry-pick a commit from one branch to another

<details>
<summary>Solution</summary>

```bash
# Create first branch
git checkout -b feature-a
echo "Feature A" > feature-a.txt
git add feature-a.txt
git commit -m "Add feature A"

# Create second branch from main
git checkout main
git checkout -b feature-b
echo "Feature B" > feature-b.txt
git add feature-b.txt
git commit -m "Add feature B"

# Find commit hash from feature-a
git log feature-a --oneline

# Cherry-pick commit from feature-a
git cherry-pick <commit-hash-from-feature-a>

# Verify feature-a.txt is now in feature-b
ls

# View history
git log --oneline --graph --all
```
</details>

---

### Exercise 16: Tags
**Objective**: Create and manage tags

1. Create lightweight tag
2. Create annotated tag
3. Push tags to remote
4. Delete tags locally and remotely

<details>
<summary>Solution</summary>

```bash
# Create lightweight tag
git tag v1.0

# Create annotated tag
git tag -a v1.1 -m "Version 1.1 release"

# List tags
git tag
git tag -l "v1.*"

# Show tag details
git show v1.1

# Push tags
git push origin v1.1
git push origin --tags  # Push all tags

# Delete local tag
git tag -d v1.0

# Delete remote tag
git push origin --delete v1.0

# Checkout tag
git checkout v1.1
git checkout -b version-1.1 v1.1  # Create branch from tag
```
</details>

---

### Exercise 17: Remote Repository
**Objective**: Work with remote repositories

(Note: For this exercise, create a repository on GitHub/GitLab)

1. Create remote repository
2. Add remote to local repository
3. Push branches to remote
4. Clone repository to different location
5. Make changes and push
6. Pull changes in original repository

<details>
<summary>Solution</summary>

```bash
# Add remote
git remote add origin https://github.com/username/repo.git

# Push to remote
git push -u origin main

# Push other branches
git push origin feature-branch

# Clone to different location
cd /tmp
git clone https://github.com/username/repo.git repo-clone
cd repo-clone

# Make changes
echo "Changes from clone" >> file1.txt
git add file1.txt
git commit -m "Update from clone"
git push origin main

# Go back to original and pull
cd /path/to/original/repo
git pull origin main

# View changes
git log --oneline
```
</details>

---

### Exercise 18: Fetch vs Pull
**Objective**: Understand difference between fetch and pull

1. Make changes in one repository
2. Use fetch in another repository
3. View fetched changes without merging
4. Merge fetched changes
5. Compare with pull

<details>
<summary>Solution</summary>

```bash
# In first repository
echo "Update" >> README.md
git add README.md
git commit -m "Update README"
git push origin main

# In second repository
# Fetch changes (doesn't merge)
git fetch origin

# View remote changes
git log origin/main
git diff main origin/main

# Changes not yet in working directory
cat README.md  # Old content

# Merge fetched changes
git merge origin/main

# Now changes are in working directory
cat README.md  # New content

# Compare with pull (fetch + merge in one command)
git pull origin main
```
</details>

---

### Exercise 19: Resolving Rebase Conflicts
**Objective**: Handle conflicts during rebase

1. Create divergent branches
2. Attempt rebase
3. Resolve conflicts
4. Complete rebase

<details>
<summary>Solution</summary>

```bash
# Create branch and commit
git checkout -b rebase-conflict
echo "Branch content" > conflict.txt
git add conflict.txt
git commit -m "Branch commit"

# Switch to main and create conflict
git checkout main
echo "Main content" > conflict.txt
git add conflict.txt
git commit -m "Main commit"

# Attempt rebase
git checkout rebase-conflict
git rebase main
# CONFLICT!

# Check status
git status

# Resolve conflict in conflict.txt
echo "Resolved content" > conflict.txt

# Stage resolved file
git add conflict.txt

# Continue rebase
git rebase --continue

# View clean history
git log --oneline --graph
```
</details>

---

### Exercise 20: Amending Commits
**Objective**: Modify the last commit

1. Make a commit
2. Realize you forgot to add a file
3. Amend the commit to include the file
4. Change the commit message

<details>
<summary>Solution</summary>

```bash
# Make initial commit
echo "Content" > file1.txt
git add file1.txt
git commit -m "Add file1"

# Forgot to add file2!
echo "More content" > file2.txt

# Add to staging
git add file2.txt

# Amend last commit
git commit --amend

# Editor opens - you can change message
# Save and close

# Verify both files in last commit
git show HEAD

# Quick amend without changing message
git commit --amend --no-edit
```
</details>

---

## Advanced Level Exercises

### Exercise 21: Git Reflog
**Objective**: Recover lost commits using reflog

1. Make commits
2. Reset hard (losing commits)
3. Use reflog to find lost commits
4. Recover them

<details>
<summary>Solution</summary>

```bash
# Make commits
echo "Important work" > important.txt
git add important.txt
git commit -m "Important commit"

git log --oneline  # Note the commit hash

# Accidentally reset hard
git reset --hard HEAD~1

# File is gone!
ls  # important.txt not present

# Use reflog to find commit
git reflog

# Output shows:
# abc123 HEAD@{0}: reset: moving to HEAD~1
# def456 HEAD@{1}: commit: Important commit

# Recover lost commit
git checkout def456
# OR create branch
git checkout -b recovered def456

# Verify file is back
ls  # important.txt is there!

# Merge back to main if needed
git checkout main
git merge recovered
```
</details>

---

### Exercise 22: Git Bisect
**Objective**: Find bug introduction using bisect

1. Create repository with several commits
2. Introduce a "bug" in one commit
3. Use bisect to find the bad commit

<details>
<summary>Solution</summary>

```bash
# Create commits
echo "Version 1" > app.txt
git add app.txt
git commit -m "v1"

echo "Version 2" >> app.txt
git commit -am "v2"

echo "Version 3" >> app.txt
git commit -am "v3"

echo "BUG" >> app.txt  # Bug introduced
git commit -am "v4"

echo "Version 5" >> app.txt
git commit -am "v5"

echo "Version 6" >> app.txt
git commit -am "v6"

# Start bisect
git bisect start

# Mark current as bad
git bisect bad

# Find good commit
git log --oneline
git bisect good <hash-of-v3>

# Git checks out middle commit
# Test for bug (check if "BUG" in file)
grep "BUG" app.txt

# If bug exists
git bisect bad

# If no bug
git bisect good

# Repeat until Git finds first bad commit
# Git will say: "abc123 is the first bad commit"

# End bisect
git bisect reset

# View the bad commit
git show abc123
```
</details>

---

### Exercise 23: Git Hooks
**Objective**: Create pre-commit hook

1. Create pre-commit hook
2. Add validation logic
3. Test hook by committing

<details>
<summary>Solution</summary>

```bash
# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# Check for console.log in JavaScript files
if git diff --cached --name-only | grep -q '\.js$'; then
    if git diff --cached | grep -q 'console.log'; then
        echo "Error: console.log found in staged files"
        echo "Please remove console.log before committing"
        exit 1
    fi
fi

echo "Pre-commit checks passed!"
exit 0
EOF

# Make hook executable
chmod +x .git/hooks/pre-commit

# Test hook
echo "console.log('test');" > test.js
git add test.js
git commit -m "Test commit"
# Hook will reject this!

# Fix and try again
echo "// console.log('test');" > test.js
git add test.js
git commit -m "Test commit"
# Hook will accept this!
```
</details>

---

### Exercise 24: Submodules
**Objective**: Add and use Git submodules

1. Create main repository
2. Add another repository as submodule
3. Clone repository with submodules
4. Update submodule

<details>
<summary>Solution</summary>

```bash
# Create main repository
mkdir main-project
cd main-project
git init

# Add submodule
git submodule add https://github.com/user/library.git libs/library

# Check .gitmodules file
cat .gitmodules

# Commit submodule
git add .gitmodules libs/library
git commit -m "Add library submodule"

# Clone repository with submodules
cd /tmp
git clone https://github.com/user/main-project.git
cd main-project

# Initialize submodules
git submodule init
git submodule update

# Or clone with submodules in one command
git clone --recursive https://github.com/user/main-project.git

# Update submodule to latest
cd libs/library
git pull origin main
cd ../..
git add libs/library
git commit -m "Update library submodule"
```
</details>

---

### Exercise 25: Worktrees
**Objective**: Work on multiple branches simultaneously

1. Create worktree for different branch
2. Work in both worktrees
3. Cleanup worktrees

<details>
<summary>Solution</summary>

```bash
# In main repository
git worktree add ../feature-worktree feature-branch

# Now you have two directories:
# - main-project (on main branch)
# - feature-worktree (on feature-branch)

# Work in feature worktree
cd ../feature-worktree
echo "Feature work" > feature.txt
git add feature.txt
git commit -m "Add feature"

# Work in main worktree
cd ../main-project
echo "Main work" > main.txt
git add main.txt
git commit -m "Add main work"

# List worktrees
git worktree list

# Remove worktree when done
git worktree remove ../feature-worktree

# Prune stale worktrees
git worktree prune
```
</details>

---

### Exercise 26: Advanced History Rewriting
**Objective**: Rewrite history to remove sensitive data

1. Commit file with "sensitive" data
2. Make more commits
3. Use filter-branch to remove sensitive file
4. Verify file is gone from history

<details>
<summary>Solution</summary>

```bash
# Create file with sensitive data
echo "API_KEY=secret123" > .env
git add .env
git commit -m "Add config"

# Make more commits
echo "Update" > file1.txt
git add file1.txt
git commit -m "Update file1"

echo "Update 2" > file2.txt
git add file2.txt
git commit -m "Update file2"

# Realize .env should not be in history!

# Option 1: filter-branch (deprecated but works)
git filter-branch --tree-filter 'rm -f .env' HEAD

# Option 2: BFG Repo Cleaner (recommended, faster)
# Download bfg.jar first
java -jar bfg.jar --delete-files .env

# Clean up
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Verify .env is gone from all commits
git log --all --full-history -- .env
# Should show nothing

# Force push to update remote
git push origin main --force
```
</details>

---

### Exercise 27: Git Blame Investigation
**Objective**: Find who changed specific lines

1. Create file with multiple contributors (simulate)
2. Use git blame to find changes
3. View commit details

<details>
<summary>Solution</summary>

```bash
# Create file
cat > code.js << EOF
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}
EOF

git add code.js
git commit -m "Add math functions"

# Modify file
cat > code.js << EOF
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

function multiply(a, b) {
    return a * b;
}
EOF

git add code.js
git commit -m "Add multiply function"

# Use git blame
git blame code.js

# Blame specific lines
git blame -L 7,9 code.js

# Show author email
git blame -e code.js

# Ignore whitespace changes
git blame -w code.js

# Show commit details
git show <commit-hash-from-blame>
```
</details>

---

### Exercise 28: Complex Merge Strategies
**Objective**: Use different merge strategies

1. Create branches with conflicts
2. Try different merge strategies
3. Understand when to use each

<details>
<summary>Solution</summary>

```bash
# Create branches
git checkout -b ours-branch
echo "From ours" > conflict.txt
git add conflict.txt
git commit -m "Ours commit"

git checkout main
git checkout -b theirs-branch
echo "From theirs" > conflict.txt
git add conflict.txt
git commit -m "Theirs commit"

# Merge with "ours" strategy (keep our version)
git checkout ours-branch
git merge -X ours theirs-branch

# Merge with "theirs" strategy (keep their version)
git reset --hard HEAD~1  # Undo merge
git merge -X theirs theirs-branch

# Merge with "recursive" strategy (default)
git reset --hard HEAD~1
git merge theirs-branch
# Resolve conflicts manually
```
</details>

---

### Exercise 29: Git Archive
**Objective**: Export repository without Git history

1. Create repository with files
2. Export to zip/tar
3. Verify exported files

<details>
<summary>Solution</summary>

```bash
# Create repository
mkdir export-demo
cd export-demo
git init

# Add files
echo "File 1" > file1.txt
echo "File 2" > file2.txt
git add .
git commit -m "Initial commit"

# Export to zip
git archive --format=zip --output=project.zip HEAD

# Export to tar.gz
git archive --format=tar --output=project.tar HEAD
gzip project.tar

# Export specific directory
git archive --format=zip --output=src.zip HEAD:src/

# Export with prefix
git archive --format=zip --prefix=project/ --output=project.zip HEAD

# Verify
unzip -l project.zip
```
</details>

---

### Exercise 30: Complete Team Workflow
**Objective**: Simulate complete team workflow

1. Setup main repository
2. Create feature branch
3. Make changes with multiple commits
4. Clean up with interactive rebase
5. Create pull request
6. Code review changes
7. Merge to main
8. Tag release

<details>
<summary>Solution</summary>

```bash
# 1. Clone repository
git clone https://github.com/team/project.git
cd project

# 2. Update main
git checkout main
git pull origin main

# 3. Create feature branch
git checkout -b feature/user-profile

# 4. Make changes (multiple commits)
echo "Profile template" > profile.html
git add profile.html
git commit -m "Add profile template"

echo "Profile styles" > profile.css
git add profile.css
git commit -m "Add profile styles"

echo "Fix typo" >> profile.html
git add profile.html
git commit -m "Fix typo in template"

# 5. Clean up commits
git rebase -i HEAD~3
# Squash last commit into second

# 6. Push feature branch
git push -u origin feature/user-profile

# 7. Create Pull Request (on GitHub/GitLab)

# 8. Code review feedback - make changes
echo "Review fix" >> profile.html
git add profile.html
git commit -m "Address review feedback"
git push origin feature/user-profile

# 9. After approval, update from main
git fetch origin
git rebase origin/main

# 10. Force push (if needed)
git push origin feature/user-profile --force-with-lease

# 11. Merge (on GitHub/GitLab)
# Or locally:
git checkout main
git pull origin main
git merge --no-ff feature/user-profile
git push origin main

# 12. Tag release
git tag -a v1.1.0 -m "Release version 1.1.0"
git push origin v1.1.0

# 13. Clean up
git branch -d feature/user-profile
git push origin --delete feature/user-profile

# 14. Update local main
git pull origin main --tags
```
</details>

---

## Tips for Practice

1. **Create Practice Repository**: Make a dedicated repo for exercises
2. **Experiment Freely**: Git is hard to break permanently (thanks to reflog!)
3. **Read Error Messages**: They're usually helpful
4. **Use --help**: `git help <command>` for detailed info
5. **Visualize**: Use `git log --graph --all` frequently
6. **Take Notes**: Document what you learn
7. **Teach Others**: Best way to solidify understanding
8. **Real Projects**: Apply to actual projects

---

## Next Steps

1. Complete all exercises in order
2. Practice each exercise until comfortable
3. Combine exercises into workflows
4. Apply to real projects
5. Read [Git Learning Roadmap](git-learning-roadmap.md) for theory
6. Check [Troubleshooting Guide](git-troubleshooting-guide.md) when stuck

**Good luck with your practice! 🚀**


---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

