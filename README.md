# 🚀 fermi — project operations dashboard

Real-time visibility into projects, teams, capacity, timelines, and risk. Built on the **Gravity** design system.

[![React 19](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://react.dev)
[![Vite 7](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)](https://vite.dev)
[![Tailwind CSS 4](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com)
[![Supabase](https://img.shields.io/badge/Supabase-REST-3FCF8E?logo=supabase&logoColor=white)](https://supabase.com)

## Modules

| Module | What it does |
|--------|-------------|
| **Dashboard** | At-a-glance metrics, project status, attention alerts |
| **Projects** | Active project tracking with status and progress |
| **Tasks** | Task management with assignment and priority |
| **Capacity** | Team workload and allocation visibility |
| **Timeline** | Project milestones and scheduling |
| **Risk** | Risk register with severity tracking |
| **Crisis** | Incident management and response |
| **Team** | Team directory and roles |

## Quick Start

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

### Environment

```
VITE_SUPABASE_URL=your-supabase-url
VITE_SUPABASE_ANON_KEY=your-anon-key
```

Auth is restricted to `@spacekayak.xyz` emails.

## Architecture

```
src/
├── components/
│   ├── dashboard/      # Metrics, status cards, alerts
│   ├── projects/       # Project list + detail views
│   ├── tasks/          # Task board and assignment
│   ├── capacity/       # Workload heatmap and allocation
│   ├── timeline/       # Gantt-style milestone view
│   ├── risk/           # Risk register and severity matrix
│   ├── crisis/         # Incident tracker and response log
│   ├── team/           # Directory, roles, availability
│   ├── settings/       # Basecamp connection and sync config
│   ├── auth/           # Login and session management
│   ├── layout/         # Shell, sidebar, navigation
│   ├── modals/         # Shared modal components
│   └── ui/             # Primitives (buttons, inputs, cards)
├── contexts/           # React context providers
├── data/               # Static data and fixtures
├── lib/                # Supabase client, utilities
└── main.jsx            # Entry point
```

## Stack

| Layer | Tech |
|-------|------|
| UI | React 19 |
| Bundler | Vite 7 |
| Styling | Tailwind CSS 4 |
| Icons | Lucide React |
| Data | Supabase (REST) |
| Sync | Basecamp 3 API |
| Design System | Gravity (Leonardo tokens) |

## Design System — Gravity

| Token | Value |
|-------|-------|
| Primary | `#2A7A5B` — forest green |
| Surface | `#F6F5F2` — warm off-white |
| Border | `#E8E5E0` |
| Serif | Cormorant Garamond |
| Body | DM Sans |
| Mono | DM Mono |
| Radius | `5px` |

## Deployment

Vercel. Pushes to `main` trigger production builds.
