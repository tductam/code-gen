---
description: Generate project structure, documentation (PRD/SRS/UI Mockup), and code from a requirements file
---

## Input

Raw arguments: `$ARGUMENTS`

### Argument Parsing

Parse `$ARGUMENTS` to extract:
1. **Requirements file path** — the first argument (required)
2. **Phase flag** — optional `--phase <name>` where `<name>` is one of: `init`, `docs`, `code`

Examples:
```
/claude-gen:generate requirements.md                    → full pipeline
/claude-gen:generate requirements.md --phase init       → Phase 1 only
/claude-gen:generate requirements.md --phase docs       → Phase 2 only
/claude-gen:generate requirements.md --phase code       → Phase 3 only
```

If no `--phase` flag is provided → run the **full pipeline** (all 3 phases sequentially).

## Pre-flight Checks

1. Verify the requirements file exists and is readable
2. Verify it is a Markdown file with meaningful content
3. Create `.generated/analysis/` and `.generated/docs/` directories if they don't exist
4. If `--phase` is provided, verify it is one of `init`, `docs`, `code`. Otherwise STOP with error.

## Git Checkpoint Helper

Before each phase, create a safety checkpoint.

### For Windows (PowerShell/CMD):

```powershell
# Checkpoint: save current state before making changes
git add -A 2>$null; if ($?) { git stash push -m "claude-gen-checkpoint: before <phase-name> -- $(Get-Date -Format o)" 2>$null }
```

If a phase **fails** (agent reports error or required output files are missing):

```powershell
# Rollback: restore state from before the failed phase
git stash pop 2>$null
```

If a phase **succeeds**, drop the stash:

```powershell
# Success: discard the checkpoint stash
git stash drop 2>$null
```

After the **final successful phase**, create a commit:

```powershell
git add -A 2>$null; if ($?) { git commit -m "claude-gen: generated from <requirements-filename>" 2>$null }
```

### For Unix/Linux/macOS (Bash):

```bash
# Checkpoint: save current state before making changes
git add -A && git stash push -m "claude-gen-checkpoint: before <phase-name> — $(date -Iseconds)" 2>/dev/null || true

# Rollback on failure
git stash pop 2>/dev/null || true

# Success: drop checkpoint
git stash drop 2>/dev/null || true

# Final commit
git add -A && git commit -m "claude-gen: generated from <requirements-filename>" 2>/dev/null || true
```

> **Note**: All git commands gracefully handle cases where git is not initialized or the working directory is not a git repo. Choose the command set appropriate for your operating system.

---

## Phase Execution

### Determine Phases to Run

| `--phase` value | Phases to run |
|----------------|--------------|
| _(not set)_ | Phase 1 → Phase 2 → Phase 3 |
| `init` | Phase 1 only |
| `docs` | Phase 2 only |
| `code` | Phase 3 only |

Execute phases **sequentially**. Each phase MUST complete before starting the next.

---

### Phase 1 — Init & Explore (via init-explorer agent)

**Skip condition**: Only run if `--phase` is `init` or not set.

**Git checkpoint**: Create checkpoint before starting.

Delegate to the **init-explorer** subagent with the following context:

```
REQUIREMENTS FILE: <requirements-file-path>
TASK: Analyze the existing codebase (if any) or initialize a new project structure based on the requirements.
OUTPUT TO: .generated/analysis/
```

**Post-phase validation**:
1. Verify `.generated/analysis/project-overview.md` exists
2. Verify `.generated/analysis/tech-stack.md` exists
3. Verify `.generated/analysis/conventions.md` exists
4. Verify `.generated/analysis/manifest.json` exists and `status` is `"complete"` or `"partial"`

If ANY check fails → **rollback** and STOP with error listing which files are missing.

If ALL checks pass → drop checkpoint stash and proceed.

---

### Phase 2 — Generate Documentation (via gen-doc agent)

**Skip condition**: Only run if `--phase` is `docs` or not set.

**Pre-phase validation** (when running standalone with `--phase docs`):
1. Verify `.generated/analysis/manifest.json` exists with valid `status`
2. If missing → STOP: `"Phase 2 requires completed Phase 1. Run: /claude-gen:generate <file> --phase init"`

**Git checkpoint**: Create checkpoint before starting.

Delegate to the **gen-doc** subagent with the following context:

```
REQUIREMENTS FILE: <requirements-file-path>
ANALYSIS DIR: .generated/analysis/
OUTPUT TO: .generated/docs/
TEMPLATES DIR: templates/
TASK: Generate PRD, SRS, UI Mockups (per screen), and API Contracts based on requirements + project analysis.
```

**Post-phase validation**:
1. Verify `.generated/docs/PRD.md` exists
2. Verify `.generated/docs/SRS.md` exists
3. Verify `.generated/docs/manifest.json` exists and `status` is `"complete"` or `"partial"`
4. Verify at least one file in `.generated/docs/ui-mockups/`
5. Verify at least one file in `.generated/docs/api-contracts/`

If ANY of checks 1-3 fail → **rollback** and STOP with error.
If checks 4-5 fail → **WARN** but continue (some projects may be FE-only or BE-only).

If critical checks pass → drop checkpoint stash and proceed.

---

### Phase 3 — Generate Code (via gen-code agent)

**Skip condition**: Only run if `--phase` is `code` or not set.

**Pre-phase validation** (when running standalone with `--phase code`):
1. Verify `.generated/analysis/manifest.json` exists with valid `status`
2. Verify `.generated/docs/manifest.json` exists with valid `status`
3. If either missing → STOP: `"Phase 3 requires completed Phase 1 and 2. Run: /claude-gen:generate <file> --phase init then --phase docs"`

**Git checkpoint**: Create checkpoint before starting.

Delegate to the **gen-code** subagent with the following context:

```
REQUIREMENTS FILE: <requirements-file-path>
ANALYSIS DIR: .generated/analysis/
DOCS DIR: .generated/docs/
TASK: Generate implementation code based on PRD, SRS, UI Mockups, and API Contracts. Follow existing codebase conventions.
```

**Wait for completion.** Drop checkpoint stash on success.

**Post-phase**: Create final git commit:
```bash
git add -A && git commit -m "claude-gen: generated from <requirements-filename>" 2>/dev/null || true
```

---

## Completion Report

After all executed phases complete, provide a summary:

### If Full Pipeline
1. **Project Analysis**: Key findings from init-explorer
2. **Documents Generated**: List all files in `.generated/docs/` with their purposes
3. **Code Generated**: List all new/modified source files with their paths
4. **Assumptions Made**: List from `.generated/docs/manifest.json` → `assumptions`
5. **Warnings**: Any warnings from manifests
6. **Next Steps**: Suggest what the user should review or do next

### If Single Phase
1. **Phase Completed**: Which phase ran
2. **Files Generated/Modified**: List of output files
3. **Status**: Complete or partial (from manifest)
4. **Next Phase**: Suggest the next phase command if applicable

Format the summary as a clean markdown table or bullet list.
