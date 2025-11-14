## Vite + NodeJS + React + TailwindCSS + Shadcn/ui + TypeScript
```bash
npm create vite@latest hr_management_dashboard --template react-ts

npm install

npm install tailwindcss @tailwindcss/vite
```

> [TailwindCSS Installation Guide](https://tailwindcss.com/docs/installation/using-vite)

### vite.config.ts

```ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'
export default defineConfig({
  plugins: [
    tailwindcss(),
  ],
})
```

### shadcn/ui Library Istallation
Add compiler options to the ```tsconfig.json```
```js
{
  "files": [],
  "references": [
    {
      "path": "./tsconfig.app.json"
    },
    {
      "path": "./tsconfig.node.json"
    }
  ],
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Add compiler options to the ```tsconfig.app.json```
```js
{
  "compilerOptions": {
    // ...
    "baseUrl": ".",
    "paths": {
      "@/*": [
        "./src/*"
      ]
    }
    // ...
  }
}
```

```bash
npm install -D @types/node
```

IF you use ```Tailwind v4```:
```bash
npx shadcn@canary init
```

Update ```vite.config.ts``` file as below:

```js

import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";
import path from "path";

// https://vite.dev/config/
export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});

```