# Claude Gen Plugin

Plugin tạo project structure, tài liệu (BRD/PRD/SRS/UI Mockup), và code từ file mô tả yêu cầu.

## Cách sử dụng

### Full Pipeline
```
/claude-gen:generate <đường-dẫn-file-yêu-cầu>
```

### Incremental (chạy từng phase)
```
/claude-gen:generate requirements.md --phase init    # Phase 1: Analyze/Init only
/claude-gen:generate requirements.md --phase docs    # Phase 2: Gen docs only
/claude-gen:generate requirements.md --phase code    # Phase 3: Gen code only
```

Ví dụ:
```
/claude-gen:generate requirements.md
/claude-gen:generate docs/feature-request.md
/claude-gen:generate requirements.md --phase docs
```

## Pipeline

```
requirements.md → [init-explorer] → [gen-doc] → [gen-code] → Done
```

1. **init-explorer**: Phân tích codebase hiện tại hoặc khởi tạo project mới
2. **gen-doc**: Tạo BRD, PRD, SRS, UI Mockup, API Contracts
3. **gen-code**: Sinh code dựa trên tài liệu + conventions hiện có

## Output Directory

Tất cả artifacts trung gian được lưu tại `.generated/`:

```
.generated/
├── analysis/              # Output Agent 1
│   ├── project-overview.md
│   ├── frontend-structure.md
│   ├── backend-structure.md
│   ├── tech-stack.md
│   ├── conventions.md
│   └── manifest.json      # Validation manifest
├── docs/                  # Output Agent 2
│   ├── BRD.md             # Business Requirements Document
│   ├── PRD.md             # Product Requirements Document
│   ├── SRS.md             # Software Requirements Specification
│   ├── manifest.json      # Validation manifest
│   ├── api-contracts/
│   │   └── <module>.md
│   └── ui-mockups/
│       └── screen-<name>.md
├── code/                  # Output Agent 3 metadata
│   └── manifest.json      # Code generation manifest
└── dependencies.json      # Dependencies to install
```

Agent 3 ghi code trực tiếp vào source tree của project.

## Safety Features

- **Validation**: Mỗi agent tạo `manifest.json` theo schema chuẩn — agent tiếp theo kiểm tra trước khi chạy
- **Rollback**: Git checkpoint (stash) được tạo trước mỗi phase — tự động rollback nếu fail
- **Non-destructive**: KHÔNG overwrite file có sẵn trừ khi user confirm
- **Cross-platform**: Hỗ trợ cả Windows (PowerShell) và Unix/Linux (Bash)
- **Dependencies Tracking**: Track và report tất cả dependencies cần cài đặt

## Platform Support

### Windows (PowerShell/CMD)
```powershell
# Git checkpoint commands
git add -A; git stash push -m "checkpoint" 2>$null
git stash pop 2>$null
git stash drop 2>$null
```

### Unix/Linux/macOS (Bash)
```bash
# Git checkpoint commands
git add -A && git stash push -m "checkpoint" 2>/dev/null || true
git stash pop 2>/dev/null || true
git stash drop 2>/dev/null || true
```

## Conventions

- Mỗi agent đọc output của agent trước qua `.generated/` + kiểm tra `manifest.json`
- Manifest format: See `schemas/manifest-schema.json` for specification
- Luôn match code style hiện tại (indent, quotes, naming)
- Templates nằm trong `templates/` — dùng làm scaffold cho documents
- File yêu cầu đầu vào phải là Markdown (.md)

## Requirements File Format

File yêu cầu đầu vào nên có cấu trúc:

```markdown
# Tên Project

## Mô tả
Mô tả ngắn gọn về project.

## Business Context (optional - cho BRD)
- **Business Problem**: Vấn đề cần giải quyết
- **Target Users**: Đối tượng người dùng
- **Business Goals**: Mục tiêu kinh doanh (với metrics)

## Features
- Feature 1: mô tả chi tiết
- Feature 2: mô tả chi tiết

## Tech Stack (optional)
- Frontend: React / Next.js / Vue...
- Backend: NestJS / Express / FastAPI...
- Database: PostgreSQL / MongoDB...

## Screens (optional - cho FE)
- Login: Mô tả màn hình
- Dashboard: Mô tả màn hình
- Settings: Mô tả màn hình

## API Modules (optional - cho BE)
- Auth: Mô tả module
- Users: Mô tả module
- Products: Mô tả module

## Non-Functional Requirements (optional)
- Performance targets
- Security requirements
- Scalability needs
- Accessibility standards

## Out of Scope (optional)
- Features không làm trong phiên bản này
```

**Example**: Xem `examples/requirements.md` để tham khảo format đầy đủ.

## Generated Documents

### BRD (Business Requirements Document)
- Executive summary & business vision
- Business objectives với target metrics
- Stakeholder analysis
- Market analysis & competitive landscape
- User personas chi tiết
- Business requirements (BR-001, BR-002, ...)
- Financial analysis & ROI
- Risks and mitigation
- Implementation timeline với Gantt chart

### PRD (Product Requirements Document)
- Product overview & success metrics
- User stories (US-001, US-002, ...) với acceptance criteria
- Feature requirements theo module
- Non-functional requirements
- Out of scope items
- Phased delivery timeline
- Cross-references với BRD

### SRS (Software Requirements Specification)
- System architecture diagram
- Frontend specification (tech stack, screens, components)
- Backend specification (tech stack, API endpoints, database)
- Data flow diagrams
- Error handling strategy
- Security specification với auth flows

### UI Mockups (Per Screen)
- Screen flow diagram
- Component tree visualization
- Desktop/Tablet/Mobile layouts (ASCII wireframes)
- Component specifications table
- State management design
- API calls per screen
- User interactions flow

### API Contracts (Per Module)
- Base path & authentication
- Common headers & response formats
- Endpoint specifications với full request/response examples
- Validation rules
- Error responses
- Rate limiting

## Dependencies Management

Plugin tự động track dependencies và tạo file `.generated/dependencies.json`:

```json
{
  "frontend": {
    "dependencies": {
      "react-query": "^4.0.0",
      "zod": "^3.22.0"
    },
    "devDependencies": {
      "@types/node": "^18.0.0"
    }
  },
  "backend": {
    "dependencies": {
      "@nestjs/common": "^10.0.0",
      "prisma": "^5.0.0"
    }
  }
}
```

Plugin sẽ report installation commands trong completion report.

## Troubleshooting

Nếu gặp lỗi trong quá trình generate, xem hướng dẫn chi tiết tại:

📖 **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)**

Quick fixes:
- Phase failed? → Check manifest for errors, rollback with `git stash pop`, re-run phase
- Validation failed? → Run missing phase: `--phase init` hoặc `--phase docs`
- Git errors? → Initialize git: `git init` hoặc skip checkpoints
- File exists? → Review warnings in manifest, manual merge hoặc delete and re-run

## Manifest Schema

Mỗi phase tạo `manifest.json` theo schema chuẩn:

```json
{
  "phase": "init | docs | code",
  "status": "complete | partial | failed",
  "timestamp": "ISO 8601",
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
  "errors": [...],
  "assumptions": ["List of assumptions made"],
  "metadata": {
    "requirements_file": "requirements.md",
    ...
  }
}
```

Full schema: `schemas/manifest-schema.json`
