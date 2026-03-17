# Fermi

**Operations, Capacity & Project Intelligence**

Internal operations dashboard for SpaceKayak — real-time visibility into projects, team capacity, timelines, risk, and crisis management. Built on the **Gravity** design system.

[![React](https://img.shields.io/badge/React-19-61DAFB?logo=react&logoColor=white)](https://react.dev)
[![Vite](https://img.shields.io/badge/Vite-7-646CFF?logo=vite&logoColor=white)](https://vite.dev)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?logo=tailwindcss&logoColor=white)](https://tailwindcss.com)

## Features

- **Dashboard** — at-a-glance metrics, project status, attention alerts
- **Projects** — active project tracking with status and progress
- **Tasks** — task management with assignment and priority
- **Capacity** — team workload and allocation visibility
- **Timeline** — project milestones and scheduling
- **Risk** — risk register with severity tracking
- **Crisis** — incident management and response
- **Team** — team directory and roles
- **Settings** — Basecamp connection and sync configuration

## Quick Start

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

## Environment Variables

Create a `.env` file:

```
VITE_SUPABASE_URL=your-supabase-url
VITE_SUPABASE_ANON_KEY=your-anon-key
```

## Auth

Sign-in restricted to `@spacekayak.xyz` email addresses only.

## Stack

| Layer | Tech |
|-------|------|
| UI | React 19 |
| Bundler | Vite 7 |
| Styling | Tailwind CSS 4 |
| Icons | Lucide React |
| Data | Supabase (REST) |
| Sync | Basecamp 3 API |
| Design System | Gravity |

## Design System (Gravity)

| Token | Value |
|-------|-------|
| Primary | `#2A7A5B` (forest green) |
| Surface | `#F6F5F2` (warm off-white) |
| Border | `#E8E5E0` |
| Serif | Cormorant Garamond |
| Body | DM Sans |
| Mono | DM Mono |
| Radius | 5px |

## Deployment

Deployed on Vercel. Pushes to `main` trigger production builds.
