# AryanVeda DMS - Product Overview (Current State)

## 1. What This System Delivers

AryanVeda DMS is a dual-branch distribution platform that combines:
- Supply chain execution (stock, pricing, orders, inventory)
- Sales-force governance (territories, hierarchy visibility, role dashboards)

It is implemented as one backend serving web and mobile clients with shared authorization rules.

## 2. Applications in This Workspace

- Backend API (`backend/`): all business workflows and authorization
- Web app (`frontend/`): operational control panel
- Mobile app (`my-expo-app/`): field-friendly operations and monitoring

## 3. Implemented Business Modules

| Module | Current Status | Key Notes |
|---|---|---|
| Auth | Implemented | mobile-first login, forgot/reset password, change-password, optional email prompt state |
| Users and hierarchy | Implemented | role-constrained creation, management listing, activation/deactivation |
| Territories | Implemented | sales-to-supply assignment bridge with scoped lookup |
| Catalog | Implemented | admin master SKU control, bulk import, assortment, pricing chain |
| Legacy SKU | Implemented (limited) | read/update active, create endpoint feature-flagged/deprecated |
| Inventory | Implemented | inventory list/value/low-stock/ledger, adjust, threshold, analytics, slow-moving trigger, intel-import |
| Orders | Implemented | primary, amount stock-in, secondary, admin + finance stock-in workflow |
| Dashboard | Implemented | role-specific KPI payloads, leaderboard, beat-wise reports |
| Notifications | Implemented | unread/list/read/read-all, POR chain recipients |
| Announcements | Implemented | admin CRUD for system-wide announcements |
| Email | Implemented | SMTP health + transactional integrations |
| Stock movement | Implemented | own/downstream views + scheduler + manual email trigger |
| Retailer Authorization | Implemented | retailer authorizes SO, SO views authorized retailers |
| Pricing | Implemented | upload pricing config, entity-specific catalog, bulk-set |
| Attendance | Implemented | clock-in/out with selfie, history, summary, admin export |
| Retailer Visits | Implemented | beat visits with photo, performance metrics |
| Location | Implemented | GPS tracking, geofence management, team live location |
| Display audit | Partial groundwork | model/repository exists, no mounted API routes yet |
| SMS | Placeholder | module folder currently empty |
| WhatsApp | Implemented (service layer) | MSG91 templates used by OTP, orders, user status, product updates, and stock alerts |

## 4. Role System Snapshot

10 roles are active in backend (NOTE: `hod` is NOT implemented):
- Root: `admin`, `finance`, `dispatch`
- Sales: `nsm`, `rsm`, `asm`, `so`
- Supply: `super_stockist`, `distributor`, `retailer`

Core hierarchy:
- `admin → nsm → rsm → asm → so`
- `admin → super_stockist → distributor`

**Note:** Earlier documentation mentioned `hod` (Head of Division) role but it was never implemented in the codebase. The sales hierarchy goes directly from `nsm` to `rsm`.

## 5. Key Operational Flows

1. User lifecycle:
- create by allowed parent roles
- scoped management list
- activation/deactivation

2. Catalog and pricing lifecycle:
- admin maintains master SKUs and company stock
- super stockist manages assortment
- pricing chain delegated from admin -> super stockist -> distributor

3. Order lifecycle:
- Distributor creates primary restock (`/orders/primary`)
- Super stockist creates amount-based stock-in (`/orders/primary/amount`)
- SO/Retailer creates secondary (`/orders/secondary`)
- Admin and Finance govern stock-in approval stages

4. Inventory lifecycle:
- stock updates with ledger trace
- threshold tuning per entity/SKU
- analytics and slow-moving checks

5. Notifications and reporting:
- in-app notification feed
- transactional email events
- WhatsApp template notifications on mapped events
- scheduled stock movement summaries
- role-chain to POR recipient expansion through territory assignments

## 6. Web and Mobile Delivery Notes

Web (`frontend/src/app`) includes all major operational modules and admin email health screen.

Mobile (`my-expo-app`) covers dashboard, users, territories, catalog, inventory, orders, notifications, stock movements, and profile; `finance` and `dispatch` are not currently modeled in mobile role type.

## 7. Current Technical Status

- Backend module architecture is active and test-heavy (`backend/tests`)
- Performance baseline reports exist in `load-test/` and `performance.md`
- Documentation has been aligned to current endpoint and role behavior (May 2026)

**Major Documentation Update (May 2026):**
- Removed references to unimplemented `hod` role
- Added missing API routes: `/announcements`, `/attendance`, `/location`, `/retailer-visits`, `/retailer-authorization`, `/pricing`
- Added missing frontend pages: `/attendance`, `/beat-visits`, `/beat-wise`, `/gps-tracking`, `/performance`, `/leaderboard`, `/outlet-wise`, `/announcement`, and admin-specific pages
- Updated hierarchy chain to reflect actual implementation: `admin → nsm → rsm → asm → so`

