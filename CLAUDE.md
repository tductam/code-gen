# Claude Gen Plugin

Plugin tạo project structure, tài liệu (PRD/SRS/UI Mockup), và code từ file mô tả yêu cầu.

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
2. **gen-doc**: Tạo PRD, SRS, UI Mockup, API Contracts
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
└── docs/                  # Output Agent 2
    ├── PRD.md
    ├── SRS.md
    ├── manifest.json       # Validation manifest
    ├── api-contracts/
    │   └── <module>.md
    └── ui-mockups/
        └── screen-<name>.md
```

Agent 3 ghi code trực tiếp vào source tree của project.

## Safety Features

- **Validation**: Mỗi agent tạo `manifest.json` — agent tiếp theo kiểm tra trước khi chạy
- **Rollback**: Git checkpoint (stash) được tạo trước mỗi phase — tự động rollback nếu fail
- **Non-destructive**: KHÔNG overwrite file có sẵn trừ khi user confirm

## Conventions

- Mỗi agent đọc output của agent trước qua `.generated/` + kiểm tra `manifest.json`
- Luôn match code style hiện tại (indent, quotes, naming)
- Templates nằm trong `templates/` — dùng làm scaffold cho documents
- File yêu cầu đầu vào phải là Markdown (.md)

## Requirements File Format

File yêu cầu đầu vào nên có cấu trúc:

```markdown
# Tên Project

## Mô tả
Mô tả ngắn gọn về project.

## Features
- Feature 1: mô tả
- Feature 2: mô tả

## Tech Stack (optional)
- Frontend: React / Next.js / Vue...
- Backend: NestJS / Express / FastAPI...
- Database: PostgreSQL / MongoDB...

## Screens (optional - cho FE)
- Login
- Dashboard
- Settings
- ...

## API Modules (optional - cho BE)
- Auth
- Users
- Products
- ...
```
