# Vue Framework — Vue 3 + TypeScript Admin Starter

**English** | [简体中文](README.zh-CN.md)

Vue Framework is a Vue 3 admin dashboard starter built with TypeScript, Vite, Ant Design Vue, Pinia, and Vue Router. It brings together single-file components, TSX examples, role-based route guards, reusable Composition API hooks, and Docker + Nginx deployment configuration.

Use it as a starting point for internal tools, admin interfaces, or learning Vue 3 application structure. Login and admin data are local demonstrations that you can replace with your own backend.

## Contents

- [Features](#features)
- [Quick start](#quick-start)
- [Demo login and routes](#demo-login-and-routes)
- [Project structure](#project-structure)
- [Customization](#customization)
- [Build and deployment](#build-and-deployment)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Vue 3 + TypeScript:** Composition API and `<script setup>` components.
- **Vite + TSX:** Vue SFC and JSX/TSX plugins, plus a legacy-browser plugin with IE 11 excluded from its targets.
- **Ant Design Vue:** UI components used in the login and TSX examples.
- **Pinia:** Authentication state with token and role restoration from `localStorage`.
- **Vue Router:** Hash routing, login guards, and `admin` role restrictions.
- **Admin page examples:** Dashboard, user management, system settings, and searchable logs.
- **React-style Vue hooks:** `useState`, `useSetState`, `useRef`, `useEffect`, `useLayoutEffect`, and `useEffectOnce`.
- **Axios request module:** Base URL, timeout, and interceptor scaffolding.
- **Static deployment:** Docker build stages and Nginx hosting with gzip configuration.

## Quick start

Use Node.js **20.19+** on the Node 20 line, or a compatible newer version. **pnpm 8** matches the committed lockfile format. Docker and CI currently use Node 20.

```bash
git clone https://github.com/Fullsize/vue-framework.git
cd vue-framework
npm install --global pnpm@8
pnpm install
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). If port 3000 is occupied, check Vite's terminal output for the selected port.

| Command | Purpose |
| --- | --- |
| `pnpm dev` | Start the development server |
| `pnpm build` | Run `vue-tsc -b`, then build into `dist/` |
| `pnpm preview` | Preview the built application locally |

The `lint` script is currently empty, and no automated test script is configured.

## Demo login and routes

Unauthenticated visits to protected pages redirect to `/login`. URLs use hash history, for example `http://localhost:3000/#/admin`.

The login form starts with username `abc` and password `123`, which produces the `user` role. Change the username to **`abc123`** to try the `admin` role. The password is not validated: a demo token is created by concatenating the username and password and stored with the role in `localStorage`.

| Route | Page | Access |
| --- | --- | --- |
| `/login` | Demo login | Public |
| `/` | Home example | Signed in |
| `/test` | TSX and state-hook example | Signed in |
| `/proxy` | Proxy example | Signed in |
| `/admin` | Admin dashboard | `admin` |
| `/admin/users` | User management example | `admin` |
| `/admin/settings` | System settings example | `admin` |
| `/admin/logs` | Log search and filtering | `admin` |
| `/forbidden` | Access-denied page | Signed in |

This login flow demonstrates client-side navigation. Replace it with backend authentication and server-side authorization for real use. Admin pages use local sample data; some actions, including saving settings and exporting logs, are placeholders. The application interface currently uses Chinese text; the README language switch changes documentation only.

## Project structure

```text
vue-framework/
├── .github/workflows/     # Build and GitHub Pages workflows
├── public/               # Public static assets
├── src/
│   ├── assets/images/    # Imported images
│   ├── components/       # Shared components
│   ├── hooks/            # Vue state and lifecycle helpers
│   ├── layout/           # App shell and RouterView
│   ├── pages/            # Login, admin, and example pages
│   ├── routes/           # Routes and navigation guards
│   ├── service/          # Axios instance and interceptors
│   ├── store/            # Pinia stores
│   ├── main.ts           # App initialization
│   └── style.css         # Global styles
├── Dockerfile            # Node build and Nginx runtime
├── nginx.conf            # Static hosting and gzip
└── vite.config.ts        # Plugins, aliases, port, asset base
```

## Customization

### Add a page

Create a component under `src/pages/`, then import it in [src/routes/base.ts](src/routes/base.ts) and add a route to the exported array:

```ts
import Reports from '@/pages/Reports.vue'

// Add to the route array:
{
  path: '/reports',
  component: Reports,
  meta: {
    requiresAuth: true,
    requireRole: ['admin'],
  },
}
```

Omit `requireRole` to allow any signed-in user. Guards live in [src/routes/index.ts](src/routes/index.ts); authentication state lives in [src/store/auth.ts](src/store/auth.ts).

### Write TSX with Vue hooks

Both `.vue` and `.tsx` components are supported. The `@` alias points to `src/`; `@images` points to `src/assets/images/`.

```tsx
import { defineComponent } from 'vue'
import { useState } from '@/hooks'

export default defineComponent({
  setup() {
    const [count, setCount] = useState(0)
    return () => (
      <button onClick={() => setCount(previous => previous + 1)}>
        Count: {count.value}
      </button>
    )
  },
})
```

The hooks use Vue refs, watchers, and lifecycle APIs. Their names are inspired by React; behavior follows the local Vue implementations. See [src/hooks/](src/hooks/) and [src/pages/Test/index.tsx](src/pages/Test/index.tsx).

### Connect a backend

Update [src/service/request.ts](src/service/request.ts) with your API URL, timeout, headers, and error handling. Its current `https://some-domain.com/api/` URL is a placeholder. Integrate your login endpoint in [src/pages/Login.vue](src/pages/Login.vue), then replace local admin data with API responses.

## Build and deployment

### Static hosting

```bash
pnpm build
pnpm preview
```

Deploy the contents of `dist/` to a static host. Vite uses `base: './'` for relative asset URLs. Hash routing handles navigation after `#` in the browser.

A GitHub Pages workflow is included for pushes to `master` and manual runs. To use it, enable GitHub Pages in repository settings and choose **GitHub Actions** as the source.

### Docker

With Docker and BuildKit available, run from the repository root:

```bash
docker build -t vue-framework .
docker run --rm -p 8080:80 vue-framework
```

Open [http://localhost:8080](http://localhost:8080). The image builds with Node 20 and serves the result with Nginx. The Dockerfile currently installs pnpm through `https://registry.npmmirror.com`; adjust the registry for your environment if needed.

## Contributing

[Open an issue](https://github.com/Fullsize/vue-framework/issues) for bugs or feature proposals, or submit a pull request with a focused change. Include reproduction steps for bugs and run `pnpm build` before submitting code changes.

## License

Licensed under the [Apache License 2.0](LICENSE).
