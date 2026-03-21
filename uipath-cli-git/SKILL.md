---
name: uipath-cli-git
tags: [uipath-dev, cli, git, version-control, source-control]
description: Execute Git operations for UiPath projects using CLI commands. Covers git init, clone, commit, push, pull, branch management, and CI/CD workflows. Aligns with the official uipathcli (studio package analyze, pack, publish, test run). Triggers on requests like "commit UiPath project", "push to git", "clone UiPath repo", "git workflow for UiPath", "analyze project in CI".
---

# UiPath CLI Git Operations

Execute Git operations for UiPath automation projects using command-line interface.

## When to Use

- "Commit UiPath project changes"
- "Push workflow to repository"
- "Clone UiPath project from Git"
- "Create branch for feature"
- "Set up Git for UiPath project"
- "CI/CD pipeline for UiPath"
- "Analyze UiPath project in GitHub Actions" / "governance check before merge"

## Quick Reference

### UiPath CLI (uipathcli) with Git

Official CLI: [github.com/UiPath/uipathcli](https://github.com/UiPath/uipathcli). Use from the **project root** (`project.json`).

| Command | Purpose |
|---------|---------|
| `uipath studio package analyze` | Governance / static analysis (policy violations, warnings) |
| `uipath studio package pack` | Build `.nupkg` |
| `uipath studio package publish` | Publish package (Orchestrator / feed per your setup) |
| `uipath studio test run` | Run Studio test cases via connected Orchestrator |
| `uipath config` | Auth (PAT, OAuth, client credentials) and profiles |

**Not the same binary:** The **`uipath`** installed via **Python/pip** (UiPath SDK) exposes commands like `pack`, `run`, `deploy` — it does **not** implement `studio package analyze`. For **Studio XAML** projects, install **uipathcli** from [GitHub releases](https://github.com/UiPath/uipathcli/releases) and ensure `uipath.exe` on `PATH` is that build. **Close Studio** (or the project) before running **`studio package analyze`**; otherwise the analyzer reports a database lock (“project is already opened in another Studio instance”).

**Review:** combine **analyze** (automated rules) with the **`uipath-code-reviewer`** skill for XAML/C# quality. See [UiPath CLI Reference](./../_shared/uipath-cli.md).

### Common Git commands for UiPath

| Command | Purpose |
|---------|---------|
| `git init` | Initialize new repository |
| `git clone` | Clone existing repository |
| `git add` | Stage changes |
| `git commit` | Commit staged changes |
| `git push` | Push to remote |
| `git pull` | Pull from remote |
| `git branch` | Manage branches |
| `git checkout` | Switch branches |
| `git merge` | Merge branches |
| `git status` | Check repository status |

### UiPath-Specific Files

| File/Folder | Git Action |
|-------------|------------|
| `*.xaml` | Track (workflow files) |
| `project.json` | Track (project config) |
| `.objects/` | Track (Object Repository) |
| `.local/` | Ignore (local settings) |
| `.screenshots/` | Ignore (debug screenshots) |
| `*.nupkg` | Ignore (packages) |

## Project Setup

### Initialize Git for UiPath Project

```bash
# Navigate to UiPath project folder
cd "C:\Users\Username\Documents\UiPath\MyProject"

# Initialize Git repository
git init

# Create .gitignore for UiPath (Git Bash / WSL)
cat > .gitignore << 'EOF'
# UiPath specific
.local/
.screenshots/
.objects/.local/
*.nupkg
*.zip

# Build outputs
bin/
obj/
output/

# IDE files
.vs/
*.user
*.suo

# OS files
Thumbs.db
.DS_Store

# Logs
*.log
logs/

# Temporary files
*.tmp
*.temp
~$*

# Credentials (never commit)
.env
*.credentials
EOF

# PowerShell (native Windows): same rules, write file explicitly
# @'
# .local/
# .screenshots/
# ... (same entries as above)
# '@ | Set-Content -Encoding utf8 .gitignore

# Stage all files
git add .

# Initial commit
git commit -m "Initial commit: UiPath project setup"
```

### Clone Existing UiPath Project

```bash
# Clone from GitHub
git clone https://github.com/org/uipath-project.git

# Clone from Azure DevOps
git clone https://dev.azure.com/org/project/_git/uipath-project

# Clone with specific branch
git clone -b develop https://github.com/org/uipath-project.git

# Clone to specific folder
git clone https://github.com/org/uipath-project.git ./my-local-folder
```

### Configure Git for UiPath Development

```bash
# Set user identity (if not set globally)
git config user.name "Developer Name"
git config user.email "developer@company.com"

# Configure line endings for Windows (UiPath runs on Windows)
git config core.autocrlf true

# Configure diff for XAML files
git config diff.xaml.textconv "cat"

# Set default branch name
git config init.defaultBranch main
```

## Daily Workflow

### Check Status

```bash
# View current status
git status

# View status in short format
git status -s

# Example output:
# M  Main.xaml
# M  project.json
# ?? NewWorkflow.xaml
```

### Stage Changes

```bash
# Stage specific file
git add Main.xaml

# Stage all XAML files
git add "*.xaml"

# Stage all changes
git add .

# Stage interactively
git add -p

# Unstage file
git reset HEAD Main.xaml
```

### Commit Changes

```bash
# Commit with message
git commit -m "Add invoice processing workflow"

# Commit with detailed message
git commit -m "Add invoice processing workflow

- Added Main.xaml with invoice extraction logic
- Updated project.json with new dependencies
- Added error handling for invalid invoices"

# Amend last commit (before push)
git commit --amend -m "Updated commit message"

# Commit all tracked changes
git commit -am "Quick fix for selector issue"
```

### Push to Remote

```bash
# Push to origin main
git push origin main

# Push current branch
git push

# Push and set upstream
git push -u origin feature/invoice-processing

# Force push (use with caution)
git push --force-with-lease origin feature/my-branch
```

### Pull from Remote

```bash
# Pull latest changes
git pull

# Pull with rebase
git pull --rebase

# Pull specific branch
git pull origin develop

# Fetch without merge
git fetch origin
```

## Branch Management

### Create and Switch Branches

```bash
# Create new branch
git branch feature/new-workflow

# Switch to branch
git checkout feature/new-workflow

# Create and switch (shorthand)
git checkout -b feature/new-workflow

# Create from specific commit
git checkout -b hotfix/urgent-fix abc1234

# List all branches
git branch -a

# Delete local branch
git branch -d feature/completed

# Delete remote branch
git push origin --delete feature/completed
```

### Merge Branches

```bash
# Switch to target branch
git checkout main

# Merge feature branch
git merge feature/new-workflow

# Merge with no fast-forward (preserves history)
git merge --no-ff feature/new-workflow

# Abort merge if conflicts
git merge --abort
```

### Resolve Merge Conflicts

```bash
# Check conflicted files
git status

# View conflicts in file
# Look for <<<<<<< HEAD, =======, >>>>>>> markers

# After resolving, stage the file
git add Main.xaml

# Complete the merge
git commit -m "Merge feature/new-workflow into main"

# Use visual merge tool
git mergetool
```

## UiPath-Specific Workflows

### Feature Development Workflow

```bash
# 1. Start from main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/JIRA-123-invoice-automation

# 3. Develop in UiPath Studio
# (Make changes to workflows)

# 4. Stage and commit regularly
git add .
git commit -m "JIRA-123: Add invoice extraction sequence"

git add .
git commit -m "JIRA-123: Add error handling for missing fields"

# 5. Push feature branch
git push -u origin feature/JIRA-123-invoice-automation

# 6. Create Pull Request (via GitHub/Azure DevOps UI)

# 7. After PR approval, merge
git checkout main
git pull origin main
git merge --no-ff feature/JIRA-123-invoice-automation
git push origin main

# 8. Delete feature branch
git branch -d feature/JIRA-123-invoice-automation
git push origin --delete feature/JIRA-123-invoice-automation
```

### Hotfix Workflow

```bash
# 1. Create hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/critical-selector-fix

# 2. Make fix
# (Edit workflow in Studio)

# 3. Commit fix
git add .
git commit -m "Hotfix: Update selector for changed UI element"

# 4. Push and merge quickly
git push -u origin hotfix/critical-selector-fix

# 5. Merge to main
git checkout main
git merge --no-ff hotfix/critical-selector-fix
git push origin main

# 6. Also merge to develop if exists
git checkout develop
git merge --no-ff hotfix/critical-selector-fix
git push origin develop

# 7. Cleanup
git branch -d hotfix/critical-selector-fix
```

### Release Workflow

```bash
# 1. Create release branch
git checkout develop
git checkout -b release/1.2.0

# 2. Update version in project.json
# (Edit projectVersion field)

# 3. Commit version bump
git add project.json
git commit -m "Bump version to 1.2.0"

# 4. Final testing and fixes

# 5. Merge to main
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags

# 6. Merge back to develop
git checkout develop
git merge --no-ff release/1.2.0
git push origin develop

# 7. Cleanup
git branch -d release/1.2.0
```

## CI/CD Integration

Use the **official [uipathcli](https://github.com/UiPath/uipathcli)** commands from the repo root. Configure auth with `uipath config` (PAT, OAuth, or client credentials) or the supported `UIPATH_*` environment variables—**never** commit secrets.

### GitHub Actions Workflow

```yaml
# .github/workflows/uipath-ci.yml
name: UiPath CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: windows-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install UiPath CLI (GitHub release)
      run: |
        Invoke-WebRequest "https://github.com/UiPath/uipathcli/releases/latest/download/uipathcli-windows-amd64.zip" -OutFile "uipathcli.zip"
        Expand-Archive -Force -Path "uipathcli.zip" -DestinationPath "$env:RUNNER_TEMP\uipathcli"
        echo "$env:RUNNER_TEMP\uipathcli" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append

    # Set UIPATH_PAT / UIPATH_ORGANIZATION / UIPATH_TENANT (or client credentials) per uipathcli README — non-interactive
    - name: Analyze project (governance)
      working-directory: ./path/to/StudioProject
      env:
        UIPATH_PAT: ${{ secrets.UIPATH_PAT }}
        UIPATH_ORGANIZATION: ${{ secrets.UIPATH_ORGANIZATION }}
        UIPATH_TENANT: ${{ secrets.UIPATH_TENANT }}
      run: uipath studio package analyze --output text

    - name: Pack
      working-directory: ./path/to/StudioProject
      env:
        UIPATH_PAT: ${{ secrets.UIPATH_PAT }}
        UIPATH_ORGANIZATION: ${{ secrets.UIPATH_ORGANIZATION }}
        UIPATH_TENANT: ${{ secrets.UIPATH_TENANT }}
      run: uipath studio package pack --output ../../artifacts/

    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: uipath-package
        path: artifacts/*.nupkg

    # Optional: run Orchestrator-backed tests when secrets and connectivity exist
    # - name: Test run
    #   working-directory: ./path/to/StudioProject
    #   run: uipath studio test run

  deploy:
    needs: build
    runs-on: windows-latest
    if: github.ref == 'refs/heads/main'

    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v4
      with:
        name: uipath-package

    - name: Publish (example)
      run: |
        # From project directory or per your release process:
        # uipath studio package publish
        echo "Add publish auth and folder options from uipathcli docs"
```

### Azure DevOps Pipeline

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
    - main
    - develop

pool:
  vmImage: 'windows-latest'

variables:
  UIPATH_PAT: $(UiPathPAT)

stages:
- stage: Build
  jobs:
  - job: BuildPackage
    steps:
    - task: PowerShell@2
      displayName: 'Install UiPath CLI'
      inputs:
        targetType: 'inline'
        script: |
          Invoke-WebRequest "https://github.com/UiPath/uipathcli/releases/latest/download/uipathcli-windows-amd64.zip" -OutFile "$(Agent.TempDirectory)\uipathcli.zip"
          Expand-Archive -Force -Path "$(Agent.TempDirectory)\uipathcli.zip" -DestinationPath "$(Agent.TempDirectory)\uipathcli"
          $env:PATH = "$(Agent.TempDirectory)\uipathcli;" + $env:PATH

    - task: PowerShell@2
      displayName: 'Analyze and pack Studio project'
      inputs:
        targetType: 'inline'
        script: |
          Set-Location "$(Build.SourcesDirectory)\path\to\StudioProject"
          uipath studio package analyze --output text
          uipath studio package pack --output "$(Build.ArtifactStagingDirectory)"

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'uipath-package'

- stage: Deploy
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployToOrchestrator
    environment: 'Production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: PowerShell@2
            displayName: 'Publish (configure per uipathcli)'
            inputs:
              targetType: 'inline'
              script: |
                # uipath studio package publish
                Write-Host "Wire authentication and target folder from uipathcli README"
```

## Best Practices

### Commit Message Convention

```bash
# Format: <type>(<scope>): <description>

# Types:
# feat: New feature
# fix: Bug fix
# refactor: Code refactoring
# docs: Documentation
# test: Adding tests
# chore: Maintenance

# Examples:
git commit -m "feat(invoice): Add OCR extraction for scanned invoices"
git commit -m "fix(selector): Update login button selector for new UI"
git commit -m "refactor(main): Split large workflow into sub-workflows"
git commit -m "docs(readme): Add setup instructions"
```

### Branch Naming Convention

```bash
# Format: <type>/<ticket>-<description>

# Examples:
feature/JIRA-123-invoice-processing
bugfix/JIRA-456-selector-timeout
hotfix/JIRA-789-critical-login-fix
release/1.2.0
```

### .gitignore for UiPath

```gitignore
# UiPath Project Files
.local/
.screenshots/
.objects/.local/
*.nupkg

# Build and Output
bin/
obj/
output/
TestResults/

# IDE and Editor
.vs/
*.user
*.suo
.idea/

# OS Generated
Thumbs.db
.DS_Store
desktop.ini

# Logs and Temp
*.log
logs/
*.tmp
*.temp
~$*

# Credentials and Secrets
.env
*.credentials
appsettings.*.json
!appsettings.json

# Package Manager
packages/

# Test Data (if sensitive)
TestData/sensitive/
```

### Pre-Commit Checklist

```bash
# Before committing, verify:

# 1. Check status
git status

# 2. Review changes
git diff

# 3. Analyze / pack (official uipathcli, from project root)
uipath studio package analyze --output text
uipath studio package pack --output ./artifacts/

# 4. Run workflow tests: Studio Test Explorer locally, or `uipath studio test run` in CI with Orchestrator

# 5. Stage and commit
git add .
git commit -m "Your descriptive message"
```

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Large file error | XAML too big | Use Git LFS or split workflow |
| Merge conflicts in XAML | Concurrent edits | Use visual diff, coordinate with team |
| Authentication failed | Token expired | Re-authenticate with `git credential-manager` |
| Push rejected | Remote has changes | `git pull --rebase` then push |
| Detached HEAD | Checkout commit directly | `git checkout main` |

### Resolve XAML Merge Conflicts

```bash
# XAML conflicts are complex - best practices:

# 1. Use visual merge tool
git mergetool

# 2. Or accept one version entirely
git checkout --ours Main.xaml    # Keep your version
git checkout --theirs Main.xaml  # Keep their version

# 3. After resolving
git add Main.xaml
git commit -m "Resolve merge conflict in Main.xaml"
```

### Undo Operations

```bash
# Undo unstaged changes
git checkout -- Main.xaml

# Undo staged changes
git reset HEAD Main.xaml

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a pushed commit
git revert abc1234
git push
```

### Clean Up Repository

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

## Git Hooks for UiPath

### Pre-Commit Hook

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Governance / analysis gate (requires uipath CLI on PATH; run from project root)
echo "Running uipath studio package analyze..."
uipath studio package analyze --output text

if [ $? -ne 0 ]; then
    echo "Analyze failed. Commit aborted."
    exit 1
fi

echo "Analyze passed."
exit 0
```

### Pre-Push Hook

```bash
#!/bin/sh
# .git/hooks/pre-push

# Optional: pack as a build check, or `uipath studio test run` if Orchestrator is configured
echo "Packaging UiPath project..."
uipath studio package pack --output ./.uipath-pack-tmp/

if [ $? -ne 0 ]; then
    echo "Pack failed. Push aborted."
    exit 1
fi

rm -rf ./.uipath-pack-tmp/
echo "Pack succeeded."
exit 0
```

## Reference Documentation

- [UiPath uipathcli (official CLI)](https://github.com/UiPath/uipathcli)
- [UiPath Studio Git Integration](https://docs.uipath.com/studio/standalone/latest/user-guide/managing-projects-git)
- [Git Authentication](https://docs.uipath.com/studio/standalone/latest/user-guide/authenticating-to-git)
- [UiPath CLI Reference](./../_shared/uipath-cli.md)
- [Git Documentation](https://git-scm.com/doc)
