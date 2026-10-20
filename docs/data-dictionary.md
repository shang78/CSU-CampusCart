# CampusCart Data Dictionary

## Campus

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `campus_id` | SERIAL | PK | Unique campus identifier. |
| `name` | VARCHAR(120) | NOT NULL, UNIQUE | Campus name. |
| `email_domain` | VARCHAR(100) | NOT NULL, UNIQUE | Allowed school e-mail domain, e.g., school.edu.ph. |

## Users

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | SERIAL | PK | Unique user identifier. |
| `campus_id` | INTEGER | FK -> campus, NOT NULL | Campus the user belongs to. |
| `school_email` | VARCHAR(255) | NOT NULL, UNIQUE | Login e-mail; domain must equal `campus.email_domain`. |
| `password_hash` | VARCHAR(255) | NOT NULL | bcrypt/argon2 hash; never returned by the API. |
| `full_name` | VARCHAR(120) | NOT NULL | Display name. |
| `role` | VARCHAR(10) | NOT NULL, DEFAULT 'buyer', CHECK IN (buyer, seller, admin) | Authorization role. |
| `student_no` | VARCHAR(30) | NOT NULL, UNIQUE | School ID number used for verification (personal data). |
| `is_verified` | BOOLEAN | NOT NULL, DEFAULT false | True after e-mail verification; required login. |

## Store

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `store_id` | SERIAL | PK | Unique store identifier. |
| `campus_id` | INTEGER | FK → campus, NOT NULL | Campus where the store operates. |
| `owner_id` | INTEGER | FK → users, NOT NULL, UNIQUE | Seller who owns the store. |
| `display_name` | VARCHAR(80) | NOT NULL | Public store name. |
| `rating_avg` | NUMERIC(3,2) | NOT NULL, DEFAULT 0, CHECK 0–5 | Cached average of its product reviews. |

## Category

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `category_id` | SERIAL | PK | Unique category identifier. |
| `parent_id` | INTEGER | FK → category (self), NULL | Parent category; NULL for top level. |
| `name` | VARCHAR(80) | NOT NULL, UNIQUE (parent_id, name) | Category name. |

## Product

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `product_id` | SERIAL | PK | Unique product identifier. |
| `store_id` | INTEGER | FK → store, NOT NULL | Selling store. |
| `category_id` | INTEGER | FK → category, NOT NULL | Product category. |
| `title` | VARCHAR(150) | NOT NULL | Listing title. |
| `description` | TEXT | NULL | Details, condition, etc. |
| `image_url` | VARCHAR(500) | NULL | Link to the product image. |
| `price` | NUMERIC(10,2) | NOT NULL, CHECK >= 0 | Current unit price (PHP). |
| `stock_qty` | INTEGER | NOT NULL, DEFAULT 0, CHECK >= 0 | Units available. |
| `status` | VARCHAR(12) | NOT NULL, DEFAULT 'active', CHECK IN (active, sold_out, hidden) | Listing visibility. |

## Cart

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `cart_id` | SERIAL | PK | Unique cart identifier. |
| `user_id` | INTEGER | FK -> users, NOT NULL, UNIQUE | Cart owner. |
| `updated_at` | TIMESTAMPTZ | NOT NULL, DEFAULT now() | Last modification time. |

## Cart_item

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `cart_item_id` | SERIAL | PK | Unique line identifier. |
| `cart_id` | INTEGER | FK → cart, NOT NULL, ON DELETE CASCADE | Parent cart. |
| `product_id` | INTEGER | FK → product, NOT NULL | Chosen product. |
| `quantity` | INTEGER | NOT NULL, CHECK > 0; UNIQUE (cart_id, product_id) | Number of units; the same product appears once per cart. |

## Orders

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `order_id` | SERIAL | PK | Unique order identifier. |
| `buyer_id` | INTEGER | FK -> users, NOT NULL | Buyer who placed the order. |
| `store_id` | INTEGER | FK -> store, NOT NULL | Store that receives the order. |
| `total_amount` | NUMERIC(10,2) | NOT NULL, CHECK >= 0 | Sum of its order items. |
| `status` | VARCHAR(12) | NOT NULL, DEFAULT 'pending', CHECK IN (pending, confirmed, ready, completed, cancelled) | Order life-cycle state. |
| `placed_at` | TIMESTAMPTZ | NOT NULL, DEFAULT now() | Time of checkout. |

## Order_item

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `order_item_id` | SERIAL | PK | Unique line identifier. |
| `order_id` | INTEGER | FK → orders, NOT NULL, ON DELETE CASCADE | Parent order. |
| `product_id` | INTEGER | FK → product, NOT NULL | Purchased product. |
| `quantity` | INTEGER | NOT NULL, CHECK > 0; UNIQUE (order_id, product_id) | Units bought. |
| `unit_price` | NUMERIC(10,2) | NOT NULL, CHECK >= 0 | Price snapshot at purchase time. |

## Payment

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `payment_id` | SERIAL | PK | Unique payment identifier. |
| `order_id` | INTEGER | FK → orders, NOT NULL, UNIQUE | Order being paid. |
| `amount` | NUMERIC(10,2) | NOT NULL, CHECK >= 0 | Amount due (equals order total). |
| `payment_method` | VARCHAR(20) | NOT NULL, CHECK IN (cash_on_pickup, ewallet) | How the buyer pays. |
| `status` | VARCHAR(12) | NOT NULL, DEFAULT 'pending', CHECK IN (pending, collected, failed) | Payment state. |
| `collected_at` | TIMESTAMPTZ | NULL | When the money was received. |
| `confirmed_by` | INTEGER | FK → users, NULL | Seller/admin who confirmed the payment. |

## Delivery

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `delivery_id` | SERIAL | PK | Unique delivery identifier. |
| `order_id` | INTEGER | FK → orders, NOT NULL, UNIQUE | Order being handed over. |
| `pickup_location` | VARCHAR(150) | NOT NULL | Meeting point on campus. |
| `scheduled_at` | TIMESTAMPTZ | NOT NULL | Agreed pickup time. |
| `seller_confirmed_at` | TIMESTAMPTZ | NULL | Set when the seller confirms the hand-over. |
| `buyer_confirmed_at` | TIMESTAMPTZ | NULL | Set when the buyer confirms receipt. |
| `status` | VARCHAR(12) | NOT NULL, DEFAULT 'scheduled', CHECK IN (scheduled, rescheduled, completed, cancelled) | Becomes 'completed' once both confirmations exist. |

## Review

| Column | Type | Key / Constraints | Description |
| :--- | :--- | :--- | :--- |
| `review_id` | SERIAL | PK | Unique review identifier. |
| `order_item_id` | INTEGER | FK -> order_item, NOT NULL, UNIQUE | Purchased line being reviewed. |
| `reviewer_id` | INTEGER | FK -> users, NOT NULL | Author (must be the order’s buyer). |
| `rating` | SMALLINT | NOT NULL, CHECK 1-5 | Star rating. |
| `comment` | TEXT | NULL | Optional written feedback. |

