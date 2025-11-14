# Next.js Tutorial

Main source: https://nextjs.org/

### Next.js project Set up
```bash
npx create-next-app@latest
```

Create an 'app' folder if it does not exists

```app/layout.tsx```
```ts
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```
```app/page.tsx```
```ts
export default function Page() {
  return <h1>Hello, Next.js!</h1>
}
```

Run the development server

1. Run npm run dev to start the development server.
2. Visit http://localhost:3000 to view your application.
3. Edit theapp/page.tsx file and save it to see the updated result in your browser.

### App Router vs Pages Router

Next.js has two different routers: App Router and the Pages Router
- App Router is a newer router that allows you to use React's latest features, such as Server Components and Streaming. 
- Pages Router is the original Next.js router, which allowed you to build server-rendered React applications and continues to be supported for older Next.js applications.

### Top-level files

- ```next.config.js```:	Configuration file for Next.js
- ```package.json```: Project dependencies and scripts
- ```instrumentation.ts```:	OpenTelemetry and Instrumentation file
- ```middleware.ts```:	Next.js request middleware
- ```.env```:	Environment variables
- ```.env.local```:	Local environment variables
- ```.env.production```:	Production environment variables
- ```.env.development```:	Development environment variables
- ```.eslintrc.json```:	Configuration file for ESLint
- ```.gitignore```:	Git files and folders to ignore
- ```next-env.d.ts```:	TypeScript declaration file for Next.js
- ```tsconfig.json```:	Configuration file for TypeScript

### Routing files 

- ```layout.tsx```: Layout
- ```page.tsx```: Page
- ```loading.tsx```: Loading UI
- ```not-found.tsx```: Not found UI
- ```error.tsx```: Error UI
- ```global-error.tsx```: Global error UI
- ```route.ts```: 	API endpoint (.js .ts file not .tsx)
- ```template.tsx```: Re-rendered layout
- ```default.tsx```: Parallel route fallback page

### Dynamic routes
- ```[folder]```: Dynamic route segment
- ```[...folder]```: Catch-all route segment
- ```[[...folder]]```: Optional catch-all route segment

### Route Groups and private folders
- ```(folder)```: Group routes without affecting routing
- ```_folder```: Opt folder and all child segments out of routing

### Parallel and Intercepted Routes
- ```@folder```: Named slot
- ```(.)folder```: Intercept same level
- ```(..)folder```: Intercept one level above
- ```(..)(..)folder```: Intercept two levels above
- ```(...)folder```: Intercept from root

### Metadata file conventions
- ```favicon.ico```: Favicon file
- ```icon(.ico .jpg .jpeg .png .svg)```: App Icon file
- ```icon(.js .ts .tsx)```: Generated App Icon

### SEO
- ```sitemap.xml```: Sitemap file
- ```sitemap(.js .ts)```: Generated Sitemap
- ```robots.txt```: Robots file
- ```robots(.js .ts)```: Generated Robots file

### Component hierarchy
- ```layout.js```
- ```template.js```
- ```error.js``` (React error boundary)
- ```loading.js``` (React suspense boundary)
- ```not-found.js``` (React error boundary)
- ```page.js``` or nested layout.js

! Even though route structure is defined through folders, a route is not publicly accessible until a <b>page.js</b> or <b>route.js</b> file is added to a route segment.

### Private folders
Private folders can be created by prefixing a folder with an underscore: <b>_folderName</b>

This indicates the folder is a private implementation detail and should <b>not</b> be considered by the <b>routing system</b>, thereby opting the folder and all its subfolders <b>out of routing</b>. 
- app/dashboard/__components => "/dashboard": Routable, "/dashboard/_components": Not Routable

### Route groups
Route groups can be created by wrapping a folder in parenthesis: <b>(folderName)</b>

This indicates the folder is for organizational purposes and should <b>not be included</b> in the <b>route's URL path</b>.

<img src="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fproject-organization-route-groups.png&w=3840&q=75" alt="Route groups" width="600"/>

### Parallel Routes

Parallel Routes allows you to **simultaneously** or conditionally **render one or more pages** within the same layout. They are useful for highly dynamic sections of an app, such as dashboards and feeds on social sites.

<img src="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fparallel-routes.png&w=3840&q=75" alt="Route groups" width="600"/>

Parallel routes are created using named slots. Slots are defined with the ```@folder``` convention. For example, the following file structure defines two slots: ```@analytics``` and ```@team```. Slots are not route segments and do **not affect the URL structure** (for ```/@analytics/views```, the URL will be ```/views``` since ```@analytics``` is a slot). The ```children``` prop is an implicit slot that does not need to be mapped to a folder. This means ```app/page.js``` is equivalent to ```app/@children/page.js```.

```app/layout.tsx```
```ts
export default function Layout({
  children,
  team,
  analytics,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <>
      {children}
      {team}
      {analytics}
    </>
  )
}
```

#### Loading and Error UI with Parallel Routes

<img src="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fparallel-routes-cinematic-universe.png&w=3840&q=75" alt="Route groups" width="600"/>

### Active state and navigation

- **Soft Navigation**: During client-side navigation, Next.js will perform a partial render, changing the subpage within the slot, while maintaining the other slot's active subpages, even if they don't match the current URL.
- **Hard Navigation**: After a full-page load (browser refresh), Next.js cannot determine the active state for the slots that don't match the current URL. Instead, it will render a default.js file for the unmatched slots, or 404 if ```default.tsx``` doesn't exist.

```app/layout.tsx```
```tsx
'use client'

// When a user navigates to app/@auth/login (or /login in the URL bar), loginSegment will be equal to the string "login".
 
import { useSelectedLayoutSegment } from 'next/navigation'
 
export default function Layout({ auth }: { auth: React.ReactNode }) {
  const loginSegment = useSelectedLayoutSegment('auth')
  // ...
}
```

### Conditional Routes

You can use Parallel Routes to conditionally render routes based on certain conditions, such as user role. For example, to render a different dashboard page for the ```/admin``` or ```/user``` roles:

<img src="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fconditional-routes-ui.png&w=3840&q=75" alt="Route groups" width="600"/>

```app/dashboard/layout.tsx```
```tsx
import { checkUserRole } from '@/lib/auth'
 
export default function Layout({
  user,
  admin,
}: {
  user: React.ReactNode
  admin: React.ReactNode
}) {
  const role = checkUserRole()
  return role === 'admin' ? admin : user
}
```

### Creating a layout
A layout is UI that is <b>shared between multiple pages</b>. On navigation, layouts preserve state, remain interactive, and do <b>not rerender</b>.

You can define a layout by default exporting a React component from a layout file. The component <b>should accept</b> a <b>children</b> prop which can be a <b>page</b> or another layout.

```app/dashboard/layout.tsx```
```ts
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        {/* Layout UI */}
        {/* Place children where you want to render a page or nested layout */}
        <main>{children}</main>
      </body>
    </html>
  )
}
```

### Linking between pages

You can use the ```<Link>``` component to navigate between routes. ```<Link>``` is a built-in Next.js component that extends the HTML ```<a>``` tag to provide prefetching and client-side navigation. For example, to generate a list of blog posts, import ```<Link>``` from next/link and pass a href prop to the component:

<img src="https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fnested-layouts.png&w=3840&q=75" alt="Route groups" width="600"/>

#### **1st Implementation:**
```app/ui/post.tsx```
```jsx
import Link from 'next/link'
 
export default async function Post({ post }) {
  const posts = await getPosts()
 
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.slug}>
          <Link href={`/blog/${post.slug}`}>{post.title}</Link>
        </li>
      ))}
    </ul>
  )
}
```
#### **Explanation:**
- This is a more conventional way to handle navigation in a blog.
- The blog posts are retrieved using `getPosts()`, and each post is mapped into a `<li>` element.
- The `Link` component uses **dynamic route-based navigation** (`/blog/[slug]`).
- Clicking on a post title will navigate to `/blog/${post.slug}` (e.g., `/blog/my-first-post`).
- This method is **SEO-friendly** and follows a clean URL structure.

#### **Use case:**
Best for blogs where each post has its own dedicated page (e.g., `/blog/my-post`).  
Ideal when you have a **file-based dynamic route** (e.g., `pages/blog/[slug].js` or `app/blog/[slug]/page.js`).  

---

#### **2nd Implementation:**
```app/ui/post.tsx```

```jsx
import Link from 'next/link'
 
export default async function Post({ post }) {
  const posts = await getPosts()

  return (
    <Link
      href={{
        pathname: '/blog',
        query: { post: `${post.slug}` }, // Corrected syntax
      }}
    >
      Blog
    </Link>
  )
}
```
#### **Explanation:**
- This navigates to the **`/blog`** page but **passes the `post.slug` as a query parameter**.
- Instead of `/blog/my-post`, it navigates to `/blog?post=my-post`.
- The blog page (`/blog`) would need to extract the query parameter (`post`) from the URL using `useRouter()` from `next/router` (for Pages Router) or `useSearchParams()` (for App Router).

#### **Use case:**
Best if the blog page (`/blog`) dynamically renders the selected post without creating separate URLs.  
Useful when `/blog` serves as a single-page interface displaying different posts based on the query.

---

#### **Which One is Better?**
- **First approach (`/blog/[slug]`)**  is the preferred and common practicefor **SEO** , clean URLs, and easy bookmarking.
- **Second approach (`/blog?post=slug`)** is useful if you want to handle navigation within a **single page**  without a full page reload.


### [How to optimize images and fonts](https://nextjs.org/docs/app/getting-started/images-and-fonts)
You can store static files, like images and fonts, under a folder called **public** in the root directory. The Next.js ```<Image>``` component extends the HTML ```<img>``` element to provide:

```app/page.tsx```
```ts
import Image from 'next/image'
 
export default function Page() {
  return <Image src="" alt="" />
}
```

#### Using Local Images
```app/page.tsx```
```tsx
import Image from 'next/image'
import profilePic from './me.png'
 
export default function Page() {
  return (
    <Image
      src={profilePic}
      alt="Picture of the author"
      // width={500} automatically provided
      // height={500} automatically provided
      // blurDataURL="data:..." automatically provided
      // placeholder="blur" // Optional blur-up while loading
    />
  )
}
```

#### Remote images

```app/page.tsx```
```tsx
import Image from 'next/image'
 
export default function Page() {
  return (
    <Image
      src="https://s3.amazonaws.com/my-bucket/profile.png"
      alt="Picture of the author"
      width={500}
      height={500}
    />
  )
}
```

Since Next.js does not have access to remote files during the build process, you'll need to provide the width, height and optional blurDataURL props manually. The width and height attributes are used to infer the correct aspect ratio of image and avoid layout shift from the image loading in.

Then, to safely allow images from remote servers, you need to define a list of supported URL patterns in next.config.js. Be as specific as possible to prevent malicious usage. For example, the following configuration will only allow images from a specific AWS S3 bucket:

```next.config.ts```
```ts
import { NextConfig } from 'next'
 
const config: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 's3.amazonaws.com',
        port: '',
        pathname: '/my-bucket/**',
        search: '',
      },
    ],
  },
}
 
export default config
```

#### Optimizing fonts

It includes built-in automatic self-hosting for any font file. This means you can optimally load web fonts with no layout shift. To start using next/font, import it from ```next/font/local``` or ```next/font/google```, call it as a function with the appropriate options, and set the className of the element you want to apply the font to. For example:

```app/layout.tsx```
```tsx
import { Geist } from 'next/font/google'
 
const geist = Geist({
  weight: '400',
  subsets: ['latin'],
})
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={geist.className}>
      <body>{children}</body>
    </html>
  )
}
```

#### Local Fonts

```app/layout.tsx```
```tsx
import localFont from 'next/font/local'
 
const myFont = localFont({
  src: './my-font.woff2',
})
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className={myFont.className}>
      <body>{children}</body>
    </html>
  )
}
```

If you want to use multiple files for a single font family, src can be an array:

```app/layout.tsx```
```ts
// ...
const roboto = localFont({
  src: [
    {
      path: './Roboto-Regular.woff2',
      weight: '400',
      style: 'normal',
    },
    {
      path: './Roboto-Italic.woff2',
      weight: '400',
      style: 'italic',
    },
    {
      path: './Roboto-Bold.woff2',
      weight: '700',
      style: 'normal',
    },
    {
      path: './Roboto-BoldItalic.woff2',
      weight: '700',
      style: 'italic',
    },
  ],
})
// ...
```

### CSS Modules - Tailwind CSS (Recommended)

```app/globals.css```
```ts
@import 'tailwindcss';
```

```app/layout.tsx```
```tsx
import type { Metadata } from 'next'
 
// These styles apply to every route in the application
import './globals.css'
 
export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
}
 
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  )
}
```

### Updating Data

You can update data in Next.js using React's Server Functions.

#### Server Components

```app/page.tsx```
```tsx
export default function Page() {
  // Server Action
  async function createPost() {
    'use server'
    // Update data
    // ...
 
  return <></>
}
```

#### Client Components

It's not possible to define Server Functions in Client Components. However, you can invoke them in Client Components by importing them from a file that has the "use server" directive at the top of it:

```app/applications.ts```
```ts
'use server'
 
export async function createPost() {}
```

```app/ui/button.tsx```
```tsx
'use client'
 
import { createPost } from '@/app/actions'
 
export function Button() {
  return <button formAction={createPost}>Create</button>
}
```

#### Invoking Server Functions

There are two main ways you can invoke a Server Function:

1. **Forms** in Server and Client Components
2. **Event Handlers** in Client Components

#### Forms
React extends the HTML ```<form>``` element to allow Server Function to be invoked with the HTML action prop.

```app/ui/form.tsx```
```tsx
import { createPost } from '@/app/actions'
 
export function Form() {
  return (
    <form action={createPost}>
      <input type="text" name="title" />
      <input type="text" name="content" />
      <button type="submit">Create</button>
    </form>
  )
}
```

```app/actions.ts```
```ts
'use server'
 
export async function createPost(formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')
 
  // Update data
  // Revalidate cache
}
```

#### Event Handlers

```app/like-button.tsx```
```tsx
'use client'
 
import { incrementLike } from './actions'
import { useState } from 'react'
 
export default function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes)
 
  return (
    <>
      <p>Total Likes: {likes}</p>
      <button
        onClick={async () => {
          const updatedLikes = await incrementLike()
          setLikes(updatedLikes)
        }}
      >
        Like
      </button>
    </>
  )
}
```

#### Pending State

```app/ui/button.tsx```
```tsx
'use client'
 
import { useActionState } from 'react'
import { createPost } from '@/app/actions'
import { LoadingSpinner } from '@/app/ui/loading-spinner'
 
export function Button() {
  const [state, action, pending] = useActionState(createPost, false)
 
  return (
    <button onClick={async () => action()}>
      {pending ? <LoadingSpinner /> : 'Create Post'}
    </button>
  )
}
```

#### Redirecting

You may want to redirect the user to a different page after performing an update. You can do this by calling redirect within the Server Function:

```app/lib/actions.ts```
```ts
'use server'
 
import { redirect } from 'next/navigation'
 
export async function createPost(formData: FormData) {
  // Update data
  // ...
 
  redirect('/posts')
}
```

### Errors

- Expected Errors
- Uncaught Exceptions

You can use the useActionState hook to manage the state of **Server Functions** and handle expected errors. Avoid using **try/catch** blocks for expected errors. Instead, you can model expected errors as return values, not as thrown exceptions.

```app/actions.ts```
```ts
'use server'
 
export async function createPost(prevState: any, formData: FormData) {
  const title = formData.get('title')
  const content = formData.get('content')
 
  const res = await fetch('https://api.vercel.app/posts', {
    method: 'POST',
    body: { title, content },
  })
  const json = await res.json()
 
  if (!res.ok) {
    return { message: 'Failed to create post' }
  }
}
```

```app/ui/form.ts```
```tsx
'use client'
 
import { useActionState } from 'react'
import { createPost } from '@/app/actions'
 
const initialState = {
  message: '',
}
 
export function Form() {
  const [state, formAction, pending] = useActionState(createPost, initialState)
 
  return (
    <form action={formAction}>
      <label htmlFor="title">Title</label>
      <input type="text" id="title" name="title" required />
      <label htmlFor="content">Content</label>
      <textarea id="content" name="content" required />
      {state?.message && <p aria-live="polite">{state.message}</p>}
      <button disabled={pending}>Create Post</button>
    </form>
  )
}
```

#### Not Found

```app/blog/[slug]/page.tsx```
```tsx
// app/blog/[slug]/page.tsx
import { getPostBySlug } from '@/lib/posts'
 
export default async function PostsPage({ params }: { params: { slug: string } }) {
  const { slug } = await params
  const post = getPostBySlug(slug)
 
  if (!post) {
    notFound()
  }
 
  return <div>{post.title}</div>
}

// app/blog/[slug]/not-found.tsx

export default function NotFound() {
  return <div>404 - Page Not Found</div>
}
```

#### Uncaught Exceptions

Uncaught exceptions are unexpected errors that indicate bugs or issues. Next.js uses error boundaries to handle uncaught exceptions. Error boundaries catch errors in their child components and display a fallback UI instead of the component tree that crashed. Create an error boundary by adding an ```error.tsx``` file inside a route segment and exporting a React component:

```app/dashboard/error.tsx```
```tsx
'use client' // Error boundaries must be Client Components
 
import { useEffect } from 'react'
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error(error)
  }, [error])
 
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </button>
    </div>
  )
}
```

#### Global Errors

You can handle errors in the root layout using the ```global-error.tsx``` file, located in the root app directory, even when leveraging internationalization. Global error UI must define its own ```<html>``` and ```<body>``` tags, since it is replacing the root layout or template when active.

```app/global-error.tsx```
```tsx
'use client' // Error boundaries must be Client Components
 
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    // global-error must include html and body tags
    <html>
      <body>
        <h2>Something went wrong!</h2>
        <button onClick={() => reset()}>Try again</button>
      </body>
    </html>
  )
}
```

...TO_BE_CONTINUED...
https://nextjs.org/docs/app/building-your-application/routing/middleware
