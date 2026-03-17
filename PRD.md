# SpaceKayak Ops — Product Requirements Document (Reverse-Engineered)

> **Status**: Audit Complete · v4 (current) · Branch: `project/spacekayak-ops`
> **Live**: https://spacekayak-ops-dun.vercel.app
> **Author**: Reverse-engineered from `spacekayak-v4.jsx` (4,580 lines)

---

## 1. Product Overview

**SpaceKayak Ops** is an internal operations platform for SpaceKayak, a design/dev agency. It manages projects, tasks, team capacity, timelines, and crisis response — replacing what was likely a mix of spreadsheets, Notion, and Slack threads.

### Who uses it

| Role | Users | Permissions |
|------|-------|-------------|
| Admin | Achyut | Full CRUD, "View As", profile switching, team management |
| Account Manager | Hari, Neel | Full CRUD, "View As", profile switching |
| Leadership | Paul, Saaket | Read-only across all projects |
| Team Member | 13 designers + devs | See only their assigned work |

### What it does today (8 modules)

1. **Dashboard** — Role-aware overview with project health, task snapshots, attention alerts
2. **Projects** — CRUD with phase tracking (Kickoff → Complete), template-based task generation
3. **Tasks** — Assignment, status tracking (7 statuses), priority, hours logging, dependencies
4. **Capacity** — Per-person workload visualization, overload warnings, reassignment suggestions
5. **Timeline** — Gantt-style project bars with today marker and 4-week capacity heatmap
6. **Risk & Resources** — Risk scoring per task, dependency chains, bottleneck detection
7. **Crisis Navigator** — Scenario-based playbooks for people, project, client, and financial crises
8. **Team Management** — Add/edit/deactivate members, roles, capacity limits

---

## 2. Current Architecture

```
┌─────────────────────────────────────────────────────┐
│           SINGLE FILE: spacekayak-v4.jsx            │
│                   4,580 lines                       │
│                                                     │
│  50+ useState hooks                                 │
│  4 useEffect (data load + 3 debounced auto-saves)   │
│  1 useMemo (workload calc)                          │
│  1 useCallback                                      │
│  8 render functions (one per tab)                   │
│  6 modal dialogs                                    │
│  ~15 business logic functions                       │
│                                                     │
│  Supabase REST (raw fetch, no SDK)                  │
│  localStorage for auth tokens                       │
│  Slack webhooks for notifications                   │
└─────────────────────────────────────────────────────┘

Tech stack: React 19 · Vite 7 · Tailwind 4 · Lucide icons
Backend: Supabase (auth + Postgres via REST)
Notifications: Slack Block Kit webhooks
```

---

## 3. Data Model

### Projects
```
{
  id: string              // "proj-1" format
  name: string            // Client name (e.g., "Assurekit")
  type: string            // Compound type (e.g., "Full Rebrand + Multi-page Website")
  isRetainer: boolean     // Ongoing retainer vs. one-off
  startDate: string       // YYYY-MM-DD
  endDate: string         // YYYY-MM-DD (current projected end)
  decidedEndDate: string  // YYYY-MM-DD (original locked deadline)
  phase: enum             // Kickoff | Discovery | Strategy | Branding | Design | Development | QA | Final Delivery | Complete
  progress: number        // 0-100
  team: {
    am: string            // Account manager name
    designTeam: string[]  // Designer names
    devTeam: string[]     // Developer names
  }
  notes: string
  isStartingSoon: boolean
  confirmedStartDate: string | null
  clientDelayDays: number // Accumulated client-caused delays
  archived: boolean
}
```

### Tasks
```
{
  id: string              // "t1" format
  projectId: string       // FK to project
  title: string
  assignedTo: string[]    // Array of team member names
  dueDate: string         // YYYY-MM-DD
  status: enum            // backlog | next-in-line | in-progress | for-review | client-delay | delayed | completed
  priority: enum          // critical | high | medium | low
  estimatedHours: number
  actualHours: number | null
  clientDelayDays: number
  dependsOn: string[]     // Task IDs this depends on
  manualStatus: boolean   // True if status was manually set (prevents auto-override)
}
```

### Team Members
```
{
  id: string              // "tm-1" format
  name: string
  email: string
  role: string            // e.g., "Head of Design", "Developer"
  type: enum              // design | dev | am
  maxProjects: number     // Capacity ceiling (1-3)
  sysRole: enum           // admin | am | leadership | team_member
  active: boolean
}
```

### Historical Data (singleton)
```
{
  completedProjects: []
  taskAccuracy: { [taskType]: { estimatedAvg, actualAvg, variancePercent } }
  teamVelocity: { [personName]: { tasksPerWeek, hoursPerWeek, accuracyRate } }
  commonDelays: [{ reason, frequency, avgDays }]
  riskPatterns: [{ pattern, occurrences, avgImpact }]
}
```

---

## 4. Feature Inventory (What Exists)

### 4.1 Authentication
- Email + password via Supabase Auth REST API
- Sign up / sign in toggle
- JWT stored in `localStorage` (token + email)
- Session restore on page load (validates token with `/auth/v1/user`)
- Email → team member name mapping for role resolution
- Logout clears localStorage and resets all state

### 4.2 Dashboard (role-aware)

**Manager/AM view:**
- 4-column stat grid: Active Projects · Overdue Tasks · Due This Week · Team Overloaded
- "Needs Attention" alert box (overdue + overloaded members)
- Portfolio Health list: each project shows name, type, phase badge, days remaining, progress bar, health status (On Track / Watch / At Risk / Completed)
- Expandable: current task + next 3 upcoming per project
- Quick actions: archive, edit, add task, view all tasks

**Individual contributor view:**
- Same layout, filtered to only their assigned projects/tasks

**"View As" (admin/AM only):**
- Impersonate any team member to see their scoped view
- Banner shows "Viewing as [Name]" with exit button

### 4.3 Projects
- Full CRUD with confirmation dialogs on delete
- Multi-type selection (e.g., "Brand Lite + Landing Page")
- Template-based auto task generation per project type:
  - Brand Lite (5 tasks), Full Rebrand (8), Landing Page (7), Full Website (26 tasks)
- Custom task creation during project setup
- Phase dropdown (9 phases), progress slider (0-100)
- Team assignment: AM (dropdown), design team (multi-select), dev team (multi-select)
- Archive/unarchive toggle
- Retainer flag

### 4.4 Tasks
- Full CRUD, linked to projects
- 7 status values with color-coded badges
- 4 priority levels with color-coded badges
- Multi-person assignment (array of names)
- Estimated hours + actual hours logging
- Dependency tracking (depends on other tasks in same project)
- Status filter bar with count badges per status
- Client delay logging: captures days, pushes project end date
- Hours logging modal: triggered when completing without actual hours
- Historical learning: completed tasks feed `taskAccuracy` and `teamVelocity`

### 4.5 Capacity
- Per-member workload calculation:
  - Project count vs. max projects
  - Active task count (8 tasks = 100%)
  - Returns `Math.max(projectPct, taskPct)` — worst case
- Capacity labels: Available → Has Capacity → Busy → At Capacity → Overloaded
- Filter buttons: All / Overloaded / At Capacity / Has Headroom / Available
- Expandable per-member detail: this week's tasks, next week's tasks, project list
- Workload warning system: triggers on 2+ projects with 4+ tasks each, or 8+ tasks in 2 weeks
- Reassignment suggestions: analyzes task type (design/dev keywords), finds people with <80% capacity

### 4.6 Timeline
- Gantt-style horizontal bar chart
- Projects as bars, color-coded by phase
- Today marker (red vertical line)
- Month markers across top
- 4-week capacity heatmap below: per-member, tasks-per-week, color gradient (gray→green→yellow→orange→red)

### 4.7 Risk & Resources
- Per-task risk scoring (0-100):
  - Days overdue (max 40 pts)
  - Blocked dependencies (15 pts each)
  - Assignee overload (20 pts)
  - Critical priority near deadline (25 pts)
- Risk levels: none / low / medium / high / critical
- Dependency chain visualization
- Overloaded team members list
- At-risk projects list
- Critical bottleneck detection (one person → multiple projects)

### 4.8 Crisis Navigator
- 5 categories × ~5 scenarios each = ~25 crisis playbooks
- Categories: People · Project Execution · Resource & Capacity · Client Relations · Financial
- Each scenario has: severity, step-by-step playbook, comms template key
- Flexibility sliders: Timeline (0-100) and Budget (0-100)
- AI recommendation engine: analyzes sliders + team state → primary action + impact prediction
- Communication templates: 4 types (medical, scope, capacity, escalation) × 3 tones (professional, casual, urgent)
- Copy-to-clipboard for templates

### 4.9 Team Management (admin/AM only)
- Add/edit/deactivate team members
- Fields: name, email, role, type (design/dev/am), max projects, system role
- Grouped display: Design Team · Dev Team · Account Managers
- Capacity bar per member
- Active/inactive toggle (soft delete)

### 4.10 Slack Integration
- Webhook-based (Block Kit payloads)
- Triggers:
  1. Task assigned → notification to assignees
  2. Task reassigned → notification to new assignees only
  3. Task marked delayed → overdue alert
- Toast feedback in UI header (green "Sent" / red "Failed", auto-dismiss 3s)
- Currently configured via env var (empty = silently disabled)

### 4.11 Data Persistence
- Debounced auto-save to Supabase:
  - Projects: 1.5s debounce
  - Tasks: 1.5s debounce
  - Team members: 2s debounce
  - Historical data: 3s debounce
- Data loads on boot from Supabase tables
- Fallback: hardcoded seed data (4 projects, 17 tasks, 16 team members) if DB empty

---

## 5. UX / UI Audit

### Layout
- **Sidebar** (240px, dark gray-900): 8 nav tabs with icons + count badges, quick stats box
- **Header** (white, sticky): logo, search bar, date, Slack toast, profile dropdown
- **Main content**: single-column, scroll, tab-switched

### Color System (current)
Uses default Tailwind palette — no brand tokens. Status colors:
- Backlog: gray · Next-in-line: purple · In Progress: yellow · For Review: indigo
- Client Delay: orange · Delayed: red · Completed: green

### Typography
- Google Fonts Inter (injected via useEffect, not CSS)
- Sizes: text-3xl (headers) → text-xs (labels)
- No type scale, inconsistent weights

### Interactions
- Click-to-expand on dashboard project cards
- Modal dialogs for all forms (6 modals)
- Filter buttons with active state highlighting
- Copy-to-clipboard for crisis templates
- Profile switcher dropdown (hover-reveal)

---

## 6. Critical Problems

### Architecture
| Problem | Impact | Severity |
|---------|--------|----------|
| 4,580-line monolithic component | Unmaintainable, impossible to test | **Critical** |
| 50+ useState hooks, no state management | Race conditions, stale closures, prop drilling nightmare | **Critical** |
| No component decomposition | Every state change re-renders entire app | **High** |
| No TypeScript | Silent type errors, no IDE support | **High** |
| No tests (zero) | Regressions invisible until production | **High** |
| No error boundaries | One crash takes down entire app | **Medium** |

### Security
| Problem | Impact | Severity |
|---------|--------|----------|
| Supabase anon key in client (even via env var, it's public) | Anyone can read/write all data | **Critical** |
| No Row Level Security (RLS) on Supabase | Any authenticated user can access all rows | **Critical** |
| JWT in localStorage | XSS → token theft | **High** |
| No password strength requirements | Weak passwords | **High** |
| No input sanitization | Potential XSS via project names/notes | **Medium** |
| No CSRF protection | Cross-site request forgery | **Medium** |
| No rate limiting | Brute force auth attacks | **Medium** |

### Data
| Problem | Impact | Severity |
|---------|--------|----------|
| Team members hardcoded as seed data | Must redeploy to change team | **High** |
| Names as foreign keys (assignedTo: ["Boris"]) | Name changes break all references | **High** |
| No data validation on write | Corrupt data can enter DB | **Medium** |
| No audit trail | Who changed what, when? Unknown | **Medium** |
| No backup/recovery strategy | Accidental deletions are permanent | **Medium** |
| Historical data as single JSON blob | Can't query, hard to scale | **Low** |

### UX
| Problem | Impact | Severity |
|---------|--------|----------|
| No loading states on data fetches | Users see stale data, think it's frozen | **High** |
| No error states for failed saves | Silent data loss | **High** |
| No mobile responsiveness | Unusable on phone/tablet | **High** |
| Forms lack validation feedback | Users submit incomplete data | **Medium** |
| No keyboard navigation | Accessibility failure | **Medium** |
| No dark mode | Eye strain for night work | **Low** |
| Default Tailwind colors (no brand) | Looks generic | **Low** |
| Timeline not interactive | Can't drag to reschedule | **Low** |

### Performance
| Problem | Impact | Severity |
|---------|--------|----------|
| No memoization on renders | Full re-render on every state change | **High** |
| No lazy loading | Entire app loaded at once (343 KB JS) | **Medium** |
| No virtualization on lists | Will slow with 50+ projects/500+ tasks | **Medium** |
| Debounced saves fire on every keystroke | Unnecessary network calls | **Low** |

---

## 7. Improvement Roadmap

### Phase 1: Foundation (Week 1-2)
> Goal: Make it safe, testable, and maintainable

- [ ] **Enable Supabase RLS** — Row-level security policies per table
- [ ] **Decompose into components** — Extract ~15 components from monolith:
  - `AuthScreen`, `Sidebar`, `Header`, `Dashboard`, `ProjectList`, `ProjectDetail`
  - `TaskList`, `TaskCard`, `CapacityView`, `TimelineGantt`, `RiskPanel`
  - `CrisisNavigator`, `TeamManager`, `ModalShell`, `ProfileSwitcher`
- [ ] **Add state management** — React Context + useReducer (or Zustand)
  - `AuthContext`, `ProjectContext`, `TaskContext`, `TeamContext`, `UIContext`
- [ ] **Add TypeScript** — Migrate JSX → TSX with strict types
- [ ] **Use IDs as foreign keys** — Replace name strings with member IDs everywhere
- [ ] **Add error boundaries** — Wrap each tab section
- [ ] **Add form validation** — Zod schemas for all inputs

### Phase 2: Data Integrity (Week 2-3)
> Goal: Data you can trust

- [ ] **Supabase SDK** — Replace raw fetch with `@supabase/supabase-js`
- [ ] **Optimistic updates** — Immediate UI feedback, rollback on failure
- [ ] **Audit logging** — Track who changed what (Supabase trigger or middleware)
- [ ] **Soft delete everywhere** — Never hard-delete, add `deleted_at` timestamps
- [ ] **Data validation** — Server-side validation via Supabase edge functions
- [ ] **Proper auth flow** — Supabase Auth with PKCE, session cookies, refresh tokens

### Phase 3: Design System (Week 3-4)
> Goal: Andromeda-grade visual quality

- [ ] **Apply Andromeda tokens** — The Swiss Editorial system is already in `index.css`
- [ ] **Component library** — Build reusable UI primitives:
  - `Badge`, `StatusDot`, `CapacityBar`, `Avatar`, `Card`, `DataTable`
  - `Modal`, `Dropdown`, `DatePicker`, `MultiSelect`, `Slider`
- [ ] **Dark mode** — CSS custom properties for theme switching
- [ ] **Responsive design** — Mobile-first layouts, collapsible sidebar
- [ ] **Motion system** — Consistent easing (already defined: ease-smooth, ease-snappy)
- [ ] **Accessibility** — ARIA labels, keyboard nav, contrast ratios, reduced-motion

### Phase 4: Features (Week 4-6)
> Goal: 10x the utility

- [ ] **Real-time updates** — Supabase Realtime subscriptions (multi-user live sync)
- [ ] **Drag-and-drop timeline** — Reschedule by dragging project bars
- [ ] **Kanban board** — Task columns by status, drag between columns
- [ ] **Time tracking** — Start/stop timer per task, not just manual entry
- [ ] **Client portal** — Read-only view for clients to see their project status
- [ ] **Reporting** — Weekly digest email, burn-down charts, velocity graphs
- [ ] **File attachments** — Link deliverables to tasks (Supabase Storage)
- [ ] **Comments/activity feed** — Per-task discussion thread
- [ ] **Slack bot (two-way)** — Slash commands to update tasks from Slack
- [ ] **Smart scheduling** — Auto-suggest task dates based on team availability + historical velocity

### Phase 5: Scale (Week 6+)
> Goal: Production-grade operations tool

- [ ] **Testing** — Unit tests (Vitest), integration tests (Testing Library), E2E (Playwright)
- [ ] **CI/CD** — GitHub Actions: lint, type-check, test, build, deploy
- [ ] **Monitoring** — Error tracking (Sentry), analytics (PostHog)
- [ ] **Performance** — Code splitting, lazy routes, list virtualization
- [ ] **Multi-tenancy** — Support multiple agencies (if expanding beyond SpaceKayak)

---

## 8. Project Types & Task Templates

These are the agency's service offerings and their standard workflows:

| Type | Template Tasks | Total Est. Hours |
|------|---------------|-----------------|
| Brand Lite | 5 tasks (workshop → handover) | ~48h |
| Full Rebrand | 8 tasks (workshop → brand book) | ~124h |
| Landing Page | 7 tasks (IA → handover) | ~78h |
| Full Website | 26 tasks (IA → documentation) | ~388h |
| Video Project | No template (custom) | varies |
| Brand + Website | Composite (both templates) | varies |
| Pitch Deck | No template | varies |
| Product Design | No template | varies |

---

## 9. Team Roster (Current)

| Name | Role | Type | Max Projects | System Role |
|------|------|------|-------------|-------------|
| Shubham | Head of Design | design | 2 | team_member |
| Navaneeth | ACD | design | 2 | team_member |
| Aditi | Brand Designer | design | 2 | team_member |
| Gayatri | Illustrator | design | 2 | team_member |
| Urja | Illustrator | design | 2 | team_member |
| Ashwin | Web Designer | design | 2 | team_member |
| Boris | Web (Extended) | design | 1 | team_member |
| Arina | Illus. (Extended) | design | 1 | team_member |
| Himanshu | Developer | dev | 2 | team_member |
| Karthick | Developer | dev | 2 | team_member |
| Prashant | Developer | dev | 2 | team_member |
| Sumit Yadav | Developer | dev | 3 | team_member |
| Ayan | Developer | dev | 2 | team_member |
| Achyut | Account Manager | am | 3 | admin |
| Hari | Account Manager | am | 3 | am |
| Neel | Account Manager | am | 3 | am |

**Leadership (read-only):** Paul, Saaket

---

## 10. Key Business Logic

### Capacity Formula
```
projectPct = (assignedProjects / maxProjects) × 100
taskPct = (activeTasks / 8) × 100
capacityPct = max(projectPct, taskPct)
```

### Risk Score (per task, 0-100)
```
+max(40) points:  days overdue (4 pts/day, cap 40)
+15 points each:  blocked by incomplete dependency
+20 points:       assignee at >80% capacity
+25 points:       critical priority + due within 3 days
```

### Health Status (per project)
```
Completed:  phase === 'Complete'
At Risk:    any delayed task OR endDate < today
Watch:      any task due within 3 days
On Track:   everything else
```

### Workload Warning Triggers
```
- Person on 2+ projects AND 4+ active tasks per project
- Person has 8+ tasks due in the next 14 days
```

---

## 11. Integration Points

| System | Direction | Method | Purpose |
|--------|-----------|--------|---------|
| Supabase Auth | Read/Write | REST API | Login, signup, session validation |
| Supabase DB | Read/Write | REST API | Projects, tasks, team, history CRUD |
| Slack | Write-only | Webhook | Task assigned, reassigned, delayed notifications |
| localStorage | Read/Write | Browser API | Auth token + email persistence |

---

*This PRD was reverse-engineered from the production codebase on 2026-03-12. It reflects the actual state of the application, not aspirational features.*
