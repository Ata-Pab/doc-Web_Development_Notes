### Error and Performance Monitoring for Web & Mobile Apps

[sentry.io](sentry.io): Fix frontend errors, API Route errors and analyze performance of the apps.

Tip: Do not use virtualized list components like FlatList (components that render only the viewvable items) in ScrollView components.

Tip: Use Next.js built-in Image component to optimize image downloading times (decrease the wait time)
```tsx
<Image
 src = {post.image}
 alt = {post.title}
 width = {500}
 height = {500}
/>
```

Tip: Use JS built-in date library instead of third party libraries like ```moment```.
```tsx
{ new Intl.DateTimeFormat("en-US", {
    year: "numeric",
    month: "long",
    day: "numeric",
}).format(post.createdAt)}
```

Tip: HTML data/inputs like ```<html><script fetch...``` can be written by malicious users to the areas like ```TextArea```, ```Input``` components and these inputs can be processed by browser. Always use ```zod``` to protect from it. (Optional) Use Arcjet Shield block, OWAPS Core Rule Set (SQL Injection, XSS, PHP/Java Code Injection, HTTProxy)

### Cross Site Scripting (XSS)

- Certainly check if the user is authenticated or not. Use RPC functions ot session control in server requests.
- Use ```zod``` form validation in Server side actions and other critical data CRUD operations
- (Optional) Use [Arcjet](https://arcjet.com/) to provide bot detection, rate limiting, e-mail verification, data redaction.
- Use ```import "server-only"``` command for server critical data. This provides that the code should be accessed by only in Server-side. Don't confuse with the ```"use server"``` tag. It's for Server-side actions.
- Always keep scret/critical constant data in ```.env``` file and access them via ```process.env.[constant]```.
- Use Arcjet Rate limiting in API requests (Fixed windows, token bucket), bot detection, shield for top 10 web application security risks (Free up to 500K requests, 5 rules). 

