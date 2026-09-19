# 📦 Stock Management System - API Documentation

A comprehensive guide and reference for all endpoints, authentication rules, JSON schemas, form-data payloads, and workflows in the Stock Management API.

---

## 📑 Table of Contents

1. [General Information & Headers](#1-general-information--headers)
2. [Global Guards & RBAC Architecture](#2-global-guards--rbac-architecture)
3. [Authentication API (`/auth`)](#3-authentication-api-auth)
4. [User Management API (`/users`)](#4-user-management-api-users)
5. [Roles & Role-Menu/Permission API (`/roles`)](#5-roles--role-menupermission-api-roles)
6. [Permissions API (`/permissions`)](#6-permissions-api-permissions)
7. [Menu Navigation API (`/menu`)](#7-menu-navigation-api-menu)
8. [Brands API (`/brands`)](#8-brands-api-brands)
9. [Categories API (`/categories`)](#9-categories-api-categories)
10. [Suppliers API (`/suppliers`)](#10-suppliers-api-suppliers)
11. [Warehouses API (`/warehouses`)](#11-warehouses-api-warehouses)
12. [Products API (`/products`)](#12-products-api-products)
13. [Product Variants API (`/product-variants`)](#13-product-variants-api-product-variants)
14. [Requests & Stock Approval Workflow API (`/requests`)](#14-requests--stock-approval-workflow-api-requests)
    - [Excel Template Download](#141-download-excel-template)
    - [Excel Import](#142-excel-import)
    - [Manual Request Creation (Multipart Form-Data)](#143-create-request-manually)
    - [Detailed Form-Data Examples by Request Type](#144-detailed-multipartform-data-examples)
    - [Approval & Rejection (Commit)](#145-commit--approve--reject-request)
15. [Mails & Notifications API (`/mails`)](#15-mails--notifications-api-mails)
16. [Direct Stock & Adjustments API (`/stock`, `/stock-adjustments`)](#16-direct-stock--adjustments-api)

---

## 1. General Information & Headers

- **Base URL**: `http://localhost:3000` (or as configured in `.env` `PORT`)
- **Default Content-Type**: `application/json` (unless file upload is required: `multipart/form-data`)
- **Authorization**: `Bearer <accessToken>`

### Standard Headers

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json
```

---

## 2. Global Guards & RBAC Architecture

The API implements three global application guards:

1. **`AccessTokenGuard`**:
   - Every route requires a valid Bearer token in the `Authorization` header.
   - Routes decorated with `@Public()` do not require a token.
2. **`MustChangePasswordGuard`**:
   - If `user.mustChangePassword === true` (e.g., first login with temporary password), access to all normal endpoints is blocked (`403 Forbidden`).
   - The user can ONLY access routes decorated with `@Public()` or `@AllowPasswordChange()` (such as `POST /auth/change-password` and `POST /auth/refresh`).
3. **`PermissionGuard`**:
   - Checks user role permissions for routes decorated with `@RequirePermission('PERMISSION_NAME')`.
   - Users with the **`SUPER_ADMIN`** role automatically bypass all permission checks.

---

## 3. Authentication API (`/auth`)

### 3.1 Login

- **Endpoint**: `POST /auth/login`
- **Access**: Public (`@Public()`)
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "staffId": "KH00001",
  "password": "Password123!"
}
```

#### Field Specifications

| Field      | Type     | Required | Description                |
| :--------- | :------- | :------- | :------------------------- |
| `staffId`  | `string` | Yes      | Staff ID (e.g., `KH00001`) |
| `password` | `string` | Yes      | Plain text password        |

#### Response (200 OK)

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsIn...",
  "refreshToken": "48b61c92-d6cb-40bc-96b6-d249d3750a98",
  "user": {
    "id": "7fa118cf-bfd3-4a11-8e5f-1550c60da6e2",
    "staffId": "KH00001",
    "status": "ACTIVE",
    "mustChangePassword": false,
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "avatar": "https://ui-avatars.com/api/?name=JD&background=random",
      "email": "admin@example.com",
      "phone": "+85512345678",
      "telegramChatId": null
    },
    "roles": ["SUPER_ADMIN"],
    "permissions": ["PRODUCT_CREATE", "PRODUCT_VIEW", "STOCK_VIEW", "..."],
    "menus": [
      {
        "id": "...",
        "name": "dashboard",
        "label": "Dashboard",
        "path": "/dashboard",
        "icon": "home",
        "sortOrder": 1,
        "children": []
      }
    ]
  }
}
```

---

### 3.2 Change Password

- **Endpoint**: `POST /auth/change-password`
- **Access**: Authenticated (`@AllowPasswordChange()`)
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "MyNewSecurePassword456!"
}
```

#### Field Specifications

| Field             | Type     | Required | Description                         |
| :---------------- | :------- | :------- | :---------------------------------- |
| `currentPassword` | `string` | Yes      | Current password                    |
| `newPassword`     | `string` | Yes      | New password (minimum 8 characters) |

#### Response (200 OK)

```json
{
  "message": "Password changed successfully"
}
```

---

### 3.3 Refresh Token

- **Endpoint**: `POST /auth/refresh`
- **Access**: Public (`@AllowPasswordChange()`)
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "refreshToken": "48b61c92-d6cb-40bc-96b6-d249d3750a98"
}
```

#### Response (200 OK)

```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsIn...",
  "refreshToken": "new-refresh-token-uuid"
}
```

---

## 4. User Management API (`/users`)

### 4.1 Get Current Profile & Permissions

- **Endpoint**: `GET /users/me`
- **Access**: Authenticated

#### Response (200 OK)

Returns user record with profile, roles array, permissions array, and hierarchical menus tree.

---

### 4.2 List All Users

- **Endpoint**: `GET /users`
- **Access**: Authenticated + `@RequirePermission('USER_VIEW')`

#### Response (200 OK)

```json
[
  {
    "id": "7fa118cf-bfd3-4a11-8e5f-1550c60da6e2",
    "staffId": "KH00001",
    "status": "ACTIVE",
    "mustChangePassword": false,
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "email": "admin@example.com",
      "phone": "+85512345678",
      "avatar": "https://..."
    },
    "userRoles": [
      {
        "role": {
          "id": "d3b07384-d113-4603-a128-490b4d455498",
          "name": "SUPER_ADMIN"
        }
      }
    ]
  }
]
```

---

### 4.3 Create User (Staff)

- **Endpoint**: `POST /users`
- **Access**: Authenticated
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "firstName": "Sokha",
  "lastName": "Chan",
  "email": "sokha.chan@example.com",
  "phone": "+85512999888",
  "telegramChatId": "987654321"
}
```

#### Field Specifications

| Field            | Type     | Required | Description                               |
| :--------------- | :------- | :------- | :---------------------------------------- |
| `firstName`      | `string` | Yes      | Max 100 characters                        |
| `lastName`       | `string` | Yes      | Max 100 characters                        |
| `email`          | `string` | Yes      | Valid email format, max 255 chars, unique |
| `phone`          | `string` | No       | Max 30 characters                         |
| `telegramChatId` | `string` | No       | Max 100 characters                        |

#### Response (201 Created)

```json
{
  "staffId": "KH00002",
  "temporaryPassword": "XyZ123!a"
}
```

_(A welcome email with login credentials is automatically dispatched to the staff email)_

---

### 4.4 Get User Roles

- **Endpoint**: `GET /users/:id/roles`
- **Access**: Authenticated
- **Param**: `:id` (User UUID)

---

### 4.5 Update User Role

- **Endpoint**: `PUT /users/:id/role`
- **Access**: Authenticated + `@RequirePermission('USER_ROLE_UPDATE')`
- **Param**: `:id` (User UUID)
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "roleId": "d3b07384-d113-4603-a128-490b4d455498"
}
```

---

## 5. Roles & Role-Menu/Permission API (`/roles`)

### 5.1 Endpoints Summary

| Method   | Endpoint                                   | Permission    | Description                                    |
| :------- | :----------------------------------------- | :------------ | :--------------------------------------------- |
| `GET`    | `/roles`                                   | -             | List all roles                                 |
| `POST`   | `/roles`                                   | `ROLE_CREATE` | Create a new custom role                       |
| `GET`    | `/roles/:id`                               | -             | Get role details                               |
| `GET`    | `/roles/:id/permissions`                   | -             | Get permissions assigned to role               |
| `POST`   | `/roles/:id/permissions`                   | `ROLE_UPDATE` | Add permissions to role                        |
| `GET`    | `/roles/:id/permissions/manage`            | `ROLE_VIEW`   | Grouped permission matrix with assigned status |
| `DELETE` | `/roles/:roleId/permissions/:permissionId` | `ROLE_UPDATE` | Remove single permission from role             |
| `GET`    | `/roles/:id/menus`                         | -             | Get role menus tree                            |
| `GET`    | `/roles/:id/menus/manage`                  | -             | Get menu checklist for role management         |
| `POST`   | `/roles/:roleId/menus/:menuId`             | -             | Assign menu item to role                       |
| `DELETE` | `/roles/:roleId/menus/:menuId`             | -             | Unassign menu item from role                   |

#### Request Bodies

- **Create Role (`POST /roles`)**:

```json
{
  "name": "WAREHOUSE_SUPERVISOR"
}
```

- **Add Permissions to Role (`POST /roles/:id/permissions`)**:

```json
{
  "permissionIds": [
    "c4a17961-0f7f-44eb-843e-c689d0208226",
    "f2963162-43bb-403d-82d2-caef61479867"
  ]
}
```

---

## 6. Permissions API (`/permissions`)

### 6.1 Endpoints Summary

| Method | Endpoint                 | Permission          | Description                                   |
| :----- | :----------------------- | :------------------ | :-------------------------------------------- |
| `GET`  | `/permissions`           | -                   | List all permissions                          |
| `GET`  | `/permissions/:id`       | -                   | Get permission by ID                          |
| `POST` | `/permissions`           | -                   | Create individual permission                  |
| `POST` | `/permissions/bulk`      | -                   | Bulk create multiple permissions              |
| `POST` | `/permissions/resources` | `PERMISSION_CREATE` | Auto-generate CRUD permissions for a resource |

#### Request Bodies

- **Create Permission (`POST /permissions`)**:

```json
{
  "resource": "audit_log",
  "action": "view"
}
```

- **Bulk Create Permissions (`POST /permissions/bulk`)**:

```json
{
  "permissions": [
    { "resource": "finance", "action": "view" },
    { "resource": "finance", "action": "create" },
    { "resource": "finance", "action": "update" },
    { "resource": "finance", "action": "delete" }
  ]
}
```

- **Auto Generate CRUD for Resource (`POST /permissions/resources`)**:

```json
{
  "resource": "shipments"
}
```

_(Generates `SHIPMENTS_CREATE`, `SHIPMENTS_VIEW`, `SHIPMENTS_UPDATE`, `SHIPMENTS_DELETE`)_

---

## 7. Menu Navigation API (`/menu`)

### 7.1 Endpoints Summary

| Method   | Endpoint    | Permission    | Description                |
| :------- | :---------- | :------------ | :------------------------- |
| `POST`   | `/menu`     | `MENU_CREATE` | Create a menu entry        |
| `GET`    | `/menu`     | `MENU_VIEW`   | Get hierarchical menu tree |
| `GET`    | `/menu/:id` | `MENU_VIEW`   | Get single menu item       |
| `PUT`    | `/menu/:id` | `MENU_UPDATE` | Update menu entry          |
| `DELETE` | `/menu/:id` | `MENU_DELETE` | Delete menu entry          |

#### Request Bodies

- **Create Menu (`POST /menu`)**:

```json
{
  "name": "stock_management",
  "label": "Stock & Warehouse",
  "path": "/stock",
  "icon": "inventory",
  "parentId": null,
  "sortOrder": 3,
  "isActive": true
}
```

- **Update Menu (`PUT /menu/:id`)**:

```json
{
  "label": "Inventory & Stock",
  "sortOrder": 1,
  "isActive": true
}
```

---

## 8. Brands API (`/brands`)

### 8.1 Endpoints Summary

| Method  | Endpoint                 | Description          |
| :------ | :----------------------- | :------------------- |
| `POST`  | `/brands`                | Create new brand     |
| `GET`   | `/brands`                | List all brands      |
| `GET`   | `/brands/:id`            | Get brand by ID      |
| `PATCH` | `/brands/:id`            | Update brand details |
| `PATCH` | `/brands/:id/deactivate` | Deactivate brand     |
| `PATCH` | `/brands/:id/activate`   | Activate brand       |

#### Request Bodies

- **Create Brand (`POST /brands`)**:

```json
{
  "code": "SAMSUNG",
  "name": "Samsung Electronics",
  "description": "Smartphones, appliances and chips",
  "isActive": true
}
```

- **Update Brand (`PATCH /brands/:id`)**:

```json
{
  "name": "Samsung Electronics Co.",
  "description": "Global electronics manufacturer"
}
```

---

## 9. Categories API (`/categories`)

### 9.1 Endpoints Summary

| Method   | Endpoint                     | Description         |
| :------- | :--------------------------- | :------------------ |
| `POST`   | `/categories`                | Create category     |
| `GET`    | `/categories`                | List all categories |
| `GET`    | `/categories/:id`            | Get category by ID  |
| `PATCH`  | `/categories/:id`            | Update category     |
| `DELETE` | `/categories/:id`            | Delete category     |
| `PATCH`  | `/categories/:id/deactivate` | Deactivate category |
| `PATCH`  | `/categories/:id/activate`   | Activate category   |

#### Request Bodies

- **Create Category (`POST /categories`)**:

```json
{
  "code": "LAPTOPS",
  "name": "Laptops & Notebooks",
  "description": "Portable personal computing devices",
  "isActive": true
}
```

- **Update Category (`PATCH /categories/:id`)**:

```json
{
  "name": "Laptops & MacBooks"
}
```

---

## 10. Suppliers API (`/suppliers`)

### 10.1 Endpoints Summary

| Method  | Endpoint                    | Description         |
| :------ | :-------------------------- | :------------------ |
| `POST`  | `/suppliers`                | Create supplier     |
| `GET`   | `/suppliers`                | List all suppliers  |
| `GET`   | `/suppliers/:id`            | Get supplier by ID  |
| `PATCH` | `/suppliers/:id`            | Update supplier     |
| `PATCH` | `/suppliers/:id/deactivate` | Deactivate supplier |
| `PATCH` | `/suppliers/:id/activate`   | Activate supplier   |

#### Request Bodies

- **Create Supplier (`POST /suppliers`)**:

```json
{
  "code": "SUP-GLOBAL-01",
  "name": "Global Tech Logistics Ltd",
  "contactPerson": "Alice Smith",
  "phone": "+85512345678",
  "email": "contact@globaltech.com",
  "address": "Phnom Penh Special Economic Zone"
}
```

---

## 11. Warehouses API (`/warehouses`)

### 11.1 Endpoints Summary

| Method  | Endpoint                     | Description          |
| :------ | :--------------------------- | :------------------- |
| `POST`  | `/warehouses`                | Create warehouse     |
| `GET`   | `/warehouses`                | List all warehouses  |
| `GET`   | `/warehouses/:id`            | Get warehouse by ID  |
| `PATCH` | `/warehouses/:id`            | Update warehouse     |
| `PATCH` | `/warehouses/:id/deactivate` | Deactivate warehouse |
| `PATCH` | `/warehouses/:id/activate`   | Activate warehouse   |

#### Request Bodies

- **Create Warehouse (`POST /warehouses`)**:

```json
{
  "code": "WH-PP-MAIN",
  "name": "Phnom Penh Central Distribution",
  "description": "Primary central warehouse facility",
  "address": "National Road 4, Phnom Penh",
  "latitude": 11.5564,
  "longitude": 104.9282,
  "contactPerson": "Dara Meng",
  "phone": "+85598765432"
}
```

---

## 12. Products API (`/products`)

_(Note: Products and stock are registered and modified through the Approval Workflow in `/requests`)_

| Method | Endpoint        | Description                                                    |
| :----- | :-------------- | :------------------------------------------------------------- |
| `GET`  | `/products`     | List all products with category, brand, supplier, and variants |
| `GET`  | `/products/:id` | Get product details by ID                                      |

---

## 13. Product Variants API (`/product-variants`)

| Method  | Endpoint                           | Description                                |
| :------ | :--------------------------------- | :----------------------------------------- |
| `GET`   | `/product-variants`                | List variants (Query: `?productId=<UUID>`) |
| `GET`   | `/product-variants/:id`            | Get variant by ID                          |
| `POST`  | `/product-variants`                | Create product variant                     |
| `PATCH` | `/product-variants/:id`            | Update variant details                     |
| `PATCH` | `/product-variants/:id/deactivate` | Deactivate variant                         |
| `PATCH` | `/product-variants/:id/activate`   | Activate variant                           |

#### Request Body (`POST /product-variants`):

```json
{
  "productId": "047c34d3-e7f0-4660-84cf-cb09ebbcbaae",
  "code": "MBP16-M3-SLV",
  "name": "MacBook Pro 16\" M3 Max Silver",
  "sku": "MBP16-M3-SLV-1TB",
  "barcode": "885909123456",
  "attributes": {
    "color": "Silver",
    "chip": "M3 Max",
    "storage": "1TB",
    "ram": "36GB"
  },
  "costPrice": 2800.0,
  "sellingPrice": 3499.0
}
```

---

## 14. Requests & Stock Approval Workflow API (`/requests`)

The core workflow engine handling product creation, variant creation/updating, stock in, stock out, stock transfer, and stock adjustments.

### Supported Request Types (`RequestType` Enum):

- `PRODUCT_CREATE`
- `PRODUCT_UPDATE`
- `VARIANT_CREATE`
- `VARIANT_UPDATE`
- `STOCK_IN`
- `STOCK_OUT`
- `STOCK_TRANSFER`
- `STOCK_ADJUSTMENT`

---

### 14.1 Download Excel Template

- **Endpoint**: `GET /requests/import/template?type=<REQUEST_TYPE>`
- **Access**: Public (`@Public()`)
- **Query Param**: `type` (one of the enum types above)
- **Response**: Binary Excel `.xlsx` file download.

---

### 14.2 Excel Import

- **Endpoint**: `POST /requests/import`
- **Access**: Authenticated
- **Content-Type**: `multipart/form-data`

#### Form-Data Fields:

| Key           | Type | Value / Description                                             |
| :------------ | :--- | :-------------------------------------------------------------- |
| `file`        | File | The filled `.xlsx` Excel template file                          |
| `requestType` | Text | The request type matching the template (e.g., `PRODUCT_CREATE`) |

---

### 14.3 Create Request Manually

- **Endpoint**: `POST /requests`
- **Access**: Authenticated
- **Content-Type**: `multipart/form-data`

#### Form-Data Keys Overview:

| Key                           | Type               | Description                                                                                                                           |
| :---------------------------- | :----------------- | :------------------------------------------------------------------------------------------------------------------------------------ |
| `requestType`                 | Text               | `PRODUCT_CREATE`, `PRODUCT_UPDATE`, `VARIANT_CREATE`, `VARIANT_UPDATE`, `STOCK_IN`, `STOCK_OUT`, `STOCK_TRANSFER`, `STOCK_ADJUSTMENT` |
| `product`                     | Text (JSON String) | Serialized product JSON object (optional)                                                                                             |
| `variants`                    | Text (JSON String) | Serialized array of variant JSON objects (optional)                                                                                   |
| `stock`                       | Text (JSON String) | Serialized array of stock transfer/in/out JSON objects (optional)                                                                     |
| `adjustments`                 | Text (JSON String) | Serialized array of stock adjustment JSON objects (optional)                                                                          |
| `approvers`                   | Text (JSON String) | Serialized array of sequential approver JSON objects (optional)                                                                       |
| `remark`                      | Text               | Optional general remark                                                                                                               |
| `image`                       | File               | Product main image                                                                                                                    |
| `variantImage[<variantCode>]` | File(s)            | Variant-specific image files (e.g. `variantImage[MBP16-M3-SLV]`)                                                                      |

---

### 14.4 Detailed `multipart/form-data` Examples

#### 💎 Scenario 1: `PRODUCT_CREATE` (Complete Product + Variants + Initial Stock + Approvers + Images)

In Postman or frontend form submission, select `form-data` body:

| Key                           | Type               | Content                              |
| :---------------------------- | :----------------- | :----------------------------------- |
| **`requestType`**             | Text               | `PRODUCT_CREATE`                     |
| **`product`**                 | Text (JSON string) | See JSON below                       |
| **`variants`**                | Text (JSON string) | See JSON below                       |
| **`stock`**                   | Text (JSON string) | See JSON below                       |
| **`approvers`**               | Text (JSON string) | See JSON below                       |
| **`remark`**                  | Text               | `Q4 Product Launch - MacBook Pro M3` |
| **`image`**                   | File               | _(Select main product picture)_      |
| **`variantImage[MBP16-BLK]`** | File               | _(Select black variant picture)_     |
| **`variantImage[MBP16-SLV]`** | File               | _(Select silver variant picture)_    |

##### `product` JSON value:

```json
{
  "productCode": "MBP-16-M3",
  "productName": "Apple MacBook Pro 16\"",
  "description": "High performance workstation laptop",
  "categoryCode": "LAPTOPS",
  "brandCode": "APPLE",
  "supplierCode": "SUP-GLOBAL-01",
  "hasVariants": true,
  "unit": "unit",
  "productSku": "MBP-16-GEN",
  "productBarcode": "194253012345",
  "productCostPrice": 2200.0,
  "productSellingPrice": 2799.0,
  "minimumStock": 5,
  "maximumStock": 50
}
```

##### `variants` JSON value:

```json
[
  {
    "variantCode": "MBP16-BLK",
    "variantName": "MacBook Pro 16\" Space Black 512GB",
    "variantSku": "MBP16-BLK-512",
    "variantBarcode": "194253012346",
    "variantAttributes": {
      "color": "Space Black",
      "storage": "512GB",
      "ram": "18GB"
    },
    "variantCostPrice": 2200.0,
    "variantSellingPrice": 2799.0
  },
  {
    "variantCode": "MBP16-SLV",
    "variantName": "MacBook Pro 16\" Silver 1TB",
    "variantSku": "MBP16-SLV-1TB",
    "variantBarcode": "194253012347",
    "variantAttributes": { "color": "Silver", "storage": "1TB", "ram": "36GB" },
    "variantCostPrice": 2600.0,
    "variantSellingPrice": 3299.0
  }
]
```

##### `stock` JSON value:

```json
[
  {
    "productCode": "MBP-16-M3",
    "variantCode": "MBP16-BLK",
    "warehouseCode": "WH-PP-MAIN",
    "quantity": 25
  },
  {
    "productCode": "MBP-16-M3",
    "variantCode": "MBP16-SLV",
    "warehouseCode": "WH-PP-MAIN",
    "quantity": 15
  }
]
```

##### `approvers` JSON value:

```json
[
  {
    "userId": "b47c34d3-e7f0-4660-84cf-cb09ebbcbaae",
    "actionType": "CERTIFIER"
  },
  {
    "userId": "c58d45e4-f8a1-5771-95df-dc10fccdcbbf",
    "actionType": "APPROVER"
  }
]
```

---

#### 💎 Scenario 2: `STOCK_ADJUSTMENT` (Stock Loss / Damage / Correction)

| Key               | Type               | Content                                |
| :---------------- | :----------------- | :------------------------------------- |
| **`requestType`** | Text               | `STOCK_ADJUSTMENT`                     |
| **`adjustments`** | Text (JSON string) | See JSON below                         |
| **`approvers`**   | Text (JSON string) | See JSON below                         |
| **`remark`**      | Text               | `Stock recount discrepancy adjustment` |

##### `adjustments` JSON value:

```json
[
  {
    "productCode": "MBP-16-M3",
    "variantCode": "MBP16-BLK",
    "warehouseCode": "WH-PP-MAIN",
    "adjustmentType": "DECREASE",
    "quantity": 1,
    "reason": "Damaged screen during warehouse inspection"
  }
]
```

_(AdjustmentType is either `"INCREASE"` or `"DECREASE"`)_

##### `approvers` JSON value:

```json
[
  {
    "userId": "c58d45e4-f8a1-5771-95df-dc10fccdcbbf",
    "actionType": "APPROVER"
  }
]
```

---

#### 💎 Scenario 3: `STOCK_TRANSFER` (Warehouse-to-Warehouse Transfer)

| Key               | Type               | Content          |
| :---------------- | :----------------- | :--------------- |
| **`requestType`** | Text               | `STOCK_TRANSFER` |
| **`stock`**       | Text (JSON string) | See JSON below   |
| **`approvers`**   | Text (JSON string) | See JSON below   |

##### `stock` JSON value:

```json
[
  {
    "productCode": "MBP-16-M3",
    "variantCode": "MBP16-BLK",
    "fromWarehouseCode": "WH-PP-MAIN",
    "toWarehouseCode": "WH-BRANCH-2",
    "quantity": 5,
    "reason": "Replenishment for Branch 2 store inventory"
  }
]
```

---

### 14.5 Commit / Approve / Reject Request

- **Endpoint**: `POST /requests/:id/commit`
- **Access**: Authenticated (User must be assigned to the current approval step)
- **Param**: `:id` (Request UUID)
- **Content-Type**: `application/json`

#### Request Body

```json
{
  "action": "APPROVE",
  "remark": "Goods physically inspected and quantities verified."
}
```

#### Field Specifications

| Field    | Type     | Required | Values / Description      |
| :------- | :------- | :------- | :------------------------ |
| `action` | `string` | Yes      | `"APPROVE"` or `"REJECT"` |
| `remark` | `string` | No       | Reason or comments        |

---

## 15. Mails & Notifications API (`/mails`)

| Method  | Endpoint              | Description                                                   |
| :------ | :-------------------- | :------------------------------------------------------------ |
| `GET`   | `/mails`              | List all system emails / notifications for the logged-in user |
| `GET`   | `/mails/unread-count` | Get total unread count: `{"unreadCount": 3}`                  |
| `PATCH` | `/mails/:id/read`     | Mark an email / notification as read                          |

---

## 16. Direct Stock & Adjustments API

### 16.1 Stock (`/stock`)

- `GET /stock`: Query stock list
- `GET /stock/:id`: Query single stock record
- `POST /stock`: Create stock record
- `PATCH /stock/:id`: Update stock record
- `DELETE /stock/:id`: Delete stock record

### 16.2 Stock Adjustments (`/stock-adjustments`)

- `GET /stock-adjustments`: List direct adjustments
- `GET /stock-adjustments/:id`: Get adjustment record
- `POST /stock-adjustments`: Create adjustment
- `PATCH /stock-adjustments/:id`: Update adjustment
- `DELETE /stock-adjustments/:id`: Delete adjustment
