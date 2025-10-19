---
id: wkf.repo-organize-files
type: workflow
title: Repository File Organization Workflow
tags: [organization, repo-cleanup, refactoring, automation, runbook]
summary: 3-phase proven process for organizing 50+ scattered files by analyzing patterns, automating migration, and safely executing with 80% reduction in search time.
version: 1
---

# Repository File Organization Workflow

**Organize 50-500+ scattered files into a maintainable structure with automated migration and safety checks.**

## 🎯 Use Case

Use this workflow when:
- Files are scattered across root directory and subdirectories
- Hard to find specific files (taking 5+ minutes)
- New team members struggle with project structure
- Import paths are inconsistent or confusing
- Documentation and scripts are mixed together

**Success Rate:** Proven 2x with 80% reduction in search time

---

## 📋 Phase 1: ANALYZE (2-3 hours)

### Step 1: Inventory Current State

**Objective:** Understand what you have and where it lives.

```bash
# Count files by type
find . -type f | wc -l

# Show directory structure
tree -L 3

# Find scattered files
find . -maxdepth 1 -type f

# Group by extension
find . -type f | sed 's/.*\.//' | sort | uniq -c | sort -rn
```

**Document:**
- Total file count
- Files in root directory
- Subdirectory structure
- File types and counts

### Step 2: Identify Patterns

**Ask these questions:**
1. What are the main **purposes/functions**? (startup, testing, deployment, docs)
2. What **temporal patterns** exist? (daily scripts, archived docs)
3. What **ownership patterns** exist? (frontend, backend, shared)
4. What makes files **hard to find**? (similar names, wrong locations, no grouping)

**Create a mapping:**
```
Current Pain Points:
- 301 docs scattered (root + /docs + /documentation)
- 115 scripts (root + /scripts + /tools)
- Hard to distinguish current vs archived
- No clear workflow order

Emerging Categories:
1. Current/active work
2. Guides and runbooks
3. Testing and quality
4. Deployment and operations
5. Historical/reference
```

### Step 3: Design Category System

**Apply these principles:**
- ✅ Organize by PURPOSE, not file type
- ✅ Use numbered prefixes for logical ordering (00-, 01-, 02-)
- ✅ Keep 2-3 levels max (avoid deep nesting)
- ✅ Use self-explanatory names

**Example Documentation Structure:**
```
docs/
├── 00-CURRENT/          # Active working docs
├── 01-GUIDES/           # How-to and tutorials
├── 02-TECHNICAL/        # Architecture, APIs
├── 03-RUNBOOKS/         # Operational procedures
├── 04-PLANNING/         # Roadmaps, specs
├── 05-ARCHIVE/          # Historical reference
└── 06-REFERENCE/        # External references
```

**Example Scripts Structure:**
```
scripts/
├── 00-STARTUP/          # Init, setup, start
├── 01-DEVELOPMENT/      # Dev tools, helpers
├── 02-DATABASE/         # Seeds, migrations
├── 03-TESTING/          # Test runners, QA
├── 04-BUILD/            # Compilation, bundling
├── 05-DEPLOYMENT/       # Deploy, release
├── 06-OPERATIONS/       # Monitoring, maintenance
├── 07-UTILITIES/        # General purpose tools
└── 08-ARCHIVE/          # Old scripts
```

**Design Checklist:**
- [ ] Categories are mutually exclusive
- [ ] Every file has an obvious home
- [ ] Room for growth (gaps in numbering)
- [ ] Team agrees on structure
- [ ] Categories match your workflow

---

## 🤖 Phase 2: AUTOMATE (2 hours)

### Step 1: Create Structure Script

**File:** `create-structure.ps1` (or `.sh`)

```powershell
param([switch]$DryRun = $true)

$directories = @(
    "docs/00-CURRENT",
    "docs/01-GUIDES", 
    "docs/02-TECHNICAL",
    # ... add all categories
    "scripts/00-STARTUP",
    "scripts/01-DEVELOPMENT"
    # ... add all categories
)

foreach ($dir in $directories) {
    if ($DryRun) {
        Write-Host "[DRY RUN] Would create: $dir" -ForegroundColor Yellow
    } else {
        New-Item -ItemType Directory -Path $dir -Force | Out-Null
        Write-Host "✓ Created: $dir" -ForegroundColor Green
    }
}
```

### Step 2: Create Migration Script

**File:** `migrate-files.ps1` (or `.sh`)

**Key features:**
- Dry-run by default
- Source → Destination → Purpose mapping
- Existence checks
- Progress tracking
- Rollback documentation

```powershell
param([switch]$DryRun = $true)
$ErrorActionPreference = "Stop"
$moveCount = 0

function Move-File {
    param([string]$Src, [string]$Dst, [string]$Purpose)
    
    $srcPath = Join-Path $baseDir $Src
    $dstPath = Join-Path $baseDir $Dst
    
    if (Test-Path $srcPath) {
        if ($DryRun) {
            Write-Host "  [DRY RUN] $Src → $Dst" -ForegroundColor Yellow
        } else {
            $dstDir = Split-Path $dstPath -Parent
            if (-not (Test-Path $dstDir)) { 
                New-Item -ItemType Directory -Path $dstDir -Force | Out-Null 
            }
            Move-Item -Path $srcPath -Destination $dstPath -Force
            Write-Host "  ✓ Moved: $Src" -ForegroundColor Green
            $script:moveCount++
        }
        if ($Purpose) { 
            Write-Host "    → Purpose: $Purpose" -ForegroundColor Gray 
        }
    } else {
        Write-Host "  ⚠ Not found: $Src" -ForegroundColor Yellow
    }
}

# Documentation Mappings
Write-Host "`n📄 DOCUMENTATION" -ForegroundColor Cyan
Move-File "README.md" "docs/00-CURRENT/README.md" "Main project readme"
Move-File "SETUP.md" "docs/01-GUIDES/SETUP.md" "Setup instructions"
Move-File "API.md" "docs/02-TECHNICAL/API.md" "API documentation"
# ... add all doc mappings

# Script Mappings
Write-Host "`n📜 SCRIPTS" -ForegroundColor Cyan
Move-File "start.ps1" "scripts/00-STARTUP/start.ps1" "Main startup script"
Move-File "test.ps1" "scripts/03-TESTING/test-all.ps1" "Test runner"
# ... add all script mappings

Write-Host "`n✨ Summary: $moveCount files moved" -ForegroundColor Cyan
if ($DryRun) {
    Write-Host "Run with -DryRun:`$false to execute for real" -ForegroundColor Yellow
}
```

### Step 3: Create Master Script

**File:** `organize-repo.ps1`

```powershell
param([switch]$DryRun = $true, [switch]$SkipBackup)

Write-Host "🚀 Repository Organization" -ForegroundColor Cyan
Write-Host "================================`n"

# 1. Backup (if not skipped)
if (-not $SkipBackup -and -not $DryRun) {
    Write-Host "📦 Creating backup..." -ForegroundColor Yellow
    $timestamp = Get-Date -Format "yyyyMMdd-HHmmss"
    $backupPath = "../repo-backup-$timestamp"
    Copy-Item -Path . -Destination $backupPath -Recurse
    Write-Host "✓ Backup created at: $backupPath`n" -ForegroundColor Green
}

# 2. Create structure
Write-Host "📁 Creating directory structure..." -ForegroundColor Yellow
& .\create-structure.ps1 -DryRun:$DryRun

# 3. Migrate files
Write-Host "`n📦 Migrating files..." -ForegroundColor Yellow
& .\migrate-files.ps1 -DryRun:$DryRun

Write-Host "`n✅ Organization complete!" -ForegroundColor Green
if ($DryRun) {
    Write-Host "This was a DRY RUN. Run with -DryRun:`$false to execute." -ForegroundColor Yellow
}
```

### Step 4: Test with Dry-Run

```bash
# Test the full process
./organize-repo.ps1 -DryRun

# Review output carefully
# Check for:
# - All files accounted for
# - Correct destinations
# - No overwrites
# - Purpose makes sense
```

---

## ✅ Phase 3: EXECUTE (1 hour)

### Step 1: Team Review

**Share dry-run output with team:**
```bash
# Generate dry-run report
./organize-repo.ps1 -DryRun > migration-plan.txt

# Review as a team
# - Does structure make sense?
# - Any files in wrong place?
# - Missing categories?
```

**Get sign-off before proceeding.**

### Step 2: Execute Migration

```powershell
# Run for real (backup created automatically)
./organize-repo.ps1 -DryRun:$false

# Monitor output for errors
# If errors occur, restore from backup
```

### Step 3: Verify Results

```bash
# Check file counts match
find docs/ -type f | wc -l
find scripts/ -type f | wc -l

# Look for orphaned files
find . -maxdepth 1 -type f

# Verify structure
tree -L 2 docs/
tree -L 2 scripts/
```

**Verification Checklist:**
- [ ] All files moved successfully
- [ ] No files left in root (except essential ones)
- [ ] Correct file counts in new structure
- [ ] No duplicate files
- [ ] All categories populated as expected

### Step 4: Update Import Paths

**Find and fix broken imports:**

```bash
# Find all import statements
grep -r "import.*from" --include="*.ts" --include="*.js"

# Find require statements
grep -r "require(" --include="*.ts" --include="*.js"

# Update paths systematically
# Old: ../../scripts/utils.js
# New: ../../scripts/07-UTILITIES/utils.js
```

**Common patterns to update:**
- Import statements (JavaScript/TypeScript)
- Require statements (Node.js)
- Script references in package.json
- Path references in documentation
- CI/CD pipeline paths
- Dockerfile COPY commands

### Step 5: Update Documentation

**Files to update:**
- `README.md` - Reference new structure
- `CONTRIBUTING.md` - Where to add new files
- `docs/00-CURRENT/PROJECT-STRUCTURE.md` - Document organization system

**Add organization guide:**
```markdown
## Repository Organization

### Documentation (`/docs`)
- `00-CURRENT/` - Active working documents
- `01-GUIDES/` - How-to guides and tutorials
- `02-TECHNICAL/` - Technical specs and architecture
- ... (explain each)

### Scripts (`/scripts`)
- `00-STARTUP/` - Initialization and startup
- `01-DEVELOPMENT/` - Development tools
- ... (explain each)

### Adding New Files
- Choose category by PURPOSE, not file type
- If unsure, check similar existing files
- Document purpose in file header
```

### Step 6: Test Application

```bash
# Run all tests
npm test
# or
pytest

# Start application
npm start
# or
python app.py

# Test key workflows:
# - Startup
# - Database operations
# - API endpoints
# - Build process
# - Deployment (in staging)
```

### Step 7: Commit Changes

```bash
# Stage changes
git add .

# Commit with detailed message
git commit -m "refactor: reorganize repository structure

- Moved 301 docs into 7 categories (00-CURRENT through 06-REFERENCE)
- Moved 115 scripts into 8 categories (00-STARTUP through 08-ARCHIVE)
- Organized by purpose with numbered prefixes
- Updated all import paths
- Added PROJECT-STRUCTURE.md documentation

Reduces search time by ~80% and improves onboarding"

# Push to feature branch for review
git push origin refactor/repo-organization
```

---

## 📊 Success Metrics

**Measure these before and after:**

### Quantitative
- ⏱️ Time to find specific file: **Before:** 5-10 min → **After:** 30 sec - 2 min
- 📁 Files in root directory: **Before:** 50+ → **After:** < 5
- 📚 Onboarding time: **Before:** 2 hours → **After:** 30 min
- 🔍 Search queries needed: **Before:** 3-5 → **After:** 0-1

### Qualitative
- ✅ New developers can find files independently
- ✅ Logical structure matches team's mental model
- ✅ Clear where to add new files
- ✅ Reduced Slack/chat questions about file locations

---

## 🚨 Common Issues

### Issue: Too many files to migrate manually

**Solution:** This is why we automate! The migration script handles hundreds of files.

### Issue: Uncertain where a file should go

**Solution:** 
1. Check file's purpose/function
2. Look at when it's used in workflow
3. Find similar existing files
4. Document your reasoning in migration script
5. Review with team

### Issue: Imports break after migration

**Solution:**
1. Use find/replace with regex
2. Update paths systematically by directory
3. Run tests after each batch
4. Use IDE refactoring tools if available

### Issue: Need to rollback

**Solution:**
```bash
# Restore from backup
rm -rf docs/ scripts/
cp -r ../repo-backup-20251019-140000/* .

# Or use git
git checkout HEAD -- .
```

---

## 💡 Pro Tips

### 1. Start with Dry-Run
Always test with dry-run first. Review output carefully.

### 2. Document Decisions
Add comments to migration script explaining why files go where.

### 3. Leave Room for Growth
Use 00, 02, 04 numbering to leave gaps for future categories.

### 4. Keep Essential Files in Root
Some files belong in root: README.md, LICENSE, package.json, .gitignore

### 5. Archive, Don't Delete
When unsure, move to ARCHIVE rather than deleting.

### 6. Update Gradually
For large repos, consider organizing in phases (docs first, then scripts).

### 7. Get Team Buy-In
Review structure with team before executing. Ownership = compliance.

---

## 📚 Related Concepts

**This workflow applies the same patterns as:**
- Clean Architecture (organize by feature/purpose)
- Domain-Driven Design (group by domain)
- Monorepo organization (workspace structure)

**Inspired by successful patterns from:**
- Large open-source projects (React, Vue, Next.js)
- Production codebases
- Developer experience best practices

---

## 🎯 Next Steps

After organizing:
1. **Document the system** - Add to CONTRIBUTING.md
2. **Set up linting** - Enforce file placement with ESLint/custom scripts
3. **Automate checks** - CI checks for files in wrong locations
4. **Regular audits** - Review structure quarterly
5. **Onboarding guide** - Update new developer docs

---

## ✅ Checklist

**Before starting:**
- [ ] Inventory current files
- [ ] Identify patterns and pain points
- [ ] Design category system
- [ ] Get team alignment on structure

**During automation:**
- [ ] Create structure script with dry-run
- [ ] Create migration script with mappings
- [ ] Create master orchestration script
- [ ] Test thoroughly with dry-run

**During execution:**
- [ ] Review dry-run output with team
- [ ] Execute migration (backup created)
- [ ] Verify file counts and structure
- [ ] Update import paths
- [ ] Update documentation
- [ ] Test application
- [ ] Commit and push

**After completion:**
- [ ] Measure success metrics
- [ ] Document new structure
- [ ] Update onboarding materials
- [ ] Set up automated checks
- [ ] Schedule regular audits

---

## 🔗 Related Arsenal Items

### Recommended Pairings

**⚙️ Rule:**
- [Repository Organization Principles](https://github.com/ChrisTansey007/ai-rules-arsenal/blob/main/windsurf/organization/repo-org-principles.md) - 7 core principles this workflow implements

**🤖 Script:**
- [Safe File Migration Script](https://github.com/ChrisTansey007/ai-scripts-arsenal/tree/main/scripts/repository-management/migrate-files) - PowerShell script used in Phase 2 & 3

**🔗 Complete Example:**
- [Repository Organization Example](https://github.com/ChrisTansey007/arsenal-integration-hub/tree/main/examples/repo-organization) - See this workflow in action

### Quick Setup

```bash
# Install all Arsenal repos
curl -sSL https://raw.githubusercontent.com/ChrisTansey007/arsenal-integration-hub/main/scripts/install-all.sh | bash

# Get this workflow + rule + script
cp ~/arsenals/ai-workflows-arsenal/windsurf/project-organization/repo-organize-files.md .windsurf/workflows/
cp ~/arsenals/ai-rules-arsenal/windsurf/organization/repo-org-principles.md .windsurf/rules/
cp ~/arsenals/ai-scripts-arsenal/scripts/repository-management/migrate-files/migrate-files-safe.ps1 ./scripts/
```

---

**Result: Organized, maintainable repository with 80% reduction in search time and clear onboarding path for new team members.** 🎉
