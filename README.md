# WinkGetJob — Project Documentation
Note: This document is a demonstration of a project from a private repository in my GitHub profile and is shared for informational purposes only.

## Overview

**WinkGetJob** is India's freelance hiring platform. Employers post jobs, review proposals, hire talent, and pay through escrow. Freelancers (job seekers) browse jobs, submit applications, track work time, and manage earnings.

This repository (`winkgetjob`) is the **frontend web application** — a Next.js app that serves public marketing pages, authenticated portals, and API route proxies to a separate backend service (`winkget-backend`).

| Item | Value |
|------|-------|
| **Project name** | `winkgetjob` |
| **Version** | `0.1.0` |
| **Primary URL (dev)** | `https://winkgetjob.vercel.app` |
| **Backend URL (prod)** | `https://winkget-backend.vercel.app` |
| **Currency** | INR (₹) — monetary values stored in paise on the backend |

---

## Technology Stack

### Core Framework

| Technology | Version | Purpose |
|------------|---------|---------|
| [Next.js](https://nextjs.org) | 16.2.4 | React framework (App Router, SSR, API routes) |
| [React](https://react.dev) | 19.2.4 | UI library |
| [TypeScript](https://www.typescriptlang.org) | ^5 | Type-safe JavaScript |
| [Tailwind CSS](https://tailwindcss.com) | ^4 | Utility-first styling (`@tailwindcss/postcss`) |

### UI & Fonts

| Technology | Purpose |
|------------|---------|
| **Poppins** (Google Font via `next/font`) | Primary typeface |
| **Tailwind CSS v4** | Layout, colors, responsive design (`app/globals.css`) |
| **Inline emoji icons** | Feature/step icons on marketing pages |

### Client Libraries

| Package | Version | Purpose |
|---------|---------|---------|
| `html-to-image` | ^1.11.13 | DOM-to-image capture for work tracking |
| `html2canvas` | ^1.4.1 | Screenshot rendering fallback |

### Tooling

| Tool | Version | Purpose |
|------|---------|---------|
| ESLint | ^9 | Linting (`eslint-config-next`) |
| PostCSS | — | CSS processing (`postcss.config.mjs`) |
| Node.js types | ^20 | Type definitions |

### External Services

| Service | Role |
|---------|------|
| **winkget-backend** | REST API, authentication, database, file storage |
| **Vercel** (typical) | Frontend deployment target |

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Browser (Client)                        │
│  React 19 + Tailwind + WorkTrackingProvider (client state)  │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│              Next.js 16 (winkgetjob)                        │
│  • App Router pages (SSR / RSC)                             │
│  • Middleware (route guards)                                │
│  • API Routes (BFF / proxy layer)                           │
│  • HttpOnly cookie session management                       │
└──────────────────────────┬──────────────────────────────────┘
                           │  BACKEND_URL
┌──────────────────────────▼──────────────────────────────────┐
│              winkget-backend (separate repo)                │
│  • /api/v1/*     — Public APIs                              │
│  • /api/user/*   — Freelancer portal                        │
│  • /api/employer/* — Employer portal                        │
│  • /api/admin/*  — Platform admin                           │
│  • /api/super-admin/* — Super admin                         │
└─────────────────────────────────────────────────────────────┘
```

### BFF Proxy Pattern

Most authenticated API calls go through Next.js route handlers in `app/api/`, which forward requests to the backend using `lib/proxy.ts`. The proxy:

- Reads JWT tokens from **HttpOnly cookies**
- Attaches `Authorization: Bearer <token>` headers
- Refreshes expired tokens automatically (employer and user)
- Returns backend responses to the client

Public endpoints under `/api/v1/` may call the backend directly or serve cached/static data.

---

## User Roles & Portals

| Role | Session cookie | Portal path | Description |
|------|----------------|-------------|-------------|
| **Job Seeker / Freelancer** | `user-session` (`seeker:…`), `user-token` | `/dashboard` | Apply to jobs, track work, earnings, messages |
| **Employer** | `user-session` (`employer:…`), `employer-token` | `/employer` | Post jobs, review applications, payments, work tracking |
| **Admin** | `admin-session` (`Admin:`, `Moderator:`, `Finance:`) | `/admin` | User management, disputes, reports, monitoring |
| **Super Admin** | `sa-session` | `/super-admin` | Full platform control, content, audit logs, escrow |

Route protection is enforced in `middleware.ts` for `/dashboard`, `/employer`, `/admin`, `/super-admin`, and `/login`.

---

## Key Features

### Public / Marketing

- Homepage with job search, categories, featured freelancers, trusted companies
- Job listings and job detail pages (`/jobs`, `/jobs/[id]`)
- Freelancer directory (`/freelancers`, `/freelancers/[id]`)
- Employer profiles (`/employers/[id]`)
- Hire talent landing page (`/hire-talent`)
- Blog (`/blog`)
- Contact, About, Support, Privacy Policy, Terms of Service
- Escrow information page (`/escrow`)
- Login and registration (`/login`, `/register`)

### Employer Portal (`/employer`)

- Dashboard and analytics
- Post and manage jobs
- Review applications and proposals (sent/received)
- Work tracking — approve/reject freelancer sessions and screenshots
- Payments, escrow funding, transactions
- Messages, reviews, profile, and settings

### Freelancer Portal (`/dashboard`)

- Dashboard overview
- Job browsing and applications
- Proposals
- **Work tracking timer** — start/pause/resume/stop with periodic screenshots
- Earnings and withdrawals
- Messages, reviews, profile, and settings

### Admin Portal (`/admin`)

- Dashboard, user management, disputes
- Payments, reports, system monitoring

### Super Admin Portal (`/super-admin`)

- Platform dashboard and health
- User and admin management
- Content management (blog, categories, companies)
- Escrow release/refund, payment transactions
- Disputes, audit logs, feature flags, settings

### Work Tracking (Notable Feature)

Real-time time tracking for freelancers on active jobs:

- Global tracking bar in navbar while a session is active
- Screen capture via `html-to-image` / `html2canvas` and optional display capture
- Screenshots saved every 30 seconds while timer is running
- Employer review and approval of tracking sessions

See `docs/work-tracking-timer.md` and related docs for implementation details.

---

## Project Structure

```
winkgetjob/
├── app/                          # Next.js App Router
│   ├── page.tsx                  # Homepage
│   ├── layout.tsx                # Root layout (Poppins font, Providers)
│   ├── globals.css               # Tailwind + theme variables
│   ├── providers.tsx             # Client providers (WorkTrackingProvider)
│   ├── components/               # Shared UI components
│   ├── dashboard/                # Freelancer portal pages
│   ├── employer/                 # Employer portal pages
│   ├── admin/                    # Admin portal pages
│   ├── super-admin/              # Super admin portal pages
│   ├── jobs/                     # Public job pages
│   ├── freelancers/              # Public freelancer pages
│   ├── employers/                # Public employer pages
│   ├── api/                      # API route handlers (BFF proxies)
│   │   ├── v1/                   # Public API (/api/v1)
│   │   ├── user/                 # Freelancer API proxy
│   │   ├── employer/             # Employer API proxy
│   │   ├── admin/                # Admin API proxy
│   │   ├── auth/                 # Admin & super-admin auth
│   │   └── super-admin/          # Super admin API proxy
│   └── lib/data.ts               # Static/mock public data helpers
├── lib/                          # Shared server/client utilities
│   ├── proxy.ts                  # Backend proxy + token refresh
│   ├── userStore.ts              # User state helpers
│   ├── user/data.ts              # Freelancer data helpers
│   ├── employer/data.ts          # Employer data helpers
│   ├── admin/data.ts             # Admin data helpers
│   ├── super-admin/data.ts       # Super admin data helpers
│   ├── hooks/                    # React hooks
│   └── workTracking/             # Timer, screenshots, provider
├── docs/                         # Feature & API requirement docs
├── middleware.ts                 # Auth route guards
├── next.config.ts                # Next.js configuration
├── tsconfig.json                 # TypeScript config (@/* path alias)
├── postcss.config.mjs            # PostCSS / Tailwind
├── eslint.config.mjs             # ESLint rules
├── BACKEND_API_REQUIREMENTS.md   # Public API spec
└── package.json
```

### Path Alias

TypeScript path alias `@/*` maps to the project root (e.g. `@/lib/proxy`).

---

## API Conventions

### Public API (`/api/v1`)

- Base path: `/api/v1`
- Used for homepage, jobs, freelancers, employers, blog, contact, auth
- Documented in `BACKEND_API_REQUIREMENTS.md`
- Standard response: `{ success, data, message? }` or paginated `{ data, pagination }`

### Authenticated APIs

| Namespace | Base path | Portal |
|-----------|-----------|--------|
| User (freelancer) | `/api/user` | `/dashboard` |
| Employer | `/api/employer` | `/employer` |
| Admin | `/api/admin` | `/admin` |
| Super Admin | `/api/super-admin` | `/super-admin` |

Detailed specs live in `docs/`:

- `docs/user-api-requirements.md`
- `docs/employer-api-requirements.md`
- `docs/admin-api-requirements.md`
- `docs/super-admin-api-requirements.md`
- `docs/work-tracking-timer.md`
- `docs/employer-job-tracking.md`
- `docs/employer-job-tracking-sessions.md`
- `docs/screenshot-session-tracking.md`

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `BACKEND_URL` | No | `https://winkget-backend.vercel.app` | Backend API base URL |
| `NODE_ENV` | Auto | — | `production` enables secure cookies |

Example (`.env.production`):

```
BACKEND_URL=https://winkget-backend.vercel.app
```

---

## Development

### Prerequisites

- Node.js 20+
- npm (or yarn/pnpm/bun)
- Running `winkget-backend` locally (default `https://winkget-backend.vercel.app`) for full functionality

### Commands

```bash
# Install dependencies
npm install

# Start development server (http://localhost:3000)
npm run dev

# Production build
npm run build

# Start production server
npm start

# Lint
npm run lint
```

### Local Setup

1. Clone the repository
2. Create `.env.local` with `BACKEND_URL=https://winkget-backend.vercel.app` (or your backend URL)
3. Start the backend service
4. Run `npm run dev`

---

## Authentication Flow

1. User logs in via `/login` or registers via `/register`
2. Next.js API route authenticates against the backend
3. JWT access + refresh tokens stored in **HttpOnly cookies** (`user-token`, `employer-token`, etc.)
4. A `user-session` cookie stores role prefix (`seeker:` or `employer:`) for middleware routing
5. API proxies attach Bearer tokens and refresh them when expired
6. Admin/super-admin use separate session cookies (`admin-session`, `sa-session`)

---

## Deployment

The frontend is designed for deployment on **Vercel** (or any Node.js host supporting Next.js 16). Set `BACKEND_URL` to the production backend URL. Cookies use `secure: true` in production.

---

## Related Repositories & Docs

| Resource | Location |
|----------|----------|
| Backend service | `winkget-backend` (separate repo, deployed at `winkget-backend.vercel.app`) |
| Public API spec | `BACKEND_API_REQUIREMENTS.md` |
| Portal API specs | `docs/*.md` |
| Agent / Next.js notes | `AGENTS.md` |

---

## License & Status

Private project (`"private": true` in `package.json`). Currently at version **0.1.0** — active development.
