# Quick Git Versioning Guide

A comprehensive guide to Git version control for everyday development workflows.

## Getting Started

### Initialize a Repository

```bash
git init
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

Creates a new Git repository in your current folder.

### Check Status

```bash
git status
```

Shows which files are modified, staged, or untracked.

## Basic Workflow

### 1. Add Files to Staging

```bash
git add filename.txt        # Add specific file
git add .                   # Add all files
git add *.js                # Add all JavaScript files
```

### 2. Commit Changes

```bash
git commit -m "Your commit message"
```

Saves your staged changes with a descriptive message.

### 3. View Commit History

```bash
git log                     # Full history
git log --oneline          # Condensed view
```

## Working with Remote Repositories

### Connect to Remote Repository

```bash
git remote add origin https://github.com/username/repo.git
```

### Push Changes

```bash
git push origin main       # Push to main branch
git push -u origin main    # Push and set upstream (first time)
```

### Pull Latest Changes

```bash
git pull origin main
```

### Clone Existing Repository

```bash
git clone https://github.com/username/repo.git
```

## Branching Basics

### Create and Switch to New Branch

```bash
git checkout -b feature-name
# or newer syntax:
git switch -c feature-name
```

### Switch Between Branches

```bash
git checkout main
git switch main
```

### Merge Branch

```bash
git checkout main
git merge feature-name
```

## Essential Commands

### See Differences

```bash
git diff                   # Unstaged changes
git diff --staged         # Staged changes
```

### Undo Changes

```bash
git checkout -- filename  # Discard unstaged changes
git reset HEAD filename    # Unstage file
git reset --soft HEAD~1    # Undo last commit (keep changes)
```

## Typical Daily Workflow

1. `git status` - Check what's changed
2. `git add .` - Stage your changes
3. `git commit -m "Description of changes"` - Commit with message
4. `git push` - Send to remote repository
5. `git pull` - Get latest changes from others

## Excluding Files (.gitignore)

Create a `.gitignore` file to exclude unwanted files:

```bash
nano .gitignore
```

### Common .gitignore Patterns

```gitignore
# Virtual environment
venv/

# Database files
chromadb/

# Log files
logs/

# Dependencies file (optional - you might want to include this)
requirements.txt

# Python cache files
__pycache__/
*.pyc
*.pyo
*.pyd
.Python

# IDE files
.vscode/
.idea/
```

## Quick Reference

| Command | Description |
|---------|-------------|
| `git init` | Initialize repository |
| `git status` | Check repository status |
| `git add .` | Stage all changes |
| `git commit -m "message"` | Commit with message |
| `git push` | Push to remote |
| `git pull` | Pull from remote |
| `git log --oneline` | View commit history |
| `git diff` | View changes |

## Best Practices

- Write clear, descriptive commit messages
- Commit frequently with small, logical changes
- Always pull before pushing to avoid conflicts
- Use branches for new features or experiments
- Keep your `.gitignore` file updated
