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
