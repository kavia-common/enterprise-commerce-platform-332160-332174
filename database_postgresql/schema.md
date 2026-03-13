# Enterprise Commerce Platform - PostgreSQL Schema

## Connection Info
Canonical connection string is in `db_connection.txt`:
```
psql postgresql://appuser:dbuser123@localhost:5000/myapp
```

## Applied Status
✅ **Schema fully applied** — all tables, indexes, triggers, constraints, and seed data are live in the `myapp` database on port `5000`.

## Extensions
- `uuid-ossp` — used for `uuid_generate_v4()` primary key defaults

## Trigger Function
`update_updated_at_column()` — Shared PL/pgSQL function that sets `NEW.updated_at = NOW()` before UPDATE. Applied to `users`, `products`, and `orders`.

## Tables

### users
| Column        | Type                     | Constraints                                              |
|---------------|--------------------------|----------------------------------------------------------|
| id            | UUID                     | PRIMARY KEY, DEFAULT uuid_generate_v4()                  |
| email         | VARCHAR(255)             | NOT NULL, UNIQUE                                         |
| password_hash | VARCHAR(255)             | NOT NULL                                                 |
| first_name    | VARCHAR(100)             | NOT NULL                                                 |
| last_name     | VARCHAR(100)             | NOT NULL                                                 |
| role          | VARCHAR(20)              | NOT NULL, DEFAULT 'customer', CHECK IN ('admin','customer') |
| is_active     | BOOLEAN                  | NOT NULL, DEFAULT TRUE                                   |
| created_at    | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                                  |
| updated_at    | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                                  |

**Indexes:** `idx_users_email`, `idx_users_role`, `idx_users_is_active`, `idx_users_created_at`
**Trigger:** `update_users_updated_at` — auto-sets `updated_at` on UPDATE

### products
| Column         | Type                     | Constraints                                  |
|----------------|--------------------------|----------------------------------------------|
| id             | UUID                     | PRIMARY KEY, DEFAULT uuid_generate_v4()      |
| name           | VARCHAR(255)             | NOT NULL                                     |
| description    | TEXT                     | nullable                                     |
| price          | NUMERIC(10,2)            | NOT NULL, CHECK >= 0                         |
| stock_quantity | INTEGER                  | NOT NULL, DEFAULT 0, CHECK >= 0              |
| category       | VARCHAR(100)             | nullable                                     |
| sku            | VARCHAR(100)             | UNIQUE, NOT NULL                             |
| image_url      | VARCHAR(500)             | nullable                                     |
| is_active      | BOOLEAN                  | NOT NULL, DEFAULT TRUE                       |
| created_at     | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                      |
| updated_at     | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                      |

**Indexes:** `idx_products_category`, `idx_products_sku`, `idx_products_is_active`, `idx_products_price`, `idx_products_created_at`, `idx_products_name`
**Trigger:** `update_products_updated_at` — auto-sets `updated_at` on UPDATE

### orders
| Column           | Type                     | Constraints                                                        |
|------------------|--------------------------|-------------------------------------------------------------------|
| id               | UUID                     | PRIMARY KEY, DEFAULT uuid_generate_v4()                            |
| user_id          | UUID                     | NOT NULL, REFERENCES users(id) ON DELETE CASCADE                   |
| status           | VARCHAR(30)              | NOT NULL, DEFAULT 'pending', CHECK IN ('pending','confirmed','processing','shipped','delivered','cancelled') |
| total_amount     | NUMERIC(12,2)            | NOT NULL, DEFAULT 0, CHECK >= 0                                    |
| shipping_address | TEXT                     | nullable                                                           |
| billing_address  | TEXT                     | nullable                                                           |
| notes            | TEXT                     | nullable                                                           |
| created_at       | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                                            |
| updated_at       | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                                            |

**Indexes:** `idx_orders_user_id`, `idx_orders_status`, `idx_orders_created_at`, `idx_orders_total_amount`
**Trigger:** `update_orders_updated_at` — auto-sets `updated_at` on UPDATE

### order_items
| Column      | Type                     | Constraints                                          |
|-------------|--------------------------|------------------------------------------------------|
| id          | UUID                     | PRIMARY KEY, DEFAULT uuid_generate_v4()              |
| order_id    | UUID                     | NOT NULL, REFERENCES orders(id) ON DELETE CASCADE    |
| product_id  | UUID                     | NOT NULL, REFERENCES products(id) ON DELETE RESTRICT |
| quantity    | INTEGER                  | NOT NULL, CHECK > 0                                  |
| unit_price  | NUMERIC(10,2)            | NOT NULL, CHECK >= 0                                 |
| total_price | NUMERIC(12,2)            | NOT NULL, CHECK >= 0                                 |
| created_at  | TIMESTAMP WITH TIME ZONE | NOT NULL, DEFAULT NOW()                              |

**Indexes:** `idx_order_items_order_id`, `idx_order_items_product_id`

## Seed Data

### Admin User
- **Email:** admin@enterprise.com
- **Password:** Admin123! (bcrypt hashed)
- **Role:** admin
- **ID:** `1af1ab54-1c73-4219-b832-dd6695e8fe10`

### Sample Customer
- **Email:** customer@example.com
- **Password:** Admin123! (bcrypt hashed)
- **Role:** customer
- **ID:** `5969e127-5483-403b-a459-828341173ed7`

### Product Catalog (5 items)
| SKU          | Name                          | Price   | Stock | Category    |
|--------------|-------------------------------|---------|-------|-------------|
| ELEC-WBH-001| Wireless Bluetooth Headphones | 79.99   | 150   | Electronics |
| FURN-EOC-001| Ergonomic Office Chair        | 249.99  | 45    | Furniture   |
| KTCH-SSW-001| Stainless Steel Water Bottle  | 24.99   | 300   | Kitchen     |
| ELEC-UDS-002| USB-C Laptop Docking Station  | 129.99  | 80    | Electronics |
| CLTH-OCT-001| Organic Cotton T-Shirt        | 29.99   | 500   | Clothing    |

### Sample Order
- **Customer:** customer@example.com (`5969e127-5483-403b-a459-828341173ed7`)
- **Order ID:** `758f20ac-693f-4c9c-b08b-b48f77b0339f`
- **Status:** confirmed
- **Total:** $104.98
- **Items:** Wireless Bluetooth Headphones ($79.99) + Stainless Steel Water Bottle ($24.99)

## How to Re-apply Schema

If you need to recreate the schema from scratch, restore from the backup:
```bash
# From the database_postgresql directory:
bash restore_db.sh
```

Or connect using the canonical connection string from `db_connection.txt`:
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp
```

To create a fresh backup of the current state:
```bash
bash backup_db.sh
```
