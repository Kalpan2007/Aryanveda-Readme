# Notifications and Email - Codebase Behavior

Current code alignment: March 2026.

This document describes how notifications are generated, stored, delivered, and consumed across backend, web, and mobile.

## 1. Delivery Channels

1. In-app notifications (`backend/src/modules/notification`)
- persisted in MongoDB `notifications` collection
- per-user unread badge count + paginated feed
- mark-one and mark-all read actions

2. Transactional SMTP emails (`backend/src/modules/email`)
- role-aware and hierarchy-aware recipient resolution
- sends are fire-and-forget (non-blocking business flow)
- admin health endpoint for SMTP verification

3. Periodic stock movement report emails (`backend/src/modules/stock-movement`)
- scheduled daily check; sends every 45 days
- admin manual trigger endpoint

## 2. API Surface

All routes below are under `/api` and require authentication unless noted.

Notifications:
- `GET /notifications/unread-count`
- `GET /notifications?page=1&limit=20&unreadOnly=false`
- `PATCH /notifications/:notificationId/read`
- `PATCH /notifications/read-all`

Email:
- `GET /email/health` (`admin` only)

Stock movement reporting:
- `GET /stock-movement/own?days=1|7|45|60`
- `GET /stock-movement/downstream?days=1|7|45|60&targetEntityId=...`
- `POST /stock-movement/trigger-email` (`admin` only)

## 3. In-App Notification Model

Defined in `backend/src/modules/notification/notification.model.ts`.

Important fields:
- `notificationId` (unique)
- `recipientEntityId`
- `type` (event enum)
- `priority` (`low | normal | high`)
- `title`, `message`
- `referenceOrderId` (nullable)
- `referenceEntityId` (nullable)
- `isRead`
- `createdAt`, `updatedAt`

Indexes:
- fast inbox query: `(recipientEntityId, isRead, createdAt desc)`
- deduplication: unique sparse index on `(recipientEntityId, referenceOrderId, type)`

Dedup behavior:
- duplicate key (`11000`) is swallowed in `createNotification(...)`
- repeated trigger attempts for same `(recipient, reference, type)` stay idempotent

## 4. In-App Notification Event Types

Currently typed and consumed in web/mobile clients:
- `order_created`
- `order_approved`
- `order_dispatched`
- `order_delivered`
- `order_cancelled`
- `user_deactivated`
- `user_activated`
- `sku_added`

## 5. Trigger Map (In-App)

### 5.1 Order flow triggers

Primary and secondary order lifecycle events are emitted by `backend/src/modules/order/order.service.ts` through helpers in `notification.service.ts`.

Core helpers:
- `notifyOrderCreatedWithPor(...)`
- `notifyOrderApprovedWithPor(...)`
- `notifyOrderDispatchedWithPor(...)`
- `notifyOrderDeliveredWithPor(...)`
- `notifyOrderCancelledWithPor(...)`
- `notifyOrderEditedWithPor(...)`

Stock-in specific helpers:
- `notifyStockInForwardedToFinance(orderId)` -> finance users
- `notifyStockInFinanceDecision(orderId, stockistEntityId, isApproved)` -> stockist
- `notifyOrderReadyForDispatch(orderId)` -> dispatch users

Secondary sales chain helper:
- `notifySecondaryOrderSalesChain(orderId, retailerId, createdByRole, totalAmount)` -> SO and ASM chain where applicable

### 5.2 User lifecycle triggers

In `backend/src/modules/user/user.service.ts`:
- deactivation triggers `notifyUserDeactivated(...)`
- activation triggers `notifyUserActivated(...)`

Behavior:
- deactivated user always receives in-app notice
- admin/deactivator/parent can also receive contextual notifications

### 5.3 Territory assignment triggers

In `backend/src/modules/territory/territory.service.ts`:
- `notifyTerritoryAssigned(agentEntityId, targetName)` on assignment creation/reactivation

### 5.4 Catalog SKU addition trigger

`notifySkuAdded(...)` sends in-app `sku_added` to recipient entity list passed by caller.

## 6. Email Trigger Map

Defined in `backend/src/modules/email/email.service.ts`.

Email categories:
- `emailOtpPassword(...)` for password reset OTP
- `emailWelcomeUser(...)` for newly created users
- `emailUserDeactivated(...)`
- `emailUserActivated(...)`
- `emailCatalogDefaultChanged(...)` (broadcast to all active users)
- `emailPriceChanged(...)` (target + owner + POR + admin CC)
- `emailThresholdHit(...)` (owner + POR + admin)
- `emailSlowMovingStock(...)` (owner + POR + admin)

Common behaviors:
- recipient normalization: lowercased, blanks skipped
- dedup on `to`/`cc` sets
- if `EMAIL_ENABLED=false`, send is skipped gracefully
- failure to send email does not fail core domain operation

## 7. POR Routing Rules Used for Emails

Recipient expansion is role-aware using territory assignments.

POR mapping in code:
- `super_stockist -> nsm + hod`
- `distributor -> rsm + asm`
- `retailer -> so + (asm on parent distributor)`

Notes:
- inactive users are filtered for POR/admin lookups
- users without valid email are excluded from mail recipients

## 8. Stock Movement Email Scheduler

Implemented in `backend/src/modules/stock-movement/stockMovement.scheduler.ts`.

Runtime behavior:
1. `startStockMovementScheduler()` starts at backend boot when email is enabled.
2. Cron runs daily at `09:00` (`0 9 * * *`).
3. It sends only if `lastSentDate` is missing or at least 45 days old.
4. Send job calls `runStockMovementEmailJob()`.
5. Each recipient gets an HTML summary via `sendStockMovementEmail(...)`.

Manual trigger:
- admin can run `POST /stock-movement/trigger-email`

## 9. Startup Sequence Relevant to Notifications/Email

At server startup (`backend/src/server.ts`):
1. connect MongoDB
2. verify SMTP connection asynchronously (`verifySmtpConnection()`)
3. start stock movement scheduler
4. start HTTP server

## 10. Client Consumption

Web:
- API wrapper: `frontend/src/services/notification.service.ts`
- UI: `frontend/src/components/layout/NotificationBell.tsx`
- page route: `/notifications`
- email health page: `/notifications/email-settings`

Mobile:
- API wrapper: `my-expo-app/src/features/notifications/notifications.api.ts`
- queries/mutations: `my-expo-app/src/features/notifications/useNotificationsQuery.ts`
- UI: `my-expo-app/src/screens/notifications/NotificationsScreen.tsx`
- email health UI: `my-expo-app/src/screens/notifications/EmailSettingsScreen.tsx`

Important:
- current implementation is in-app feed + email only
- there is no push notification gateway (FCM/APNS/Expo push) in backend runtime

## 11. Optional Email and Business Safety

Email is optional for many users (`/auth/email-preference` flow).

Current safety guarantees:
- missing recipient email does not block order/user/inventory workflows
- mail failures are logged and skipped
- in-app notifications continue independently of SMTP health

## 12. Operational Checklist

- Set SMTP env vars: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM_EMAIL`.
- Keep `EMAIL_ENABLED=true` for mail delivery.
- Validate `GET /email/health` using admin auth.
- Ensure territory assignments are accurate (POR expansion depends on them).
- Monitor logs for `[EmailService]`, `[StockMovementScheduler]`, `[StockMovementEmail]` entries.

