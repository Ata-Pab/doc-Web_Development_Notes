## Hosting Multiple Projects on a single domain (Using Roots)

<b>React Project Names</b>: ```menupier_menu```, ```menucreatorweb```<br/>
<b>Firebase Project</b>: ```MenuPier```<br/>
<b>Linked WebApp with MenuPier Project</b>: ```menu_pier```<br/>
<b>Linked Firebase Hosting Sites with menu_pier app</b>: ```menupier-4868b```<br/>
<b>Linked Domains for Hosting Sites</b>: ```menupier-4868b.web.app```, ```menupier-4868b.firebaseapp.com```, ```menupier.com```<br/>
<b>Linked Roots for Domains</b>: ```.../menu``` and ```.../enterprise```, e.g. "https://menupier.com/menu/", "https://menupier.com/enterprise/".

### Do before Build

* If you explicitly use ```Router``` in the project, set "Router basename" which will be the root for the domain before build.

App.tsx in ```menupier_menu```
```ts
...
function App() {
...
    return (
    <Router basename="/menu">
    ...
    </Router>
    );
...
}
```    

App.tsx in ```menucreatorweb```
```ts
...
function App() {
...
    return (
    <Router basename="/enterprise">
    ...
    </Router>
    );
...
}
```

* If you use <b>vite</b> to create/build project, add ```base``` attribute to the ```defineConfig``` in <b>```vite.config.ts```</b>. 

vite.config.ts in ```menupier_menu```
```ts
export default defineConfig({
  base: "/menu/",
  plugins: [react()],
})
```

vite.config.ts in ```menucreatorweb```
```ts
export default defineConfig({
  base: "/enterprise/",
  plugins: [react()],
})
```

* If you use <b>i18n</b> internationalization library in the project, modify ```backend``` prop of the ```i18n.init``` method. 

i18n.ts in ```menupier_menu```
```ts
...
i18n
  .use(HttpBackend) // Load translations using http (e.g., from /public/locales)
  .use(LanguageDetector) // Detect the user's language
  .use(initReactI18next) // Bind with React
  .init({
    fallbackLng: "tr", // Default language
    debug: true,
    interpolation: {
      escapeValue: false, // React already escapes values to prevent XSS
    },
    backend: {
      loadPath: "/menu/locales/{{lng}}/translation.json", 
      // Previous path: "/locales/{{lng}}/translation.json"
    },
  });

export default i18n;
```

* <b>Build both of the projects.</b>

* <b>Initialize Firebase Hosting and set necessary configs in one of these projects</b>.

### Prepare Deploy Folder 

After built all projects, move ```dist``` folders into another folder which the deploy process will be in.

Folder Strucutre:
```bash
/Menupier_Deploy/
├── firebase.json
├── .firebaserc
├── dist/
│   ├── menu/
│   │   ├── assets/
│   │   ├── locales/
│   │   ├── index.html
│   ├── enterprise/
│   │   ├── assets/
│   │   ├── logo_icon.png/
│   │   ├── index.html
```

### Prepare firebase.json file

firebase.json
```json
{
  "hosting": 
    {
      "public": "dist",
      "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
      "rewrites": [
        {
          "source": "/menu/**",
          "destination": "/menu/index.html"
        },
        {
          "source": "/enterprise/**",
          "destination": "/enterprise/index.html"
        }
      ]
    }
}
```

### Deploy Projects


```bash
firebase deploy
```

## Extra

* If you get "<i>Info: The current domain is not authorized for OAuth operations. This will prevent signInWithPopup, signInWithRedirect, linkWithPopup and linkWithRedirect from working. Add your domain (menupier.com) to the OAuth redirect domains list in the Firebase console -> Authentication -> Settings -> Authorized domains tab.</i>"
error, <b>add custom domain to the Firebase project Authentication</b>.