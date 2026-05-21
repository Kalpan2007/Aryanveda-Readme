# Mobile Feature Parity Report (Web vs Mobile)

Snapshot: March 2026

## 1. Scope Audited

- Web routes and services from `frontend/src/app` and `frontend/src/services`
- Mobile screens/navigation/API wrappers from `my-expo-app/src`
- Backend endpoint contracts from mounted route files

## 2. Role Coverage Parity

Backend roles:
- `admin`, `finance`, `dispatch`, `nsm`, `hod`, `rsm`, `asm`, `so`, `super_stockist`, `distributor`, `retailer`

Mobile role enum currently supports:
- `admin`, `nsm`, `hod`, `rsm`, `asm`, `so`, `super_stockist`, `distributor`, `retailer`

Not in mobile enum:
- `finance`, `dispatch`

## 3. Feature Parity Matrix

| Module | Web Status | Mobile Status | Notes |
|---|---|---|---|
| Login/session restore/logout | Complete | Complete | shared auth endpoints |
| Forgot/reset password | Complete | Complete | OTP flow implemented |
| Change password | Complete | Complete | mobile via Profile screen |
| Optional email preference prompt | Complete | Partial | web uses `/auth/email-preference`; mobile not currently wired |
| Dashboard | Complete | Complete | role-mapped KPI cards implemented |
| User management | Complete | Complete | list/create/toggle covered |
| Territories | Complete | Complete | list/create/deactivate covered |
| Catalog core | Complete | Complete | list/detail/principal/filter and actions covered |
| Catalog admin bulk import | Complete | Missing | mobile does not use bulk import endpoints |
| SKU legacy module | Complete | Partial | mobile list only, no create/update |
| Inventory core + analytics | Complete | Complete | list/detail/value/low-stock/ledger/analytics |
| Inventory initialize | Service-ready | Missing | no active mobile call |
| Slow-moving trigger | Not wired in web UI | Present in mobile API layer | endpoint available on backend |
| Orders create/list/detail/status | Complete | Complete | includes admin approve/reject |
| Admin handover to finance | Complete | Missing | mobile not wired |
| Finance edit/approve/reject | Complete | Missing | finance/dispatch role not in mobile enum |
| Notifications | Complete | Complete | unread/list/read/read-all |
| Email health | Complete | Complete | admin only |
| Stock movement own/downstream/trigger | Complete | Complete | covered in both |
| Audits UI | Placeholder page | Missing | backend routes not mounted |

Notification channel parity notes:
- both web and mobile consume the same in-app feed APIs
- both web and mobile expose email health check UI for admin users
- neither client currently uses backend push channels (FCM/APNS/Expo push); implementation is feed + SMTP email

## 4. Navigation Parity

Web navigation includes:
- dashboard, users, territories, catalog, skus, inventory, inventory analytics, stock movements, orders, notifications, email settings

Mobile drawer includes:
- dashboard, users, territories, catalog, inventory, stock movements, orders, notifications, email settings, profile

Observation:
- Mobile uses a broader drawer-first navigation model and keeps SKU in catalog flow.

## 5. Highest-Impact Gaps

1. Finance/dispatch workflows on mobile:
- no mobile role support for `finance`, `dispatch`
- finance order stage actions not exposed

2. Catalog import parity:
- mobile lacks admin bulk import preview/import flows

3. Optional email preference parity:
- mobile does not currently implement `/auth/email-preference` flow

4. Audits parity:
- web has placeholder only; mobile has no corresponding screen

## 6. Recommended Next Tasks

1. Add mobile enum and guarded screens for `finance` and `dispatch` if required operationally.
2. Implement mobile stock-in finance stage actions (`admin-handover-finance`, `finance-*`).
3. Add mobile admin bulk catalog import support if mobile admin operation is needed.
4. Add optional email preference capture flow in mobile first-login/profile.
5. Create shared audits placeholder in mobile until backend audit endpoints are exposed.

