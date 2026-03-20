# FlowVoice - Workflow-Based Task Management System

> A full-stack, voice-enabled task management SaaS with role-based workflows, approval chains, and real-time charts.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18 + Vite + Tailwind CSS |
| Backend | Node.js + Express.js |
| Database | MongoDB + Mongoose |
| Auth | JWT (Bearer tokens, localStorage) |
| Voice | Web Speech API (SpeechRecognition) |
| Charts | Chart.js + react-chartjs-2 |

---

## Quick Start

### Prerequisites
- Node.js 18+ and npm
- MongoDB (local or Atlas)

### 1 - Install dependencies

```bash
# Backend
cd backend
npm install

# Frontend
cd ../frontend
npm install
```

### 2 - Configure environment variables

```bash
cd backend
cp .env.example .env
```

Edit `backend/.env`:

```env
PORT=5000
MONGODB_URI=mongodb://localhost:27017/flowvoice
JWT_SECRET=your_super_secret_jwt_key_change_in_production
JWT_EXPIRE=7d
NODE_ENV=development
```

> For MongoDB Atlas: replace MONGODB_URI with your Atlas connection string.

### 3 - Seed the database

```bash
cd backend
npm run seed
```

This creates 4 users, 1 workflow, and 10 sample tasks.

### 3.1 - Apply organization/workspace schema (Supabase)

Run [backend/schema.supabase.organizations.sql](backend/schema.supabase.organizations.sql) in your Supabase SQL editor.

This adds:
- organizations
- organization_memberships
- organization_invitations
- workspaces
- workspace_members
- tasks.workspace_id (for workspace-scoped task assignment)

It also initializes:
- a default `General` workspace for each organization
- owner membership in the default workspace
- `workspace_tasks` view (workspace-scoped logical task table)

### 3.2 - Load organization/workspace dummy data

Run [backend/seed.supabase.organizations.sql](backend/seed.supabase.organizations.sql) in Supabase SQL editor.

This seeds:
- 2 organizations
- owners/admins/employees
- workspace memberships
- sample invitations
- workspace-scoped tasks + history

Seed login password for dummy users: `password123`

### 4 - Run the servers

**Terminal 1 - Backend (port 5000):**
```bash
cd backend
npm run dev
```

**Terminal 2 - Frontend (port 5173):**
```bash
cd frontend
npm run dev
```

Open **http://localhost:5173**

---

## Default Test Credentials

All passwords: **password123**

| Email | Role | Capabilities |
|-------|------|-------------|
| admin@flowvoice.com | Admin | Full control: manage users, workflows, all tasks |
| manager@flowvoice.com | Manager | Create/assign tasks, approve/reject submissions |
| john@flowvoice.com | Employee | View assigned tasks, update status, submit for approval |
| alice@flowvoice.com | Employee | View assigned tasks, update status, submit for approval |

---

## Application Workflow

```
Task Created (by Manager/Admin)
        |
    Assigned  <-----------------------+
        |                             |
   In Progress  (Employee starts)     |
        |                             | Rejection sends back
 Pending Approval (Employee submits)  |
        |                             |
  Manager/Admin reviews               |
        |                             |
Completed (approved) OR Rejected -----+
```

### Role Permissions

| Action | Employee | Manager | Admin |
|--------|----------|---------|-------|
| View own tasks | Yes | Yes | Yes |
| View all tasks | No | Yes | Yes |
| Create tasks | No | Yes | Yes |
| Assign tasks | No | Yes | Yes |
| Update task status | Self only | Yes | Yes |
| Approve/Reject | No | Yes | Yes |
| Manage users | No | No | Yes |
| Manage workflows | No | No | Yes |

---

## Voice Commands

Click the **microphone button** (bottom-right corner) or the Voice button in the top bar.

> Voice recognition requires Chrome or Edge browser.

### Supported Commands

| Command Pattern | Example | Action |
|----------------|---------|--------|
| create task [title] | "Create task Fix login bug" | Opens new task |
| create task [title] priority [level] | "Create task API docs priority high" | Creates with priority |
| show my tasks | "Show my tasks" | Navigate to tasks (filtered) |
| show tasks | "Show all tasks" | Navigate to all tasks |
| show dashboard | "Show dashboard" | Navigate to dashboard |
| show team | "Show team" | Navigate to team page |
| update task [id] to [status] | "Update task abc123 to in progress" | Update task status |
| approve task [id] | "Approve task abc123" | Approve pending task |
| reject task [id] | "Reject task abc123" | Reject pending task |
| search tasks [query] | "Search tasks dashboard" | Filter tasks by query |

### Status Values for Voice
- "in progress" or "start" -> in_progress
- "pending approval" or "submit" -> pending_approval  
- "complete" or "done" -> completed

---

## API Reference

### Authentication
- POST /api/auth/register  - { name, email, password, role, organizationName?, organizationSlug?, registerAsOrganizationOwner? }
- POST /api/auth/login     - { email, password }
- GET  /api/auth/me        - Authorization: Bearer token
- PUT  /api/auth/profile   - { name, avatar }

Notes:
- Base role is assigned automatically as employee on normal registration.
- Owner flow: send registerAsOrganizationOwner=true (or organizationName). The backend creates organization + owner membership and promotes the user to admin app role.
- Organization admin invites are handled with organization_invitations; invited users can self-register/login and accept invite.

### Organizations and Workspaces
- POST /api/organizations                            - Create organization (owner)
- GET  /api/organizations/me                         - Current organization summary
- POST /api/organizations/invitations                - Invite user to organization
- GET  /api/organizations/invitations/my             - Logged-in user's pending org invites
- POST /api/organizations/invitations/:id/accept     - Accept organization invite
- POST /api/organizations/workspaces                 - Create workspace in organization
- GET  /api/organizations/workspaces                 - List organization workspaces
- GET  /api/organizations/workspaces/:workspaceId/members
- POST /api/organizations/workspaces/:workspaceId/members

### Tasks
- GET    /api/tasks              - ?status=&priority=&search=&page=&limit=
- POST   /api/tasks              - { title, description, priority, assignee, dueDate, workspaceId }
- GET    /api/tasks/:id
- PUT    /api/tasks/:id          - Update task details
- DELETE /api/tasks/:id          - Admin/Manager only
- PUT    /api/tasks/:id/status   - { status, comment }
- PUT    /api/tasks/:id/approve  - { comment } (Admin/Manager)
- PUT    /api/tasks/:id/reject   - { comment } (Admin/Manager)
- GET    /api/tasks/:id/history

### Users (Admin only except /employees)
- GET    /api/users/employees    - All employees (any auth user)
- GET    /api/users              - All users
- GET    /api/users/:id
- PUT    /api/users/:id
- DELETE /api/users/:id

### Workflows (Admin only)
- GET    /api/workflows
- POST   /api/workflows          - { name, description, steps[], isDefault }
- PUT    /api/workflows/:id
- DELETE /api/workflows/:id

### Dashboard
- GET /api/dashboard/stats       - Returns task counts, charts data, recent tasks

---

## Folder Structure

```
tmsv/
|-- backend/
|   |-- config/db.js               Supabase config check
|   |-- controllers/               Supabase business logic
|   |-- middleware/auth.js         JWT protect + authorize
|   |-- routes/                    Express routers
|   |-- routes/voice.js            Whisper transcription route
|   |-- controllers/voiceController.js  Whisper transcription handler
|   |-- seed.js                    Supabase database seeder
|   |-- server.js                  Express entry point
|   |-- .env.example
|   `-- package.json
|
|-- frontend/
|   |-- src/
|   |   |-- components/
|   |   |   |-- charts/            TaskCharts (doughnut + bar)
|   |   |   |-- common/            Layout, Sidebar, TopBar, Modal
|   |   |   |-- tasks/             TaskCard, TaskStatusBadge, WorkflowTimeline
|   |   |   `-- voice/             VoiceCommand (FAB + modal)
|   |   |-- context/               AuthContext, NotificationContext
|   |   |-- hooks/useVoice.js      Whisper + Web Speech API hook
|   |   |-- pages/                 All page components
|   |   |-- services/api.js        Axios API client
|   |   |-- App.jsx                Routes + guards
|   |   `-- index.css              Tailwind + custom styles
|   |-- vite.config.js             Proxy /api -> :5000
|   `-- package.json
|
`-- README.md
```

---

## Security Notes

- Passwords hashed with **bcrypt (10 rounds)**
- JWT tokens stored in localStorage (use httpOnly cookies in production)
- Protected API routes validate token on every request
- Role-based middleware prevents unauthorized access
- Input validation on all auth routes

---

## Workflow Customization

Admins can create custom workflows via the Workflows page:
1. Define workflow name and description
2. Add steps with names and order numbers
3. Mark steps as "approval required"
4. Set as the default workflow for new tasks

---

## Browser Support for Voice

Whisper runs on the backend, so any browser with microphone access can use voice commands.
The app also falls back to the Web Speech API when configured.
And The LLM will complie the text into the json file and that will be considered as a task dynamically.