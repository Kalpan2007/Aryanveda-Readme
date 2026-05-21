![Nimson Logo](./assets/Nimson_Logo.png)

# AryanVeda DMS

**"Empowering Distribution Networks with Digital Intelligence"**

---

A comprehensive multi-role distribution management platform that connects sales planning, supply-chain execution, stock visibility, field force automation, and notifications in one unified system.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Project Structure](#3-project-structure)
4. [User Roles & Hierarchy](#4-user-roles--hierarchy)
5. [Feature Modules](#5-feature-modules)
   - [Authentication & Security](#51-authentication--security)
   - [User Management](#52-user-management)
   - [Attendance & Field Work](#53-attendance--field-work)
   - [GPS & Location Tracking](#54-gps--location-tracking)
   - [Beat & Visit Management](#55-beat--visit-management)
   - [Order Management](#56-order-management)
   - [Inventory Management](#57-inventory-management)
   - [SKU & Catalog](#58-sku--catalog)
   - [Territory Management](#59-territory-management)
   - [Notifications](#510-notifications)
   - [Dashboard & Analytics](#511-dashboard--analytics)
   - [Retailer-SO Authorization](#512-retailer-so-authorization)
   - [Pricing Management](#513-pricing-management)
6. [Order Flow](#6-order-flow)
7. [Notification System](#7-notification-system)
8. [API Endpoints Overview](#8-api-endpoints-overview)
9. [Frontend Pages](#9-frontend-pages)
10. [Mobile App Features](#10-mobile-app-features)
11. [Quick Start](#11-quick-start)
12. [Testing](#12-testing)

---

## 1. Project Overview

AryanVeda DMS digitizes the complete distributor network workflow:

- Build a strict role and hierarchy-based operating model
- Make every order auditable from create to delivery
- Keep inventory movement accurate by unit conversion and ledger tracking
- Route operational visibility to the right people (participants + POR chain)
- Deliver notifications in-app, by email, and by WhatsApp with role-aware recipients
- Track field force with attendance, GPS, beat visits, and geofencing

---

## 2. Tech Stack

### Backend
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?logo=express&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?logo=jsonwebtokens&logoColor=white)
![Nodemailer](https://img.shields.io/badge/Nodemailer-22A6F2?logo=gmail&logoColor=white)
![MSG91](https://img.shields.io/badge/WhatsApp-MSG91-25D366?logo=whatsapp&logoColor=white)
![Cloudinary](https://img.shields.io/badge/Cloudinary-3448C5?logo=cloudinary&logoColor=white)
![k6](https://img.shields.io/badge/k6-7D64FF?logo=k6&logoColor=white)

### Web Frontend
![Next.js](https://img.shields.io/badge/Next.js-000000?logo=next.js&logoColor=white)
![React](https://img.shields.io/badge/React-61DAFB?logo=react&logoColor=black)
![TanStack Query](https://img.shields.io/badge/TanStack%20Query-FF4154?logo=reactquery&logoColor=white)
![Zustand](https://img.shields.io/badge/Zustand-443E38?logo=react&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-06B6D4?logo=tailwindcss&logoColor=white)

### Mobile App
![Expo](https://img.shields.io/badge/Expo-000020?logo=expo&logoColor=white)
![React Native](https://img.shields.io/badge/React%20Native-20232A?logo=react&logoColor=61DAFB)
- **Navigation**: React Navigation (Drawer + Tab + Stack)
- **State**: Zustand

---

## 3. Project Structure

```
├── backend/              # API server, workflows, notifications
├── frontend/             # Web operations panel (Next.js)
├── my-expo-app/          # Mobile app (React Native)
├── load-test/            # k6 performance test scripts
├── doc/                  # Documentation
│   └── main_docs/
│       ├── PRD/          # Product requirements
│       ├── api/          # API documentation
│       ├── features/     # Feature specs
│       ├── architecture/ # Architecture docs
│       ├── technical/   # Technical docs
│       └── guides/       # User guides
├── assets/               # Brand assets (logo)
└── data/                 # Price lists, SKU data, user exports
```

---

## 4. User Roles & Hierarchy

### Role Branches

| Branch | Roles |
|--------|-------|
| Root | `admin`, `finance`, `dispatch` |
| Sales Force | `nsm`, `hod`, `rsm`, `asm`, `so` |
| Supply Chain | `super_stockist`, `distributor`, `retailer` |

### Hierarchy Chain

```
admin → nsm → rsm → asm → so
admin → super_stockist → distributor
```

### Role Descriptions

| Role | Code | Description |
|------|------|-------------|
| Admin | `admin` | Full system control, user management, catalog CRUD, stock-in approval |
| Finance | `finance` | Financial review of stock-in orders |
| Dispatch | `dispatch` | Dispatch stock after approval |
| National Sales Manager | `nsm` | Regional oversight, RSM creation |
| Regional Sales Manager | `rsm` | Area supervision, ASM management |
| Area Sales Manager | `asm` | Team management, SO oversight |
| Sales Officer | `so` | Field execution, retailer visits, secondary order creation |
| Super Stockist | `super_stockist` | Primary stock holder, distributor management, assortment control |
| Distributor | `distributor` | Secondary distribution, retailer network |
| Retailer | `retailer` | End retailer, self-ordering, SO authorization |

---

## 5. Feature Modules

### 5.1 Authentication & Security

| Feature | Description |
|---------|-------------|
| JWT Authentication | Token-based auth with refresh mechanism |
| Login with OTP | Phone-based OTP login |
| Password Reset | OTP-based password reset |
| Role-Based Access | Middleware-based role guards |
| Hierarchy Scope | Data visibility based on sales hierarchy |
| Email Preference | User can set email notification preferences |

**API Endpoints:**
- `POST /api/auth/login` - Login
- `POST /api/auth/send-otp` - Send OTP
- `POST /api/auth/reset-password` - Reset password
- `POST /api/auth/change-password` - Change password
- `GET /api/auth/me` - Get current user

---

### 5.2 User Management

| Feature | Description |
|---------|-------------|
| Create User | Create users with role assignment |
| User List | Role-scoped user listing |
| Subordinate Chain | View sales hierarchy (NSM→HOD→RSM→ASM→SO) |
| User Status Toggle | Activate/deactivate users |
| Distributor List | List distributors under SS |
| Retailer Network | View retailer's connected network |
| User Count KPI | Count users by role |

**API Endpoints:**
- `POST /api/users` - Create user
- `GET /api/users` - List users
- `GET /api/users/management` - Role-scoped management
- `GET /api/users/subordinates` - Sales hierarchy
- `GET /api/users/distributors` - Distributor list
- `PATCH /api/users/:entityId/status` - Toggle status

---

### 5.3 Attendance & Field Work

| Feature | Description |
|---------|-------------|
| Clock-In/Clock-Out | Daily attendance marking with selfie photo |
| Attendance History | View personal attendance records |
| Attendance Summary | Monthly/yearly attendance stats |
| Admin View | Admin can view all attendance records |
| Attendance Export | Admin can export attendance CSV |
| Role Scope | SO, ASM, RSM can mark attendance |

**API Endpoints:**
- `GET /api/attendance/today-status` - Today's status
- `GET /api/attendance/my` - My attendance history
- `GET /api/attendance/my-summary` - My summary
- `POST /api/attendance/mark` - Clock in/out with selfie
- `GET /api/attendance/admin/history` - Admin view
- `GET /api/attendance/admin/summary` - Admin summary
- `GET /api/attendance/admin/export` - Export CSV

---

### 5.4 GPS & Location Tracking

| Feature | Description |
|---------|-------------|
| GPS Tracking | Real-time location submission by field staff |
| Location History | View own location history |
| Geofencing | Create virtual boundaries for territories |
| Team Live Location | Admin view team live locations |
| Geofence Check | Check if location is within geofence |
| Geofence Management | Create, update, delete geofences |

**API Endpoints:**
- `POST /api/location/track` - Submit GPS location
- `GET /api/location/my-history` - My location history
- `POST /api/location/check-geofence` - Check geofence
- `GET /api/location/admin/team-live` - Team live locations
- `POST /api/location/geofences` - Create geofence
- `GET /api/location/geofences` - List geofences
- `PATCH /api/location/geofences/:geofenceId` - Update geofence

---

### 5.5 Beat & Visit Management

| Feature | Description |
|---------|-------------|
| Retailer Visit | SO visits retailer with photo proof |
| Beat Visit | SO marks daily beat with photo |
| Visit Performance | SO's visit metrics and KPIs |
| Admin Performance View | Admin view all visit performance |
| Visit Export | Export visit data to CSV |

**API Endpoints:**
- `POST /api/retailer-visits/` - Create retailer visit
- `POST /api/retailer-visits/beat` - Create beat visit
- `GET /api/retailer-visits/performance` - SO performance
- `GET /api/retailer-visits/admin/performance` - Admin view
- `GET /api/retailer-visits/admin/export` - Export data

---

### 5.6 Order Management

| Feature | Description |
|---------|-------------|
| Primary Order | Distributor creates primary order to SS |
| Amount Stock-In | SS creates amount-based stock-in order |
| Secondary Order | SO/Retailer creates secondary order |
| Order Approval | SS approves primary orders |
| Stock-In Workflow | Multi-stage: admin → finance → dispatch |
| Order Status Update | Dispatch, deliver, cancel orders |
| Order Summary | Role-scoped order KPI counts |
| Pending Approvals | Admin list pending approvals |
| Order Preview | SS stock check before approval |

**Order States:**
- `CREATED` - Order created
- `IN_FINANCE` - Under finance review
- `APPROVED` - Approved
- `DISPATCHED` - Dispatched to destination
- `DELIVERED` - Delivered to entity
- `CANCELLED` - Cancelled

**API Endpoints:**
- `POST /api/orders/primary` - Create primary order
- `POST /api/orders/primary/amount` - Amount stock-in
- `POST /api/orders/secondary` - Create secondary order
- `GET /api/orders` - List orders
- `GET /api/orders/:orderId` - Order detail
- `POST /api/orders/:orderId/approve` - SS approve
- `POST /api/orders/:orderId/admin-approve` - Admin approve stock-in
- `POST /api/orders/:orderId/admin-reject` - Admin reject
- `PATCH /api/orders/:orderId/finance-edit` - Finance edit
- `POST /api/orders/:orderId/finance-approve` - Finance approve
- `PATCH /api/orders/:orderId/status` - Update status

---

### 5.7 Inventory Management

| Feature | Description |
|---------|-------------|
| Inventory View | View inventory for any entity |
| Inventory Value | Total stock value calculation |
| Inventory Ledger | Full movement history |
| Low Stock Alert | Items below threshold |
| Stock Adjustment | Manual inventory correction |
| Threshold Settings | Set low-stock alerts per SKU |
| Inventory Analytics | Dashboard with slow-moving analysis |
| Inventory Intel | Bulk import inventory data |
| Admin Override | Admin view any user's inventory |

**API Endpoints:**
- `GET /api/inventory/:entityId` - View inventory
- `GET /api/inventory/:entityId/low-stock` - Low stock
- `GET /api/inventory/:entityId/value` - Total value
- `GET /api/inventory/:entityId/ledger` - Ledger history
- `PATCH /api/inventory/:entityId/:skuId/adjust` - Adjust stock
- `PATCH /api/inventory/:entityId/:skuId/threshold` - Set threshold
- `GET /api/inventory/analytics` - Analytics dashboard
- `POST /api/inventory/intel/import` - Bulk import

---

### 5.8 SKU & Catalog

| Feature | Description |
|---------|-------------|
| Master SKU CRUD | Admin creates/updates master SKU |
| Company Stock | Admin manages company-level stock |
| Principal Management | Brand/principal categorization |
| Category Management | Product category hierarchy |
| Pricing Chain | Per-dozen pricing (SS, Distributor, Retail) |
| Assortment | SS enables/disables SKU for distributors |
| Bulk Import | Import SKU data via CSV/Excel |
| Pricing Catalog | Entity-specific pricing display |

**API Endpoints:**
- `POST /api/catalog/admin/sku` - Create master SKU
- `PATCH /api/catalog/admin/sku/:masterSkuId` - Update SKU
- `POST /api/catalog/admin/add-stock` - Add stock
- `POST /api/catalog/admin/bulk-import` - Bulk import
- `GET /api/catalog/pricing/:targetEntityId` - Get pricing
- `PUT /api/catalog/pricing/:targetEntityId/bulk` - Bulk set price
- `POST /api/catalog/assortment/enable` - Enable assortment
- `POST /api/catalog/assortment/disable` - Disable assortment

---

### 5.9 Territory Management

| Feature | Description |
|---------|-------------|
| Territory Assignment | Assign territories to agents |
| Territory List | View all territory assignments |
| Agent Territories | Get territories for specific agent |
| Territory Update | Modify territory assignments |
| Territory Delete | Deactivate territory |
| Supply Chain Mapping | Map supply nodes to sales agents |

**API Endpoints:**
- `POST /api/territories` - Create assignment
- `GET /api/territories` - List territories
- `GET /api/territories/agent/:entityId` - Agent's territories
- `PUT /api/territories/:assignmentId` - Update
- `DELETE /api/territories/:assignmentId` - Delete

---

### 5.10 Notifications

| Feature | Description |
|---------|-------------|
| In-App Notifications | Real-time notification delivery |
| Unread Count | Badge count for unread |
| Mark as Read | Individual and bulk read |
| Email Notifications | Transactional emails (SMTP) |
| WhatsApp Notifications | MSG91 WhatsApp templates |
| Role-Based Routing | POR chain + hierarchy recipients |

**Notification Types:**
- `order_created`
- `order_approved`
- `order_dispatched`
- `order_delivered`
- `order_cancelled`
- `user_activated`
- `user_deactivated`
- `stock_threshold_alert`
- `slow_moving_alert`

**API Endpoints:**
- `GET /api/notifications/unread-count` - Unread count
- `GET /api/notifications` - List notifications
- `PATCH /api/notifications/:notificationId/read` - Mark read
- `PATCH /api/notifications/read-all` - Mark all read

---

### 5.11 Dashboard & Analytics

| Feature | Description |
|---------|-------------|
| Role-Specific Dashboard | KPI dashboard based on role |
| Order KPIs | Order counts by status |
| Inventory Analytics | Stock value, slow-moving items |
| Performance Metrics | Sales performance tracking |
| Attendance Reports | Field force attendance analytics |
| Visit Performance | Beat visit completion metrics |

**API Endpoints:**
- `GET /api/dashboard` - Role-specific dashboard

---

### 5.12 Retailer-SO Authorization

| Feature | Description |
|---------|-------------|
| Retailer Authorizes SO | Retailer grants ordering rights to SO |
| View Authorized SO | Retailer sees their authorized SO |
| View Authorized Retailers | SO sees their retailers |
| Available SO List | Retailer views available SOs |
| Revoke Authorization | Retailer can revoke SO access |

**API Endpoints:**
- `POST /api/retailer-authorization/authorize` - Authorize SO
- `GET /api/retailer-authorization/my-so` - My authorized SO
- `GET /api/retailer-authorization/my-retailers` - My retailers
- `GET /api/retailer-authorization/available-sos` - Available SOs
- `DELETE /api/retailer-authorization/revoke` - Revoke

---

### 5.13 Pricing Management

| Feature | Description |
|---------|-------------|
| Upload Pricing | Upload pricing config file |
| Scoped Pricing | Entity-specific pricing |
| Bulk Pricing | Set prices for multiple SKUs |
| Hierarchical Pricing | SS→Distributor→Retail pricing chain |

**API Endpoints:**
- `POST /api/pricing/upload` - Upload pricing
- `GET /api/pricing/catalog` - Scoped pricing list
- `PUT /api/pricing/catalog/bulk` - Bulk set prices

---

## 6. Order Flow

### Primary Order (Distributor → Super Stockist)

1. Distributor creates `POST /api/orders/primary` → Status: `CREATED`
2. SS reviews and approves `POST /api/orders/:orderId/approve` → Status: `APPROVED`
3. SS dispatches `PATCH /api/orders/:orderId/status` → Status: `DISPATCHED`
4. Distributor confirms delivery → Status: `DELIVERED`

### Stock-In Order (SS Amount-Based)

1. SS creates `POST /api/orders/primary/amount` → Status: `CREATED`
2. Admin: approve / reject / handover to finance
3. Finance: edit / approve / reject
4. Dispatch: dispatch after approval
5. SS confirms delivery

### Secondary Order (SO/Retailer → Distributor)

1. SO or Retailer creates `POST /api/orders/secondary` → Status: `CREATED`
2. Distributor dispatches → Status: `DISPATCHED` (deduct stock)
3. Retailer confirms → Status: `DELIVERED` (increment stock)

---

## 7. Notification System

### Recipient Logic

1. Direct participants (`fromEntityId`, `toEntityId`, actor)
2. POR chain roles based on relationship
3. Territory assignment expansion
4. Deduplication before delivery

### Notification Channels

| Channel | Format |
|---------|--------|
| In-App | Real-time, unread badges |
| Email | SMTP with transactional templates |
| WhatsApp | MSG91 templates (order status, OTP, alerts) |

---

## 8. API Endpoints Overview

| Module | Routes Count | Key Routes |
|--------|--------------|-------------|
| Auth | 8 | login, send-otp, reset-password, change-password, me |
| User | 15 | create, list, management, subordinates, status |
| Attendance | 10 | mark, today-status, history, summary, export |
| Location | 10 | track, geofence, team-live, check-geofence |
| Retailer Visit | 6 | visit, beat, performance, admin-view, export |
| Order | 20 | primary, primary/amount, secondary, approve, status |
| Inventory | 18 | view, adjust, analytics, ledger, intel-import |
| Catalog | 16 | SKU CRUD, assortment, pricing, bulk-import |
| Territory | 7 | create, list, update, delete, agent-territories |
| Notification | 6 | list, read, unread-count, read-all |
| Stock Movement | 5 | own, downstream, trigger-email, admin-view |
| Pricing | 5 | upload, catalog, bulk, entity-pricing |
| Retailer Auth | 6 | authorize, revoke, my-so, my-retailers |
| Email | 3 | health, send-test, preferences |
| Announcement | 5 | create, list, read, delete |
| WhatsApp | 0 | Service layer (no mounted routes) |

---

## 9. Frontend Pages

### Public Routes
- `/login` - Login page
- `/forgot-password` - Forgot password

### Dashboard & Home
- `/dashboard` - Role-specific KPI dashboard
- `/leaderboard` - Sales leaderboard

### Field Work
- `/attendance` - Attendance clock-in/out with selfie photo
- `/beat-visits` - Beat visit management with photo proof
- `/beat-wise` - Beat-wise performance view
- `/gps-tracking` - GPS location tracking history

### Management
- `/users` - User management (list, create, toggle status)
- `/territories` - Territory management (assignments, CRUD)
- `/admin/attendance` - Admin attendance view and export
- `/admin/team-location` - Team live location tracking
- `/admin/geofence` - Geofence management (create, update, delete)
- `/admin/performance` - Admin performance metrics view

### Orders
- `/orders` - Order list with filters
- `/orders/new/primary` - Create primary order (Distributor → SS)
- `/orders/new/secondary` - Create secondary order (SO/Retailer → Distributor)
- `/orders/[id]` - Order detail view

### Inventory & Stock
- `/inventory` - Entity inventory management
- `/inventory-intel` - Bulk inventory import
- `/inventory-analytics` - Analytics dashboard with slow-moving analysis
- `/stock-movements` - Stock movement log (own + downstream)

### Catalog & SKU
- `/catalog` - Product catalog management
- `/catalog/product-catalogs` - Product catalogs list
- `/skus` - SKU master list
- `/skus/new` - Create new SKU
- `/skus/[skuId]/edit` - Edit SKU

### Announcements
- `/announcement` - System announcements management

### Analytics & Performance
- `/performance` - Performance metrics
- `/outlet-wise` - Outlet-wise analytics
- `/admin/inventory` - Admin inventory overview

---

## 10. Mobile App Features

### Navigation
- Drawer navigation with role-based access
- Bottom tab navigation (Home, Orders, Stock, Catalog)
- Stack navigation for details

### Mobile Screens

| Screen | Description |
|--------|-------------|
| LoginScreen | Phone + OTP login |
| ForgotPasswordScreen | Password recovery |
| ResetPasswordScreen | Set new password |
| ProfileScreen | User profile |
| DashboardScreen | KPI dashboard |
| OrdersScreen | Order list with tabs |
| CreateOrderScreen | Create new order |
| OrderDetailScreen | Order details |
| InventoryScreen | Inventory overview |
| InventoryListScreen | SKU inventory list |
| InventoryItemScreen | Single item detail |
| CatalogScreen | Product catalog |
| CatalogListScreen | SKU list |
| CatalogItemScreen | SKU detail |
| StockMovementsScreen | Movement history |
| SkuMasterScreen | SKU management |
| UsersScreen | User management |
| TerritoriesScreen | Territory view |
| NotificationsScreen | Notification list |
| EmailSettingsScreen | Email preferences |

---

## 11. Quick Start

### Backend
```bash
cd backend
npm install
npm run dev
```

### Frontend (Web)
```bash
cd frontend
npm install
npm run dev
```

### Mobile App
```bash
cd my-expo-app
npm install
npm run start
```

### Seed Data
```bash
cd backend
npx ts-node scripts/mobil_seed.ts
```

---

## 12. Testing

### Backend Tests
```bash
cd backend
npm test
```

### Load Testing
```bash
cd load-test
k6 run k6-load-test.js
```

---

## Documentation Index

| Location | Description |
|----------|-------------|
| `doc/main_docs/PRD/` | Product requirements |
| `doc/main_docs/api/` | API documentation |
| `doc/main_docs/features/` | Feature specs |
| `doc/main_docs/architecture/` | Architecture |
| `doc/main_docs/technical/` | Technical docs |
| `doc/main_docs/guides/` | User guides |

---

*Last Updated: May 2026*