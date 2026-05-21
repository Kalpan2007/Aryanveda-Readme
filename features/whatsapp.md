# WhatsApp Notification Matrix (Implemented)

Current code alignment: March 2026.

This file lists every WhatsApp notification currently implemented in the backend, including trigger source, template name, payload shape, and status values.

## 1. Global Routing Behavior

- Provider: MSG91 bulk WhatsApp API (`/api/v5/whatsapp/whatsapp-outbound-message/bulk/`)
- Integration layer: `backend/src/modules/whatsapp/whatsapp.service.ts`
- Default mode: notification is sent to the target user phone passed by the event.
- If `WHATSAPP_ALSO_SEND_TO_ADMIN=true`, notification is sent to both target user and `WHATSAPP_ADMIN_TEST_PHONE`.
- If `WHATSAPP_FORCE_ADMIN_TEST_MODE=true`, every WhatsApp notification is redirected to `WHATSAPP_ADMIN_TEST_PHONE` only.
- Duplicate recipient protection is enabled (if user phone equals admin phone, only one send target is kept).
- If `WHATSAPP_ENABLED=false`, WhatsApp sends are skipped.

## 2. Template Catalog

Template key to MSG91 template name mapping:

1. `AUTH_OTP` -> `auth_otp_code`
2. `ORDER_UPDATE` -> `order_status_update`
3. `USER_STATUS` -> `user_account_status_`
4. `PRODUCT_UPDATE` -> `catalog_product_update`
5. `STOCK_ALERT` -> `stock_notification`

Source: `backend/src/modules/whatsapp/whatsapp.templates.ts`

## 3. Implemented WhatsApp Notifications

### 3.1 OTP (Auth)

- Function: `sendOtpWhatsApp(phone, otpCode, validityMinutes)`
- Template: `auth_otp_code`
- Trigger source: password-reset OTP flow in `backend/src/modules/auth/auth.service.ts`
- Components sent:
  1. `body_1` (`otp_code`)
  2. `button_1` (url button value from `otp_button_value` or `MSG91_OTP_BUTTON_URL_VALUE` fallback)

Note: Validity minutes are intentionally not sent to match provider template shape.

### 3.2 Order Lifecycle Updates

- Function: `sendOrderStatusWhatsApp(phone, orderId, status, details)`
- Template: `order_status_update`
- Trigger source: `notifyOrderWithPor(...)` in `backend/src/modules/notification/notification.service.ts`
- Fired by:
  1. `notifyOrderCreatedWithPor`
  2. `notifyOrderApprovedWithPor`
  3. `notifyOrderDispatchedWithPor`
  4. `notifyOrderDeliveredWithPor`
  5. `notifyOrderCancelledWithPor`
  6. `notifyOrderEditedWithPor`
- Components sent:
  1. `body_status` (`parameter_name: status`)
  2. `body_details` (`parameter_name: details`)
  3. `body_order_id` (`parameter_name: order_id`)
- Status values used today:
  1. `created`
  2. `approved`
  3. `dispatched`
  4. `delivered`
  5. `cancelled`
  6. `edited`

### 3.3 User Activation/Deactivation

- Function: `sendUserStatusWhatsApp(phone, status, actor)`
- Template: `user_account_status_`
- Trigger source: `notifyUserActivated(...)`, `notifyUserDeactivated(...)`
- Components sent:
  1. `body_status` (`parameter_name: status`)
  2. `body_actor` (`parameter_name: actor`)
- Status values used today:
  1. `activated`
  2. `deactivated`

### 3.4 Product/Catalog Updates

- Function: `sendProductUpdateWhatsApp(phone, productName, action, skuId)`
- Template: `catalog_product_update`
- Trigger sources:
  1. catalog create/update paths in `backend/src/modules/catalog/catalog.service.ts`
  2. notification-level SKU add in `notifySkuAdded(...)`
- Components sent:
  1. `body_product_name` (`parameter_name: product_name`)
  2. `body_sku_id` (`parameter_name: sku_id`)
  3. `body_action` (`parameter_name: action`)
- Action values used today:
  1. `added`
  2. `updated`

### 3.5 Stock Alerts and Stock Movement Signals

- Function: `sendStockAlertWhatsApp(phone, entity, type, summary)`
- Template: `stock_notification`
- Trigger sources:
  1. threshold hit in `backend/src/modules/inventory/inventory.service.ts`
  2. slow-moving stock in `backend/src/modules/email/email.service.ts`
  3. 45-day report-ready scheduler in `backend/src/modules/stock-movement/stockMovement.scheduler.ts`
- Components sent:
  1. `body_summary` (`parameter_name: summary`)
  2. `body_type` (`parameter_name: type`)
  3. `body_entity` (`parameter_name: entity`)
- Type values used today:
  1. `threshold_reached`
  2. `slow_moving`
  3. `report_ready`

## 4. In-App and Email vs WhatsApp Coverage

Covered in WhatsApp now:

1. OTP reset notifications
2. Order lifecycle updates (all mapped statuses)
3. User activation/deactivation updates
4. Product update announcements
5. Stock threshold/slow-moving/report-ready alerts

In-app flows that do not yet have dedicated WhatsApp templates today:

1. `notifyStockInForwardedToFinance`
2. `notifyStockInFinanceDecision`
3. `notifyOrderReadyForDispatch`
4. `notifyTerritoryAssigned`
5. `notifySecondaryOrderSalesChain`

These currently stay as in-app notifications (and some have email support) unless mapped to new WhatsApp templates.

## 5. Operational Checklist

1. Set `WHATSAPP_ENABLED=true`
2. Set valid `MSG91_AUTH_KEY`, `MSG91_INTEGRATED_NUMBER`, `MSG91_NAMESPACE`
3. Keep `WHATSAPP_DEBUG=true` while validating payloads
4. For test mode, use:
   - `WHATSAPP_FORCE_ADMIN_TEST_MODE=true`
   - `WHATSAPP_ADMIN_TEST_PHONE=<admin_mobile_with_country_code>`
5. For production copy-to-admin mode, use:
  - `WHATSAPP_FORCE_ADMIN_TEST_MODE=false`
  - `WHATSAPP_ALSO_SEND_TO_ADMIN=true`
  - `WHATSAPP_ADMIN_TEST_PHONE=<admin_mobile_with_country_code>`
6. For user-only mode, use:
  - `WHATSAPP_FORCE_ADMIN_TEST_MODE=false`
  - `WHATSAPP_ALSO_SEND_TO_ADMIN=false`
7. Monitor MSG91 delivery reports using returned `request_id`
