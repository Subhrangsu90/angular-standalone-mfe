# 🧩 Standalone MFE Creation Process with Classic Module Federation (Angular)

This guide documents the step-by-step process for setting up a shell and two micro frontends (`mfe1`, `mfe2`) using standalone components with `@angular-architects/module-federation`.

---

## ✅ Step 1: Create Applications

```bash
ng new shell
ng new mfe1
ng new mfe2
```

---

## ✅ Step 2: Add Module Federation Support

```bash
ng add @angular-architects/module-federation@19.0.1 --type host --port 4200
ng add @angular-architects/module-federation@19.0.1 --type remote --port 4201
ng add @angular-architects/module-federation@19.0.1 --type remote --port 4202
```

---

## ✅ Step 3: Create `mfe-manifest` in Shell

**Directory Structure:**

```
shell/
├── src/
│   └── app/
│       └── mfe-manifest/
│           ├── manifest.json
│           └── manifest.local.json
```

**`manifest.local.json`**

```json
{
  "mfe1": "http://localhost:4201/remoteEntry.js",
  "mfe2": "http://localhost:4202/remoteEntry.js"
}
```

Add the manifest folder to assets in `angular.json` under `architect > build > options > assets`:

```json
{
  "glob": "*.json",
  "input": "src/mfe-manifest",
  "output": "./mfe-manifest"
}
```

---

## ✅ Step 4: Add Environment Configuration in Shell

**`src/environments/environment.ts`**

```ts
export const environment = {
  production: false,
  mfeManifest: "manifest.local.json",
};
```

---

## ✅ Step 5: Bootstrap Shell with Federation

**`src/main.ts`**

```ts
import { initFederation } from "@angular-architects/module-federation";
import { environment } from "./environments/environment";

initFederation(`mfe-manifest/${environment.mfeManifest}`)
  .catch((err) => console.error(err))
  .then(() => import("./bootstrap"))
  .catch((err) => console.error(err));
```

---

## ✅ Step 6: Define Routes in Shell

**`src/app/app.routes.ts`**

```ts
import { loadRemoteModule } from "@angular-architects/module-federation";
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "mfe1",
    loadChildren: () => loadRemoteModule("mfe1", "./Routes").then((r) => r.routes),
  },
  {
    path: "mfe2",
    loadChildren: () => loadRemoteModule("mfe2", "./Routes").then((r) => r.routes),
  },
];
```

---

## ✅ Step 7: Expose Routes from Remote Apps

**In `webpack.config.ts` of `mfe1` and `mfe2`:**

```ts
exposes: {
  './Routes': './src/app/app.routes.ts' // or your actual routing file
}
```

Make sure the exposed file (`app.routes.ts`) exists and exports a valid `Routes` array:

```ts
import { Routes } from "@angular/router";
import { SomeComponent } from "./some.component";

export const routes: Routes = [
  {
    path: "",
    component: SomeComponent,
  },
];
```

---

## ✅ Final Notes

- Ensure each MFE is running on its respective port (`4201`, `4202`).
- `loadRemoteModule` uses the manifest to resolve the correct remote entry.
- Each remote should be built and served independently before testing the shell.
