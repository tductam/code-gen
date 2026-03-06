# Error Recovery & Troubleshooting Guide

## Quick Reference

| Issue | Cause | Solution |
|-------|-------|----------|
| "init-explorer analysis is missing" | Phase 1 not run or failed | Run `--phase init` first |
| "documentation is missing" | Phase 2 not run or failed | Run `--phase docs` first |
| Git checkpoint failed | Not a git repository | Initialize git or skip checkpoints |
| Files from previous run exist | Re-running plugin | Use incremental mode or clean `.generated/` |
| Manifest validation failed | Corrupted/incomplete manifest | Delete manifest and re-run phase |
| Agent timeout | Large codebase or slow system | Run phases separately with `--phase` flag |
| Dependencies conflict | Version mismatch | Check manifest and manually resolve |

---

## Recovery Workflows

### Scenario 1: Phase Failed Mid-Execution

**Symptoms**:
- Agent reported errors
- Manifest shows `status: "failed"` or `status: "partial"`
- Some files generated, others missing

**Recovery Steps**:

1. **Check the manifest** to see what completed:
   ```powershell
   Get-Content .generated/analysis/manifest.json
   # or for docs phase:
   Get-Content .generated/docs/manifest.json
   ```

2. **Review errors** in manifest:
   ```json
   {
     "errors": [
       {
         "code": "MISSING_FILE",
         "message": "Could not find tech-stack.md"
       }
     ]
   }
   ```

3. **Rollback if needed** (git checkpoint created before phase):
   ```powershell
   # Check stash list
   git stash list

   # Find the checkpoint stash (e.g., stash@{0})
   # Pop it to restore previous state
   git stash pop stash@{0}
   ```

4. **Clean partial output**:
   ```powershell
   # Remove failed phase output
   Remove-Item -Recurse -Force .generated/analysis
   # or
   Remove-Item -Recurse -Force .generated/docs
   # or
   Remove-Item -Recurse -Force .generated/code
   ```

5. **Fix the root cause** (see Troubleshooting section below)

6. **Re-run the phase**:
   ```
   /claude-gen:generate requirements.md --phase init
   ```

---

### Scenario 2: Agent Validation Failed

**Symptoms**:
- Error: "Cannot proceed: init-explorer analysis is missing"
- Error: "Cannot proceed: documentation is missing"
- Manifest not found or invalid

**Recovery Steps**:

1. **Verify manifest exists**:
   ```powershell
   Test-Path .generated/analysis/manifest.json
   Test-Path .generated/docs/manifest.json
   ```

2. **If manifest is missing** → Previous phase didn't complete:
   ```
   # Run the missing phase
   /claude-gen:generate requirements.md --phase init
   # then
   /claude-gen:generate requirements.md --phase docs
   ```

3. **If manifest is corrupted** → Delete and re-run:
   ```powershell
   Remove-Item .generated/analysis/manifest.json
   /claude-gen:generate requirements.md --phase init
   ```

---

### Scenario 3: Git Checkpoint Issues

**Symptoms**:
- "fatal: not a git repository"
- Git stash commands fail

**Why it happens**:
- Working directory is not a git repository
- Git is not installed

**Recovery Steps**:

1. **Option A: Initialize git** (recommended):
   ```powershell
   git init
   git add .
   git commit -m "Initial commit before code-gen"
   ```

2. **Option B: Skip git checkpoints**:
   - Git commands in the plugin use `2>$null` to ignore errors
   - Plugin will continue without checkpoints
   - ⚠️ **WARNING**: No rollback available if phase fails

3. **Manually checkpoint before each phase** (if git not available):
   ```powershell
   # Create backup
   Copy-Item -Recurse . ../code-gen-backup
   ```

---

### Scenario 4: Incremental Updates (Adding Features)

**Symptoms**:
- Want to add new feature to existing codebase
- Previous `.generated/` output exists
- Afraid of overwriting existing work

**Strategy**:

1. **Keep existing analysis** (if project structure unchanged):
   - `.generated/analysis/` can be reused
   - Skip `--phase init`

2. **Regenerate docs** with updated requirements:
   ```powershell
   # Backup existing docs
   Copy-Item -Recurse .generated/docs .generated/docs.backup

   # Update requirements.md with new features

   # Regenerate docs
   /claude-gen:generate requirements.md --phase docs
   ```

3. **Generate only new code**:
   - `gen-code` agent skips overwriting existing files by default
   - Review warnings in manifest for skipped files
   - Manually merge new code with existing code if needed

4. **Compare outputs**:
   ```powershell
   # Compare old vs new docs
   code --diff .generated/docs.backup/PRD.md .generated/docs/PRD.md
   ```

---

### Scenario 5: Dependency Conflicts

**Symptoms**:
- Code generated but imports fail
- "Cannot find module" errors
- Version conflicts with existing dependencies

**Recovery Steps**:

1. **Check dependencies manifest**:
   ```powershell
   Get-Content .generated/code/manifest.json | Select-String -Pattern "dependencies"
   ```

2. **Review existing dependencies**:
   ```powershell
   # Frontend
   Get-Content package.json

   # Backend (if separate)
   Get-Content backend/package.json
   ```

3. **Resolve conflicts**:
   - If package exists with different version → Choose compatible version
   - If package missing → Install from manifest

4. **Install dependencies**:
   ```powershell
   # Get installation commands from manifest warnings
   # Example:
   npm install react-query@^4.0.0 zod@^3.22.0
   npm install -D @types/node@^18.0.0
   ```

5. **Re-run build** to verify:
   ```powershell
   npm run build
   # or
   npm run type-check
   ```

---

## Troubleshooting Common Errors

### Error: "Requirements file not found"

**Cause**: File path incorrect or file doesn't exist

**Solution**:
```powershell
# Check file exists
Test-Path requirements.md

# Use absolute path if needed
/claude-gen:generate D:\path\to\requirements.md

# Or relative path from workspace root
/claude-gen:generate docs/requirements.md
```

---

### Error: "Manifest validation failed"

**Cause**: Manifest missing required fields or corrupted

**Solution**:
```powershell
# View manifest
Get-Content .generated/analysis/manifest.json

# Check schema compliance
# Required fields: phase, status, timestamp, version
# If invalid → delete and re-run phase
Remove-Item .generated/analysis/manifest.json
/claude-gen:generate requirements.md --phase init
```

---

### Error: "Agent timeout" or "Agent did not respond"

**Cause**: 
- Very large codebase (init phase)
- Too many documents to generate (docs phase)
- Too many files to create (code phase)

**Solution**:
1. **Run phases separately** instead of full pipeline:
   ```
   /claude-gen:generate requirements.md --phase init
   # Wait for completion, then:
   /claude-gen:generate requirements.md --phase docs
   # Wait for completion, then:
   /claude-gen:generate requirements.md --phase code
   ```

2. **Reduce scope** in requirements.md:
   - Fewer screens → Fewer UI mockups
   - Fewer API modules → Fewer API contracts
   - Split into multiple requirements files (MVP first)

---

### Error: "No frontend/backend detected"

**Cause**: Init-explorer couldn't find source files (in explorer mode)

**Solution**:
1. **Check project structure**:
   ```powershell
   Get-ChildItem -Recurse -Include "*.tsx","*.ts","*.jsx","*.js","package.json"
   ```

2. **If files exist but not detected** → Check directory structure:
   - Expected: `src/`, `app/`, `pages/`, or similar
   - If non-standard → Agent may set `status: "partial"`

3. **If truly empty project** → Runs in init mode (scaffolds new project)

---

### Error: "File already exists" / "Overwrite skipped"

**Cause**: gen-code agent found existing file and skipped overwrite (safety feature)

**Solution**:
1. **Review the warning** in manifest:
   ```json
   {
     "warnings": [
       {
         "code": "OVERWRITE_SKIPPED",
         "message": "Skipped overwriting existing file",
         "context": {
           "file": "src/app/layout.tsx"
         }
       }
     ]
   }
   ```

2. **Manually merge** new code:
   - Agent may provide a diff or proposed changes in the report
   - Use a merge tool or manually copy needed parts

3. **Or delete file** and re-run (if you want to replace):
   ```powershell
   Remove-Item src/app/layout.tsx
   /claude-gen:generate requirements.md --phase code
   ```

---

### Error: "Template not found"

**Cause**: Template file missing from `templates/` directory

**Solution**:
```powershell
# Verify templates exist
Get-ChildItem templates/

# Expected files:
# - brd-template.md
# - prd-template.md
# - srs-template.md
# - api-spec-template.md
# - ui-mockup-template.md

# If missing → Re-download plugin or restore from repo
```

---

## Best Practices for Error Prevention

### 1. Start Small
- Test with a small requirements file first
- Add features incrementally
- Validate each phase before proceeding

### 2. Use Version Control
- Always initialize git before running plugin
- Commit after each successful phase
- Enables easy rollback

### 3. Incremental Execution
- Use `--phase` flag for large projects
- Review output after each phase
- Adjust requirements if needed

### 4. Keep Requirements Clear
- Be specific about tech stack
- Provide detailed screen descriptions
- Specify API contracts clearly

### 5. Monitor Manifests
- Check `status` after each phase
- Review `warnings` array
- Document `assumptions`

### 6. Backup Before Regeneration
```powershell
# Before re-running any phase
Copy-Item -Recurse .generated .generated.backup
Copy-Item -Recurse src src.backup  # if re-running code gen
```

---

## Manual Cleanup

### Clean All Generated Files
```powershell
# Remove all generated artifacts
Remove-Item -Recurse -Force .generated

# Remove generated code (BE CAREFUL - review first!)
# Only if you want to start fresh
git checkout -- src/  # Restore from git
```

### Clean Specific Phase
```powershell
# Clean init phase
Remove-Item -Recurse -Force .generated/analysis

# Clean docs phase
Remove-Item -Recurse -Force .generated/docs

# Clean code phase (review carefully!)
Remove-Item -Recurse -Force .generated/code
```

---

## Getting Help

### 1. Check Manifest for Clues
```powershell
# View full manifest
Get-Content .generated/analysis/manifest.json | ConvertFrom-Json | Format-List

# Check errors
Get-Content .generated/analysis/manifest.json | Select-String -Pattern "errors"

# Check warnings
Get-Content .generated/analysis/manifest.json | Select-String -Pattern "warnings"

# Check assumptions
Get-Content .generated/analysis/manifest.json | Select-String -Pattern "assumptions"
```

### 2. Review Agent Instructions
- Read `.generated/analysis/` files for init phase insights
- Read `.generated/docs/` for generated specs
- Check if assumptions match your expectations

### 3. Validate Requirements File
- Is it valid Markdown?
- Are all sections present?
- Is tech stack specified?

### 4. Report Issue
Include:
- Requirements file (sanitized)
- Manifest files from all phases
- Error messages
- Plugin version (`plugin.json` → `version`)

---

## Emergency Rollback

If everything goes wrong:

```powershell
# 1. Restore from git checkpoint
git stash list
git stash pop stash@{0}  # Use appropriate stash

# 2. Or hard reset to before plugin run
git log --oneline  # Find commit before "claude-gen: generated from..."
git reset --hard <commit-hash>

# 3. Or restore from manual backup
Remove-Item -Recurse -Force .generated
Remove-Item -Recurse -Force src
Copy-Item -Recurse ..\backup\src .\src
```

---

**Last Updated**: March 6, 2026  
**Plugin Version**: 1.0.0
