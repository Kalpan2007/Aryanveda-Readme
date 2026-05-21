# Admin User Management - Edit and Delete

## Overview

Admin users have the ability to edit and delete other users in the system. This feature provides user management capabilities for system administrators to maintain user data and remove inactive or unauthorized accounts.

## Architecture

### Backend Implementation

#### Routes

| Method | Route | Access | Handler |
|---|---|---|---|
| PATCH | `/api/users/:entityId` | `admin` | `editUserHandler` |
| DELETE | `/api/users/:entityId` | `admin` | `deleteUserHandler` |

**File:** `backend/src/modules/user/user.routes.ts` (lines 171-183)

#### Controllers

**File:** `backend/src/modules/user/user.controller.ts`

- `editUserHandler` (line 493)
- `deleteUserHandler` (line 502)

#### Service Layer

**File:** `backend/src/modules/user/user.service.ts`

- `editUser` (line 1826)
- `deleteUser` (line 1893)

### Edit User Feature

#### Endpoint

**Endpoint:** `PATCH /api/users/:entityId`

**Access:** Admin only.

#### Request Body

```json
{
  "name": "Updated User Name",
  "email": "newemail@example.com",
  "phone": "9876543210"
}
```

**Editable Fields:**
- `name`: User's full name (must be non-empty)
- `email`: User's email address (must be unique if provided)
- `phone`: User's phone number (must be 10 digits if provided)

#### Validation Rules

1. **Name:** Must be non-empty string
2. **Email:** Must be valid email format, must be unique across all users
3. **Phone:** Must be exactly 10 digits
4. **Self-Edit Protection:** Admin cannot edit their own account
5. **Uniqueness:** Email and phone must not conflict with existing users

#### Response

**Success (200):**
```json
{
  "success": true,
  "data": {
    "entityId": "user_123",
    "name": "Updated User Name",
    "email": "newemail@example.com",
    "phone": "9876543210",
    "role": "distributor",
    "status": "active",
    "updatedAt": "2026-05-17T10:30:00.000Z"
  }
}
```

**Error (400):**
```json
{
  "success": false,
  "error": "Email already exists"
}
```

**Error (403):**
```json
{
  "success": false,
  "error": "Cannot edit your own account"
}
```

#### Implementation Details

**File:** `backend/src/modules/user/user.service.ts` (lines 1826-1889)

```typescript
async function editUser(entityId: string, adminId: string, updates: EditUserInput) {
  // 1. Prevent self-edit
  if (entityId === adminId) {
    throw new ForbiddenError('Cannot edit your own account');
  }

  // 2. Validate input fields
  validateEditInput(updates);

  // 3. Check email uniqueness if email is being updated
  if (updates.email) {
    await checkEmailUniqueness(updates.email, entityId);
  }

  // 4. Check phone uniqueness if phone is being updated
  if (updates.phone) {
    await checkPhoneUniqueness(updates.phone, entityId);
  }

  // 5. Update user in database
  const updatedUser = await updateUserById(entityId, updates);

  return updatedUser;
}
```

### Delete User Feature

#### Endpoint

**Endpoint:** `DELETE /api/users/:entityId`

**Access:** Admin only.

#### Request

No request body required. User ID is passed in the URL path.

#### Validation Rules

1. **Self-Delete Protection:** Admin cannot delete their own account
2. **Admin Protection:** Cannot delete other admin users
3. **Existence Check:** User must exist

#### Response

**Success (200):**
```json
{
  "success": true,
  "data": {
    "message": "User deleted successfully",
    "entityId": "user_123"
  }
}
```

**Error (403):**
```json
{
  "success": false,
  "error": "Cannot delete your own account"
}
```

**Error (403):**
```json
{
  "success": false,
  "error": "Cannot delete admin users"
}
```

**Error (404):**
```json
{
  "success": false,
  "error": "User not found"
}
```

#### Implementation Details

**File:** `backend/src/modules/user/user.service.ts` (lines 1893-1913)

```typescript
async function deleteUser(entityId: string, adminId: string) {
  // 1. Prevent self-delete
  if (entityId === adminId) {
    throw new ForbiddenError('Cannot delete your own account');
  }

  // 2. Fetch user to check role
  const user = await getUserById(entityId);
  if (!user) {
    throw new NotFoundError('User not found');
  }

  // 3. Prevent deleting other admins
  if (user.role === UserRole.ADMIN) {
    throw new ForbiddenError('Cannot delete admin users');
  }

  // 4. Hard delete user from database
  await deleteById(entityId);

  return { entityId };
}
```

### Frontend Implementation

#### Users Page

**File:** `frontend/src/app/users/users-client.tsx`

- Edit mutation (lines 180-228)
- Delete mutation (lines 180-228)
- `EditUserModal` component (lines 1032-1145)

**Edit User Modal Features:**
- Form with name, email, phone fields
- Validation feedback
- Submit and cancel buttons
- Loading states during API calls

**Delete User Flow:**
- Confirmation dialog before deletion
- Shows user name and role being deleted
- Success/error notifications

#### User Table

**File:** `frontend/src/components/UserTable.tsx`

- Edit and delete buttons shown only when:
  - `isAdmin=true` (user is admin)
  - `user.role !== ADMIN` (target user is not admin)
- Buttons appear in actions column (lines 383-412)

#### User Service

**File:** `frontend/src/services/user.service.ts`

- `editUser(entityId, data)`: Sends PATCH request to update user
- `deleteUser(entityId)`: Sends DELETE request to remove user

### Important Notes

1. **Hard Delete:** User deletion is a hard delete (`deleteById`). The user is permanently removed from the database with no audit trail. Consider implementing soft-delete for compliance.

2. **Limited Editable Fields:** Admin can only edit `name`, `email`, and `phone`. Cannot edit:
   - `state`
   - `beat`
   - `role`
   - `parentId`
   - `status` (use separate status toggle endpoint)
   - Other profile fields

3. **No Bulk Edit:** No bulk edit capability exists for admin user management. Only bulk activate/deactivate is available via the status toggle endpoint.

4. **No Cascade Delete:** When a user is deleted, their related data (orders, inventory, etc.) is not automatically handled. This may leave orphaned references.

5. **Audit Trail:** No audit log is created for edit or delete operations. Consider adding audit logging for compliance.

## Testing

### Manual Testing Steps

#### Edit User
1. Login as admin user
2. Navigate to Users page
3. Find a non-admin user in the table
4. Click Edit button
5. Modify name, email, or phone
6. Submit the form
7. Verify changes appear in the table
8. Try editing your own account (should fail)
9. Try using duplicate email (should fail)

#### Delete User
1. Login as admin user
2. Navigate to Users page
3. Find a non-admin user in the table
4. Click Delete button
5. Confirm deletion in dialog
6. Verify user is removed from the table
7. Try deleting your own account (should fail)
8. Try deleting another admin (should fail)

### API Testing

```bash
# Edit user
curl -X PATCH "http://localhost:3000/api/users/user_123" \
  -H "Authorization: Bearer <admin_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated Name",
    "email": "newemail@example.com",
    "phone": "9876543210"
  }'

# Delete user
curl -X DELETE "http://localhost:3000/api/users/user_123" \
  -H "Authorization: Bearer <admin_token>"
```

## Future Enhancements

1. **Soft Delete:** Implement soft-delete mechanism to maintain audit trail and allow user recovery
2. **Extended Edit Fields:** Allow admin to edit additional fields like state, beat, role, parent
3. **Bulk Edit:** Add bulk edit capability for multiple users
4. **Audit Logging:** Log all edit and delete operations for compliance
5. **Cascade Handling:** Handle related data when user is deleted (orders, inventory, etc.)
6. **User Deactivation:** Prefer deactivation over deletion for inactive users
7. **Edit History:** Track changes to user data over time
8. **Role-Based Edit Restrictions:** Allow certain roles to edit specific fields only
