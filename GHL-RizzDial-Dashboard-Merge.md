# LeadPulse — Complete Project Documentation

> **Last updated:** 2026-04-08
> **Production domain:** `https://dashboard.evrythingai.com`
> **Preview domain:** `https://nurture-radar.lovable.app`

---

## Table of Contents

1. [Project Vision](#1-project-vision)
2. [Tech Stack](#2-tech-stack)
3. [Architecture Overview](#3-architecture-overview)
4. [User Roles & RBAC](#4-user-roles--rbac)
5. [Authentication](#5-authentication)
6. [Routing & Page Map](#6-routing--page-map)
7. [Agency Onboarding Flow](#7-agency-onboarding-flow)
8. [Client Setup Wizard (9-Step)](#8-client-setup-wizard-9-step)
9. [Dashboard Pages — Agency](#9-dashboard-pages--agency)
10. [Dashboard Pages — Location Analytics](#10-dashboard-pages--location-analytics)
11. [Client Embed Portal](#11-client-embed-portal)
12. [Founder Admin Panel](#12-founder-admin-panel)
13. [Integrations](#13-integrations)
14. [Database Schema](#14-database-schema)
15. [Edge Functions](#15-edge-functions)
16. [White-Label / Branding System](#16-white-label--branding-system)
17. [Health Score Engine](#17-health-score-engine)
18. [AI Insights & Recommendations](#18-ai-insights--recommendations)
19. [Data Sync Architecture](#19-data-sync-architecture)
20. [Security & RLS](#20-security--rls)
21. [UI Components & Design System](#21-ui-components--design-system)
22. [Buttons & Actions Reference](#22-buttons--actions-reference)
23. [Hard Rules & Constraints](#23-hard-rules--constraints)

---

## 1. Project Vision

LeadPulse is a **unified performance analytics platform** for sales agencies. It replaces the GoHighLevel (GHL) dashboard by consolidating CRM events, dialer metrics (RizzDial), and marketing data into a single "source of truth." The goal is to maximize client retention through a high-performance, enterprise-grade interface.

**Key value props:**
- Single dashboard for all business metrics
- Real-time health monitoring across all client locations
- Automated AI-driven recommendations
- White-labeled for each agency
- Embeddable client portal (no login required)
- Scalable to thousands of sales companies

---

## 2. Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 18, TypeScript 5, Vite 5 |
| Styling | Tailwind CSS v3, shadcn/ui components |
| State | TanStack React Query, React Context |
| Routing | React Router v6, lazy loading |
| Backend | Lovable Cloud (Supabase) |
| Auth | Supabase Auth (email/password) |
| Database | PostgreSQL via Supabase |
| Edge Functions | Deno (Supabase Edge Functions) |
| Charts | Recharts |
| Theming | next-themes (light/dark mode) |
| Date utils | date-fns |

---

## 3. Architecture Overview

```
Frontend (React SPA) → Supabase JS Client (RLS) + Edge Functions (service_role)
  → Lovable Cloud (Supabase: Postgres, Auth, Storage, Edge Functions)
    → External: GHL OAuth, RizzDial API
```

---

## 4. User Roles & RBAC

Roles stored in `user_roles` table (never on profile/users).

| Role | Scope | Capabilities |
|------|-------|-------------|
| `founder_admin` | Global | View/manage ALL agencies. Impersonation. |
| `agency_owner` | Agency | Full CRUD on agency, locations, members. |
| `agency_admin` | Agency | Same as owner minus billing/deletion. |
| `agency_member` | Agency | View agency data, manage assigned locations. |
| `client_admin` | Location | View own location, manage settings. |
| `client_viewer` | Location | View-only. |

**Key functions:** `is_founder`, `is_agency_owner`, `is_location_owner`, `has_role`, `has_agency_role`

---

## 5. Authentication

- Email/password via Supabase Auth
- Signup stores `full_name` + `agency_name` in metadata
- Email confirmation required (no auto-confirm)
- `AuthContext` provides user/session/signOut
- `ProtectedRoute` guards authenticated pages
- Password reset via `/reset-password`

---

## 6. Routing & Page Map

Public: `/auth`, `/reset-password`, `/auth/connect/callback`, `/embed/dashboard`
Protected: `/onboarding`, `/setup`, `/admin`, `/dashboard`, `/dashboard/clients`, `/dashboard/clients/:locationId`, `/dashboard/overview`, `/dashboard/speed`, `/dashboard/nurture`, `/dashboard/pipeline`, `/dashboard/health`, `/dashboard/settings`

Layout: ThemeProvider → QueryClient → Auth → Impersonation → Router → (ProtectedRoute → Location → Branding → AppShell)

---

## 7. Agency Onboarding (5 steps)

1. Welcome
2. Agency Profile (name, logo, branding)
3. Connect CRM
4. Invite Team
5. Add First Client

On finish: sets `onboarding_completed=true`, routes to `/dashboard`.

---

## 8. Client Setup Wizard (9 Steps)

| # | Step | Weight | Critical |
|---|------|--------|----------|
| 1 | Connect CRM | 15% | Yes |
| 2 | Snapshot Structure (18 pipeline stages) | 14% | Yes |
| 3 | Tag Validation (32 tags) | 14% | Yes |
| 4 | Workflow Validation (15 workflows) | 10% | Yes |
| 5 | Disposition Form | 10% | No |
| 6 | Calling / Dialer | 10% | No |
| 7 | Historical Sync | 14% | Yes |
| 8 | Live Metric Sanity | 8% | No |
| 9 | Final Readiness | 5% | Yes |

**18 Pipeline Stages:** New Lead, Contacted, Qualified, Waiting for Appointment, Appointment Set, Showed, Sold, Transferred, No Show, Canceled, Rescheduled, Not Interested, DND, Nurture, Inbound, Bad Number, Follow Up/Interested, Disqualified

**15 Required Workflows:** Opt-In, Nurture SMS/Email, Nurture Calls, Inbound, Kill Switch, Appt Confirm+Reminder, Disposition Form Submit, No Show, AI Transcript, Text AI ON, Text AI OFF, Send to Power Dialer, Dialer Dispositions, DB Reactivation, Remove Tags

**Sync:** 120s budget per chunk, resume cursors, 365-day default range.

---

## 9. Agency Dashboard Pages

**Agency Overview** (`/dashboard`): KPIs (Total Locations, Total Leads 30d, Avg Health, Needs Attention), sortable locations table, Needs Attention queue (Response/No-Show/Stuck/Disposition).

**Clients** (`/dashboard/clients`): Management table with actions (View, Setup, Settings, Copy Embed URL, Deactivate).

**Client Settings** (`/dashboard/clients/:id`): Basic info, CRM connection, tag mappings, pipeline mappings, custom metrics, benchmarks, embed token, dialer config.

---

## 10. Location Analytics Pages

**TopBar controls:** Search, Location selector, Date range (7/30/90d/YTD), Theme toggle, Notifications.

- **Overview**: 6 primary KPIs + 8 secondary KPIs, 14-day trend, AI Insights panel, export buttons
- **Speed to Lead**: Response distribution, daily trend, calling stats
- **Nurture**: Funnel New→Ready to Book, effectiveness metrics
- **Pipeline**: Full funnel, revenue calc, dispositions, stage table
- **Health Monitor**: CRM/Webhook/Data Freshness/Tag Coverage cards, webhook heatmap

---

## 11. Embed Portal

`/embed/dashboard?token={embed_token}` — 24-byte token, no-login, validated by `embed-data`/`embed-save` edge functions.

Pages (Analytics group): Overview, Speed, Pipeline, Nurture, Dispositions, Health. (Config group): Setup Wizard, Settings. Analytics locked until setup complete. Agency branding applied.

---

## 12. Founder Admin (`/admin`)

View all agencies/locations, impersonate any agency (ImpersonationContext + banner).

---

## 13. Integrations

**GHL:** OAuth 2.0 (preferred) or API key. Dynamic redirect URI resolution. Edge functions: `ghl-oauth-start`, `ghl-oauth-callback`, `ghl-refresh-token`, `ghl-webhook`. Webhook events: contacts, appointments, opportunities, tags, calls.

**RizzDial:** PULL model via `rizdial-sync`/`rizdial-ingest`. Global static bearer token. Metrics: leads, calls, speed_to_lead, pickup_rate, conversation_rate (1m–5m), appt/callback/transfer rates. Branded generically as "Dialer".

**AI:** Lovable AI Gateway via `analyze-metrics` edge function.

---

## 14. Database Schema (28 tables)

agencies, agency_members, ai_agents, appointments, benchmarks, call_events, canonical_disposition_mappings, canonical_stage_mappings, canonical_tag_mappings, client_access, contacts, custom_metrics, dialer_metrics, dispositions, ghl_oauth_tokens, lead_source_mappings, locations, messaging_config, onboarding_checklist_items, onboarding_notes, onboarding_steps, pipeline_events, pipeline_step_mappings, profiles, setup_validations, snapshot_imports, sync_log, tag_events, tag_mappings, user_roles, webhook_log, workflow_confirmations

View: `locations_public` (strips sensitive fields)

Enum `app_role`: founder_admin | agency_owner | agency_admin | agency_member | client_viewer | client_admin

---

## 15. Edge Functions

ghl-oauth-start, ghl-oauth-callback, ghl-refresh-token, ghl-webhook, test-ghl-connection, validate-ghl-account, sync-ghl-contacts, sync-ghl-data, rizdial-ingest, rizdial-sync, embed-data, embed-save, analyze-metrics, test-webhook

All have `verify_jwt = false`.

---

## 16. White-Label / Branding

Per-agency: name, dashboard_title, primary_color (hex→HSL CSS vars), logo_url, favicon_url, custom_domain. Upload to Supabase storage bucket `agency-assets`. BrandingProvider applies runtime.

---

## 17. Health Score Engine

0-100 composite:
- Speed to Lead 30% (<5m=100, ≤15m=75, ≤60m=50, >60m=25)
- Booking Rate 25% (target 30%)
- Show Rate 20% (target 75%)
- Nurture 25%

Labels: ≥75 HEALTHY, ≥50 AT RISK, <50 FAILING

---

## 18. AI Insights & Recommendations

Rule-based (`src/lib/recommendations.ts`): marketing/sales_followup categories × critical/warning/info severities. AI narrative via `AIInsightsPanel` → `analyze-metrics`.

---

## 19. Data Sync

Methods: Manual, Historical Import, Webhook real-time, Scheduled daily (pg_cron 12:00 UTC). Chunked 120s, resume cursors, smart resume. Classifies contacts by tag_mappings + lead_source_mappings.

---

## 20. Security & RLS

Patterns: owner-based, client_access, founder override, service_role. SECURITY DEFINER functions prevent recursion. External URL validation via `navigateToTrustedExternalUrl`.

**Hard rules:** No window.location.origin for public URLs; all redirects use dashboard.evrythingai.com; no roles on profiles; no anonymous signups; no auto-confirm.

---

## 21. UI & Design System

Design tokens in index.css (light/dark). Full shadcn/ui suite. Custom: KpiCard, DashboardHeader, SystemHealthBanner, NeedsAttentionQueue, LeadSourceBreakdown, DispositionBreakdown, AIInsightsPanel, AddCustomMetricDialog, HealthRing, OnboardingStepper. Theme toggle via next-themes.

---

## 22. Buttons & Actions

(Full reference preserved — global controls, agency overview, clients, overview, settings, setup wizard, embed portal actions. See original doc for exhaustive table.)

---

## 23. Hard Rules & Constraints

**Branding:** No Lovable branding anywhere user-facing. Always `https://dashboard.evrythingai.com`. Embed uses agency custom_domain.

**Security:** No roles on profile table. No anonymous signups. No auto-confirm. Server-side validation only.

**Database:** Never edit `src/integrations/supabase/client.ts` or `types.ts`. Never edit `.env`. No FKs to `auth.users`. No modifying reserved schemas. Use triggers not CHECK for time validations.

**Dialer:** Branded "Dialer" in UI (never RizzDial). Base URL normalized.

**Data:** Supabase 1000-row default — paginate for large sets. Server-side filtering for 100k+ locations.
