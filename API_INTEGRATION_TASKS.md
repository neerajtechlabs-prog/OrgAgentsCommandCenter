# API Integration Tasks

This file summarizes frontend integration work needed against backend APIs based on the frontend `PROGRESS_TRACKING.md` and backend `PROGRESS_TRACKER.md` documents.

Priority legend: High / Medium / Low

## Today's API work log — 2026-07-18
- **Authentication APIs**
  - Hardened login/refresh/logout flow around the auth endpoints and cookie-based session handling.
  - Standardized cookie configuration to reduce 401 and refresh-token failures during local development.
- **Dashboard APIs**
  - Integrated the dashboard UI with real summary and workload data from the backend.
  - Made the dashboard summary endpoint resilient to missing tenant booking tables so it returns safe fallback data instead of failing with a 500.
- **Build/runtime stabilization**
  - Resolved the backend cookie-parser import/build issue so the API server compiles and starts correctly.
- **Current status**
  - Auth and dashboard API wiring are now in place and ready for live verification against the running backend.

- **High Priority**
  - `POST /auth/login` and `POST /auth/refresh` — Authentication
    - Used in: Frontend login flow (`src/features/auth`) and all protected API calls.
    - Notes: Frontend currently mocks auth; integrate real login to obtain access/refresh tokens and persist via `lib/api/axios` interceptors.

  - `GET /api/lab-profile` and `PUT /api/lab-profile` — Owner / Lab profile
    - Used in: Owner Profile page (`src/pages/dashboard/masters/owner-profile`) and Settings > Lab Profile.
    - Status: Implemented in backend and wired to frontend in this sprint; verify mapping of `lab.config` fields.

  - `GET/POST/PUT/DELETE /api/doctors` — Doctor Master CRUD
    - Used in: Doctor Master page (create/edit/list), booking form doctor selector.
    - Notes: Frontend UI exists; wire list and save endpoints; add optimistic updates and error handling.

  - `GET/POST /api/patients` — Patient registration & lookup
    - Used in: Patient registration form, booking form (patient autocomplete), patient detail pages.
    - Notes: Frontend needs patient create and search integration plus UID generation display.

  - Booking endpoints
    - `GET /api/bookings` (list with filters & pagination) — Booking list page
    - `POST /api/bookings` — Create booking (Booking form)
    - `GET /api/bookings/:id` — Booking detail (receipt, report links)
    - `POST /api/bookings/:id/receipts` — Create receipt/payment (Balance Receipt page)
    - Used in: Booking form, Booking list, Balance Receipt, Booking detail, Print/receipt flows.
    - Notes: Wire multi-selector values (tests/doctors/centres) with backend list endpoints; implement save/validation feedback.

- **Medium Priority**
  - `GET /api/tests` and related test catalog endpoints (search, CSV import)
    - Used in: Booking form test selector, Test Master pages.
    - Notes: Backend supports test catalog; frontend should use server search for large catalogs instead of local mocks.

  - `GET /api/reports` and `POST /api/reports` — Request PDF generation and check status
    - Used in: Booking detail (Request report), Reports page, public report preview (`/reports/public/:token`).
    - Notes: Wire request/report status polling and provide download link when ready.

  - `GET /api/mis/day-collection` and related MIS endpoints
    - Used in: Dashboard MIS / Day collection pages (frontend planned Week 12).
    - Notes: Backend MIS module exists; integrate the summary, register, and export endpoints and wire Excel export flow.

  - `GET /api/dashboard/workload` and `GET /api/dashboard/summary`
    - Used in: Workload dashboard, results entry page for TAT and counts.
    - Notes: `summary` endpoint implemented backend-side; front-end must consume and map `workload` and `recentActivity` when available.

- **Low Priority**
  - `GET /api/notifications` and `POST /api/notifications` — Notification logs and send
    - Used in: Notifications admin UI, test trigger flows.

  - `GET /api/labs` and lab CRUD (already exist under `labs` controller)
    - Used in: Admin lab listing and multi-centre setup in settings.

Cross-cutting items & integration notes
- Authentication: update `src/lib/api/axios` to use real login/refresh tokens already present; ensure `X-Tenant-Slug` header is set in requests (already handled by axios interceptor).
- Error handling: show validation errors returned by backend near form fields; use toast or inline alerts for network errors.
- Role gating: ensure PUT/POST endpoints that require `LAB_ADMIN`/`SUPER_ADMIN` show UI controls only to those roles (frontend must read role from `auth` store).
- Feature flags / gradual rollout: for large flows (Booking, PDF reports), integrate incrementally and use mock fallbacks until end-to-end behaviors are stabilized.

Missed/Not-yet-implemented APIs (identified from docs)
- Owner Profile: implemented (backend + frontend wired in this sprint).
- Doctor Master save/list: backend exists but frontend integration pending.
- Patient registration: backend exists; frontend integration pending (auto-complete & UID display).
- Booking dropdown data: endpoints for tests/doctors/centres are available but not wired into booking selectors (frontend uses mocks).

Suggested next actions (ordered)
1. Integrate auth (`/auth/login`) and update `auth` store + `api` interceptor flows (High).
2. Wire Doctor Master list/save endpoints and add list caching (High).
3. Wire Patient create/search and booking patient autocomplete (High).
4. Integrate booking create/list and receipt create flows; validate print/receipt endpoints (Critical).
5. Integrate reports (request + status + public URL) and show report-ready notifications (Medium).
6. Integrate MIS/day-collection and export in Week 12 (Medium).

If you want, I can open PRs for the frontend wiring (start with auth + doctor master) — which should I implement next? 
