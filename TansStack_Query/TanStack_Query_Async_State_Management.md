## TanStack Query (React Query) Library

> **References:**
> - [TanStack Query v5](https://tanstack.com/query/latest/docs/framework/react/overview)
> - [TanStack Query for React Native](https://tanstack.com/query/latest/docs/framework/react/react-native)
> - [TanStack Query Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)

There's **no global** state to manage, **reducers**, normalization systems or heavy configurations to understand. Simply pass a function that resolves your data or throws an error. It makes **fetching**, **caching**, **synchronizing** and **updating** server state in the web applications a breeze.

Features

- Backend agnostic
- Auto Caching
- Auto Refetching
- Polling/Realtime Queries
- Parallel Queries
- Automatic Garbage Collection
- Paginated/Cursor Queries
- Load-More/Infinite Scroll Queries
- Request Cancellation
- Render-as-you-fetch
- Prefetching
- Variable-length Parallel Queries
- Offline Support
- SSR Support
- Data Selectors

Traditional state management libraries are great for working with **client state**, they are not so great at working with **async** or **server state**. Server state:

- **Caching** (the hardest thing to do in programming)
- **Deduping**/Singularization multiple requests for the same data into a single request
- **Updating** "out of date" data in the background
- Performance optimizations like **pagination** and **lazy loading** data
- Managing memory and **garbage collection** of server state
- **Memoizing** query results with structural sharing
- Save on **bandwidth** and increase **memory performance**

### Installation

```bash
npm install @tanstack/react-query
```

And also it is recommended to use **ESLint Plugin Query** to catch bugs and inconsistencies:

```bash
npm install -D @tanstack/eslint-plugin-query
```

Install DevTools for debugging 

```bash
npm i @tanstack/react-query-devtools
```

⚠ For Next 13+ App Dir you must install it as a dev dependency for it to work.


Coding Example
```tsx
import {
  useQuery,
  useMutation,
  useQueryClient,
  QueryClient,
  QueryClientProvider,
} from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools' // Import devtools
import { getTodos, postTodo } from '../my-api'

/* This code snippet very briefly illustrates the 3 core concepts of React Query:
    - Queries
    - Mutations
    - Query Invalidation

By default, React Query Devtools are only included in bundles when process.env.NODE_ENV === 'development'
*/

// Create a client
const queryClient = new QueryClient()

function App() {
  return (
    // Provide the client to your App
    <QueryClientProvider client={queryClient}>
      <Todos />
    </QueryClientProvider>
  )
}

function Todos() {
  // Access the client
  const queryClient = useQueryClient()

  // Queries
  const query = useQuery({ queryKey: ['todos'], queryFn: getTodos })

  // Mutations
  const mutation = useMutation({
    mutationFn: postTodo,
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })

  return (
    <div>
      <ul>{query.data?.map((todo) => <li key={todo.id}>{todo.title}</li>)}</ul>

      <button
        onClick={() => {
          mutation.mutate({
            id: Date.now(),
            title: 'Do Laundry',
          })
        }}
      >
        Add Todo
      </button>
    </div>
  )
}

render(<App />, document.getElementById('root'))
```

### [Devtools](https://tanstack.com/query/latest/docs/framework/react/devtools) Integration

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* The rest of your application */}
      <ReactQueryDevtools initialIsOpen={false} />  {/* Floating Mode */}
    </QueryClientProvider>
  )
}
```

### TanStack Defaults

Query instances via ```useQuery``` or ```useInfiniteQuery``` by default consider **cached** data as stale. To change this behavior, you can configure your queries both globally and per-query using the ```staleTime``` option. Specifying a longer ```staleTime``` means queries **will not refetch** their data as often.

