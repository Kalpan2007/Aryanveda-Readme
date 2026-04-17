# AryanVeda DMS

AryanVeda DMS is a multi-role distribution management platform that connects sales planning, supply-chain execution, stock visibility, and notifications in one system.

## 1. Project Motive

The motive of this project is to digitize the full distributor network workflow in one place:

1. Build a strict role and hierarchy based operating model.
2. Make every order auditable from create to delivery.
3. Keep inventory movement accurate by unit conversion and ledger tracking.
4. Route operational visibility to the right people (participants + POR chain).
5. Deliver notifications in-app, by email, and by WhatsApp with role-aware recipients.

## 2. Tech Stack (With Logos)

### Backend

![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?logo=express&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?logo=jsonwebtokens&logoColor=white)
![Nodemailer](https://img.shields.io/badge/Nodemailer-22A6F2?logo=gmail&logoColor=white)

### Web

![Next.js](https://img.shields.io/badge/Next.js-000000?logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black)
![TanStack Query](https://img.shields.io/badge/TanStack%20Query-FF4154?logo=reactquery&logoColor=white)
![Zustand](https://img.shields.io/badge/Zustand-443E38?logo=react&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-06B6D4?logo=tailwindcss&logoColor=white)

### Mobile

![Expo](https://img.shields.io/badge/Expo-000020?logo=expo&logoColor=white)
![React Native](https://img.shields.io/badge/React%20Native-20232A?logo=react&logoColor=61DAFB)

### Messaging and Reporting

![MSG91](https://img.shields.io/badge/WhatsApp-MSG91-25D366?logo=whatsapp&logoColor=white)
![SMTP](https://img.shields.io/badge/Email-SMTP-0A66C2?logo=maildotru&logoColor=white)
![k6](https://img.shields.io/badge/Load%20Test-k6-7D64FF?logo=k6&logoColor=white)

## 3. Monorepo Structure

- `backend/` - API, workflows, notification orchestration, scheduler
- `frontend/` - web operations panel
- `my-expo-app/` - mobile app
- `load-test/` - k6 scripts and performance reports
- root docs - architecture, endpoints, roles, notification notes

## 4. Role Model and Hierarchy (11 Roles)

### Branches

| Branch | Roles |
|---|---|
| Root | `admin`, `finance`, `dispatch` |
| Sales Force | `nsm`, `hod`, `rsm`, `asm`, `so` |
| Supply Chain | `super_stockist`, `distributor`, `retailer` |

### Hierarchy

- `admin -> nsm -> hod -> rsm -> asm -> so`
- `admin -> super_stockist -> distributor`
- retailer visibility is connected through hierarchy plus territory scope.

## 5. Role Access: Pages and Functional Scope

The table below combines what each role can see and what each role can do across the website.

| Role | Main Pages Seen | Core Functional Access |
|---|---|---|
| `admin` | dashboard, users, territories, catalog, orders, notifications, stock-movements, email-settings | create/manage root and branch users, full territory control, master catalog CRUD, company stock add/update, stock-in admin actions (approve/reject/handover), read all orders, trigger stock movement email, SMTP health check |
| `finance` | dashboard, orders, notifications | finance edit stock-in orders, finance approve/reject, read finance-visible stock-in flow |
| `dispatch` | dashboard, orders, notifications | dispatch stock-in orders after finance/admin stage, can participate in finance-edit endpoint scope |
| `nsm` | dashboard, users, territories, orders, stock-movements, notifications | create HOD users, manage scoped users, territory assignments, sales-scope order visibility, downstream stock movement view |
| `hod` | dashboard, users, territories, orders, stock-movements, notifications | create RSM users, scoped user status management, scoped territory and order visibility |
| `rsm` | dashboard, users, territories, orders, stock-movements, notifications | create ASM users, scoped user management, territory operations for own tree |
| `asm` | dashboard, users, territories, orders, stock-movements, notifications | create SO users, territory management, chain-level order visibility |
| `so` | dashboard, users, orders, stock-movements, notifications | create secondary orders for retailers in scope, manage own/child visibility, participate in retail fulfillment chain |
| `super_stockist` | dashboard, users, catalog, skus, inventory, inventory-analytics, orders, stock-movements, notifications | create distributors, assortment enable/disable, pricing to distributors, amount stock-in creation, approve regular primary orders, inventory adjustment and threshold operations |
| `distributor` | dashboard, users, catalog, skus, inventory, inventory-analytics, orders, stock-movements, notifications | create primary orders, dispatch secondary orders, pricing to retailers, inventory and threshold management in scope |
| `retailer` | dashboard, skus, inventory, inventory-analytics, orders, stock-movements, notifications | create secondary self-orders, receive and confirm delivery, inventory threshold updates for own node |

## 6. Order Flow: End-to-End by Chain Role

Order states in system:

- `CREATED`
- `IN_FINANCE`
- `APPROVED`
- `DISPATCHED`
- `DELIVERED`
- `CANCELLED`

### A) Regular Primary Flow (Distributor -> Super Stockist)

1. Distributor creates `POST /orders/primary` -> status `CREATED`.
2. Super Stockist reviews and approves `POST /orders/:orderId/approve` -> status `APPROVED`.
3. Super Stockist dispatches `PATCH /orders/:orderId/status` -> `DISPATCHED`.
4. Distributor confirms delivery -> `DELIVERED`.

Stock movement behavior:

- no inventory movement at create.
- inventory deduct/increment happens on approval for regular primary.

### B) Stock-In Primary Flow (Super Stockist self stock-in)

1. Super Stockist creates amount order `POST /orders/primary/amount` -> `CREATED`.
2. Admin can:
	- approve (`admin-approve`) or
	- reject (`admin-reject`) or
	- hand over to finance (`admin-handover-finance`) -> `IN_FINANCE`.
3. Finance can edit/approve/reject.
4. Dispatch dispatches stock-in after approval stage.
5. Super Stockist confirms delivery.

Stock movement behavior:

- inventory increment for stock-in happens on admin approval.

### C) Secondary Flow (Distributor -> Retailer, created by SO or Retailer)

1. SO or Retailer creates `POST /orders/secondary` -> `CREATED`.
2. Distributor dispatches -> `DISPATCHED` (deduct distributor stock).
3. Retailer confirms delivery -> `DELIVERED` (increment retailer stock).

Stock movement behavior:

- no movement on create.
- deduct on dispatch and increment on delivery.

## 7. Notification Routing: Who Receives What

### Recipient Logic

For order lifecycle notifications, recipients are built in layers:

1. direct participants (`fromEntityId`, `toEntityId`, actor)
2. POR chain roles based on relationship
3. territory assignment expansion
4. deduplication before delivery

POR role mapping in active implementation:

- stock-in flow -> `nsm`, `hod`
- `super_stockist -> distributor` -> `rsm`, `asm`
- `distributor -> retailer` -> `asm`, `so`

### In-Site Notification Format

In-app payload structure includes:

- `type` (example: `order_created`, `order_approved`, `order_dispatched`, `order_delivered`, `order_cancelled`)
- `priority` (`low`, `normal`, `high`)
- `title`
- `message`
- `referenceOrderId`
- `recipientEntityId`

### Email Notification Format

Order emails use transactional subject lines similar to:

- `[Nimson DMS] Order Created - <ORDER_ID>`
- `[Nimson DMS] Order Approved - <ORDER_ID>`
- `[Nimson DMS] Order Dispatched - <ORDER_ID>`

User and stock emails include dedicated templates for:

- welcome
- activation/deactivation
- threshold alerts
- slow moving alerts
- stock movement periodic reports

### WhatsApp Notification Format

Order template: `order_status_update`

Mapped parameters sent:

1. `status`
2. `details`
3. `order_id`

Current order status values sent:

- `created`, `approved`, `dispatched`, `delivered`, `cancelled`, `edited`

Typical details block format includes lines like:

- `Update: <event>`
- `Route: <source> to <destination>`
- `Quantity: <value>`
- `Value: <amount>`
- `Requested By: <name>`
- `Updated By: <name>`

Other WhatsApp templates:

- OTP: `auth_otp_code`
- user status: `user_account_status_`
- product update: `catalog_product_update`
- stock alerts: `stock_notification`

## 8. Agent Role Working Model (NSM, HOD, RSM, ASM, SO)

### How Agent Chain Works

1. Sales hierarchy is created by parent roles (`nsm -> hod -> rsm -> asm -> so`).
2. Territory assignments connect sales users to supply nodes.
3. Assigned chain determines visibility and POR expansion in notifications.
4. Agent-visible order access is scope-based, not global.

### Agent Responsibilities Summary

- `nsm`: national oversight, HOD creation, top-level territory and chain visibility.
- `hod`: regional governance, RSM creation, scoped chain controls.
- `rsm`: area supervision, ASM creation, regional visibility.
- `asm`: ground team manager, SO creation, territory and order supervision.
- `so`: frontline role, creates secondary orders, manages retailer execution in scope.

### SKU and Agent Interaction

- Agents do not run master catalog admin CRUD.
- Agents consume SKU data through scoped order and catalog views.
- SO directly uses SKU data in secondary order creation and downstream fulfillment flow.

## 9. SKU Information Model

There are two SKU tracks in codebase:

1. Master Catalog (`/catalog`)
	- admin-governed master SKU records
	- company stock
	- principal/category
	- per-dozen pricing chain (`SS`, `Distributor`, `Retail`)
	- assortment enable/disable at super stockist level

2. Legacy SKU (`/skus`)
	- legacy read/update path still active
	- create is feature-flagged/deprecated in current behavior

Pricing and assortment flow:

- admin -> super stockist -> distributor pricing chain
- super stockist controls assortment visibility for downstream ordering

Unit modes in order engine:

- `piece`, `dozen`, `jar`
- order-unit quantity is preserved, movement quantity is converted to pieces for ledger/inventory accuracy

## 10. Backend Design and Module Architecture

Backend modules mounted in app:

- `auth`
- `user`
- `territory`
- `catalog`
- `sku`
- `inventory`
- `order`
- `dashboard`
- `notification`
- `email`
- `stock-movement`

Additional module status:

- `whatsapp` implemented as service integration (no dedicated public routes)
- `display-audit` groundwork present but not mounted as API
- `sms` placeholder only

Request pipeline summary:

1. CORS and request parsing
2. public health and auth public endpoints
3. global auth middleware for protected routes
4. role and hierarchy guards
5. centralized error handling

## 11. Frontend Design and Module Architecture

Web app architecture:

- Next.js App Router
- service-layer API adapters in `frontend/src/services`
- TanStack Query for server state
- Zustand for auth/session state
- role-gated navigation in `frontend/src/components/layout/nav-items.ts`

Core web routes:

- auth: `/login`, `/forgot-password`
- operations: `/dashboard`, `/users`, `/territories`, `/catalog`, `/skus`, `/inventory`, `/inventory-analytics`, `/orders`, `/stock-movements`, `/notifications`, `/notifications/email-settings`

Mobile app architecture:

- Expo + React Native
- role-gated drawer navigation
- shared backend authorization model

## 12. Quick Start

Backend:

```bash
cd backend
npm install
npm run dev
```

Web:

```bash
cd frontend
npm install
npm run dev
```

Mobile:

```bash
cd my-expo-app
npm install
npm run start
```

Seed:

```bash
cd backend
npx ts-node scripts/mobil_seed.ts
```

## 13. Test Commands

Backend tests:

```bash
cd backend
npm test
```

Load test:

```bash
cd load-test
k6 run k6-load-test.js
```

## 14. Documentation Index

- `Roles.md` - full role and access mapping
- `endpoints.md` - full backend routes
- `use_endpoints.md` - endpoint usage by web and mobile
- `backend.md` - backend architecture reference
- `frontend.md` - frontend architecture reference
- `notification.md` - in-app and email behavior
- `whatsapp.md` - WhatsApp template and trigger matrix
- `mobile-feature-parity.md` - web/mobile parity tracking
- `performance.md` - performance and load notes
- `Project.md` and `PRD.md` - product baseline and requirements

