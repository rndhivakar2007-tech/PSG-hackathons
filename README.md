# SafeHer AI

**Your Safety. Your Control. Your Confidence.**

SafeHer AI is a full-stack women's safety, emergency response, consent-based location sharing, smart navigation, and AI safety intelligence platform delivered as a responsive web application and installable Progressive Web App (PWA).

---

## Core Capabilities

- **Confirmation-Protected Emergency SOS (`/sos`, `/emergencies`)**: Tactile 3D SOS trigger (`Button3DSOS`) with confirmation protection, direct national emergency call shortcut (`112` default, configurable per country), live GPS capture, automatic 4-hour revocable location sharing, honest multi-channel delivery status logs, and full event timelines.
- **Trusted Contact Network & Contact Portal (`/contacts`, `/trusted-portal`)**: Tokenized contact invitations (7-day expiry), primary responder designation, per-channel notification toggles (`notify_sos`, `notify_checkin`, `notify_location`), verification/revocation controls, and one-tap SOS acknowledgement by verified contacts.
- **Consent-Based Temporary Location Sharing (`/map`, `/share/:token`)**: Cryptographic 48-character hex share tokens, configurable expiry (`5–480` minutes), `exact` or `approximate` precision controls, instant one-click revocation, and a dedicated token-verified public viewer (`410 Gone` on expiry/revocation).
- **Interactive Safety Map (`SafetyMapCanvas` on `/map`)**: Built with Leaflet and OpenStreetMap tiles, OpenStreetMap Nominatim location search proxy, `ResizeObserver` tile synchronization, verified safety resources (12 pre-seeded reference venues across Bengaluru, New Delhi, San Francisco, and New York), and administrator-verified incident markers.
- **Smart Navigation & OSRM Route Comparison (`/navigation`)**: Multi-route comparison (up to 3 alternatives for `foot` or `driving`) using OSRM geometry with Haversine corridor matching against verified safety resources (`1.2 km` radius) and verified incident reports (`0.8 km` radius).
- **Gemini AI Safety Intelligence (`/navigation`, `/assistant`)**: Server-side `@google/genai` (`gemini-4-hydrogen`) structured JSON route assessments (`Lower reported concern`, `Moderate reported concern`, `Elevated reported concern`, or `Insufficient information`) with transparent missing-data disclosures and a context-aware SafeHer Assistant that prioritizes immediate action steps in imminent danger.
- **Automated Server-Scheduled Safety Check-ins (`/checkin`)**: Background scheduler (`checkinScheduler.ts`) running every 15 seconds to transition overdue journeys, send reminder alerts, and escalate past configurable grace periods when permitted by user privacy settings.
- **Community Incident Reporting & Moderation (`/reports`, `/admin/incidents`)**: Categorized incident submission (`harassment`, `stalking_concerns`, `unsafe_locations`, `poor_lighting`, `suspicious_activity`, `other_safety_concerns`), anonymous reporting support, pre-review editing lock, and mandatory admin verification before public map visibility.
- **Enterprise 3D UI (Zero-Image Design)**: CSS perspective/elevation system (`TiltCard3D`, `LayerTransition3D`, `HeroShield3D`, `AssistantShield3D`, `Button3DSOS`) with reduced-motion and touch-device safeguards—built strictly without stock photos or decorative images.
- **Administration & Super Administration (`/admin/*`)**: Role-protected KPI overview, Recharts analytics, user suspension and role promotion, incident moderation queue, emergency monitor, safety resources CRUD, audit logs, and live platform configuration.
- **Installable Mobile PWA**: `vite-plugin-pwa` Workbox service worker, Web App Manifest, offline fallback (`/offline.html`), offline banner (`OfflineIndicator`), and Android/Desktop/iOS install flows (`PWAInstallButton`).

---

## Architecture & Tech Stack

- **Frontend (`src/`)**: React 19, TypeScript, Vite 8, Tailwind CSS v4, Motion (`motion/react`), React Router DOM v7, TanStack React Query v5, React Hook Form + Zod, Leaflet `1.9.4`, Recharts `3.11`, Lucide React.
- **Backend (`server.ts`, `server/`)**: Express REST API on port `3000` (`tsx` runtime in dev/prod), Vite middleware integration in dev, `scrypt` password hashing (`crypto.timingSafeEqual`), SHA-256 hashed Bearer session tokens stored strictly in-memory on the client, sliding-window per-IP rate limiting, Zod validation, and audit logging.
- **Database**:
  - **SQLite (`server/db.ts` -> `safeher.db`)**: WAL mode with foreign keys enabled across 15 tables: `profiles`, `sessions`, `password_resets`, `trusted_contacts`, `contact_invitations`, `emergency_events`, `location_shares`, `location_updates`, `safety_checkins`, `incident_reports`, `safety_resources`, `notifications`, `ai_assessments`, `audit_logs`, and `platform_config`.
  - **Supabase Ready (`supabase/`)**: PostgreSQL schema with Row Level Security (RLS) across 12 core tables in `supabase/migrations/001_initial_schema.sql` and a Deno Edge Function reference in `supabase/functions/gemini-safety/index.ts`.

---

## Roles & Access Control

| Role | Capabilities |
| :--- | :--- |
| `registered_user` | Personal safety workspace (`/dashboard`, `/sos`, `/map`, `/navigation`, `/assistant`, `/checkin`, `/contacts`, `/reports`, `/notifications`, `/emergencies`, `/profile`, `/settings`) |
| `trusted_contact` | Authorized responder views on `/trusted-portal` (invitations, authorized SOS acknowledgements, shared coordinates) |
| `administrator` | Admin dashboard (`/admin`), user status moderation (`/admin/users`), incident review (`/admin/incidents`), emergency monitor (`/admin/emergencies`), safety resources CRUD (`/admin/resources`), analytics (`/admin/analytics`) |
| `super_administrator` | All administrator permissions plus role management, audit log inspection (`/admin/audit-logs`), and live platform configuration (`/admin/config`) |

---

## Environment Variables

Copy `.env.example` to `.env` (or configure Secrets in your hosting environment):

| Variable | Required | Description |
| :--- | :--- | :--- |
| `GEMINI_API_KEY` | Yes (for AI) | Server-side Google Gemini API key (`@google/genai`). Never exposed to the browser. |
| `APP_URL` | Optional | Public URL of the hosted application. |
| `ADMIN_BOOTSTRAP_PASSCODE` | Optional | Passcode gating initial Super Administrator creation on `/admin-setup`. |
| `OSRM_BASE_URL` | Optional | Custom OSRM server base URL (defaults to `https://router.project-osrm.org`). |
| `RESEND_API_KEY` | Optional | Transactional email API key (when unset, verification/reset preview tokens are returned in dev and email dispatch reports `not_configured`). |
| `TWILIO_ACCOUNT_SID`, `TWILIO_AUTH_TOKEN`, `TWILIO_FROM_NUMBER` | Optional | SMS gateway credentials (when unset, SMS dispatch logs `not_configured`). |
| `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` | Optional | Hosted Supabase PostgreSQL/Auth deployment variables. |

---

## Getting Started

```bash
# 1. Install dependencies
npm install

# 2. Start development server (Express + Vite middleware on http://localhost:3000)
npm run dev
```

### Build, Type-Check & Production Start

```bash
# TypeScript validation
npm run lint

# Production build (Vite bundle + PWA service worker)
npm run build

# Run production server (serves dist/ and API on port 3000)
npm start
```

---

## Key Project Paths

```text
├── server.ts                          # Express + Vite/static entry point (port 3000)
├── server/
│   ├── auth.ts                        # scrypt hashing, session auth, RBAC middleware, rate limiter, audit logs
│   ├── db.ts                          # SQLite schema (15 tables) & reference data seeder
│   ├── routes/
│   │   ├── authRoutes.ts              # Registration, login, sessions, password reset, onboarding/profile, admin bootstrap
│   │   ├── safetyRoutes.ts            # Dashboard, contacts, trusted portal, SOS, location shares, check-ins, incidents, notifications
│   │   ├── geoAiRoutes.ts             # Verified resources, Nominatim geocoding, OSRM routing, Gemini AI endpoints
│   │   └── adminRoutes.ts             # Overview KPIs, user/role management, incident moderation, resource CRUD, audit logs, config
│   └── services/
│       ├── checkinScheduler.ts        # 15-second background timer for overdue & escalated check-ins
│       └── gemini.ts                  # Server-side @google/genai client & structured safety schemas
├── supabase/
│   ├── migrations/001_initial_schema.sql
│   └── functions/gemini-safety/index.ts
├── public/                            # PWA icons & offline.html fallback
└── src/
    ├── App.tsx                        # 30 client routes with ProtectedRoute role enforcement
    ├── index.css                      # Tailwind v4, design tokens & 3D surface utilities
    ├── components/                    # AppShell, PublicNavbar/Footer, SafetyMapCanvas, Depth3D, UI primitives
    ├── contexts/AuthContext.tsx       # Session state & authentication methods
    ├── hooks/                         # useGeolocation, useOnlineStatus, usePWAInstall
    ├── pages/                         # public/, user/, trusted-contact/, and admin/ pages
    └── services/api.ts                # In-memory Bearer token API client
```

---

## Safety & AI Disclaimer

AI-assisted route safety assessments and SafeHer Assistant responses are informational only, grounded strictly in available database and routing records, and never guarantee personal safety. In imminent danger, contact local emergency services directly (`112` in India/EU, `911` in US/Canada, `999` in UK).

