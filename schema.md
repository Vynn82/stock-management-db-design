

# Stock Management System — Final Database Schema

## 1. Authentication

### `users`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | User ID |
| `username` | VARCHAR(100) | UNIQUE, NOT NULL | Login username |
| `email` | VARCHAR(255) | UNIQUE | User email |
| `password_hash` | VARCHAR(255) | NOT NULL | Hashed password |
| `status` | VARCHAR(20) | NOT NULL | User status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

### `user_profiles`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Profile ID |
| `user_id` | UUID | FK, UNIQUE, NOT NULL | FK → users.id |
| `first_name` | VARCHAR(100) | NOT NULL | First name |
| `last_name` | VARCHAR(100) | NOT NULL | Last name |
| `phone` | VARCHAR(30) | NULL | Phone number |
| `avatar_url` | VARCHAR(500) | NULL | Profile image |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

### `sessions`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Session ID |
| `user_id` | UUID | FK, NOT NULL | FK → users.id |
| `refresh_token_hash` | VARCHAR(255) | NOT NULL | Hashed refresh token |
| `expires_at` | TIMESTAMP | NOT NULL | Expiration time |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `revoked_at` | TIMESTAMP | NULL | Revoked time |


---

# 2. RBAC

## `roles`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Role ID |
| `name` | VARCHAR(100) | UNIQUE, NOT NULL | Role name |
| `description` | VARCHAR(255) | NULL | Role description |
| `status` | VARCHAR(20) | NOT NULL | Role status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `permissions`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Permission ID |
| `key` | VARCHAR(100) | UNIQUE, NOT NULL | Permission key |
| `name` | VARCHAR(100) | NOT NULL | Permission name |
| `description` | VARCHAR(255) | NULL | Permission description |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

### Example permission keys

- `product:create`
- `product:read`
- `product:update`
- `product:delete`
- `stock:read`
- `stock:adjust`
- `stock:transfer`
- `purchase:create`
- `purchase:approve`
- `sale:create`
- `sale:read`

## `user_roles`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `user_id` | UUID | PK, FK | FK → users.id |
| `role_id` | UUID | PK, FK | FK → roles.id |

## `role_permissions`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `role_id` | UUID | PK, FK | FK → roles.id |
| `permission_id` | UUID | PK, FK | FK → permissions.id |

## `menus`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Menu ID |
| `key` | VARCHAR(100) | UNIQUE, NOT NULL | Menu key |
| `name` | VARCHAR(100) | NOT NULL | Menu name |
| `path` | VARCHAR(255) | NULL | Frontend route |
| `parent_id` | UUID | FK, NULL | Parent menu |
| `display_order` | INT | NOT NULL | Menu ordering |
| `icon` | VARCHAR(100) | NULL | Menu icon |
| `status` | VARCHAR(20) | NOT NULL | Menu status |

### Menu data

| Key | Name | Path | Display Order |
|---|---|---|---:|
| `dashboard` | Dashboard | `/dashboard` | 1 |
| `products` | Products | `/products` | 2 |
| `stock` | Stock | `/stock` | 3 |
| `purchases` | Purchases | `/purchases` | 4 |
| `sales` | Sales | `/sales` | 5 |
| `reports` | Reports | `/reports` | 6 |
| `users` | Users | `/users` | 7 |

## `role_menus`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `role_id` | UUID | PK, FK | FK → roles.id |
| `menu_id` | UUID | PK, FK | FK → menus.id |


---

# 3. Product Master

## `categories`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Category ID |
| `name` | VARCHAR(100) | UNIQUE, NOT NULL | Category name |
| `description` | VARCHAR(255) | NULL | Category description |
| `status` | VARCHAR(20) | NOT NULL | Category status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `brands`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Brand ID |
| `name` | VARCHAR(100) | UNIQUE, NOT NULL | Brand name |
| `description` | VARCHAR(255) | NULL | Brand description |
| `status` | VARCHAR(20) | NOT NULL | Brand status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `units`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Unit ID |
| `name` | VARCHAR(50) | UNIQUE, NOT NULL | Unit name |
| `symbol` | VARCHAR(20) | UNIQUE, NOT NULL | Unit symbol |
| `status` | VARCHAR(20) | NOT NULL | Unit status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

### Example units

| Name | Symbol |
|---|---|
| Piece | `pcs` |
| Kilogram | `kg` |
| Gram | `g` |
| Liter | `L` |
| Meter | `m` |
| Box | `box` |
| Carton | `ctn` |

## `products`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Product ID |
| `sku` | VARCHAR(100) | UNIQUE, NOT NULL | Stock keeping unit |
| `name` | VARCHAR(255) | NOT NULL | Product name |
| `category_id` | UUID | FK, NOT NULL | FK → categories.id |
| `brand_id` | UUID | FK, NULL | FK → brands.id |
| `unit_id` | UUID | FK, NOT NULL | FK → units.id |
| `description` | TEXT | NULL | Product description |
| `cost_price` | DECIMAL(15,2) | NOT NULL | Product cost |
| `sale_price` | DECIMAL(15,2) | NOT NULL | Selling price |
| `status` | VARCHAR(20) | NOT NULL | Product status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |


---

# 4. Warehouse & Stock

## `warehouses`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Warehouse ID |
| `code` | VARCHAR(50) | UNIQUE, NOT NULL | Warehouse code |
| `name` | VARCHAR(100) | NOT NULL | Warehouse name |
| `location` | VARCHAR(255) | NULL | Warehouse location |
| `status` | VARCHAR(20) | NOT NULL | Warehouse status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `stock_balances`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Stock balance ID |
| `warehouse_id` | UUID | FK, NOT NULL | FK → warehouses.id |
| `product_id` | UUID | FK, NOT NULL | FK → products.id |
| `quantity` | DECIMAL(15,3) | NOT NULL | Current stock quantity |
| `updated_at` | TIMESTAMP | NOT NULL | Last updated time |

### Constraint

`UNIQUE (warehouse_id, product_id)`

## `stock_movements`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Movement ID |
| `warehouse_id` | UUID | FK, NOT NULL | FK → warehouses.id |
| `product_id` | UUID | FK, NOT NULL | FK → products.id |
| `type` | VARCHAR(30) | NOT NULL | Movement type |
| `quantity` | DECIMAL(15,3) | NOT NULL | Movement quantity |
| `reference_type` | VARCHAR(50) | NULL | Related document type |
| `reference_id` | UUID | NULL | Related document ID |
| `created_by` | UUID | FK, NOT NULL | FK → users.id |
| `created_at` | TIMESTAMP | NOT NULL | Created time |

### Movement Types

- `PURCHASE`
- `SALE`
- `ADJUSTMENT`
- `TRANSFER_IN`
- `TRANSFER_OUT`


---

# 5. Request & Approval

## `requests`

Main business request table.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Request ID |
| `request_no` | VARCHAR(50) | UNIQUE, NOT NULL | Request number |
| `request_type` | VARCHAR(50) | NOT NULL | Type of request |
| `requester_id` | UUID | FK, NOT NULL | User who creates request |
| `status` | VARCHAR(20) | NOT NULL | Request status |
| `description` | TEXT | NULL | Request description |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

### Request Status

- `PENDING`
- `APPROVED`
- `REJECTED`

### Request Types

- `PRODUCT_CREATE`
- `PRODUCT_UPDATE`
- `PRODUCT_IMPORT`
- `STOCK_ADJUSTMENT`
- `STOCK_TRANSFER`

## `request_items`

Stores the actual requested data/change.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Request item ID |
| `request_id` | UUID | FK, NOT NULL | FK → requests.id |
| `product_id` | UUID | FK, NULL | Existing product, NULL for product creation |
| `warehouse_id` | UUID | FK, NULL | Related warehouse |
| `quantity` | DECIMAL(15,3) | NULL | Requested quantity |
| `action_type` | VARCHAR(30) | NOT NULL | Requested action |
| `reason` | TEXT | NULL | Reason for request |

## `request_attachments`

Stores files uploaded to a request so certifiers and approvers can review them.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Attachment ID |
| `request_id` | UUID | FK, NOT NULL | FK → requests.id |
| `file_name` | VARCHAR(255) | NOT NULL | Original file name |
| `file_path` | VARCHAR(500) | NOT NULL | Stored file location |
| `file_type` | VARCHAR(100) | NOT NULL | MIME type |
| `file_size` | BIGINT | NOT NULL | File size in bytes |
| `uploaded_by` | UUID | FK, NOT NULL | FK → users.id |
| `created_at` | TIMESTAMP | NOT NULL | Upload time |

## `request_approvers`

Stores the approval chain.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `request_id` | UUID | PK, FK | FK → requests.id |
| `user_id` | UUID | PK, FK | FK → users.id |
| `action_type` | VARCHAR(20) | NOT NULL | CERTIFIER / APPROVER |
| `status` | VARCHAR(20) | NOT NULL | PENDING / APPROVED / REJECTED |
| `comment` | TEXT | NULL | Approval/rejection comment |
| `acted_at` | TIMESTAMP | NULL | Action time |

### Example

| REQUEST_ID | USER_ID | ACTION_TYPE | STATUS |
|---|---|---|---|
| 1001 | U001 | CERTIFIER | APPROVED |
| 1001 | U002 | APPROVER | PENDING |

### Approval Rules

1. At least one `CERTIFIER`.
2. At least one `APPROVER`.
3. `APPROVER` must be the final role.
4. Multiple `APPROVER`s are allowed.
5. The same user cannot appear twice.
6. Requester cannot appear in approval steps.
7. Certifier/Approver must be valid users.


---

# 6. Suppliers & Purchasing

## `suppliers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Supplier ID |
| `code` | VARCHAR(50) | UNIQUE, NOT NULL | Supplier code |
| `name` | VARCHAR(255) | NOT NULL | Supplier name |
| `phone` | VARCHAR(30) | NULL | Phone number |
| `email` | VARCHAR(255) | NULL | Email |
| `address` | VARCHAR(500) | NULL | Address |
| `status` | VARCHAR(20) | NOT NULL | Supplier status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `purchase_orders`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Purchase order ID |
| `order_no` | VARCHAR(50) | UNIQUE, NOT NULL | Purchase order number |
| `supplier_id` | UUID | FK, NOT NULL | FK → suppliers.id |
| `warehouse_id` | UUID | FK, NOT NULL | FK → warehouses.id |
| `status` | VARCHAR(20) | NOT NULL | Purchase status |
| `total_amount` | DECIMAL(15,2) | NOT NULL | Total amount |
| `requested_by` | UUID | FK, NOT NULL | User who created order |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `purchase_order_items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Item ID |
| `purchase_order_id` | UUID | FK, NOT NULL | FK → purchase_orders.id |
| `product_id` | UUID | FK, NOT NULL | FK → products.id |
| `quantity` | DECIMAL(15,3) | NOT NULL | Purchase quantity |
| `unit_cost` | DECIMAL(15,2) | NOT NULL | Cost per unit |
| `total_cost` | DECIMAL(15,2) | NOT NULL | Total cost |


---

# 7. Customers & Sales

## `customers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Customer ID |
| `code` | VARCHAR(50) | UNIQUE, NOT NULL | Customer code |
| `name` | VARCHAR(255) | NOT NULL | Customer name |
| `phone` | VARCHAR(30) | NULL | Phone number |
| `email` | VARCHAR(255) | NULL | Email |
| `address` | VARCHAR(500) | NULL | Address |
| `status` | VARCHAR(20) | NOT NULL | Customer status |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `sales_orders`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Sales order ID |
| `order_no` | VARCHAR(50) | UNIQUE, NOT NULL | Sales order number |
| `customer_id` | UUID | FK, NULL | FK → customers.id |
| `warehouse_id` | UUID | FK, NOT NULL | FK → warehouses.id |
| `status` | VARCHAR(20) | NOT NULL | Sales status |
| `total_amount` | DECIMAL(15,2) | NOT NULL | Total amount |
| `created_by` | UUID | FK, NOT NULL | User who created sale |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `sales_order_items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Item ID |
| `sales_order_id` | UUID | FK, NOT NULL | FK → sales_orders.id |
| `product_id` | UUID | FK, NOT NULL | FK → products.id |
| `quantity` | DECIMAL(15,3) | NOT NULL | Sale quantity |
| `unit_price` | DECIMAL(15,2) | NOT NULL | Selling price |
| `total_price` | DECIMAL(15,2) | NOT NULL | Total price |


---

# 8. Stock Transfer

## `stock_transfers`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Transfer ID |
| `transfer_no` | VARCHAR(50) | UNIQUE, NOT NULL | Transfer number |
| `from_warehouse_id` | UUID | FK, NOT NULL | Source warehouse |
| `to_warehouse_id` | UUID | FK, NOT NULL | Destination warehouse |
| `status` | VARCHAR(20) | NOT NULL | Transfer status |
| `requested_by` | UUID | FK, NOT NULL | User who requested transfer |
| `created_at` | TIMESTAMP | NOT NULL | Created time |
| `updated_at` | TIMESTAMP | NOT NULL | Updated time |

## `stock_transfer_items`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Transfer item ID |
| `transfer_id` | UUID | FK, NOT NULL | FK → stock_transfers.id |
| `product_id` | UUID | FK, NOT NULL | FK → products.id |
| `quantity` | DECIMAL(15,3) | NOT NULL | Transfer quantity |


---

# 9. Audit

## `audit_logs`

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | UUID | PK | Audit log ID |
| `user_id` | UUID | FK, NOT NULL | User who performed action |
| `action` | VARCHAR(50) | NOT NULL | Action performed |
| `entity_type` | VARCHAR(100) | NOT NULL | Entity/table name |
| `entity_id` | UUID | NOT NULL | Affected record ID |
| `old_value` | JSONB | NULL | Previous value |
| `new_value` | JSONB | NULL | New value |
| `ip_address` | VARCHAR(45) | NULL | User IP |
| `created_at` | TIMESTAMP | NOT NULL | Created time |


---

# 10. Complete Table List

## Authentication

1. `users`
2. `user_profiles`
3. `sessions`

## RBAC

4. `roles`
5. `permissions`
6. `user_roles`
7. `role_permissions`
8. `menus`
9. `role_menus`

## Product Master

10. `categories`
11. `brands`
12. `units`
13. `products`

## Warehouse & Stock

14. `warehouses`
15. `stock_balances`
16. `stock_movements`

## Request & Approval

17. `requests`
18. `request_items`
19. `request_approvers`

## Purchasing

20. `suppliers`
21. `purchase_orders`
22. `purchase_order_items`

## Sales

23. `customers`
24. `sales_orders`
25. `sales_order_items`

## Stock Transfer

26. `stock_transfers`
27. `stock_transfer_items`

## Audit

28. `audit_logs`


---

# 11. Main Relationships

users
  ├── user_profiles
  ├── sessions
  ├── user_roles
  │      └── roles
  │             ├── role_permissions
  │             │      └── permissions
  │             └── role_menus
  │                    └── menus
  ├── requests
  │      └── request_approvers
  └── audit_logs

categories
  └── products

brands
  └── products

units
  └── products

products
  ├── request_items
  ├── stock_balances
  ├── stock_movements
  ├── purchase_order_items
  ├── sales_order_items
  └── stock_transfer_items

warehouses
  ├── stock_balances
  ├── stock_movements
  ├── request_items
  ├── purchase_orders
  ├── sales_orders
  └── stock_transfers

suppliers
  └── purchase_orders
       └── purchase_order_items

customers
  └── sales_orders
       └── sales_order_items


---

# 12. Request → Approval → Execution Flow

                         REQUEST
                            │
                            ▼
                    PENDING / WAITING
                            │
                   ┌────────┴────────┐
                   │                 │
                REJECTED          APPROVED
                   │                 │
                   ▼                 ▼
              KEEP HISTORY      EXECUTE REQUEST
                                     │
                             ┌───────┴───────┐
                             ▼               ▼
                          PRODUCT          STOCK
                                             │
                                             ▼
                                      STOCK_MOVEMENTS


---

# 13. Approval Flow

REQUEST
   │
   ▼
CERTIFIER
   │
   ├── REJECTED
   │       │
   │       ▼
   │   REQUEST = REJECTED
   │
   └── APPROVED
           │
           ▼
        APPROVER
           │
           ├── REJECTED
           │       │
           │       ▼
           │   REQUEST = REJECTED
           │
           └── APPROVED
                   │
                   ▼
             REQUEST = APPROVED
                   │
                   ▼
             EXECUTE REQUEST
                   │
             ┌─────┴─────┐
             ▼           ▼
          PRODUCT       STOCK
                           │
                           ▼
                    STOCK_MOVEMENTS


---

# 14. Important Business Rule

RBAC and Approval are two different things.

RBAC:

    "Can this user perform this action?"

Approval:

    "Has this specific request been authorized?"

The backend checks RBAC first.

Then the request goes through the approval workflow.

Only after the final APPROVER approves the request does the backend execute the actual business change.


---

# 15. Example — Create Product Request

User U001 wants to create a new product.

REQUEST:

| Column | Value |
|---|---|
| request_no | REQ-000001 |
| request_type | PRODUCT_CREATE |
| requester_id | U001 |
| status | PENDING |

REQUEST_ITEM:

| Column | Value |
|---|---|
| request_id | REQ-000001 |
| product_id | NULL |
| warehouse_id | NULL |
| quantity | NULL |
| action_type | CREATE |
| reason | New product |

REQUEST_APPROVERS:

| Request | User | Action | Status |
|---|---|---|---|
| REQ-000001 | U002 | CERTIFIER | APPROVED |
| REQ-000001 | U003 | APPROVER | PENDING |

After U003 approves:

REQUEST:

    status = APPROVED

Then backend executes:

    INSERT INTO products ...

If stock is also part of the request:

    INSERT INTO stock_balances ...

And if stock is affected:

    INSERT INTO stock_movements ...


---

# 16. Example — Update Product Request

Existing product:

    Product ID = P001

User U001 requests:

    Sale Price = 15.00
    Name = New Product Name

REQUEST:

| Column | Value |
|---|---|
| request_type | PRODUCT_UPDATE |
| requester_id | U001 |
| status | PENDING |

REQUEST_ITEM:

| Column | Value |
|---|---|
| request_id | REQ-000002 |
| product_id | P001 |
| action_type | UPDATE |
| reason | Update product information |

After final approval:

    UPDATE products
    SET name = ...,
        sale_price = ...
    WHERE id = P001;

The update happens only after the approval process finishes.


---

# 17. Example — Stock Adjustment Request

User requests:

    Product = P001
    Warehouse = W001
    Quantity = +20

REQUEST:

    request_type = STOCK_ADJUSTMENT
    status = PENDING

REQUEST_ITEM:

    product_id = P001
    warehouse_id = W001
    quantity = 20
    action_type = ADJUST

After final approval:

    UPDATE stock_balances
    SET quantity = quantity + 20
    WHERE warehouse_id = W001
      AND product_id = P001;

Then create:

    stock_movements

with:

    type = ADJUSTMENT
    quantity = 20
    reference_type = REQUEST
    reference_id = request_id


---

# 18. Important Execution Rule

The final approval and business update should be treated as one transaction.

Example:

    BEGIN TRANSACTION

        1. Approve request
        2. Execute request
        3. Update product/stock
        4. Create stock movement
        5. Create audit log

    COMMIT

If anything fails:

    ROLLBACK


---

# 19. Final Architecture

                         ┌───────────────┐
                         │     USER      │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │     RBAC      │
                         │               │
                         │ Roles         │
                         │ Permissions   │
                         │ Menus         │
                         └───────┬───────┘
                                 │
                                 ▼
                         ┌───────────────┐
                         │    REQUEST    │
                         └───────┬───────┘
                                 │
                                 ▼
                       ┌───────────────────┐
                       │ REQUEST_APPROVERS │
                       └─────────┬─────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                 REJECTED                  APPROVED
                    │                         │
                    ▼                         ▼
                 HISTORY                 EXECUTION
                                              │
                               ┌──────────────┴──────────────┐
                               │                             │
                               ▼                             ▼
                           PRODUCTS                       STOCK
                                                               │
                                                               ▼
                                                        STOCK_MOVEMENTS
