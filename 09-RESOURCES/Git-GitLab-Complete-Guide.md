# Git & GitLab - Hướng Dẫn Toàn Diện

## Mục Lục

1. [Git Basics](#git-basics)
2. [Git Branching](#git-branching)
3. [Git Remote](#git-remote)
4. [Git Advanced](#git-advanced)
5. [GitLab](#gitlab)
6. [Git Workflows](#git-workflows)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)

---

# Git Basics

## Giới Thiệu

**Git** là hệ thống quản lý phiên bản phân tán (Distributed Version Control System).

### Tại Sao Cần Git?

✅ **Version Control**: Theo dõi lịch sử thay đổi
✅ **Collaboration**: Làm việc nhóm hiệu quả
✅ **Backup**: Lưu trữ code an toàn
✅ **Branching**: Phát triển tính năng độc lập
✅ **Rollback**: Quay lại phiên bản cũ dễ dàng

---

## Setup & Configuration

### Install Git

```bash
# Windows
# Download từ https://git-scm.com/download/win

# macOS
brew install git

# Linux (Ubuntu/Debian)
sudo apt-get install git

# Linux (Fedora)
sudo dnf install git

# Kiểm tra version
git --version
```

### Initial Configuration

```bash
# Set user name
git config --global user.name "Your Name"

# Set email
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# Set default branch name
git config --global init.defaultBranch main

# Enable color
git config --global color.ui auto

# View all config
git config --list

# View specific config
git config user.name
git config user.email

# Config levels
git config --global   # User level (~/.gitconfig)
git config --local    # Repository level (.git/config)
git config --system   # System level (/etc/gitconfig)
```

### SSH Key Setup

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent
eval "$(ssh-agent -s)"

# Add SSH key
ssh-add ~/.ssh/id_ed25519

# Copy public key
cat ~/.ssh/id_ed25519.pub
# Paste vào GitLab/GitHub Settings > SSH Keys

# Test connection
ssh -T git@gitlab.com
ssh -T git@github.com
```

---

## Repository Basics

### Initialize Repository

```bash
# Create new repository
git init

# Create with specific branch name
git init -b main

# Clone existing repository
git clone https://gitlab.com/username/repo.git
git clone git@gitlab.com:username/repo.git

# Clone to specific folder
git clone https://gitlab.com/username/repo.git my-folder

# Clone specific branch
git clone -b develop https://gitlab.com/username/repo.git

# Shallow clone (only recent history)
git clone --depth 1 https://gitlab.com/username/repo.git
```

### Basic Commands

```bash
# Check status
git status

# Short status
git status -s

# Add files to staging
git add file.txt                # Add specific file
git add .                       # Add all files
git add *.js                    # Add all .js files
git add src/                    # Add entire folder

# Remove from staging
git reset file.txt              # Unstage file
git reset                       # Unstage all

# Commit changes
git commit -m "Commit message"

# Commit with description
git commit -m "Title" -m "Description"

# Add and commit in one command
git commit -am "Message"        # Only for tracked files

# Amend last commit
git commit --amend -m "New message"

# Amend without changing message
git commit --amend --no-edit

# View commit history
git log

# Compact log
git log --oneline

# Graph view
git log --graph --oneline --all

# Show specific number of commits
git log -n 5

# Show commits by author
git log --author="John"

# Show commits in date range
git log --since="2024-01-01" --until="2024-12-31"

# Show file changes
git log --stat

# Show detailed changes
git log -p

# Show specific file history
git log -- file.txt
```

---

## Working with Changes

### View Changes

```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged
git diff --cached

# Show changes in specific file
git diff file.txt

# Show changes between commits
git diff commit1 commit2

# Show changes between branches
git diff main develop

# Show only file names
git diff --name-only

# Show word-level diff
git diff --word-diff
```

### Discard Changes

```bash
# Discard changes in working directory
git checkout -- file.txt        # Old way
git restore file.txt            # New way (Git 2.23+)

# Discard all changes
git checkout -- .
git restore .

# Unstage file
git reset HEAD file.txt         # Old way
git restore --staged file.txt   # New way

# Discard staged and unstaged changes
git reset --hard HEAD

# Discard last commit (keep changes)
git reset --soft HEAD~1

# Discard last commit (discard changes)
git reset --hard HEAD~1
```

### Stash Changes

```bash
# Stash current changes
git stash

# Stash with message
git stash save "Work in progress"

# Stash including untracked files
git stash -u

# List stashes
git stash list

# Show stash content
git stash show
git stash show -p              # Show detailed changes

# Apply stash (keep in stash list)
git stash apply
git stash apply stash@{0}      # Apply specific stash

# Pop stash (remove from list)
git stash pop

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch new-branch
```

---

# Git Branching

## Branch Basics

```bash
# List branches
git branch                      # Local branches
git branch -r                   # Remote branches
git branch -a                   # All branches

# Create branch
git branch feature-login

# Create and switch to branch
git checkout -b feature-login   # Old way
git switch -c feature-login     # New way (Git 2.23+)

# Switch branch
git checkout main               # Old way
git switch main                 # New way

# Rename branch
git branch -m old-name new-name

# Rename current branch
git branch -m new-name

# Delete branch
git branch -d feature-login     # Safe delete (merged only)
git branch -D feature-login     # Force delete

# Delete remote branch
git push origin --delete feature-login
git push origin :feature-login  # Alternative syntax
```

## Merging

```bash
# Merge branch into current branch
git merge feature-login

# Merge with commit message
git merge feature-login -m "Merge feature-login"

# Merge without fast-forward
git merge --no-ff feature-login

# Abort merge
git merge --abort

# Continue merge after resolving conflicts
git merge --continue

# Squash merge (combine all commits into one)
git merge --squash feature-login
git commit -m "Add login feature"
```

## Rebasing

```bash
# Rebase current branch onto main
git rebase main

# Interactive rebase (last 3 commits)
git rebase -i HEAD~3

# Continue rebase after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip

# Abort rebase
git rebase --abort

# Rebase onto specific commit
git rebase --onto main feature-a feature-b
```

### Interactive Rebase Commands

```bash
# In interactive rebase editor:
pick    # Use commit
reword  # Use commit, but edit message
edit    # Use commit, but stop for amending
squash  # Combine with previous commit
fixup   # Like squash, but discard message
drop    # Remove commit

# Example:
pick abc123 Add login feature
squash def456 Fix login bug
reword ghi789 Update documentation
drop jkl012 Remove debug code
```

## Cherry-Pick

```bash
# Apply specific commit to current branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick without committing
git cherry-pick -n abc123

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

---

# Git Remote

## Remote Basics

```bash
# List remotes
git remote
git remote -v                   # Show URLs

# Add remote
git remote add origin https://gitlab.com/username/repo.git

# Change remote URL
git remote set-url origin https://gitlab.com/username/new-repo.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin upstream

# Show remote info
git remote show origin
```

## Fetch & Pull

```bash
# Fetch from remote (don't merge)
git fetch origin

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune

# Pull (fetch + merge)
git pull origin main

# Pull with rebase
git pull --rebase origin main

# Pull all branches
git pull --all

# Set upstream branch
git branch --set-upstream-to=origin/main main

# Pull with upstream
git pull
```

## Push

```bash
# Push to remote
git push origin main

# Push all branches
git push --all origin

# Push tags
git push --tags

# Push and set upstream
git push -u origin main

# Force push (dangerous!)
git push --force origin main

# Force push with lease (safer)
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete feature-login

# Push specific tag
git push origin v1.0.0
```

---

# Git Advanced

## Tags

```bash
# List tags
git tag

# Create lightweight tag
git tag v1.0.0

# Create annotated tag
git tag -a v1.0.0 -m "Version 1.0.0"

# Tag specific commit
git tag v1.0.0 abc123

# Show tag info
git show v1.0.0

# Delete tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push --tags

# Checkout tag
git checkout v1.0.0
```

## Submodules

```bash
# Add submodule
git submodule add https://gitlab.com/username/lib.git libs/mylib

# Initialize submodules
git submodule init

# Update submodules
git submodule update

# Clone with submodules
git clone --recursive https://gitlab.com/username/repo.git

# Update all submodules
git submodule update --remote

# Remove submodule
git submodule deinit libs/mylib
git rm libs/mylib
rm -rf .git/modules/libs/mylib
```

## Git Hooks

```bash
# Hooks location
.git/hooks/

# Common hooks:
pre-commit          # Before commit
prepare-commit-msg  # Before commit message editor
commit-msg          # After commit message
post-commit         # After commit
pre-push            # Before push
post-merge          # After merge

# Example pre-commit hook
#!/bin/sh
npm run lint
npm run test

# Make hook executable
chmod +x .git/hooks/pre-commit
```

## Reflog

```bash
# Show reflog
git reflog

# Show reflog for specific branch
git reflog show main

# Recover deleted branch
git reflog
git checkout -b recovered-branch abc123

# Recover deleted commits
git reflog
git cherry-pick abc123
```

## Bisect (Find Bug)

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark old commit as good
git bisect good abc123

# Git will checkout middle commit
# Test and mark as good or bad
git bisect good   # or git bisect bad

# Continue until bug is found

# Reset bisect
git bisect reset
```

## Worktree

```bash
# Create worktree
git worktree add ../feature-branch feature-branch

# List worktrees
git worktree list

# Remove worktree
git worktree remove ../feature-branch

# Prune worktrees
git worktree prune
```

---

# GitLab

## GitLab Basics

### GitLab vs GitHub

| Feature                | GitLab             | GitHub                 |
| ---------------------- | ------------------ | ---------------------- |
| **CI/CD**              | Built-in           | GitHub Actions         |
| **Self-hosted**        | ✅ Free            | ❌ Enterprise only     |
| **Issue Tracking**     | Advanced           | Basic                  |
| **Wiki**               | ✅                 | ✅                     |
| **Container Registry** | ✅ Built-in        | ✅                     |
| **Security Scanning**  | ✅                 | ✅ (Advanced Security) |
| **Price**              | Free tier generous | Free tier limited      |

---

## GitLab CI/CD

### .gitlab-ci.yml Basics

```yaml
# Basic pipeline
stages:
  - build
  - test
  - deploy

build-job:
  stage: build
  script:
    - echo "Building the app..."
    - npm install
    - npm run build
  artifacts:
    paths:
      - dist/

test-job:
  stage: test
  script:
    - echo "Running tests..."
    - npm run test

deploy-job:
  stage: deploy
  script:
    - echo "Deploying..."
    - npm run deploy
  only:
    - main
```

### Advanced Pipeline

```yaml
# Variables
variables:
  NODE_VERSION: "18"
  DOCKER_IMAGE: "node:${NODE_VERSION}"

# Cache
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/
    - .npm/

# Before script (runs before every job)
before_script:
  - npm ci --cache .npm --prefer-offline

# Stages
stages:
  - install
  - lint
  - test
  - build
  - deploy

# Install dependencies
install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

# Lint
lint:
  stage: lint
  script:
    - npm run lint
  dependencies:
    - install

# Unit tests
test:unit:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# E2E tests
test:e2e:
  stage: test
  script:
    - npm run test:e2e
  only:
    - main
    - merge_requests

# Build
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  only:
    - main
    - tags

# Deploy to staging
deploy:staging:
  stage: deploy
  script:
    - echo "Deploying to staging..."
    - npm run deploy:staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Deploy to production
deploy:production:
  stage: deploy
  script:
    - echo "Deploying to production..."
    - npm run deploy:production
  environment:
    name: production
    url: https://example.com
  only:
    - main
  when: manual # Require manual trigger
```

### Docker Build & Push

```yaml
# Build and push Docker image
docker-build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  variables:
    DOCKER_DRIVER: overlay2
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

### Multi-Environment Deployment

```yaml
# Deploy to multiple environments
.deploy_template: &deploy_template
  stage: deploy
  script:
    - echo "Deploying to $ENVIRONMENT..."
    - ssh $SSH_USER@$SSH_HOST "cd /var/www/$ENVIRONMENT && git pull && npm install && pm2 restart app"

deploy:dev:
  <<: *deploy_template
  variables:
    ENVIRONMENT: "dev"
    SSH_HOST: "dev.example.com"
  environment:
    name: development
    url: https://dev.example.com
  only:
    - develop

deploy:staging:
  <<: *deploy_template
  variables:
    ENVIRONMENT: "staging"
    SSH_HOST: "staging.example.com"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

deploy:production:
  <<: *deploy_template
  variables:
    ENVIRONMENT: "production"
    SSH_HOST: "example.com"
  environment:
    name: production
    url: https://example.com
  only:
    - tags
  when: manual
```

---

## GitLab Merge Requests

### Create Merge Request

```bash
# Push branch
git push -u origin feature-login

# Create MR via CLI (using glab)
glab mr create --title "Add login feature" --description "Implements user login"

# Or via GitLab UI
# Go to repository → Merge Requests → New Merge Request
```

### MR Best Practices

```markdown
## Merge Request Template

### Description

Brief description of changes

### Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

### Checklist

- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] All tests passing
- [ ] No new warnings

### Screenshots (if applicable)

Add screenshots here

### Related Issues

Closes #123
```

### MR Commands

```bash
# Approve MR
glab mr approve 123

# Merge MR
glab mr merge 123

# Close MR
glab mr close 123

# Reopen MR
glab mr reopen 123

# View MR
glab mr view 123

# List MRs
glab mr list

# Checkout MR locally
glab mr checkout 123
```

---

## GitLab Issues

### Create Issue

```bash
# Via CLI
glab issue create --title "Fix login bug" --description "Login fails on mobile"

# With labels
glab issue create --title "Add feature" --label "enhancement,high-priority"

# Assign to user
glab issue create --title "Bug fix" --assignee @username
```

### Issue Commands

```bash
# List issues
glab issue list

# View issue
glab issue view 123

# Close issue
glab issue close 123

# Reopen issue
glab issue reopen 123

# Update issue
glab issue update 123 --title "New title"

# Add comment
glab issue note 123 "This is a comment"
```

---

# Git Workflows

## Git Flow

```
main (production)
  ↓
develop (integration)
  ↓
feature/* (new features)
release/* (release preparation)
hotfix/* (urgent fixes)
```

### Git Flow Commands

```bash
# Start feature
git checkout develop
git checkout -b feature/login

# Finish feature
git checkout develop
git merge --no-ff feature/login
git branch -d feature/login
git push origin develop

# Start release
git checkout develop
git checkout -b release/1.0.0

# Finish release
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Version 1.0.0"
git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# Hotfix
git checkout main
git checkout -b hotfix/critical-bug
# Fix bug
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix 1.0.1"
git checkout develop
git merge --no-ff hotfix/critical-bug
git branch -d hotfix/critical-bug
```

---

## GitHub Flow (Simpler)

```
main (production)
  ↓
feature/* (all changes)
```

### GitHub Flow Process

```bash
# 1. Create branch
git checkout -b feature/new-feature

# 2. Make changes and commit
git add .
git commit -m "Add new feature"

# 3. Push and create PR
git push -u origin feature/new-feature

# 4. Review and merge
# Merge via GitHub/GitLab UI

# 5. Delete branch
git branch -d feature/new-feature
git push origin --delete feature/new-feature
```

---

## Trunk-Based Development

```
main (always deployable)
  ↓
short-lived feature branches (< 1 day)
```

### Trunk-Based Process

```bash
# 1. Pull latest
git checkout main
git pull origin main

# 2. Create short-lived branch
git checkout -b feature/quick-fix

# 3. Make small changes
git add .
git commit -m "Quick fix"

# 4. Push and merge quickly
git push -u origin feature/quick-fix
# Create MR and merge immediately

# 5. Delete branch
git branch -d feature/quick-fix
```

---

# Best Practices

## Commit Messages

### Conventional Commits

```bash
# Format
<type>(<scope>): <subject>

<body>

<footer>

# Types
feat:     # New feature
fix:      # Bug fix
docs:     # Documentation
style:    # Formatting
refactor: # Code restructuring
test:     # Tests
chore:    # Maintenance

# Examples
feat(auth): add login functionality

fix(api): resolve null pointer exception

docs(readme): update installation instructions

style(css): format button styles

refactor(utils): simplify date formatting

test(auth): add login tests

chore(deps): update dependencies
```

### Good Commit Messages

```bash
# ✅ GOOD
git commit -m "feat(auth): implement JWT authentication"
git commit -m "fix(api): handle null response from user endpoint"
git commit -m "docs(readme): add setup instructions"

# ❌ BAD
git commit -m "fix"
git commit -m "update"
git commit -m "changes"
git commit -m "asdfasdf"
```

---

## .gitignore

```bash
# Node.js
node_modules/
npm-debug.log
yarn-error.log
.env
.env.local

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

# Testing
coverage/
.nyc_output/

# Temporary
*.tmp
*.temp
.cache/
```

### Global .gitignore

```bash
# Create global gitignore
git config --global core.excludesfile ~/.gitignore_global

# Add to ~/.gitignore_global
.DS_Store
.vscode/
.idea/
*.swp
```

---

## Git Aliases

```bash
# Add aliases to ~/.gitconfig
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
git config --global alias.amend 'commit --amend --no-edit'

# Usage
git co main          # git checkout main
git br               # git branch
git ci -m "message"  # git commit -m "message"
git st               # git status
git unstage file.txt # git reset HEAD file.txt
git last             # git log -1 HEAD
git visual           # git log --graph --oneline --all
git amend            # git commit --amend --no-edit
```

---

## Branch Naming

```bash
# Convention
<type>/<description>

# Types
feature/    # New features
bugfix/     # Bug fixes
hotfix/     # Urgent fixes
release/    # Release branches
docs/       # Documentation
refactor/   # Code refactoring
test/       # Tests

# Examples
feature/user-authentication
bugfix/login-error
hotfix/critical-security-patch
release/v1.2.0
docs/api-documentation
refactor/database-queries
test/integration-tests
```

---

# Troubleshooting

## Common Issues

### Merge Conflicts

```bash
# When merge conflict occurs
git status  # See conflicted files

# Open conflicted file, you'll see:
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name

# Resolve conflicts manually, then:
git add resolved-file.txt
git commit -m "Resolve merge conflict"

# Or abort merge
git merge --abort
```

### Undo Commits

```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Undo last commit (keep changes in working directory)
git reset --mixed HEAD~1

# Undo multiple commits
git reset --soft HEAD~3

# Undo pushed commit (create new commit)
git revert HEAD
git push origin main
```

### Fix Wrong Branch

```bash
# Committed to wrong branch
git log  # Copy commit hash

# Switch to correct branch
git checkout correct-branch
git cherry-pick abc123

# Go back and remove from wrong branch
git checkout wrong-branch
git reset --hard HEAD~1
```

### Recover Deleted Branch

```bash
# Find deleted branch commit
git reflog

# Recreate branch
git checkout -b recovered-branch abc123
```

### Clean Repository

```bash
# Remove untracked files (dry run)
git clean -n

# Remove untracked files
git clean -f

# Remove untracked files and directories
git clean -fd

# Remove ignored files too
git clean -fdx
```

### Large File Issues

```bash
# Remove large file from history
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD

# Or use BFG Repo-Cleaner (faster)
bfg --delete-files large-file.zip
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# Use Git LFS for large files
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add file.psd
git commit -m "Add large file with LFS"
```

### Sync Fork

```bash
# Add upstream remote
git remote add upstream https://gitlab.com/original/repo.git

# Fetch upstream
git fetch upstream

# Merge upstream changes
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

---

## Git Performance

### Speed Up Git

```bash
# Enable parallel index preload
git config --global core.preloadindex true

# Enable file system cache
git config --global core.fscache true

# Increase pack window
git config --global pack.windowMemory 256m
git config --global pack.packSizeLimit 256m

# Garbage collection
git gc --aggressive --prune=now

# Optimize repository
git repack -a -d --depth=250 --window=250
```

---

## Useful Commands

```bash
# Show who changed each line
git blame file.txt

# Show file at specific commit
git show abc123:file.txt

# Search in commits
git log --all --grep="search term"

# Search in code
git grep "search term"

# Show branches containing commit
git branch --contains abc123

# Show commits not in main
git log main..feature-branch

# Show commits in main but not in feature
git log feature-branch..main

# Count commits
git rev-list --count HEAD

# Show commit stats
git shortlog -sn

# Show repository size
git count-objects -vH

# Archive repository
git archive --format=zip --output=repo.zip HEAD
```

---

## Git Cheat Sheet

### Daily Commands

```bash
# Start work
git pull origin main
git checkout -b feature/new-feature

# During work
git status
git add .
git commit -m "feat: add new feature"
git push -u origin feature/new-feature

# Update branch
git checkout main
git pull origin main
git checkout feature/new-feature
git rebase main

# Finish work
git checkout main
git merge feature/new-feature
git push origin main
git branch -d feature/new-feature
```

### Emergency Commands

```bash
# Undo last commit
git reset --soft HEAD~1

# Discard all changes
git reset --hard HEAD

# Stash everything
git stash -u

# Recover deleted branch
git reflog
git checkout -b recovered abc123

# Fix merge conflict
git merge --abort
git rebase --abort
```

---

## Resources

- [Git Documentation](https://git-scm.com/doc)
- [GitLab Documentation](https://docs.gitlab.com/)
- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)

---

**Lưu Ý**: Luôn backup code quan trọng và cẩn thận với các lệnh `--force`, `--hard`, và `filter-branch`!
