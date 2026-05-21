# Backend Architecture - AryanVeda DMS

Current code alignment: May 2026 (updated to match actual codebase)

## 1. Runtime Stack

- Node.js + Express
- TypeScript
- MongoDB + Mongoose
- JWT auth middleware
- role and hierarchy guards
- Nodemailer SMTP integration
- scheduler for stock movement emails

## 2. Request Pipeline (`backend/src/app.ts`)

1. CORS and request logging
2. JSON + cookie parsing
3. `GET /api/health` (public)
4. `/api/auth/*` router (public + protected mix)
5. Global `authMiddleware` for all remaining `/api/*`
6. Protected module routers
7. Centralized error handler

## 3. Mounted Module Routers

- `/api/auth` - Authentication (login, OTP, password reset)
- `/api/users` - User management (CRUD, hierarchy)
- `/api/skus` - Legacy SKU management (list, get, update)
- `/api/catalog` - Master SKU catalog (admin CRUD, assortment, pricing)
- `/api/inventory` - Inventory management (view, adjust, ledger, analytics, intel-import)
- `/api/dashboard` - Role-specific KPI dashboard
- `/api/orders` - Order management (primary, secondary, amount stock-in, status updates)
- `/api/notifications` - In-app notifications (list, read, unread-count)
- `/api/announcements` - System announcements (admin CRUD)
- `/api/territories` - Territory assignments (CRUD, scoped listing)
- `/api/email` - Email service (health check, send-test, preferences)
- `/api/stock-movement` - Stock movement log (own, downstream, scheduler)
- `/api/retailer-authorization` - Retailer-SO authorization (authorize, revoke, list)
- `/api/pricing` - Pricing management (upload, catalog, bulk-set)
- `/api/attendance` - Attendance tracking (mark, history, summary, export)
- `/api/retailer-visits` - Beat/Retailer visit tracking (visit, beat, performance)
- `/api/location` - GPS/Geofencing (track, geofence, team-live, check)

## 4. Authorization Model

Backend uses layered control:
- role guard: `allowRoles(...)` middleware
- hierarchy scoping: `hierarchyScopeMiddleware()` and service-level checks
- territory assignment checks for cross-branch access

Source principle:

`effective access = role permission + hierarchy scope + territory scope`

### Active User Roles (10 roles)
- Root branch: `admin`, `finance`, `dispatch`
- Sales branch: `nsm`, `rsm`, `asm`, `so`
- Supply chain branch: `super_stockist`, `distributor`, `retailer`

### Hierarchy Chain
- Sales: `admin → nsm → rsm → asm → so`
- Supply: `admin → super_stockist → distributor`

## 5. Auth and Identity

Implemented auth features:
- login, refresh, logout
- forgot password OTP (`send-otp`) and reset-password
- current user profile (`me`, including inactive)
- change-password
- optional email preference update

Identity notes:
- mobile-first login strategy is active
- email is optional for many users
- `hasSeenEmailPrompt` is tracked in user model

## 6. Domain Module Summary

| Module | Status | Key Features |
|---|---|---|
| `auth` | Implemented | login/refresh/logout/me/password/OTP/email-preference/change-password |
| `user` | Implemented | create users, management listing, hierarchy listing, status toggle, subordinates |
| `territory` | Implemented | assignment CRUD, scoped listing, agent territories |
| `catalog` | Implemented | admin master SKU control, stock controls, bulk import, pricing, assortment |
| `sku` | Implemented (legacy path) | list/get, update, feature-flagged deprecated create |
| `inventory` | Implemented | inventory list/detail/value/low-stock/ledger, adjust, threshold, analytics, intel-import |
| `order` | Implemented | primary/secondary/amount-stock-in workflows, approval, status transitions |
| `dashboard` | Implemented | role-specific KPI payload, leaderboard, beat-wise reports |
| `notification` | Implemented | unread/list/read/read-all, POR chain recipients |
| `announcement` | Implemented | admin create, list, read, delete announcements |
| `email` | Implemented | SMTP health, send-test, user preferences |
| `stock-movement` | Implemented | own/downstream movement, scheduler, manual email trigger |
| `retailer-authorization` | Implemented | authorize/revoke SO, list authorized SOs/retailers |
| `pricing` | Implemented | upload pricing config, entity-specific catalog, bulk-set |
| `attendance` | Implemented | clock-in/out with selfie, history, summary, admin export |
| `retailer-visits` | Implemented | retailer visits, beat visits, performance metrics |
| `location` | Implemented | GPS tracking, geofence management, team live location |
| `display-audit` | Partial | model/repository present, no mounted routes |
| `sms` | Placeholder | folder currently empty |
| `whatsapp` | Implemented (service layer) | MSG91 template sender for OTP, orders, user status, product updates, stock alerts |

## 7. Order Workflow States

Current status enum:
- `CREATED` - Order created by requester
- `IN_FINANCE` - Stock-in order forwarded to finance for review
- `APPROVED` - Order approved by supplier/SS/finance
- `DISPATCHED` - Stock dispatched by supplier/dispatch role
- `DELIVERED` - Order delivered and confirmed by receiver
- `CANCELLED` - Order cancelled by creator or rejected by approver

### Status Transitions
- PRIMARY orders: CREATED → IN_FINANCE → APPROVED → DISPATCHED → DELIVERED
- SECONDARY orders: CREATED → DISPATCHED → DELIVERED

Additional stock-in controls:
- admin approve/reject/handover to finance
- finance edit/approve/reject
- dispatch role can dispatch stock-in orders

## 8. Inventory and Alerting

Implemented capabilities:
- inventory ledger retrieval
- low-stock threshold updates
- stock adjustments
- slow-moving stock check trigger
- inventory analytics endpoint + principal filters

Alerting integration:
- notifications module for in-app feed
- email module and stock movement module for mail flows
- whatsapp service for template messages across auth and business events
- threshold-hit emails are triggered from inventory deduction checks
- stock movement scheduler runs daily and dispatches mail every 45 days when due

Notification delivery shape:
- in-app notifications are persisted in MongoDB (`notifications` collection)
- deduplication for repeated events is enforced by `(recipientEntityId, referenceOrderId, type)` unique sparse index
- current channels are in-app feed + SMTP email (no push gateway in backend runtime)

## 9. Boot and Operations (`backend/src/server.ts`)

Startup sequence:
1. connect database
2. verify SMTP (non-blocking)
3. start stock movement scheduler
4. start HTTP server

## 10. Test Surface

`backend/tests` contains broad API/model coverage including:
- auth, users, catalog, inventory, orders, sku
- integration and phase test suites
- replica set smoke and index verification tests

