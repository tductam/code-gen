---
description: Generate project structure, documentation (PRD/SRS/UI Mockup), and code from a requirements file
argument-hint: <path-to-requirements-file>
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, TodoWrite, LS
---

## Input

Requirements file path: `$ARGUMENTS`

Read the requirements file at `$ARGUMENTS` first. If the file does not exist or is empty, STOP and report the error.

## Pre-flight Checks

1. Verify the file at `$ARGUMENTS` exists and is readable
2. Verify it is a Markdown file with meaningful content
3. Create `.generated/analysis/` and `.generated/docs/` directories if they don't exist

## Orchestration Pipeline

Execute the following phases **sequentially**. Each phase MUST complete before starting the next.

---

### Phase 1 — Init & Explore (via init-explorer agent)

Delegate to the **init-explorer** subagent with the following context:

```
REQUIREMENTS FILE: $ARGUMENTS
TASK: Analyze the existing codebase (if any) or initialize a new project structure based on the requirements.
OUTPUT TO: .generated/analysis/
```

**Wait for completion.** Verify that `.generated/analysis/project-overview.md` exists before proceeding.

---

### Phase 2 — Generate Documentation (via gen-doc agent)

Delegate to the **gen-doc** subagent with the following context:

```
REQUIREMENTS FILE: $ARGUMENTS
ANALYSIS DIR: .generated/analysis/
OUTPUT TO: .generated/docs/
TEMPLATES DIR: templates/
TASK: Generate PRD, SRS, UI Mockups (per screen), and API Contracts based on requirements + project analysis.
```

**Wait for completion.** Verify that `.generated/docs/PRD.md` and `.generated/docs/SRS.md` exist before proceeding.

---

### Phase 3 — Generate Code (via gen-code agent)

Delegate to the **gen-code** subagent with the following context:

```
REQUIREMENTS FILE: $ARGUMENTS
ANALYSIS DIR: .generated/analysis/
DOCS DIR: .generated/docs/
TASK: Generate implementation code based on PRD, SRS, UI Mockups, and API Contracts. Follow existing codebase conventions.
```

**Wait for completion.**

---

## Completion Report

After all 3 phases complete, provide a summary:

1. **Project Analysis**: Key findings from init-explorer
2. **Documents Generated**: List all files in `.generated/docs/`
3. **Code Generated**: List all new/modified source files
4. **Next Steps**: Suggest what the user should review or do next

Format the summary as a clean markdown table or bullet list.
