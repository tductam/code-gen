# Task Management System

## Mô tả

Hệ thống quản lý công việc (Task Management System) giúp team quản lý và theo dõi tiến độ công việc một cách hiệu quả. Hệ thống cho phép tạo project, phân công task, theo dõi deadline và báo cáo tiến độ.

## Business Context

**Business Problem**: Team hiện đang sử dụng spreadsheet và email để quản lý task, dẫn đến việc mất thông tin, không đồng bộ và khó theo dõi tiến độ.

**Target Users**: 
- Project Managers: Người quản lý và phân bổ công việc
- Team Members: Người thực hiện công việc
- Stakeholders: Người theo dõi tiến độ dự án

**Business Goals**:
- Tăng 40% hiệu suất làm việc của team
- Giảm 60% thời gian meeting để sync công việc
- Tăng transparency trong quản lý dự án

## Features

### Core Features (MVP)

#### 1. Authentication & Authorization
- Đăng ký tài khoản bằng email/password
- Đăng nhập với JWT token
- Quên mật khẩu và reset password
- Phân quyền: Admin, Project Manager, Member

#### 2. Project Management
- Tạo project mới với tên, mô tả, deadline
- Thêm/xóa thành viên vào project
- Xem danh sách projects (grid/list view)
- Filter projects theo status (Active, Completed, Archived)
- Archive/Unarchive project

#### 3. Task Management
- Tạo task với: title, description, assignee, due date, priority, status
- Priority levels: Low, Medium, High, Urgent
- Status: To Do, In Progress, Review, Done
- Gán task cho member
- Comment vào task
- Upload attachments (images, documents)
- Drag & drop để thay đổi status (Kanban board)
- Filter và sort tasks

#### 4. Dashboard & Reporting
- Dashboard overview: tổng số tasks, tasks đúng hạn/trễ hạn
- Biểu đồ thống kê: task by status, task by priority
- My Tasks: danh sách task được gán cho user hiện tại
- Calendar view: xem tasks theo timeline

#### 5. Notifications
- Notification khi được gán task mới
- Notification khi task sắp đến deadline (1 ngày trước)
- Notification khi có comment mới trên task của mình
- Real-time notification (WebSocket)

### Future Features (Post-MVP)

- Time tracking cho mỗi task
- Gantt chart để xem project timeline
- Integration với Slack/Teams
- Export reports to PDF/Excel
- Mobile app (iOS/Android)
- Recurring tasks (daily, weekly, monthly)
- Task templates
- Subtasks

## Tech Stack

### Frontend
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **UI Library**: Tailwind CSS + shadcn/ui
- **State Management**: Zustand
- **Form Handling**: React Hook Form + Zod
- **Data Fetching**: TanStack Query (React Query)
- **Real-time**: Socket.io-client

### Backend
- **Framework**: NestJS
- **Language**: TypeScript
- **Database**: PostgreSQL 15
- **ORM**: Prisma
- **Authentication**: JWT (access token + refresh token)
- **File Upload**: AWS S3 or local storage
- **Real-time**: Socket.io
- **API Style**: RESTful API

### DevOps
- **Container**: Docker + Docker Compose
- **CI/CD**: GitHub Actions
- **Hosting**: 
  - Frontend: Vercel
  - Backend: Railway / Render
  - Database: Railway / Supabase

## Screens (Frontend)

### Public Screens
1. **Login** (`/login`)
   - Email/password form
   - "Forgot password" link
   - "Sign up" link

2. **Register** (`/register`)
   - Full name, email, password form
   - Email verification

3. **Forgot Password** (`/forgot-password`)
   - Email input
   - Send reset link

4. **Reset Password** (`/reset-password?token=xxx`)
   - New password form

### Authenticated Screens

5. **Dashboard** (`/dashboard`)
   - Overview statistics
   - Recent projects
   - My tasks today
   - Quick actions

6. **Projects List** (`/projects`)
   - Grid/List view toggle
   - Filter by status
   - Search projects
   - "Create new project" button

7. **Project Detail** (`/projects/[id]`)
   - Project header: name, description, members
   - Kanban board: columns for To Do, In Progress, Review, Done
   - Drag & drop tasks between columns
   - Filter/sort controls

8. **Task Detail Modal** (Modal overlay)
   - Task information
   - Description editor
   - Comments section
   - Attachments list
   - Activity log

9. **My Tasks** (`/tasks`)
   - List of tasks assigned to current user
   - Group by: Today, This Week, Upcoming, Overdue
   - Filter by project, status, priority

10. **Calendar View** (`/calendar`)
    - Monthly calendar with tasks marked on dates
    - Click date to see tasks

11. **Reports** (`/reports`)
    - Select project
    - Date range selector
    - Charts: completion rate, workload distribution
    - Export button

12. **Settings** (`/settings`)
    - Profile settings (name, email, avatar)
    - Password change
    - Notification preferences

## API Modules (Backend)

### Auth Module
- `POST /api/auth/register` - Register new user
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout
- `POST /api/auth/refresh` - Refresh access token
- `POST /api/auth/forgot-password` - Send reset password email
- `POST /api/auth/reset-password` - Reset password with token

### Users Module
- `GET /api/users/me` - Get current user profile
- `PATCH /api/users/me` - Update profile
- `PATCH /api/users/me/password` - Change password
- `GET /api/users` - Get users list (for project member selection)

### Projects Module
- `GET /api/projects` - Get all projects (with pagination, filter)
- `POST /api/projects` - Create new project
- `GET /api/projects/:id` - Get project by ID
- `PATCH /api/projects/:id` - Update project
- `DELETE /api/projects/:id` - Delete project
- `POST /api/projects/:id/members` - Add member to project
- `DELETE /api/projects/:id/members/:userId` - Remove member
- `PATCH /api/projects/:id/archive` - Archive project

### Tasks Module
- `GET /api/tasks` - Get tasks (filter by project, assignee, status, priority)
- `POST /api/tasks` - Create new task
- `GET /api/tasks/:id` - Get task by ID
- `PATCH /api/tasks/:id` - Update task
- `DELETE /api/tasks/:id` - Delete task
- `POST /api/tasks/:id/comments` - Add comment
- `GET /api/tasks/:id/comments` - Get comments
- `POST /api/tasks/:id/attachments` - Upload attachment
- `DELETE /api/tasks/:id/attachments/:attachmentId` - Delete attachment

### Notifications Module
- `GET /api/notifications` - Get notifications for current user
- `PATCH /api/notifications/:id/read` - Mark notification as read
- `PATCH /api/notifications/read-all` - Mark all as read

### Statistics Module
- `GET /api/statistics/dashboard` - Dashboard statistics
- `GET /api/statistics/projects/:id` - Project statistics

## Database Schema (High-level)

### Users
- id, email, password_hash, full_name, avatar_url, role, created_at, updated_at

### Projects
- id, name, description, start_date, end_date, status, created_by, created_at, updated_at

### ProjectMembers (join table)
- id, project_id, user_id, role (owner/member), joined_at

### Tasks
- id, project_id, title, description, assignee_id, creator_id, status, priority, due_date, created_at, updated_at

### Comments
- id, task_id, user_id, content, created_at, updated_at

### Attachments
- id, task_id, filename, file_url, file_size, uploaded_by, uploaded_at

### Notifications
- id, user_id, type, title, content, related_task_id, is_read, created_at

## Non-Functional Requirements

### Performance
- Page load time: < 2 seconds (LCP)
- API response time: < 500ms (p95)
- Support 100 concurrent users

### Security
- Password hashing: bcrypt (10 rounds)
- JWT tokens: access token (15 min), refresh token (7 days)
- HTTPS only
- Input validation on all endpoints
- XSS protection
- CSRF protection

### Scalability
- Database indexing on frequently queried fields
- Pagination for all list endpoints (default: 20 items)
- Caching for static data (user profiles, project lists)

### Accessibility
- WCAG 2.1 AA compliance
- Keyboard navigation
- Screen reader support
- Color contrast ratio: 4.5:1

### Browser Support
- Chrome (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)
- Edge (latest 2 versions)

## Assumptions

- Users have valid email addresses
- File uploads limited to 10MB per file
- Maximum 50 members per project
- Maximum 500 tasks per project
- Notification retention: 30 days

## Out of Scope (for MVP)

- Mobile native apps
- Third-party integrations (Slack, Teams, etc.)
- Advanced reporting (custom reports, export to Excel)
- Time tracking
- Gantt charts
- Team calendar (shared calendar view)
- Multi-language support
- Dark mode (add later)

## Success Metrics

### Launch Metrics (First 3 Months)
- 50+ active users
- 100+ projects created
- 1000+ tasks created
- 80% user retention after 1 month
- Average response time < 300ms

### Quality Metrics
- 95% uptime
- < 1% error rate
- Zero critical security vulnerabilities

---

**Document Version**: 1.0  
**Created**: March 6, 2026  
**Last Updated**: March 6, 2026
