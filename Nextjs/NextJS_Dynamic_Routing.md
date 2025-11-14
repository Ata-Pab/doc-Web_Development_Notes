## 1. **Dynamic Route Segments**
Dynamic route segments allow you to define parts of a route dynamically using square brackets (`[]`) in the file or folder names.

### Example:
You can create a file structure like this:

```
app/
├── page.tsx
├── product/
│   ├── [id]/
│   │   ├── page.tsx
```

### Code Example:
```tsx
// app/product/[id]/page.tsx
import { useParams } from 'next/navigation';

export default function ProductPage() {
  const params = useParams(); // Get dynamic params

  return (
    <div>
      <h1>Product ID: {params.id}</h1>
    </div>
  );
}
```

### Explanation:
- The route `/product/[id]` will match URLs like:
  - `/product/1`
  - `/product/2`
- The `id` parameter can be accessed via `useParams()` or via the `props` of the `Page` component.


## 2. **Catch-All Segments**
Catch-all segments allow you to create routes that match multiple dynamic segments.  
You can define catch-all routes using `[...name]`.

### Example:
Create a catch-all route structure like this:

```
app/
├── docs/
│   ├── [...slug]/
│   │   ├── page.tsx
```

### Code Example:
```tsx
// app/docs/[...slug]/page.tsx
import { useParams } from 'next/navigation';

export default function DocsPage() {
  const params = useParams();

  return (
    <div>
      <h1>Docs Path: {params.slug?.join('/')}</h1>
    </div>
  );
}
```

### Explanation:
- The route `/docs/[...slug]` will match URLs like:
  - `/docs/installation`
  - `/docs/guides/setup`
  - `/docs/api/v1/endpoints`
- The `params.slug` will return an **array** of path segments:
  - `/docs/installation` → `['installation']`
  - `/docs/guides/setup` → `['guides', 'setup']`
- If you want to create an **optional catch-all segment**, you can use `[...[slug]]`.


## 3. **Optional Catch-All Segments**  
Optional catch-all segments allow the route to match both with and without additional segments.  
You can define an optional catch-all route using `[[...name]]`.

### Example:
Create an optional catch-all route structure like this:

```
app/
├── blog/
│   ├── [[...slug]]/
│   │   ├── page.tsx
```

### Code Example:
```tsx
// app/blog/[[...slug]]/page.tsx
import { useParams } from 'next/navigation';

export default function BlogPage() {
  const params = useParams();

  return (
    <div>
      <h1>
        {params.slug
          ? `Blog Path: ${params.slug.join('/')}`
          : 'Default Blog Page'}
      </h1>
    </div>
  );
}
```

### Explanation:
- The route `/blog/[[...slug]]` will match:
  - `/blog` → `params.slug` will be `undefined`
  - `/blog/post` → `params.slug` will be `['post']`
  - `/blog/2025/nextjs` → `params.slug` will be `['2025', 'nextjs']`
- This is useful for defining both a base page and specific detail pages under the same route.


## 4. **Parallel Routes**
Parallel routes allow you to define multiple independent layouts or components at the same path.  
You define parallel routes using the `@folder` convention.

### Example:
```tsx
// app/@header/page.tsx
export default function Header() {
  return <div>Header</div>;
}

// app/@main/page.tsx
export default function Main() {
  return <div>Main</div>;
}

// app/layout.tsx
export default function Layout({
  children,
  header,
  main,
}: {
  children: React.ReactNode;
  header: React.ReactNode;
  main: React.ReactNode;
}) {
  return (
    <div>
      <div>{header}</div>
      <div>{main}</div>
      <div>{children}</div>
    </div>
  );
}
```

###  Explanation:
- The `@header` and `@main` folders define parallel routes.
- The `layout.tsx` will render them **in parallel** alongside the children route.

---

## **Summary**

| Type | Pattern | Example | Params Format |
|-------|---------|---------|---------------|
| **Dynamic Route Segments** | `[id]` | `/product/1` → `params.id` = `'1'` | `string` |
| **Catch-All Segments** | `[...slug]` | `/docs/guides/setup` → `params.slug` = `['guides', 'setup']` | `string[]` |
| **Optional Catch-All Segments** | `[[...slug]]` | `/blog`, `/blog/post/nextjs` → `params.slug` = `['post', 'nextjs']` or `undefined` | `string[] \| undefined` |
| **Parallel Routes** | `@header`, `@main` | `/header`, `/main` rendered in parallel | - |

- Use **dynamic segments** for structured data like product IDs, user IDs, etc.  
- Use **catch-all segments** for nested or deeply structured content (like documentation).  
- Use **optional catch-all segments** when you want to define a base route alongside detailed subroutes.  
- Use **parallel routes** for complex layouts or UI components that need to load independently.

---
