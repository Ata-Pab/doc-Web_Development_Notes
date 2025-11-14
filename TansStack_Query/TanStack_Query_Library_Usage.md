## **Setup TanStack Query**
1. Install TanStack Query:
```bash
npm install @tanstack/react-query
```

2. Install the DevTools (optional but very useful):
```bash
npm install @tanstack/react-query-devtools
```

---

## **Set Up a Query Client**
In your Next.js app, you need to wrap your components with a `QueryClientProvider`:

### `app/providers.tsx`
```tsx
"use client";

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactNode, useState } from 'react';

export default function Providers({ children }: { children: ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### `app/layout.tsx`
```tsx
import Providers from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## **Fetching Data Example (Using Supabase)**
Here's how to use `useQuery` to fetch data from Supabase:

### Example: Fetching Meals from Supabase
```tsx
import { useQuery } from '@tanstack/react-query';
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_KEY!
);

const fetchMeals = async () => {
  const { data, error } = await supabase.from('meals').select('*');
  if (error) throw new Error(error.message);
  return data;
};

export default function Meals() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['meals'],
    queryFn: fetchMeals,
  });

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      {data?.map((meal) => (
        <div key={meal.id}>
          <h2>{meal.name}</h2>
          <p>{meal.price}</p>
        </div>
      ))}
    </div>
  );
}
```

### **Explanation:**
- `queryKey` – A unique key for caching the data.
- `queryFn` – Function that fetches the data.
- `data` – Fetched data.
- `isLoading` – Indicates loading state.
- `isError` – Indicates error state.
- `error` – Holds the error message if fetching fails.

---

## **Refetching Data**
You can refetch data automatically when the user focuses the window or pull-to-refresh:
```tsx
const { data, refetch } = useQuery({
  queryKey: ['meals'],
  queryFn: fetchMeals,
  refetchOnWindowFocus: true, // Default is true
  refetchOnMount: true, // Refetch when component mounts
});
```

---

## **Mutating Data (Inserting, Updating, Deleting)**
To modify data, use `useMutation`.

### Example: Inserting a Meal
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

const addMeal = async (meal) => {
  const { data, error } = await supabase.from('meals').insert(meal);
  if (error) throw new Error(error.message);
  return data;
};

export default function AddMeal() {
  const queryClient = useQueryClient();

  const mutation = useMutation({
    mutationFn: addMeal,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['meals'] }); // Refetch after adding
    },
  });

  const handleAdd = () => {
    mutation.mutate({ name: 'Pizza', price: 12.99 });
  };

  return (
    <div>
      <button onClick={handleAdd} disabled={mutation.isLoading}>
        {mutation.isLoading ? 'Adding...' : 'Add Meal'}
      </button>
      {mutation.isError && <p>Error: {mutation.error.message}</p>}
    </div>
  );
}
```

### **Explanation:**
- `useMutation` – Handles mutations (inserts/updates/deletes).
- `mutationFn` – Function that performs the mutation.
- `onSuccess` – Called after a successful mutation.
- `queryClient.invalidateQueries` – Forces a refetch after mutation.

---

## **Optimistic Updates**
If you want to show the update *before* the server confirms it:

```tsx
const mutation = useMutation({
  mutationFn: addMeal,
  onMutate: async (newMeal) => {
    // Cancel any outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['meals'] });

    // Snapshot the previous value
    const previousMeals = queryClient.getQueryData(['meals']);

    // Optimistically update to the new value
    queryClient.setQueryData(['meals'], (old) => [...(old || []), newMeal]);

    return { previousMeals };
  },
  onError: (err, newMeal, context) => {
    queryClient.setQueryData(['meals'], context.previousMeals);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['meals'] });
  },
});
```

---

## **Handling Pagination and Infinite Scroll**
For paginated data:
```tsx
const fetchMeals = async ({ pageParam = 0 }) => {
  const { data, error } = await supabase
    .from('meals')
    .select('*')
    .range(pageParam, pageParam + 10);

  if (error) throw new Error(error.message);
  return data;
};

const { data, fetchNextPage } = useInfiniteQuery({
  queryKey: ['meals'],
  queryFn: fetchMeals,
  getNextPageParam: (lastPage, allPages) => lastPage.length ? allPages.length * 10 : undefined,
});
```

---

## **When to Use TanStack Query**
**Use it for:**
- Data fetching and caching from APIs.
- Synchronizing server state with client state.
- Handling real-time updates and invalidations.

**Avoid it for:**
- Global state like theme, user auth status → (Use Zustand or Context API for that).

---

##  **Next Steps**
- Start with fetching data using `useQuery`.  
- Use `useMutation` for inserts, updates, and deletes.  
- Add optimistic updates and pagination when needed.  
- Use DevTools (`ReactQueryDevtools`) to monitor queries and mutations:  

### Add DevTools to `Providers.tsx`:
```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  {children}
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

---

That's an excellent question! Let's go deeper into how **TanStack Query** handles data updates, changes, and caching — especially when working with **Supabase** or similar databases.

## **1. Does TanStack Query Automatically Detect Changes?**
No, **TanStack Query does not automatically detect changes** in the backend or database. However, you can configure it to stay in sync with the backend through various strategies:

### **Automatic Refetching on Events:**
TanStack Query can refetch data automatically when:
- **Window Focus:** When the user switches back to the tab.
- **Reconnect:** When the user regains internet connection.
- **Time Interval:** Periodic background refetching.
- **Manual Invalidations:** After a mutation or manual trigger.

### Example:
To enable automatic background refetching:
```tsx
const { data } = useQuery({
  queryKey: ['meals'],
  queryFn: fetchMeals,
  refetchOnWindowFocus: true, // When the window gains focus
  refetchOnReconnect: true, // When internet reconnects
  refetchInterval: 10000, // Refetch every 10 seconds
});
```

---

## **2. Can TanStack Query Handle Partial Row Updates (PATCH)?**
TanStack Query itself **does not perform partial updates directly** — but it gives you the tools to handle them **manually** using `useMutation` and `queryClient.setQueryData()`.

### **How to Handle Partial Updates Efficiently (PATCH):**
1. Perform a `PATCH` request to update only the changed properties.
2. Use `queryClient.setQueryData()` to update only the changed properties **in the cache** (without refetching the whole row).
3. This allows for an **instant UI update** while the server processes the update.

---

### Example: Efficiently Updating a Single Column in Supabase
Suppose you want to update the `price` of a meal without refetching all the data:

```tsx
const updateMealPrice = async ({ id, price }) => {
  const { data, error } = await supabase
    .from('meals')
    .update({ price })
    .eq('id', id);

    /* OR you can add a LIMIT and RETURNING to confirm the update to prevent accidental updates
    const { data, error } = await supabase
      .from('meals')
      .update({ price })
      .eq('id', id)
      .limit(1) // Ensure you don't accidentally update multiple rows
      .select('id, price');
    */

  if (error) throw new Error(error.message);
  return data;
};

const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: updateMealPrice,
  onMutate: async ({ id, price }) => {
    // Cancel any outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['meals'] });

    // Snapshot the previous value
    const previousMeals = queryClient.getQueryData(['meals']);

    // Optimistically update the value
    queryClient.setQueryData(['meals'], (old) =>
      old.map((meal) =>
        meal.id === id ? { ...meal, price } : meal
      )
    );

    return { previousMeals };
  },
  onError: (err, newMeal, context) => {
    // Rollback to the previous value on error
    queryClient.setQueryData(['meals'], context.previousMeals);
  },
  onSettled: () => {
    // Optionally refetch to make sure the cache is consistent
    queryClient.invalidateQueries({ queryKey: ['meals'] });
  },
});
```

### Explanation:
- `onMutate` – Before making the server call, we update the cache optimistically.
- `queryClient.setQueryData()` – Directly modifies the cached data.
- `onError` – Restores the cache if the mutation fails.
- `onSettled` – Ensures the cache remains consistent after the mutation.

This is more efficient than re-fetching everything because it updates only the affected row **in memory** without needing to hit the server again.

---

## **3. How to Update Multiple Rows Efficiently**
If you need to update multiple rows at once (for example, increasing the price of all meals by 10%), you can batch the updates like this:

### Example: Bulk Update Without Refetching:
```tsx
const updateAllMealPrices = async (priceIncrease) => {
  const { data, error } = await supabase
    .from('meals')
    .update({ price: priceIncrease })
    .gt('price', 0);

  if (error) throw new Error(error.message);
  return data;
};

const mutation = useMutation({
  mutationFn: updateAllMealPrices,
  onMutate: async (priceIncrease) => {
    await queryClient.cancelQueries({ queryKey: ['meals'] });

    const previousMeals = queryClient.getQueryData(['meals']);

    queryClient.setQueryData(['meals'], (old) =>
      old.map((meal) => ({
        ...meal,
        price: meal.price + priceIncrease,
      }))
    );

    return { previousMeals };
  },
  onError: (err, newMeals, context) => {
    queryClient.setQueryData(['meals'], context.previousMeals);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['meals'] });
  },
});
```

This avoids multiple network calls and updates the cache immediately for a smoother UI experience.

---

## **4. How to Sync Data Automatically with Supabase (Real-Time)**
Supabase supports **Postgres Realtime** which means you can subscribe to row changes and automatically update the client-side state without manual refetching.

### Example: Real-Time Subscription with Supabase + TanStack Query
You can integrate Supabase's real-time subscription with TanStack Query's cache:

```tsx
useEffect(() => {
  const subscription = supabase
    .from('meals')
    .on('INSERT', (payload) => {
      queryClient.setQueryData(['meals'], (old) => [...(old || []), payload.new]);
    })
    .on('UPDATE', (payload) => {
      queryClient.setQueryData(['meals'], (old) =>
        old.map((meal) =>
          meal.id === payload.new.id ? payload.new : meal
        )
      );
    })
    .on('DELETE', (payload) => {
      queryClient.setQueryData(['meals'], (old) =>
        old.filter((meal) => meal.id !== payload.old.id)
      );
    })
    .subscribe();

  return () => {
    subscription.unsubscribe();
  };
}, [queryClient]);
```

### Explanation:
- `on('INSERT')` → When a new row is inserted, add it to the cache.
- `on('UPDATE')` → When a row is updated, modify only the affected row.
- `on('DELETE')` → When a row is deleted, remove it from the cache.

This keeps the cache updated **without any manual fetching** — Supabase pushes changes automatically.

---

## **5. How to Handle Conditional or Dependent Queries**
You can define a query as **dependent** on another query’s result or value:

### Example: Fetch Meals After User Logs In:
```tsx
const { data: user } = useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
});

const { data: meals } = useQuery({
  queryKey: ['meals'],
  queryFn: fetchMeals,
  enabled: !!user, // Only fetch meals if the user is logged in
});
```

`enabled` controls whether the query should run or not.

---

## **6. Pagination and Infinite Scroll (Efficient)**
You can efficiently handle pagination and infinite scroll using `useInfiniteQuery`:

### Example: Infinite Scrolling with Supabase
```tsx
const { data, fetchNextPage } = useInfiniteQuery({
  queryKey: ['meals'],
  queryFn: ({ pageParam = 0 }) =>
    supabase.from('meals').select('*').range(pageParam, pageParam + 10),
  getNextPageParam: (lastPage, allPages) =>
    lastPage.length ? allPages.length * 10 : undefined,
});
```

This fetches only the next page without over-fetching.

---

## **Best Practices**
- Use **onMutate + optimistic updates** for smooth UX.  
- Keep `queryKey` consistent across queries and mutations.  
- Use **real-time subscriptions** with Supabase to keep cache fresh.  
- For **massive datasets**, use pagination or infinite scrolling.  
- Use `refetchOnWindowFocus` and `refetchInterval` carefully to avoid unnecessary traffic.  

---

## Let's create a **professional-grade Menu Manager** component using:

- **Next.js** – Framework for server-side rendering and routing.  
- **Shadcn-UI** – For building a clean and modern form.  
- **TanStack Query** – For fetching, updating, inserting, and deleting data.  
- **Supabase** – For backend data storage.  

---

## **Goal:**
- Create a `MenuManager` page to:
  - Fetch the list of meals.
  - Add a new meal.
  - Update an existing meal.
  - Delete a meal.
  - Use optimistic updates for smooth UI.

---

## **Project Structure**
```
src
├── app
│   ├── menu-manager
│   │   ├── page.tsx
│   │   ├── MealForm.tsx
│   │   └── MealList.tsx
└── components
    └── ui
        └── Button.tsx
```

---

## **Install Dependencies**
```bash
npm install @tanstack/react-query @supabase/supabase-js shadcn-ui
```

---

## **Set Up Supabase Client**
Create a `supabase.ts` file:

### `src/lib/supabase.ts`
```ts
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_KEY!
);
```

---

## **Set Up TanStack Query Provider**
### `src/app/providers.tsx`
```tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactNode, useState } from 'react';

export default function Providers({ children }: { children: ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

### `src/app/layout.tsx`
```tsx
import Providers from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## **Fetch Meals**
Create a `useMeals` hook to fetch meals from Supabase.

### `src/hooks/useMeals.ts`
```ts
import { useQuery } from '@tanstack/react-query';
import { supabase } from '@/lib/supabase';

const fetchMeals = async () => {
  const { data, error } = await supabase.from('meals').select('*');
  if (error) throw new Error(error.message);
  return data;
};

export const useMeals = () => {
  return useQuery({
    queryKey: ['meals'],
    queryFn: fetchMeals,
  });
};
```

---

## **Insert Meal**
### `src/hooks/useInsertMeal.ts`
```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { supabase } from '@/lib/supabase';

const insertMeal = async (meal: { name: string; price: number }) => {
  const { data, error } = await supabase.from('meals').insert([meal]);
  if (error) throw new Error(error.message);
  return data;
};

export const useInsertMeal = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: insertMeal,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['meals'] });
    },
  });
};
```

---

## **Update Meal**
### `src/hooks/useUpdateMeal.ts`
```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { supabase } from '@/lib/supabase';

const updateMeal = async (meal: { id: number; name: string; price: number }) => {
  const { data, error } = await supabase
    .from('meals')
    .update({ name: meal.name, price: meal.price })
    .eq('id', meal.id);
  if (error) throw new Error(error.message);
  return data;
};

export const useUpdateMeal = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateMeal,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['meals'] });
    },
  });
};
```

---

## **Delete Meal**
### `src/hooks/useDeleteMeal.ts`
```ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { supabase } from '@/lib/supabase';

const deleteMeal = async (id: number) => {
  const { data, error } = await supabase.from('meals').delete().eq('id', id);
  if (error) throw new Error(error.message);
  return data;
};

export const useDeleteMeal = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteMeal,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['meals'] });
    },
  });
};
```

---

## **Meal Form**
### `src/app/menu-manager/MealForm.tsx`
```tsx
'use client';

import { useInsertMeal } from '@/hooks/useInsertMeal';
import { useState } from 'react';
import { Button } from '@/components/ui/Button';

export default function MealForm() {
  const [name, setName] = useState('');
  const [price, setPrice] = useState<number>(0);
  const { mutate, isLoading } = useInsertMeal();

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    mutate({ name, price });
    setName('');
    setPrice(0);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Meal Name"
      />
      <input
        type="number"
        value={price}
        onChange={(e) => setPrice(Number(e.target.value))}
        placeholder="Price"
      />
      <Button type="submit" disabled={isLoading}>
        {isLoading ? 'Saving...' : 'Save Meal'}
      </Button>
    </form>
  );
}
```

---

## **Meal List**
### `src/app/menu-manager/MealList.tsx`
```tsx
'use client';

import { useMeals } from '@/hooks/useMeals';
import { useDeleteMeal } from '@/hooks/useDeleteMeal';

export default function MealList() {
  const { data, isLoading } = useMeals();
  const { mutate: deleteMeal } = useDeleteMeal();

  if (isLoading) return <div>Loading...</div>;

  return (
    <ul>
      {data?.map((meal) => (
        <li key={meal.id}>
          {meal.name} - ${meal.price}
          <button onClick={() => deleteMeal(meal.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## **Final Menu Manager Page**
### `src/app/menu-manager/page.tsx`
```tsx
'use client';

import MealForm from './MealForm';
import MealList from './MealList';

export default function MenuManager() {
  return (
    <div className="space-y-8">
      <h1 className="text-2xl font-bold">Menu Manager</h1>
      <MealForm />
      <MealList />
    </div>
  );
}
```

---

## **What This Covers:**
- Efficient fetching with `useQuery`  
- Inserting, updating, deleting with `useMutation`  
- Cache invalidation with `invalidateQueries()`  
- Real-time UI updates with optimistic mutations  

---

### Update DB for Only Changed Values

```tsx
'use client';

import { useUpdateMeal } from '@/hooks/useUpdateMeal';
import { useState, useEffect } from 'react';
import { Button } from '@/components/ui/Button';

interface MealFormProps {
  meal: { id: number; name: string; price: number };
}

export default function MealForm({ meal }: MealFormProps) {
  const [name, setName] = useState(meal.name);
  const [price, setPrice] = useState(meal.price);
  const [isChanged, setIsChanged] = useState(false);

  const { mutate, isLoading } = useUpdateMeal();

  useEffect(() => {
    if (name !== meal.name || price !== meal.price) {
      setIsChanged(true);
    } else {
      setIsChanged(false);
    }
  }, [name, price, meal.name, meal.price]);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    if (isChanged) {
      const updatedFields: Record<string, any> = {};
      if (name !== meal.name) updatedFields.name = name;
      if (price !== meal.price) updatedFields.price = price;

      mutate({ id: meal.id, ...updatedFields });
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Meal Name"
      />
      <input
        type="number"
        value={price}
        onChange={(e) => setPrice(Number(e.target.value))}
        placeholder="Price"
      />
      <Button type="submit" disabled={!isChanged || isLoading}>
        {isLoading ? 'Saving...' : isChanged ? 'Save Changes' : 'No Changes'}
      </Button>
    </form>
  );
}

```