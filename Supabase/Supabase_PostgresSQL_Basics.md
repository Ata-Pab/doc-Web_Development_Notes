### **Core Concepts**
1. **Databases and Tables:** A PostgreSQL database contains schemas, which organize tables storing data.
2. **Schemas:** Default is `public`, but you can create your own.
3. **Indexes:** Improve query performance by optimizing lookups.
4. **Constraints:** Ensure data integrity (e.g., `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`).
5. **Data Types:** Supports standard types (`INT`, `TEXT`, `BOOLEAN`), JSON (`JSONB`), and more.
6. **Transactions:** Ensure atomicity (ACID compliance).
7. **Joins:** Combine data from multiple tables (`INNER JOIN`, `LEFT JOIN`, etc.).
8. **Extensions:** PostgreSQL supports extensions like `pgcrypto` (encryption) and `postgis` (geospatial).

---

### **Most Used PostgreSQL/SQL Commands for Real-World Applications**
Here are essential SQL commands frequently used in **real-world applications** like **Supabase-based web and mobile apps**.

#### **1. Database and Table Management**
```sql
-- Create a new database
CREATE DATABASE my_database;

-- Create a new table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Drop a table
DROP TABLE users;
```

---

#### **2. Inserting Data**
```sql
-- Insert a single record
INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

-- Insert multiple records
INSERT INTO users (name, email) VALUES 
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com');
```

---

#### **3. Selecting Data**
```sql
-- Select all records
SELECT * FROM users;

-- Select specific columns
SELECT name, email FROM users;

-- Select with filtering
SELECT * FROM users WHERE email = 'john@example.com';

-- Select with ordering
SELECT * FROM users ORDER BY created_at DESC;
```

---

#### **4. Updating Data**
```sql
-- Update a user's email
UPDATE users SET email = 'john.doe@example.com' WHERE id = 1;
```

---

#### **5. Deleting Data**
```sql
-- Delete a single record
DELETE FROM users WHERE id = 1;

-- Delete all records (use cautiously)
DELETE FROM users;
```

---

#### **6. Indexing (For Performance Optimization)**
```sql
-- Create an index on the email column for faster lookups
CREATE INDEX idx_users_email ON users(email);
```

---

#### **7. Relationships (Foreign Keys)**
```sql
-- Create a posts table linked to users
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

---

#### **8. Aggregation and Grouping**
```sql
-- Count total users
SELECT COUNT(*) FROM users;

-- Group by and count
SELECT user_id, COUNT(*) FROM posts GROUP BY user_id;
```

---

#### **9. Joins (Fetching Related Data)**
```sql
-- Get all users with their posts
SELECT users.name, posts.content 
FROM users
JOIN posts ON users.id = posts.user_id;
```

---

#### **10. JSON and JSONB (Common in Supabase)**
```sql
-- Create a table with JSONB column
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    order_details JSONB NOT NULL
);

-- Insert JSON data
INSERT INTO orders (user_id, order_details) 
VALUES (1, '{"items": [{"name": "Pizza", "price": 10.99}, {"name": "Soda", "price": 2.50}]}');

-- Query JSON data
SELECT order_details->'items' FROM orders;
```

---

#### **11. Transactions (Ensuring Data Consistency)**
```sql
BEGIN;  -- Start transaction

UPDATE users SET email = 'newemail@example.com' WHERE id = 1;
INSERT INTO logs (message) VALUES ('User email updated');

COMMIT;  -- Commit changes
-- ROLLBACK;  -- Rollback in case of failure
```

---

#### **12. Supabase-Specific Features**
Since you're using **Supabase**, here are some additional commands you might use:

- **Enable Row-Level Security (RLS)**
```sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
```

- **Create a Policy to Allow User-Specific Data Access**
```sql
CREATE POLICY "Users can see their own data"
ON users
FOR SELECT
USING (auth.uid() = id);
```

- **Using Supabase Auth for User Data**
```sql
SELECT * FROM auth.users;
```

#### **Default TABLESPACE**
In PostgreSQL, a tablespace is a storage location where database objects such as tables and indexes are physically stored. By default, every database object is stored in the default tablespace, which is usually called <b>pg_default</b>.

- Every PostgreSQL database has a default tablespace, which is typically pg_default.
- It stores all tables and indexes unless another tablespace is specified.
- The physical storage location of pg_default is inside the PostgreSQL data directory.
 
#### **What Does `TABLESPACE pg_default` Mean?**
```sql
TABLESPACE pg_default;
```
- This **explicitly specifies** that the table should be stored in **`pg_default`**, the default tablespace.
- However, this is **redundant** because if no tablespace is specified, PostgreSQL **automatically** stores the table in `pg_default`.

---

#### **What is the Default Tablespace?**
- Every PostgreSQL database has a **default tablespace**, which is typically **`pg_default`**.
- It stores **all tables and indexes** unless another tablespace is specified.
- The **physical storage location** of `pg_default` is inside the **PostgreSQL data directory**.

---

#### **When Should You Use a Custom Tablespace?**
##### **Use Custom Tablespaces If:**
- You want to store **large tables on a separate disk** (for better performance).  
- You need **different storage configurations** (e.g., fast SSDs for indexes).  
- You want to **distribute database storage across multiple locations**.

##### **Stick to `pg_default` If:**
- You're running a standard application (like a Supabase project).  
- You don’t need special storage management.  
- Your database is small to medium-sized.

---

#### **Example: Creating a Custom Tablespace**
If you have a separate storage disk (e.g., `/mnt/fast_disk`), you can create a new tablespace:

```sql
CREATE TABLESPACE fast_storage LOCATION '/mnt/fast_disk';
```
Then, you can create a table using that tablespace:

```sql
CREATE TABLE large_table (
  id SERIAL PRIMARY KEY,
  data TEXT
) TABLESPACE fast_storage;
```
Now, PostgreSQL will store `large_table` on the `/mnt/fast_disk` location instead of `pg_default`.

---

#### **Should You Worry About Tablespaces in Supabase?**
**No.**  
- Supabase handles database storage internally.
- You don’t have access to custom tablespaces in Supabase.
- **All tables in Supabase are stored in `pg_default`.**

So, the `TABLESPACE pg_default;` in your SQL is **not necessary**—PostgreSQL will automatically use it.

---

#### **Conclusion**
- **`pg_default`** is the **default tablespace** where PostgreSQL stores tables and indexes.
- **You don’t need to specify it**—PostgreSQL uses it automatically.
- **Custom tablespaces** are only useful if you have advanced storage needs (not needed in Supabase).

```sql
create table public.users (
  id uuid not null,
  created_at timestamp with time zone not null default now(),
  email text null,
  name text null,
  description text null,
  avatar character varying null,
  updated_at timestamp with time zone null,
  provider text null,
  constraint users_pkey primary key (id),
  constraint users_id_fkey foreign KEY (id) references auth.users (id) on delete CASCADE
) TABLESPACE pg_default;
```

---

### **PostgREST & RPC Calls in Supabase – API-like Interactions**
Supabase **automatically** provides a RESTful API for your PostgreSQL database using **PostgREST**. This means that **every table and function in your database can be accessed as an API endpoint** without needing to write a backend manually.

---

## **What is PostgREST?**
**PostgREST** is a tool that turns your **PostgreSQL database into a RESTful API automatically**. Supabase uses this to generate API endpoints for every table in your database.

**Why is it useful?**
- You don’t need to write an API manually.
- You can perform CRUD operations (Create, Read, Update, Delete) over HTTP.
- Secure access via **Row-Level Security (RLS)**.
- Supports authentication with **Supabase Auth**.

---

## **How Does PostgREST Work?**
### **Basic API Calls**
Let’s assume you have a **users** table in your Supabase database:

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL
);
```

Once you create this table, Supabase automatically generates API endpoints!

### **Example API Calls Using PostgREST**
#### **🔹 GET (Fetch All Users)**
```bash
curl "https://your-project-ref.supabase.co/rest/v1/users" \
  -H "apikey: your-anon-key"
```
- Returns all users in JSON format.

#### **GET (Fetch a Specific User)**
```bash
curl "https://your-project-ref.supabase.co/rest/v1/users?id=eq.1" \
  -H "apikey: your-anon-key"
```
- Fetches user with `id = 1`.

#### **POST (Insert a New User)**
```bash
curl -X POST "https://your-project-ref.supabase.co/rest/v1/users" \
  -H "apikey: your-anon-key" \
  -H "Content-Type: application/json" \
  -d '{"name": "John Doe", "email": "john@example.com"}'
```
- Inserts a new user.

#### **PATCH (Update a User’s Name)**
```bash
curl -X PATCH "https://your-project-ref.supabase.co/rest/v1/users?id=eq.1" \
  -H "apikey: your-anon-key" \
  -H "Content-Type: application/json" \
  -d '{"name": "Johnny"}'
```
- Updates `name` where `id = 1`.

#### **DELETE (Remove a User)**
```bash
curl -X DELETE "https://your-project-ref.supabase.co/rest/v1/users?id=eq.1" \
  -H "apikey: your-anon-key"
```
- Deletes `id = 1`.

**You can use these API calls directly in your React, React Native, or mobile app without building a backend!**

---

## **What is an RPC Call? (Remote Procedure Call)**
### **Why Use RPC Instead of Basic API Calls?**
- When you need **custom business logic** inside your database.
- When a **single API call should execute multiple SQL statements**.
- To **optimize performance** by reducing multiple API requests into one.

### **Creating an RPC (Stored Procedure) in Supabase**
RPC calls allow you to define **custom SQL functions** and call them via HTTP.

### **Example: Get Users by Email**
```sql
CREATE FUNCTION get_user_by_email(email TEXT)
RETURNS SETOF users AS $$
  SELECT * FROM users WHERE users.email = email;
$$ LANGUAGE sql STABLE;
```

### **Calling the RPC via API**
```bash
curl "https://your-project-ref.supabase.co/rest/v1/rpc/get_user_by_email" \
  -H "apikey: your-anon-key" \
  -H "Content-Type: application/json" \
  -d '{"email": "john@example.com"}'
```
**Returns user details for the given email**.

---

## **When to Use PostgREST vs. RPC Calls?**
| Use Case | PostgREST | RPC (Stored Procedures) |
|----------|----------|------------------------|
| Fetch all users | ✅ `GET /users` | ❌ Not needed |
| Fetch a user by email | ✅ `GET /users?email=eq.john@example.com` | ✅ `RPC get_user_by_email` |
| Complex queries (e.g., multiple joins) | ❌ Not efficient | ✅ Optimized with SQL functions |
| Batch processing (updating multiple rows) | ❌ Slow multiple API calls | ✅ Single RPC execution |
| Business logic (e.g., updating user & logging action) | ❌ Needs multiple API calls | ✅ Single RPC function |

---

## **Using Supabase Client in React/React Native**
### **Fetching Data Using PostgREST (REST API)**
```tsx
import { createClient } from '@supabase/supabase-js';

const supabase = createClient('https://your-project-ref.supabase.co', 'your-anon-key');

const fetchUsers = async () => {
  const { data, error } = await supabase.from('users').select('*');
  if (error) console.error(error);
  return data;
};
```

### **Calling an RPC Function**
```tsx
const getUserByEmail = async (email) => {
  const { data, error } = await supabase.rpc('get_user_by_email', { email });
  if (error) console.error(error);
  return data;
};
```


### **PostgREST (REST API)**
- Automatically generates API endpoints for every table.  
- Best for simple CRUD operations.  
- Use `.select()`, `.insert()`, `.update()`, `.delete()` in Supabase Client.  

### **RPC Calls (Stored Procedures)**
- Best for **complex SQL logic** that involves multiple queries.  
- Optimized performance (fewer API calls).  
- Allows custom business logic inside the database.  