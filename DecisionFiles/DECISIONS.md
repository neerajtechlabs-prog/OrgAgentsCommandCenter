# Project Decisions Log

## [2026-07-07] Used PostgreSQL instead of MongoDB
**Context:** Need relational data with strong consistency for user transactions.
**Decision:** PostgreSQL
**Alternatives considered:** MongoDB, MySQL
**Reason:** Better support for complex joins and ACID guarantees needed for payments.

## [2026-07-08] API auth via JWT
**Context:** Need stateless auth for mobile + web clients.
**Decision:** JWT with refresh tokens
**Reason:** Avoids server-side session storage, scales better horizontally.

## [2026-07-07] RBAC for backend APIs
**Context:** Need controlled access to tenant-sensitive and admin-only endpoints.
**Decision:** Use JWT authentication plus role-based guards and role decorators.
**Alternatives considered:** Simple auth-only protection, per-route manual checks
**Reason:** Provides consistent enforcement, easier auditing, and cleaner long-term maintainability.

## [2026-07-07] Tenant-based multi-tenancy architecture
**Context:** Need isolated data per lab/tenant while sharing the same application codebase.
**Decision:** Use tenant-specific PostgreSQL schemas with a public tenants registry.
**Alternatives considered:** Separate databases per tenant, shared schema with tenant_id column
**Reason:** Balances isolation, operational simplicity, and scalability for the PathCare multi-tenant model.

## [2026-07-07] Persisted and validated refresh tokens
**Context:** JWT refresh tokens alone don't support revocation (e.g., logout from all devices, token compromise).
**Decision:** Store refresh token hashes + family in a `refresh_tokens` table per tenant; validate on refresh; support rotation chains and bulk revocation.
**Alternatives considered:** Stateless JWT-only refresh; Redis-based token blacklist
**Reason:** Database-backed validation enables immediate revocation, device management, and audit trails. Rotation chains (token families) mitigate compromise. Easier to integrate with existing audit logging.

## [2026-07-07] Critical-only audit logging policy
**Context:** Logging every request or non-security-critical event causes performance impact and audit table bloat.
**Decision:** Restrict audit logging to security-sensitive and admin actions: login/logout/refresh, role changes, user CRUD, tenant access, role denials.
**Alternatives considered:** Log everything; no audit logging; separate audit queue
**Reason:** Balances security visibility with performance. Minimal overhead while capturing all meaningful security events. Easy to expand list if new sensitive actions added later.

## [2026-07-07] Audit log retrieval endpoint for admin inspection
**Context:** Admins need to inspect audit logs directly without database access to verify security events and debug access issues.
**Decision:** Expose GET /audit endpoint protected by JWT + RBAC (SUPER_ADMIN, LAB_ADMIN roles only) with optional filters (limit, action, userId).
**Alternatives considered:** No UI access (DB-only); full unrestricted access
**Reason:** Provides admin visibility into security events while maintaining access control. Filters reduce noise and make troubleshooting easier. Follows principle of least privilege.

## [2026-07-07] Basic login lockout policy
**Context:** The authentication flow needed a first-layer defense against brute-force and password spraying attempts without introducing excessive user friction.
**Decision:** Lock out a user/IP combination after 5 failed login attempts within a 15-minute window, and reset the counter after a successful login.
**Alternatives considered:** No lockout; lockout after 3 attempts; lockout for 24 hours
**Reason:** Five attempts is a practical balance for healthcare staff, reducing brute-force risk while avoiding overly aggressive lockouts from ordinary typos. The window keeps the protection bounded and manageable for support teams.

## [2026-07-09] Tenant-scoped lab configuration CRUD
**Context:** PathCare needed a first-class way to manage lab profiles, departments, and sample types per tenant without leaking data across tenants.
**Decision:** Implement lab, department, and sample type CRUD as tenant-scoped entities with dedicated repositories, services, DTOs, controllers, and module wiring in the backend.
**Alternatives considered:** Store these details in a shared schema with tenant_id filtering, or keep them as unstructured configuration without dedicated CRUD APIs.
**Reason:** Tenant-scoped entities preserve multi-tenancy isolation, provide a clean API surface for lab administration, and align with the existing schema-per-tenant architecture.

## [2026-07-09] Tenant-scoped test catalog CRUD and import workflow
**Context:** The Week 5 test catalog module needed a consistent way to manage tests and parameters per tenant, while also supporting bulk onboarding from CSV files.
**Decision:** Implement tenant-scoped test catalog CRUD with dedicated entities, repositories, services, and controllers, and add CSV-based import plus search endpoints for test discovery.
**Alternatives considered:** Keep test catalog data in a shared table, or support CRUD only without bulk import/search.
**Reason:** The tenant-scoped model preserves isolation, while search and CSV import reduce manual setup effort and make the catalog usable for real lab workflows.

## [2026-07-09] Tenant-scoped doctor and patient master data
**Context:** Week 6 required a reliable way to manage doctors and patients per tenant, including registration, lookup, and history access without cross-tenant leakage.
**Decision:** Implement doctor and patient records as tenant-scoped entities with dedicated CRUD APIs, search endpoints, and a patient UID generation flow.
**Alternatives considered:** Keep doctor/patient data in shared tables with tenant filtering, or only support in-memory/demo data.
**Reason:** Tenant-scoped master data keeps the platform compliant with multi-tenancy isolation and supports real operational workflows such as registration and patient lookup.

## [2026-07-09] Patient UID generation as a deterministic registration identifier
**Context:** Patient registration needed a consistent identifier that could be generated during onboarding and used across the booking and reporting flow.
**Decision:** Generate patient UIDs using the format PT-YYYYMMDD-XXXX, derived from the registration date and a monotonic sequence per tenant.
**Alternatives considered:** Random UUID-only identifiers, or manual entry-based IDs.
**Reason:** The structured UID is human-readable for support teams, easy to search, and still deterministic enough for downstream workflows.

## [2026-07-09] Seeded doctor and patient reference data per tenant
**Context:** Week 6 needed realistic sample data for UI validation and demo readiness without depending on manual imports.
**Decision:** Seed each tenant schema with representative doctor and patient records during bootstrap, alongside existing lab and test catalog seed data.
**Alternatives considered:** No seed data, or only seed users without domain records.
**Reason:** Seeded reference data accelerates local testing and keeps the demo environment closer to real production usage.

## [2026-07-09] Tenant-scoped booking workflow for Week 7
**Context:** Week 7 needed a booking lifecycle that could support patient selection, test selection, booking number assignment, and payment confirmation while remaining tenant-safe.
**Decision:** Implement bookings and booking_tests as tenant-scoped entities with a dedicated booking service, controller, booking-number generation, QR/barcode generation, and payment validation workflow.
**Alternatives considered:** Keep booking data in a shared table with tenant filtering, or implement booking creation without structured identifiers and payment tracking.
**Reason:** A dedicated tenant-scoped booking model supports real operational flows, keeps the multi-tenant isolation model intact, and provides the data needed for later reporting and receipt workflows.

## [2026-07-09] Deterministic booking number generation
**Context:** Booking records needed a human-readable reference number that could be used by staff, patients, and downstream integrations without relying on random IDs.
**Decision:** Generate booking numbers in the format TENANTCODE-YYYYMMDD-XXXX using the tenant slug, booking date, and a monotonic sequence.
**Alternatives considered:** Random UUID-only identifiers, or manual booking numbering.
**Reason:** Deterministic numbers are easier for support teams to read and communicate, while still being unique and compatible with future automation.


## [2026-07-09] Booking list filters & pagination
**Context:** The UI requires listing bookings with search, filtering by status/date ranges, and pagination for large datasets.
**Decision:** Implement server-side filtering and offset-based pagination in the booking repository (`page` + `perPage`) with optional filters: free-text search (bookingNumber, phone, email, patientId via ILIKE), status, and preferredDate range.
**Alternatives considered:** Cursor-based pagination, client-side pagination after fetching larger result sets.
**Reason:** Offset-based pagination is simple to implement and integrates with the existing UI patterns (page numbers). It provides predictable behavior for staff-facing listing screens and is sufficient for expected dataset sizes; cursor pagination can be added later if performance requires it.


## [2026-07-09] Cancellation flow & audit logging
**Context:** Cancellation of bookings must be auditable and must prevent invalid state transitions (e.g., cancelling already cancelled bookings or creating receipts for cancelled bookings).
**Decision:** Make cancellation an atomic update to the `Booking` entity setting `status = CANCELLED` and `cancellationRemark`, validate a non-empty remark, then create an audit log entry that records `oldValues` and `newValues` (including the remark). The operation is synchronous and guarded by NotFound/BadRequest validations in the service layer.
**Alternatives considered:** Record cancellations in a separate table and asynchronously reconcile, or enqueue cancellation via job queue.
**Reason:** Synchronous, atomic updates simplify UX (immediate feedback) and ensure the audit trail captures the exact state change. This design reduces complexity and avoids race conditions during cancellation and subsequent actions (receipts, reports).


## [2026-07-09] Balance receipt handling (atomic receipts)
**Context:** Receipts must update both `BookingReceipt` and the corresponding `Booking`'s `paidAmount`, `paymentVerified`, and potentially `status` without leaving the system in an inconsistent state.
**Decision:** Implement receipt creation inside a tenant-scoped database transaction (`tenantDS.manager.transaction`) that: validates `amount <= remaining`, saves a new `BookingReceipt` with a generated `receiptNumber`, updates the `Booking`'s `paidAmount`, `paymentVerified`, and `status` (CONFIRMED when fully paid). The service validates inputs and throws on invalid operations.
**Alternatives considered:** Offload receipt processing to a queue or perform non-transactional two-step writes.
**Reason:** A DB transaction guarantees atomicity and prevents race conditions or partial updates (double application, lost updates). This approach provides a stronger correctness model for financial data and simplifies client-side error handling.
