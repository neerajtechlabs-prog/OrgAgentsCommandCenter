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
