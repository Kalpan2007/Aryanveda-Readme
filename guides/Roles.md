# Roles and Access Matrix (Current Codebase)

This document reflects the implemented role behavior in backend, web, and mobile.

## 1. Role Inventory

| Role | Code | Branch |
|---|---|---|
| Admin | `admin` | Root |
| Finance | `finance` | Root |
| Dispatch | `dispatch` | Root |
| National Sales Manager | `nsm` | Sales Force |
| Regional Sales Manager | `rsm` | Sales Force |
| Area Sales Manager | `asm` | Sales Force |
| Sales Officer | `so` | Sales Force |
| Super Stockist | `super_stockist` | Supply Chain |
| Distributor | `distributor` | Supply Chain |
| Retailer | `retailer` | Supply Chain |

**Note:** `hod` (Head of Division) role is NOT implemented in the codebase despite earlier documentation. The actual sales hierarchy is: `admin → nsm → rsm → asm → so`

## 2. Hierarchy and Manageable Child Roles

Hierarchy chain:
- `admin → nsm → rsm → asm → so`
- `admin → super_stockist → distributor`

Manageable child roles (`MANAGEABLE_CHILD_ROLES`):

| Parent Role | Can Create / Manage |
|---|---|
| `admin` | `finance`, `dispatch`, `nsm`, `rsm`, `asm`, `super_stockist`, `distributor`, `so`, `retailer` |
| `nsm` | `rsm` |
| `rsm` | `asm` |
| `asm` | `so` |
| `super_stockist` | `distributor` |
| `distributor` | `so`, `retailer` |
| `so` | `retailer` |
| `finance`, `dispatch`, `retailer` | None |

## 3. Feature to Role Authorization (Backend)

| Feature / Action | Allowed Roles |
|---|---|
| Create users (`POST /users`) | `admin`, `nsm`, `rsm`, `asm`, `super_stockist`, `distributor`, `so` |
| Toggle user status (`PATCH /users/:entityId/status`) | `admin`, `super_stockist`, `distributor`, `so`, `rsm`, `asm` |
| Territory CRUD (`/territories`) | `admin`, `nsm`, `rsm`, `asm` |
| Catalog admin CRUD + bulk import | `admin` |
| Assortment (`/catalog/assortment/*`) | `super_stockist` |
| Pricing chain (`/catalog/pricing/*`) | `admin`, `super_stockist`, `distributor` |
| Legacy SKU create/update (`/skus`) | `super_stockist` (create is feature-flagged) |
| Inventory analytics (`/inventory/analytics*`) | `super_stockist`, `distributor`, `retailer` |
| Inventory adjust (`/inventory/:entityId/:skuId/adjust`) | `super_stockist` |
| Inventory threshold update | `super_stockist`, `distributor`, `retailer` |
| Create primary order (`/orders/primary`) | `distributor` |
| Create amount stock-in (`/orders/primary/amount`) | `super_stockist` |
| Create secondary order (`/orders/secondary`) | `so`, `retailer` |
| SS order approve (`/orders/:orderId/approve`) | `super_stockist` |
| Admin stock-in controls (`admin-approve`, `admin-reject`, `admin-handover-finance`) | `admin` |
| Finance stock-in controls (`finance-approve`, `finance-reject`) | `finance` |
| Finance edit (`/orders/:orderId/finance-edit`) | `finance`, `super_stockist`, `dispatch` |
| Notifications list/read | All authenticated roles |
| Email health (`/email/health`) | `admin` |
| Stock movement own | `super_stockist`, `distributor`, `retailer` |
| Stock movement downstream | `admin`, `nsm`, `rsm`, `asm`, `so`, `super_stockist`, `distributor` |
| Trigger stock movement email | `admin` |
| Attendance mark (`/attendance/mark`) | `so`, `asm` |
| Attendance admin view (`/attendance/admin/*`) | `admin` |
| Retailer visit create (`/retailer-visits`) | `so` |
| Location track (`/location/track`) | `so`, `asm` |
| Geofence CRUD (`/location/geofences`) | `admin` |
| Retailer-SO authorization (`/retailer-authorization/*`) | `retailer`, `so` |
| Pricing upload (`/pricing/upload`) | `admin` |
| Announcement CRUD (`/announcements/*`) | `admin` |

## 4. Web Navigation to Role Matrix

Derived from `frontend/src/components/layout/nav-items.ts` and actual page implementations.

| Web Feature | Roles |
|---|---|
| Dashboard | All 10 roles |
| Leaderboard | `admin`, `nsm`, `rsm`, `asm`, `super_stockist` |
| User Management | `admin`, `nsm`, `rsm`, `asm`, `super_stockist`, `distributor`, `so` |
| Territories | `admin`, `nsm`, `rsm`, `asm` |
| SKU Catalog | `admin`, `super_stockist`, `distributor` |
| SKU Master | `super_stockist`, `distributor`, `retailer` |
| Inventory | `super_stockist`, `distributor`, `retailer` |
| Inventory Intel | `super_stockist`, `distributor`, `retailer` |
| Inventory Analytics | `super_stockist`, `distributor` |
| Stock Movements | `admin`, `nsm`, `rsm`, `asm`, `so`, `super_stockist`, `distributor`, `retailer` |
| Orders | All 10 roles |
| Notifications | All 10 roles |
| Email Settings | `admin` |
| Attendance | `so`, `asm`, `admin` |
| Beat Visits | `so`, `asm`, `admin` |
| GPS Tracking | `so`, `admin` |
| Admin Geofence | `admin` |
| Admin Team Location | `admin` |
| Admin Performance | `admin` |
| Announcements | `admin` |
| Performance | `admin`, `nsm`, `rsm`, `asm` |
| Outlet-wise | `admin`, `super_stockist` |

## 5. Mobile Access Matrix

Derived from `my-expo-app/src/navigation/roleRoutes.ts`.

Mobile supported roles:
- `admin`, `nsm`, `super_stockist`, `rsm`, `distributor`, `asm`, `so`, `retailer`

Roles not in mobile enum:
- `finance`, `dispatch`, `hod` (not implemented)

| Mobile Feature | Roles |
|---|---|
| Dashboard | all mobile-supported roles |
| Orders | all mobile-supported roles |
| Inventory | `super_stockist`, `distributor`, `so` |
| Catalog | `admin`, `super_stockist`, `distributor`, `so`, `retailer` |
| Users | `admin`, `nsm`, `rsm`, `asm`, `super_stockist`, `distributor`, `so` |
| Territories | `admin`, `nsm`, `rsm`, `asm` |
| Stock Movements | all mobile-supported roles |
| Notifications | all mobile-supported roles |
| Email Settings | `admin` |
| Profile | all mobile-supported roles |

## 6. POR (Person of Responsibility) Routing

Used for stock alert and stock movement email CC behavior:

| Entity | POR Chain |
|---|---|
| `super_stockist` | `nsm` + `rsm` |
| `distributor` | `rsm` + `asm` |
| `retailer` | `asm` + `so` |

How role chain connects to POR in runtime:
1. Order notification flows call `notifyOrderWithPor(...)`.
2. Direct participants are included first (`fromEntityId`, `toEntityId`, optional actor).
3. POR roles are derived from chain relationship:
   - stock-in flow -> `nsm`, `rsm`, `asm`
   - `super_stockist -> distributor` -> `rsm`, `asm`
   - `distributor -> retailer` -> `asm`, `so`
4. Territory assignments are queried to find connected agents with those POR roles.
5. Recipients are deduplicated and then used for in-app notifications and WhatsApp delivery (if phone + config allow).

**Note:** The `hod` role is NOT implemented, so POR routing does not include it.

Implementation source:
- `backend/src/modules/notification/notification.service.ts`
- `resolvePorRoles(...)`
- `resolveConnectedPorRecipients(...)`

## 7. Access Principle

Effective access is enforced server-side as:

`Role Permission` + `Hierarchy Scope` + `Territory Assignment Scope`

UI role visibility (web/mobile) does not replace backend authorization.

