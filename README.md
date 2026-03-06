# Claude Gen Plugin

> Generate project structure, documentation (BRD/PRD/SRS/UI Mockup), and code from a requirements file.

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
| 2 | **gen-doc** | Generate BRD, PRD, SRS, UI Mockups, API Contracts |
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
├── docs/                  # Phase 2 output
│   ├── BRD.md             # Business Requirements Document
│   ├── PRD.md             # Product Requirements Document
│   ├── SRS.md             # Software Requirements Specification
│   ├── manifest.json
│   ├── api-contracts/
│   │   └── <module>.md
│   └── ui-mockups/
│       └── screen-<name>.md
├── code/                  # Phase 3 metadata
│   └── manifest.json
└── dependencies.json      # Dependencies to install
```

Phase 3 writes code directly into the project source tree.

## Safety Features

- **Validation**: Each agent produces a `manifest.json` — the next agent checks it before running
- **Rollback**: Git checkpoint (stash) created before each phase — auto-rollback on failure
- **Non-destructive**: Existing files are NOT overwritten without user confirmation
- **Cross-platform**: Supports Windows (PowerShell) and Unix/Linux/macOS (Bash)
- **Dependencies Tracking**: Tracks and reports all dependencies that need to be installed

## Requirements File Format

```markdown
# Project Name

## Description
Brief project description.

## Business Context (optional - for BRD)
- **Business Problem**: Problem to solve
- **Target Users**: User personas
- **Business Goals**: Objectives with metrics

## Features
- Feature 1: detailed description
- Feature 2: detailed description

## Tech Stack (optional)
- Frontend: React / Next.js / Vue...
- Backend: NestJS / Express / FastAPI...
- Database: PostgreSQL / MongoDB...

## Screens (optional - for FE)
- Login: Screen description
- Dashboard: Screen description
- Settings: Screen description

## API Modules (optional - for BE)
- Auth: Module description
- Users: Module description
- Products: Module description

## Non-Functional Requirements (optional)
- Performance targets
- Security requirements
- Scalability needs
- Accessibility standards

## Out of Scope (optional)
- Features not included in this version
```

**Example**: See [examples/requirements.md](examples/requirements.md) for a complete example.

## Generated Documents

### BRD (Business Requirements Document)
- Executive summary & business vision
- Business objectives with target metrics
- Stakeholder analysis
- Market analysis & competitive landscape
- Detailed user personas
- Business requirements (BR-001, BR-002, ...)
- Financial analysis & ROI
- Risks and mitigation strategies
- Implementation timeline with Gantt chart

### PRD (Product Requirements Document)
- Product overview & success metrics
- User stories (US-001, US-002, ...) with acceptance criteria
- Feature requirements by module
- Non-functional requirements
- Out of scope items
- Phased delivery timeline
- Cross-references to BRD

### SRS (Software Requirements Specification)
- System architecture diagram
- Frontend specification (tech stack, screens, components)
- Backend specification (tech stack, API endpoints, database)
- Data flow diagrams
- Error handling strategy
- Security specification with auth flows

### UI Mockups (Per Screen)
- Screen flow diagram
- Component tree visualization
- Desktop/Tablet/Mobile layouts (ASCII wireframes)
- Component specifications table
- State management design
- API calls per screen
- User interaction flows

### API Contracts (Per Module)
- Base path & authentication
- Common headers & response formats
- Endpoint specifications with full request/response examples
- Validation rules
- Error responses
- Rate limiting

## Troubleshooting

If you encounter errors during generation, see the detailed guide at:

📖 **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**

Quick fixes:
- Phase failed? → Check manifest for errors, rollback with `git stash pop`, re-run phase
- Validation failed? → Run missing phase: `--phase init` or `--phase docs`
- Git errors? → Initialize git: `git init` or skip checkpoints
- File exists? → Review warnings in manifest, manual merge or delete and re-run

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
│   ├── brd-template.md       # Business Requirements template
│   ├── prd-template.md       # Product Requirements template
│   ├── srs-template.md       # Software Requirements template
│   ├── ui-mockup-template.md # UI Mockup template
│   └── api-spec-template.md  # API Contract template
├── schemas/
│   └── manifest-schema.json  # Manifest validation schema
├── examples/
│   └── requirements.md       # Example requirements file
├── docs/
│   └── TROUBLESHOOTING.md    # Error recovery guide
├── settings.json             # Default plugin settings
├── CLAUDE.md                 # Plugin instructions
└── README.md                 # This file
```

## Manifest Schema

Each phase generates a `manifest.json` following a standardized schema:

```json
{
  "phase": "init | docs | code",
  "status": "complete | partial | failed",
  "timestamp": "ISO 8601 datetime",
  "version": "1.0.0",
  "files_generated": [
    {
      "path": "relative/path",
      "type": "created | modified | deleted",
      "size_bytes": 1234
    }
  ],
  "warnings": [
    {
      "code": "WARNING_CODE",
      "message": "Description",
      "context": {}
    }
  ],
  "errors": [],
  "assumptions": ["List of assumptions made"],
  "metadata": {...}
}
```

Full schema specification: [schemas/manifest-schema.json](schemas/manifest-schema.json)

### Templates

Modify files in `templates/` to customize document output format:

- `prd-template.md` — PRD structure
- `srs-template.md` — SRS structure
- `ui-mockup-template.md` — UI mockup layout
- `api-spec-template.md` — API contract format

## License

MIT
