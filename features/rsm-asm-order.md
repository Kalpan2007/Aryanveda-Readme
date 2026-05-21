# RSM/ASM Order from Distributor Feature

## Overview

This feature allows Regional Sales Managers (RSM) and Area Sales Managers (ASM) to place orders on behalf of distributors in their assigned territories. This enables sales managers to initiate orders for distributors who may not have direct system access or need assistance with ordering.

## Architecture

### Backend Implementation

#### Routes

| Method | Route | Access | Handler |
|---|---|---|---|
| GET | `/api/orders/sales-user/connected-distributors` | `rsm`, `asm` | `getConnectedDistributorsHandler` |
| POST | `/api/orders/sales-user` | `rsm`, `asm` | `createSalesUserOrderHandler` |

**File:** `backend/src/modules/order/order.routes.ts` (lines 144-156)

#### Controllers

**File:** `backend/src/modules/order/order.controller.ts`

- `getConnectedDistributorsHandler` (line 365)
- `createSalesUserOrderHandler` (line 378)

#### Service Layer

**File:** `backend/src/modules/order/order.service.ts`

- `getConnectedDistributorsForSalesUser` (line 2785)
- `createSalesUserOrder` (line 2856)

### How It Works

#### 1. Get Connected Distributors

**Endpoint:** `GET /api/orders/sales-user/connected-distributors`

**Access:** Restricted to `RSM` and `ASM` roles only.

**Logic:**
- Finds distributors via territory assignments linked to the sales user
- Returns only active distributors
- Territory assignments connect RSM/ASM to specific geographic areas
- Distributors assigned to those territories are returned as "connected"

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "entityId": "dist_123",
      "name": "Distributor Name",
      "email": "dist@example.com",
      "phone": "9876543210",
      "state": "Maharashtra",
      "beat": "Mumbai Central",
      "status": "active"
    }
  ]
}
```

#### 2. Create Sales User Order

**Endpoint:** `POST /api/orders/sales-user`

**Access:** Restricted to `RSM` and `ASM` roles only.

**Request Body:**
```json
{
  "distributorId": "dist_123",
  "items": [
    {
      "skuId": "sku_456",
      "quantity": 10,
      "unitPrice": 150.00
    },
    {
      "skuId": "sku_789",
      "quantity": 5,
      "unitPrice": 200.00
    }
  ],
  "notes": "Urgent order for month-end stock"
}
```

**Validation:**
- Validates that the distributor is connected via territory assignment
- Validates items array is non-empty
- Resolves SKU prices from catalog
- Validates quantities are positive integers

**Order Creation:**
- Creates a PRIMARY order with:
  - `fromEntityId`: distributor's entityId
  - `toEntityId`: RSM/ASM's entityId
  - `createdByRole`: `RSM` or `ASM`
  - `status`: `CREATED`
  - `orderType`: `PRIMARY`
- Sends notifications via POR (Parent-Organization-Root) chain

**Response:**
```json
{
  "success": true,
  "data": {
    "orderId": "order_abc123",
    "status": "CREATED",
    "totalAmount": 2500.00,
    "itemCount": 2
  }
}
```

### Frontend Implementation

#### Orders Page

**File:** `frontend/src/app/orders/orders-client.tsx`

- RSM/ASM distributor selector UI (lines 339-345, 866-947)
- Shows connected distributors in dropdown
- Filters orders by selected distributor

#### Sales User Order Creation

**File:** `frontend/src/app/orders/new/sales-user/page.tsx`

**File:** `frontend/src/app/orders/new/sales-user/sales-user-order-client.tsx`

- Order creation form for RSM/ASM
- Distributor selection dropdown
- SKU item picker with quantity and price
- Order summary and submission

#### Order Service

**File:** `frontend/src/services/order.service.ts`

- `getConnectedDistributors` (line 229)
- `createSalesUserOrder` (line 235)

### Order Visibility

RSM/ASM users see orders in their sales scope through the `getSalesScopeEntityIds` function in the order service. This includes:
- Subordinates in their hierarchy
- Territory targets (distributors assigned to their territories)

### Important Notes

1. **Finance Visibility:** Finance role explicitly excludes RSM/ASM-created orders from their view (`createdByRole: { $nin: [UserRole.RSM, UserRole.ASM] }` in order listing). This is intentional as these orders follow a different approval flow.

2. **Order Status:** RSM/ASM orders are created as `CREATED` primary orders. The distributor must dispatch them, but there is no dedicated RSM/ASM-specific approval step.

3. **Territory Dependency:** The feature relies on proper territory assignments. Without territory assignments linking RSM/ASM to distributors, the connected distributors list will be empty.

4. **Notifications:** Orders created by RSM/ASM trigger notifications through the POR chain to inform relevant parties (distributor, super stockist, admin).

## Testing

### Manual Testing Steps

1. Login as RSM or ASM user
2. Navigate to Orders > New Order > Sales User Order
3. Verify connected distributors appear in dropdown
4. Select a distributor
5. Add items to the order
6. Submit the order
7. Verify order appears in order list
8. Verify notifications are sent

### API Testing

```bash
# Get connected distributors
curl -X GET "http://localhost:3000/api/orders/sales-user/connected-distributors" \
  -H "Authorization: Bearer <rsm_or_asm_token>"

# Create order
curl -X POST "http://localhost:3000/api/orders/sales-user" \
  -H "Authorization: Bearer <rsm_or_asm_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "distributorId": "dist_123",
    "items": [
      {"skuId": "sku_456", "quantity": 10, "unitPrice": 150}
    ]
  }'
```

## Future Enhancements

1. **Approval Flow:** Add dedicated approval step for RSM/ASM orders before they reach distributor
2. **Order Limits:** Implement credit limits or order value caps per distributor
3. **Order Templates:** Allow RSM/ASM to save order templates for frequent orders
4. **Bulk Orders:** Support bulk order creation across multiple distributors
5. **Order Tracking:** Enhanced tracking for RSM/ASM-created orders through dispatch and delivery
