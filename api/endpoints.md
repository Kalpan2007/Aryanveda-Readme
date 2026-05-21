# Backend API Endpoints (Current Code)

Base path: `/api`

All routes except `GET /health` and public auth endpoints run behind authentication.

## 1. System

| Method | Route | Access |
|---|---|---|
| GET | `/health` | Public |

## 2. Auth (`/auth`)

| Method | Route | Access |
|---|---|---|
| POST | `/auth/login` | Public (rate-limited) |
| POST | `/auth/refresh` | Public |
| POST | `/auth/logout` | Public |
| POST | `/auth/send-otp` | Public (rate-limited) |
| POST | `/auth/reset-password` | Public (rate-limited) |
| GET | `/auth/me` | Authenticated (active/inactive allowed) |
| POST | `/auth/change-password` | Authenticated |
| POST | `/auth/email-preference` | Authenticated |

## 3. Users (`/users`)

| Method | Route | Access |
|---|---|---|
| POST | `/users` | `admin`, `nsm`, `hod`, `super_stockist`, `rsm`, `asm`, `so` |
| GET | `/users` | `admin`, `finance`, `dispatch` |
| GET | `/users/management` | `admin`, `nsm`, `hod`, `rsm`, `asm`, `so`, `super_stockist`, `distributor`, `retailer`, `finance`, `dispatch` |
| GET | `/users/count-by-role` | `admin`, `finance`, `dispatch` |
| GET | `/users/subordinates` | `nsm`, `rsm`, `asm` |
| GET | `/users/distributors` | `super_stockist`, `admin`, `finance`, `dispatch`, `so` |
| GET | `/users/agents` | `distributor`, `admin`, `finance`, `dispatch` |
| GET | `/users/retailers` | `distributor`, `so`, `super_stockist`, `admin`, `finance`, `dispatch` |
| PATCH | `/users/:entityId/status` | `admin`, `super_stockist`, `distributor`, `so`, `rsm`, `asm` |
| GET | `/users/beats` | All authenticated users |
| GET | `/users/beats/search` | All authenticated users |
| POST | `/users/import-excel` | `admin` |
| GET | `/users/:entityId` | Authenticated (hierarchy-scoped) |
| PATCH | `/users/:entityId` | `admin` (edit name, email, phone) |
| DELETE | `/users/:entityId` | `admin` (hard delete, cannot delete self or other admins) |
| GET | `/users/retailer/my-network` | `retailer` |

## 4. Territories (`/territories`)

| Method | Route | Access |
|---|---|---|
| POST | `/territories` | `admin`, `nsm`, `hod`, `rsm`, `asm` |
| GET | `/territories` | Authenticated (service-scoped) |
| GET | `/territories/agent/:entityId` | Authenticated (service-scoped) |
| GET | `/territories/:assignmentId` | Authenticated (service-scoped) |
| PUT | `/territories/:assignmentId` | `admin`, `nsm`, `hod`, `rsm`, `asm` |
| DELETE | `/territories/:assignmentId` | `admin`, `nsm`, `hod`, `rsm`, `asm` |

## 5. Catalog (`/catalog`)

| Method | Route | Access |
|---|---|---|
| POST | `/catalog/admin/sku` | `admin` |
| PATCH | `/catalog/admin/sku/:masterSkuId` | `admin` |
| PATCH | `/catalog/admin/stock/:masterSkuId` | `admin` |
| POST | `/catalog/admin/add-stock` | `admin` |
| POST | `/catalog/admin/bulk-import/preview` | `admin` |
| POST | `/catalog/admin/bulk-import` | `admin` |
| GET | `/catalog/pricing/:targetEntityId` | `admin`, `super_stockist`, `distributor` |
| PUT | `/catalog/pricing/:targetEntityId/bulk` | `admin`, `super_stockist`, `distributor` |
| PUT | `/catalog/pricing/:targetEntityId/:masterSkuId` | `admin`, `super_stockist`, `distributor` |
| GET | `/catalog/principals` | `super_stockist`, `admin` |
| GET | `/catalog/assortment` | `super_stockist` |
| POST | `/catalog/assortment/enable` | `super_stockist` |
| POST | `/catalog/assortment/disable` | `super_stockist` |
| POST | `/catalog/assortment/bulk` | `super_stockist` |
| GET | `/catalog` | `admin`, `super_stockist`, `distributor` |
| GET | `/catalog/:masterSkuId` | `admin`, `super_stockist`, `distributor` |

## 6. Legacy SKU (`/skus`)

| Method | Route | Access |
|---|---|---|
| GET | `/skus` | Authenticated |
| GET | `/skus/:skuId` | Authenticated |
| POST | `/skus` | `super_stockist` (feature-flagged deprecated create) |
| PATCH | `/skus/:skuId` | `super_stockist` |

## 7. Inventory (`/inventory`)

| Method | Route | Access |
|---|---|---|
| GET | `/inventory/analytics` | `super_stockist`, `distributor`, `retailer` |
| GET | `/inventory/analytics/principals` | `super_stockist`, `distributor`, `retailer` |
| POST | `/inventory/initialize` | `super_stockist` |
| POST | `/inventory/check-slow-moving` | `admin` |
| GET | `/inventory/:entityId` | Authenticated + hierarchy scoped |
| GET | `/inventory/:entityId/low-stock` | Authenticated + hierarchy scoped |
| GET | `/inventory/:entityId/value` | Authenticated + hierarchy scoped |
| GET | `/inventory/:entityId/ledger` | Authenticated + hierarchy scoped |
| GET | `/inventory/:entityId/:skuId` | Authenticated + hierarchy scoped |
| PATCH | `/inventory/:entityId/:skuId/adjust` | `super_stockist` + hierarchy scoped |
| PATCH | `/inventory/:entityId/:skuId/threshold` | `super_stockist`, `distributor`, `retailer` + hierarchy scoped |

## 8. Dashboard (`/dashboard`)

| Method | Route | Access |
|---|---|---|
| GET | `/dashboard` | Authenticated |

## 9. Orders (`/orders`)

| Method | Route | Access |
|---|---|---|
| POST | `/orders/primary` | `distributor` |
| POST | `/orders/primary/amount` | `super_stockist` |
| POST | `/orders/secondary` | `so`, `retailer` |
| GET | `/orders/so/retailer/:retailerId` | `so` |
| GET | `/orders/summary` | All authenticated users |
| GET | `/orders/pending-approval` | `admin` |
| GET | `/orders` | All authenticated users |
| GET | `/orders/:orderId` | All authenticated users |
| GET | `/orders/:orderId/approval-preview` | `super_stockist` |
| POST | `/orders/:orderId/approve` | `super_stockist` |
| POST | `/orders/:orderId/admin-approve` | `admin` |
| POST | `/orders/:orderId/admin-reject` | `admin` |
| POST | `/orders/:orderId/admin-handover-finance` | `admin` |
| PATCH | `/orders/:orderId/finance-edit` | `finance`, `super_stockist`, `dispatch` |
| POST | `/orders/:orderId/finance-approve` | `finance` |
| POST | `/orders/:orderId/finance-reject` | `finance` |
| PATCH | `/orders/:orderId/status` | All authenticated users |
| GET | `/orders/sales-user/connected-distributors` | `rsm`, `asm` |
| POST | `/orders/sales-user` | `rsm`, `asm` |

## 10. Notifications (`/notifications`)

| Method | Route | Access |
|---|---|---|
| GET | `/notifications/unread-count` | Authenticated |
| PATCH | `/notifications/read-all` | Authenticated |
| GET | `/notifications` | Authenticated |
| PATCH | `/notifications/:notificationId/read` | Authenticated |

Query contract for `GET /notifications`:
- `page` (default `1`)
- `limit` (default `20`, max `50`)
- `unreadOnly` (`true|false`, default `false`)

Response contract for `GET /notifications`:
- `success`
- `data` (notification list)
- `total`
- `unreadCount`
- `page`
- `limit`

Other notification responses:
- `GET /notifications/unread-count` -> `{ success, data: { count } }`
- `PATCH /notifications/read-all` -> `{ success, data: { markedCount } }`

## 11. Email (`/email`)

| Method | Route | Access |
|---|---|---|
| GET | `/email/health` | `admin` |

Notes:
- endpoint checks SMTP connectivity through configured transporter
- returns `503` when SMTP verification fails or email is disabled

## 12. Stock Movement (`/stock-movement`)

| Method | Route | Access |
|---|---|---|
| GET | `/stock-movement/own` | `super_stockist`, `distributor`, `retailer` |
| GET | `/stock-movement/downstream` | `admin`, `nsm`, `hod`, `super_stockist`, `rsm`, `distributor`, `asm`, `so` |
| POST | `/stock-movement/trigger-email` | `admin` |

Notes:
- `days` query for movement endpoints allows: `1`, `7`, `45`, `60`
- `POST /stock-movement/trigger-email` manually executes the 45-day email job immediately

## 13. Notes

- `display-audit` models exist but no mounted routes currently.
- `sms` remains a placeholder with no mounted routes.
- `whatsapp` has no mounted HTTP routes, but WhatsApp sending is implemented in service/event flows (`backend/src/modules/whatsapp`).

