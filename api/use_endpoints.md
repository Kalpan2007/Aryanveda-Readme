# Endpoint Usage Map (Web + Mobile)

This maps implemented backend endpoints to current web and mobile client usage.

Legend:
- Web: called by `frontend/src/services/*`
- Mobile: called by `my-expo-app/src/api/*` or `my-expo-app/src/features/*`

## 1. Auth

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `POST /auth/login` | Yes | Yes | Mobile currently sends `email` field alias for login value |
| `POST /auth/refresh` | Yes | Yes | |
| `POST /auth/logout` | Yes | Yes | |
| `POST /auth/send-otp` | Yes | Yes | |
| `POST /auth/reset-password` | Yes | Yes | |
| `GET /auth/me` | Yes | Yes | |
| `POST /auth/change-password` | Yes | Yes | |
| `POST /auth/email-preference` | Yes | No | web-first optional email prompt |

## 2. Users

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `POST /users` | Yes | Yes | |
| `GET /users` | Yes | Yes (`all` scope) | |
| `GET /users/management` | Yes | No | role-scoped management list |
| `GET /users/count-by-role` | Yes | No | admin KPI support |
| `GET /users/subordinates` | Yes | Yes | |
| `GET /users/distributors` | Yes | Yes | |
| `GET /users/agents` | Yes | Yes | |
| `GET /users/retailers` | Yes | Yes | |
| `PATCH /users/:entityId/status` | Yes | Yes | |
| `GET /users/:entityId` | Yes | No | |

## 3. Territories

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `POST /territories` | Yes | Yes | |
| `GET /territories` | Yes | Yes | |
| `GET /territories/agent/:entityId` | Yes | No | web service supports it |
| `GET /territories/:assignmentId` | Yes | No | web service supports it |
| `PUT /territories/:assignmentId` | Yes | No | web service supports it |
| `DELETE /territories/:assignmentId` | Yes | Yes | |

## 4. Catalog and Pricing

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `POST /catalog/admin/sku` | Yes | Yes | |
| `PATCH /catalog/admin/sku/:masterSkuId` | Yes | No | mobile does not currently call update endpoint |
| `PATCH /catalog/admin/stock/:masterSkuId` | Yes | Yes | |
| `POST /catalog/admin/add-stock` | Yes | Yes | |
| `POST /catalog/admin/bulk-import/preview` | Yes | No | |
| `POST /catalog/admin/bulk-import` | Yes | No | |
| `GET /catalog/pricing/:targetEntityId` | Yes | No | mobile currently sets prices without fetching table |
| `PUT /catalog/pricing/:targetEntityId/bulk` | Yes | Yes | |
| `PUT /catalog/pricing/:targetEntityId/:masterSkuId` | Yes | Yes | |
| `GET /catalog/principals` | Yes | Yes | |
| `GET /catalog/assortment` | No | No | exposed by backend, not used in current clients |
| `POST /catalog/assortment/enable` | Yes | Yes | |
| `POST /catalog/assortment/disable` | Yes | Yes | |
| `POST /catalog/assortment/bulk` | Yes | No | |
| `GET /catalog` | Yes | Yes | |
| `GET /catalog/:masterSkuId` | Yes | Yes | |

## 5. Legacy SKU

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `GET /skus` | Yes | Yes | |
| `GET /skus/:skuId` | Yes | No | |
| `POST /skus` | Yes | No | deprecated/feature-flagged |
| `PATCH /skus/:skuId` | Yes | No | |

## 6. Inventory

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `GET /inventory/analytics` | Yes | Yes | |
| `GET /inventory/analytics/principals` | Yes | Yes | |
| `POST /inventory/initialize` | Yes | No | defined in web service, not active in current UI |
| `POST /inventory/check-slow-moving` | No | Yes | currently wired in mobile API layer only |
| `GET /inventory/:entityId` | Yes | Yes | |
| `GET /inventory/:entityId/low-stock` | Yes | Yes | |
| `GET /inventory/:entityId/value` | Yes | Yes | |
| `GET /inventory/:entityId/ledger` | Yes | Yes | |
| `GET /inventory/:entityId/:skuId` | Yes | Yes | |
| `PATCH /inventory/:entityId/:skuId/adjust` | Yes | Yes | |
| `PATCH /inventory/:entityId/:skuId/threshold` | Yes | Yes | |

## 7. Dashboard

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `GET /dashboard` | Yes | Yes | |

## 8. Orders

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `POST /orders/primary` | Yes | Yes | |
| `POST /orders/primary/amount` | Yes | Yes | |
| `POST /orders/secondary` | Yes | Yes | |
| `GET /orders/summary` | Yes | Yes | |
| `GET /orders/pending-approval` | Yes | Yes | |
| `GET /orders` | Yes | Yes | |
| `GET /orders/:orderId` | Yes | Yes | |
| `POST /orders/:orderId/approve` | Yes | Yes | |
| `POST /orders/:orderId/admin-approve` | Yes | Yes | |
| `POST /orders/:orderId/admin-reject` | Yes | Yes | |
| `POST /orders/:orderId/admin-handover-finance` | Yes | No | |
| `PATCH /orders/:orderId/finance-edit` | Yes | No | |
| `POST /orders/:orderId/finance-approve` | Yes | No | |
| `POST /orders/:orderId/finance-reject` | Yes | No | |
| `PATCH /orders/:orderId/status` | Yes | Yes | |

## 9. Notifications

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `GET /notifications/unread-count` | Yes | Yes | |
| `PATCH /notifications/read-all` | Yes | Yes | |
| `GET /notifications` | Yes | Yes | |
| `PATCH /notifications/:notificationId/read` | Yes | Yes | |

Notification usage notes:
- both clients implement in-app feed consumption (`unread-count`, list, read, read-all)
- both clients rely on backend-persisted notification feed; no push socket channel is used

## 10. Email and Stock Movement

| Endpoint | Web | Mobile | Notes |
|---|---|---|---|
| `GET /email/health` | Yes | Yes | |
| `GET /stock-movement/own` | Yes | Yes | |
| `GET /stock-movement/downstream` | Yes | Yes | |
| `POST /stock-movement/trigger-email` | Yes | Yes | |

## 11. Unmounted / Planned Domains

Not currently available as backend endpoints:
- display audit API routes (model layer exists only)
- sms integration routes
- whatsapp integration routes

