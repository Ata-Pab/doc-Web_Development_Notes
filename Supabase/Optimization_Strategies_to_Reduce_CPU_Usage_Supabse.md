> **Scenario:** Customers frequently fetch the same path:  
> `/restaurantID/menuID/categories/...`  
> (This likely includes menus, categories, meals, translations, etc.)

---

## Optimization Strategies to Reduce CPU Usage

### 1. **Materialized Views (PostgreSQL)**

**Use Case:** Frequently queried, rarely changing data like menus, categories, and meals.

```sql
CREATE MATERIALIZED VIEW menu_snapshot AS
SELECT 
  r.id AS restaurant_id,
  m.id AS menu_id,
  c.id AS category_id,
  c.name AS category_name,
  me.id AS meal_id,
  me.price,
  mt.name AS meal_name
FROM restaurants r
JOIN menus m ON m.restaurant_id = r.id
JOIN categories c ON c.menu_id = m.id
JOIN meals me ON me.category_id = c.id
JOIN meal_translations mt ON mt.meal_id = me.id
WHERE mt.language_code = 'en';
```

You can refresh it when data changes:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY menu_snapshot;
```

> Pros:
> - Avoids repeated joins at runtime
> - Much faster reads
> - Reduces CPU spikes

> Cons:
> - Needs to be refreshed manually or via trigger/scheduled task

---

### 2. **PostgREST Caching (Supabase Edge Cache or CDN)**

Supabase provides caching headers via PostgREST. You can set cache control headers in PostgREST or cache it on a reverse proxy (like Vercel, Cloudflare, etc.).

#### Example:

```ts
fetch('/restaurant/xyz/menu', {
  headers: {
    'Cache-Control': 'public, max-age=300' // 5 minutes
  }
});
```

Or cache it on the frontend/backend (e.g., SWR, React Query, etc.).

> Offloads repeat traffic from hitting the database
> Super low latency for clients

---

### 3. **Stored Procedures or RPC Functions (with light logic)**

Instead of many client queries hitting multiple endpoints, you can **wrap your complex logic in an RPC**:

```sql
CREATE FUNCTION get_menu_data(restaurant_id UUID, lang TEXT)
RETURNS TABLE (...) AS $$
BEGIN
  RETURN QUERY
  SELECT ...
  FROM ...
  WHERE ...
END;
$$ LANGUAGE plpgsql;
```

> Minimizes roundtrips and CPU cost per client
> Efficient batch execution
> Easier to monitor and optimize a single entry point

---

### 4. **Limit Result Size + Use Indexed Pagination**

If you list meals/categories, **don’t return everything at once**. Use:

- Pagination (`limit`, `offset`, or keyset pagination)
- Query indexes on `menu_id`, `restaurant_id`, etc.

```sql
SELECT * FROM meals
WHERE category_id = ...
ORDER BY position
LIMIT 20 OFFSET 0;
```

> Reduces data transfer
> Reduces CPU time from sorting large lists

---

### 5. **Denormalized Cache Table (Advanced / Precomputed JSON)**

For *super-read-heavy* use cases (like QR menus), you can create a cache table with a full JSON blob:

```sql
CREATE TABLE menu_cache (
  restaurant_id UUID PRIMARY KEY,
  language_code TEXT,
  menu_json JSONB,
  updated_at TIMESTAMP
);
```

You pre-render it (via trigger, cron, or Supabase Edge Function) and serve it instantly:

```sql
SELECT menu_json FROM menu_cache
WHERE restaurant_id = 'xyz' AND language_code = 'en';
```

> Super fast
> CPU usage near-zero
> Needs logic to update when menu data changes

---

## Monitor High-CPU Queries

Use Supabase Dashboard → **Database → Logs → Performance**  
Watch for:

- Repeated long-running queries
- Unindexed WHERE clauses
- Large JOIN chains

Add indexes on:

- `restaurant_id`
- `menu_id`
- `category_id`
- Composite indexes for filtering/sorting

---

## Summary

| Optimization | Reduces CPU? | Easy to Implement? | Notes |
|--------------|---------------|--------------------|-------|
| Materialized Views | ✅✅ | 🟡 Medium | Great for static or rarely updated data |
| CDN/Edge Caching | ✅✅✅ | ✅ Easy | Works great for all clients |
| RPC Functions | ✅ | ✅ Medium | Consolidates logic and reduces client compute |
| Pagination + Indexing | ✅ | ✅ Easy | Always recommended |
| Precomputed JSON | ✅✅✅ | 🔴 Complex | Best for very high traffic routes |

---
