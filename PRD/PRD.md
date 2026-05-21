# AryanVeda DMS - PRD (Current Implementation Baseline)

Version: March 2026

This PRD is aligned to the currently implemented codebase and should be treated as the source baseline for next phases.

## 1. Product Objective

Build a role-safe distribution management platform that:
- controls stock and pricing across a hierarchical network,
- links sales supervision with supply execution,
- keeps every movement auditable,
- supports both web and mobile operations on a shared backend.

## 2. Scope

In scope:
- 11-role authorization model
- hierarchy + territory scoped access
- catalog, inventory, orders, dashboard, notifications
- admin and finance stock-in governance
- web and mobile clients

Partially in scope:
- display-audit domain model exists but no mounted API yet

Out of scope for current release:
- active SMS/WhatsApp channel workflows (module placeholders only)

## 3. Personas and Roles

- `admin`: global governance, approvals, global reporting
- `finance`: finance-stage stock-in review and decision
- `dispatch`: stock-in process operational support
- `nsm`, `hod`, `rsm`, `asm`, `so`: sales hierarchy and territory execution
- `super_stockist`, `distributor`, `retailer`: supply chain execution

## 4. Functional Requirements (Implemented)

FR-1 Authentication:
- login, refresh, logout, profile (`me`)
- forgot/reset password via OTP email
- change password
- optional first-login email preference

FR-2 Authorization:
- endpoint role guards
- hierarchy scope checks
- territory assignment scope checks

FR-3 User Management:
- scoped user creation and management
- role-aware visibility list (`/users/management`)
- activation/deactivation

FR-4 Territory Management:
- create/list/get/update/deactivate assignment
- assignment links sales roles to supply entities

FR-5 Catalog Management:
- admin master SKU CRUD
- admin company stock update/add
- admin bulk import preview/import
- principal listing and catalog browsing

FR-6 Assortment and Pricing:
- super stockist assortment enable/disable/bulk
- entity pricing read/set/bulk set

FR-7 Legacy SKU Compatibility:
- list/get SKU available
- update available
- create endpoint deprecated and feature-flagged

FR-8 Inventory Operations:
- inventory list/detail/value/low-stock/ledger
- stock initialize and adjust
- per-SKU threshold update
- slow-moving check trigger
- inventory analytics + principal filters

FR-9 Order Operations:
- primary order create (distributor)
- amount stock-in create (super stockist)
- secondary order create (SO/retailer)
- list/detail/summary
- standard status updates

FR-10 Stock-In Governance:
- admin approve/reject/handover to finance
- finance edit/approve/reject

FR-11 Notifications:
- unread count
- list
- mark one read
- mark all read

FR-12 Email:
- SMTP health endpoint
- transactional email integration for business events

FR-13 Stock Movement Reporting:
- own movement and downstream movement APIs
- admin manual email trigger
- scheduler starts at backend boot

FR-14 Dashboard:
- role-scoped KPI payload from `/dashboard`

FR-15 Web Delivery:
- operational pages for dashboard, users, territories, catalog, skus, inventory, orders, stock movements, notifications
- audits page currently placeholder

FR-16 Mobile Delivery:
- role-gated drawer and tab navigation
- feature modules for dashboard, users, territories, catalog, inventory, orders, notifications, stock movements, profile

## 5. Non-Functional Requirements

NFR-1 Security:
- JWT auth middleware
- rate limits for auth/OTP flows
- password hashing and role checks

NFR-2 Data Integrity:
- inventory ledger tracking
- order status transition validation
- hierarchy and territory guardrails

NFR-3 Performance:
- optimized DB pool/compression/lean query strategy from performance tuning report

NFR-4 Operability:
- health endpoint
- SMTP health endpoint
- scheduler bootstrap and centralized logging

NFR-5 Maintainability:
- modular backend routes/services
- typed frontend/mobile service layers

## 6. Acceptance Criteria (Current Release)

- All route modules mounted in backend are documented and role-mapped.
- Web and mobile feature visibility follows role matrices in code.
- Order and inventory workflows are fully role-scoped and API-enforced.
- Notification and email flows are operational with graceful email-optional behavior.
- Docs stay synchronized with code-level endpoint and role behavior.

## 7. Next Phase Candidates

- expose display-audit APIs and UI workflows
- activate SMS/WhatsApp modules with real providers
- expand mobile parity for finance/dispatch if required by operations
- deeper analytics and drilldown reporting

