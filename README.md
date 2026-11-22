# MCB Frontend Starter

A clean starting point for **MCB Bank Limited** web apps: sign-in, profile, a sliding navigation
drawer and top bar, plus ready-to-copy page templates and a reusable component kit.

| | |
|---|---|
| Language / build | React 19 · JavaScript (JSX) · Vite 8 |
| Styling | Tailwind CSS v4 - white, gray, MCB blue `#005BAA`, MCB green `#50B848` |
| Routing / data | React Router 8 · TanStack Query 5 · Axios |
| State | Zustand (session, drawer, toasts) |
| Tests | Vitest + Testing Library |

**Data sources**
- **Authentication** uses the real API (`BISP_Backend`): login, current user, logout, change password.
- **Everything else** (menu, home page, notifications, activity, sample lists) comes from JSON files in
  `src/data/` through a small mock layer that behaves like an API. Switching a screen to a real
  endpoint later is a one-line change in its `*.api.js` file.

---

## 1. Quick start

```powershell
npm install
copy .env.example .env
npm run dev                 # http://localhost:5173
```

The dev server forwards `/api/*` to the backend (`VITE_DEV_PROXY_TARGET`, default `http://localhost:8000`):

```powershell
cd ..\BISP_Backend
php artisan serve
```

Test sign-in (seeded users, password `Password@123`): `admin@bisp.test`, `ibmaker@bisp.test`, or an employee code such as `IBM0001`.

| Script | Purpose |
|---|---|
| `npm run dev` | Dev server with hot reload + API proxy |
| `npm run build` | Production build to `dist/` |
| `npm run preview` | Serve the production build |
| `npm run test` | Unit and component tests |
| `npm run validate` | test → build (use in CI) |

---

## 2. What's included

| Screen | Path | Notes |
|---|---|---|
| Login | `/login` | Email or employee code, session-expired message, return to the requested page |
| Home | `/` | Welcome banner, stats, chart, quick links, announcements (`data/home.json`) |
| My profile | `/profile` | Overview · Security (change password, real API) · Activity (`data/activity.json`) |
| List page template | `/templates/list` | Search, filters, sorting, paging, row selection, row actions (`data/employees.json`) |
| Form page template | `/templates/form` | Sectioned form, every field type, validation, simulated save |
| UI components | `/templates/components` | Live gallery of every component with usage hints |

Layout: a white **sliding drawer** (pushes content on desktop, slides over it on mobile, closes on
Esc / backdrop / navigation) and a white **top bar** with notifications (`data/notifications.json`)
and the user menu.

---

## 3. Folder structure

```
src/
├─ main.jsx               Entry: styles + bootstrap + <App/>
├─ app/                   App.jsx, Providers.jsx, router.jsx (all routes), bootstrap.js
├─ api/
│  ├─ client.js           Axios instance: token, 401 handling, api.get/post/upload helpers
│  ├─ errors.js           ApiError - one error shape for the whole app
│  ├─ queryClient.js      TanStack Query setup + global error toasts
│  ├─ mock.js             mockResponse / mockPaginate / mockMutation (JSON behaves like an API)
│  ├─ auth.api.js         REAL endpoints: login, me, logout, change-password
│  └─ content.api.js      JSON-backed data (home, notifications, activity, employees)
├─ data/                  JSON files: navigation, home, notifications, activity, employees
├─ hooks/
│  ├─ queries/            useAuth.js, useContent.js, keys.js
│  ├─ useForm.js          Form state + validation + server (422) errors
│  ├─ useTableState.js    Paging / sorting / filters kept in the URL
│  └─ …                   usePermission, useDownload, useDebounce, useDocumentTitle…
├─ components/
│  ├─ ui/                 Reusable kit: Button, TextField, SelectField, SearchSelect, Modal, FormModal,
│  │                      DataTable, FilterBar, Tabs, Dropdown, Popover, Badge, Card, StatCard,
│  │                      charts, Timeline, Stepper, Icon, Toaster…
│  ├─ layout/             AppLayout, Drawer, Topbar, PageHeader, AuthLayout
│  └─ guards/             RequireAuth, GuestOnly, RequireRole, <Can>
├─ pages/
│  ├─ auth/  home/  profile/  errors/
│  └─ templates/          ListTemplatePage, FormTemplatePage, ComponentsPage  ← copy these
├─ store/                 auth.store (session), ui.store (drawer), toast.store
├─ constants/             routes.js, roles.js, colors.js
├─ config/                env.js, app.config.js (name, login texts…), navigation.js
├─ styles/index.css       Tailwind + MCB colour tokens (the only place for colours)
├─ utils/                 format (dates, PKR, CNIC, IBAN), validators, download, cn()
└─ assets/brand/          Official MCB logos
```

---

## 4. Adding a new page

1. **Copy a template** from `src/pages/templates/` into `src/pages/<area>/MyPage.jsx`.
2. **Data**
   - From JSON: add a file to `src/data/`, a function in `src/api/content.api.js` using `mockResponse` /
     `mockPaginate`, and a hook in `src/hooks/queries/useContent.js`.
   - From a real API: create `src/api/<area>.api.js` with `api.get(...)` calls and hooks in
     `src/hooks/queries/use<Area>.js`.
3. **Route** - in `src/app/router.jsx`: `const MyPage = lazy(() => import('@/pages/<area>/MyPage'))`
   and `{ path: ROUTES.myPage, element: <MyPage /> }`; add the path to `constants/routes.js`.
4. **Menu** - add `{ "label", "path", "icon", "roles"? }` to `src/data/navigation.json`.
5. **Restrict by role** (optional) - wrap the element in `<RequireRole roles={['admin']}>`.

### Moving a JSON screen to a real API
Only the function body changes, the page stays the same:

```js
// src/api/content.api.js
listEmployees: (params) => api.getPaginated('/employees', params),   // was mockPaginate(...)
```

---

## 5. Building blocks cheat-sheet

| Need | Use |
|---|---|
| Form | `const form = useForm({ initialValues, validate: (v) => validateFields(v, SCHEMA) })` + `<TextField {...form.field('email')} />` |
| Rules | `rules.required('Email')`, `rules.email()`, `rules.cnic()`, `rules.mobile()`, `rules.strongPassword()`, `rules.sameAs('password')`, `rules.maxFileSizeMb(2)` |
| Form in a dialog | `<FormModal form={form} onSubmit={save}>`; export the dialog wrapped in `mountWhenOpen()` |
| List | `const table = useTableState({ filterKeys })` + `<FilterBar>` + `<DataTable state={table} meta={data.meta}>` |
| Save + toast | `useApiMutation({ mutationFn, invalidate })` shows the server message and refreshes data |
| Notification | `toast.success('Saved')` / `toast.error(…)` |
| Status pill | `<StatusBadge map={STATUS} value={row.status} />` |
| Icon | `<Icon name="upload" />` - all names are shown on the UI components page |

Errors are shown once: field errors (422) under the field, permission / conflict / network errors as a
toast, anything else at the top of the form. An expired session returns the user to the login page and
back to the same page after sign-in.

---

## 6. Branding & configuration

- Colours: `src/styles/index.css` (`@theme` block). Logos: `src/assets/brand/`.
- App name, login texts, support email: `src/config/app.config.js` and `VITE_APP_NAME` in `.env`.

| Variable | Default | Notes |
|---|---|---|
| `VITE_APP_NAME` | `BISP Account Opening` | Browser title / headings |
| `VITE_API_BASE_URL` | `/api/v1` | Absolute URL in production |
| `VITE_DEV_PROXY_TARGET` | `http://localhost:8000` | Dev server only |
| `VITE_API_TIMEOUT` | `60000` | ms |
| `VITE_ENABLE_QUERY_DEVTOOLS` | `true` | Development only |

## 7. Deployment

- `npm run build` → static files in `dist/`. Serve `index.html` for unknown paths (IIS URL Rewrite / nginx `try_files $uri /index.html`).
- If the frontend and API use different domains, allow the frontend origin in Laravel `config/cors.php`.
- The session token is stored in sessionStorage (cleared when the tab closes).
