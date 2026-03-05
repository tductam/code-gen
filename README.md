# Claude Gen Plugin

> Generate project structure, documentation (PRD/SRS/UI Mockup), and code from a requirements file.

## Installation

### Via Marketplace

```bash
# Add the marketplace
/plugin marketplace add tductam/code-gen

# Install the plugin
/plugin install claude-gen@claude-gen-marketplace
```

### Via GitHub

```bash
/plugin marketplace add https://github.com/tductam/code-gen.git
/plugin install claude-gen@claude-gen-marketplace
```

### Via Local Path (for development)

```bash
claude --plugin-dir ./path-to/code-gen
```

## Usage

### Full Pipeline

```
/claude-gen:generate <path-to-requirements-file>
```

### Incremental (run individual phases)

```
/claude-gen:generate requirements.md --phase init    # Phase 1: Analyze/Init only
/claude-gen:generate requirements.md --phase docs    # Phase 2: Gen docs only
/claude-gen:generate requirements.md --phase code    # Phase 3: Gen code only
```

### Examples

```
/claude-gen:generate requirements.md
/claude-gen:generate docs/feature-request.md
/claude-gen:generate requirements.md --phase docs
```

## Pipeline

```
requirements.md → [init-explorer] → [gen-doc] → [gen-code] → Done
```

| Phase | Agent | Description |
|-------|-------|-------------|
| 1 | **init-explorer** | Analyze existing codebase or scaffold new project |
| 2 | **gen-doc** | Generate PRD, SRS, UI Mockups, API Contracts |
| 3 | **gen-code** | Generate implementation code following conventions |

## Output

All intermediate artifacts are saved to `.generated/`:

```
.generated/
├── analysis/              # Phase 1 output
│   ├── project-overview.md
│   ├── frontend-structure.md
│   ├── backend-structure.md
│   ├── tech-stack.md
│   ├── conventions.md
│   └── manifest.json
└── docs/                  # Phase 2 output
    ├── PRD.md
    ├── SRS.md
    ├── manifest.json
    ├── api-contracts/
    │   └── <module>.md
    └── ui-mockups/
        └── screen-<name>.md
```

Phase 3 writes code directly into the project source tree.

## Safety Features

- **Validation**: Each agent produces a `manifest.json` — the next agent checks it before running
- **Rollback**: Git checkpoint (stash) created before each phase — auto-rollback on failure
- **Non-destructive**: Existing files are NOT overwritten without user confirmation

## Requirements File Format

```markdown
# Project Name

## Description
Brief project description.

## Features
- Feature 1: description
- Feature 2: description

## Tech Stack (optional)
- Frontend: React / Next.js / Vue...
- Backend: NestJS / Express / FastAPI...
- Database: PostgreSQL / MongoDB...

## Screens (optional)
- Login
- Dashboard
- Settings

## API Modules (optional)
- Auth
- Users
- Products
```

## Plugin Structure

```
code-gen/
├── .claude-plugin/
│   ├── plugin.json           # Plugin manifest
│   └── marketplace.json      # Marketplace definition
├── skills/
│   └── generate/
│       └── SKILL.md          # Main generate skill
├── agents/
│   ├── init-explorer.md      # Phase 1: Codebase analysis
│   ├── gen-doc.md            # Phase 2: Documentation generation
│   └── gen-code.md           # Phase 3: Code generation
├── templates/
│   ├── prd-template.md
│   ├── srs-template.md
│   ├── ui-mockup-template.md
│   └── api-spec-template.md
├── settings.json             # Default plugin settings
├── CLAUDE.md                 # Plugin instructions
└── README.md                 # This file
```

## Customization

### Templates

Modify files in `templates/` to customize document output format:

- `prd-template.md` — PRD structure
- `srs-template.md` — SRS structure
- `ui-mockup-template.md` — UI mockup layout
- `api-spec-template.md` — API contract format

## License

MIT
