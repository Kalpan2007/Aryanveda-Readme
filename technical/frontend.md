# Frontend Architecture - AryanVeda DMS Web

Current alignment: May 2026 (updated to match actual codebase)

## 1. Stack

- Next.js (App Router)
- React + TypeScript
- TanStack Query
- Zustand
- Axios service layer
- React Hook Form + Zod
- Tailwind CSS

## 2. App Structure (`frontend/src/app`)

Auth routes:
- `/(auth)/login` - Login page
- `/(auth)/forgot-password` - Forgot password page

Dashboard & Analytics:
- `/dashboard` - Role-specific KPI dashboard
- `/leaderboard` - Sales performance leaderboard
- `/performance` - Performance metrics
- `/outlet-wise` - Outlet-wise analytics
- `/inventory-analytics` - Inventory analytics with slow-moving analysis
- `/beat-wise` - Beat-wise performance view

Field Operations:
- `/attendance` - Attendance clock-in/out with selfie photo
- `/beat-visits` - Beat visit management with photo proof
- `/gps-tracking` - GPS location history

Management:
- `/users` - User management
- `/territories` - Territory assignment management

Admin-specific:
- `/admin/attendance` - Admin attendance view and export
- `/admin/team-location` - Team live location tracking
- `/admin/geofence` - Geofence management
- `/admin/performance` - Admin performance metrics
- `/admin/inventory` - Admin inventory overview

Orders:
- `/orders` - Order list
- `/orders/new/primary` - Create primary order
- `/orders/new/secondary` - Create secondary order
- `/orders/[id]` - Order detail

Inventory:
- `/inventory` - Entity inventory management
- `/inventory-intel` - Bulk inventory import

Catalog & SKU:
- `/catalog` - Product catalog
- `/catalog/product-catalogs` - Product catalogs
- `/skus` - SKU master list
- `/skus/new` - Create new SKU
- `/skus/[skuId]/edit` - Edit SKU

Other:
- `/stock-movements` - Stock movement log
- `/notifications` - In-app notifications
- `/announcement` - System announcements

## 3. Session and Auth Handling

- Auth state is maintained in `src/store/auth.store.ts`.
- `AuthProvider` restores token and validates via `/api/auth/me`.
- Login uses backend `/auth/login`.
- Forgot/reset password uses OTP endpoints.
- Optional email prompt is handled through `/auth/email-preference`.
- Deactivated account behavior is handled in app layout/auth flow.

## 4. Role-Based Navigation

Navigation is defined in `src/components/layout/nav-items.ts`.

Visible menu is role-gated for:
- Dashboard
- User Management
- Territories
- SKU Catalog
- SKU Master
- Inventory
- Inventory Intel
- Stock Movements
- Orders
- Notifications
- Email Settings (admin only)

Important:
- frontend role-gating is UX guidance,
- backend authorization is still the source of truth.

## 5. Service Layer (`frontend/src/services` or `frontend/src/api`)

Active service modules:
- `auth.service.ts` - Authentication (login, OTP, password reset)
- `user.service.ts` - User management (CRUD, hierarchy)
- `territory.service.ts` - Territory assignments
- `catalog.service.ts` - Product catalog management
- `sku.service.ts` - SKU management
- `inventory.service.ts` - Inventory operations
- `order.service.ts` - Order operations (primary, secondary)
- `notification.service.ts` - In-app notifications
- `dashboard.service.ts` - Dashboard KPIs
- `email.service.ts` - Email health check
- `stock-movement.service.ts` - Stock movement logs
- `attendance.service.ts` - Attendance clock-in/out
- `location.service.ts` - GPS and geofence operations
- `retailer-visit.service.ts` - Beat and visit tracking
- `announcement.service.ts` - Announcement management

Service layer centralizes API contracts and keeps pages/components thin.

## 6. Implemented Feature Areas

| Feature Area | Web Status |
|---|---|
| Authentication and recovery | Implemented |
| Role dashboards | Implemented |
| Leaderboard | Implemented |
| User and hierarchy management | Implemented |
| Territory assignment management | Implemented |
| Catalog and pricing controls | Implemented |
| SKU legacy management UI | Implemented |
| Inventory and analytics | Implemented |
| Inventory Intel (bulk import) | Implemented |
| Order creation/approval/status lifecycle | Implemented |
| Primary order (Distributor → SS) | Implemented |
| Amount-based stock-in (SS) | Implemented |
| Secondary order (SO/Retailer → Distributor) | Implemented |
| Attendance clock-in/out with selfie | Implemented |
| Beat and visit tracking | Implemented |
| GPS tracking history | Implemented |
| Geofence management | Implemented |
| Team live location | Implemented |
| Notifications feed | Implemented |
| Email health check screen | Implemented |
| Stock movement reporting screen | Implemented |
| Announcements management | Implemented |
| Performance metrics | Implemented |
| Outlet-wise analytics | Implemented |
| Beat-wise reports | Implemented |
| Retailer-SO authorization | Implemented |
| Display audits page | Placeholder (coming phase) |

## 6A. Notification and Email UX Behavior

- notification bell uses lightweight unread count polling (`GET /notifications/unread-count`)
- bell dropdown loads latest notifications via paginated list endpoint (`GET /notifications`)
- individual read and bulk read actions call:
	- `PATCH /notifications/:notificationId/read`
	- `PATCH /notifications/read-all`
- email settings page is admin-only in backend (`GET /email/health`)
- web supports optional email capture/update via `POST /auth/email-preference`

Current delivery scope in web experience:
- in-app notification feed
- SMTP-backed emails
- no browser push or WebSocket live stream path is currently implemented

## 7. API Client Behavior

`src/lib/api-client.ts` handles:
- base URL (`NEXT_PUBLIC_API_URL` + `/api`)
- auth header injection
- consistent error handling
- unauthorized/deactivation flow handling

## 8. Current Gaps / Planned Expansion

- `/audits` is currently UI placeholder only.
- Some backend endpoints are service-ready but not wired in all screens (see `use_endpoints.md`).

