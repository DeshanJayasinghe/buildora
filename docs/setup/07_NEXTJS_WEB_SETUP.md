# Next.js Web Application Setup

Target:

```text
apps/web
```

## 1. Scaffold

From repository root:

```bash
pnpm create next-app apps/web --ts --eslint --tailwind --app --src-dir --import-alias "@/*" --use-pnpm
```

If the CLI initializes a nested Git repository, remove only that nested `.git` folder:

```bash
rm -rf apps/web/.git
```

Do not remove the root repository `.git`.

## 2. Verify

```bash
pnpm --filter web dev
```

Open:

```text
http://localhost:3000
```

Then:

```bash
pnpm --filter web build
```

## 3. Recommended web dependencies

Add only when needed:

```bash
pnpm --filter web add zod
pnpm --filter web add @tanstack/react-query
pnpm --filter web add zustand
pnpm --filter web add react-hook-form @hookform/resolvers
```

Usage rules:

- TanStack Query: server/API state.
- Zustand: ephemeral UI/editor state only.
- Building Model domain state must not be duplicated into a general Zustand store.
- React Hook Form: ordinary forms.
- Zod: boundary/input schemas.

## 4. Folder baseline

```text
apps/web/src/
├── app/
├── components/
├── features/
├── lib/
└── providers/
```

Keep CAD and 3D engines out of general page components. They belong in workspace packages.

## 5. App routes

Initial:

```text
/
 /sign-in
 /sign-up
 /dashboard
 /projects
 /projects/[projectId]
```

Later:

```text
/projects/[projectId]/2d
/projects/[projectId]/3d
/projects/[projectId]/quantities
/projects/[projectId]/boq
/projects/[projectId]/cost
```

## 6. Health page

Create a basic application page and ensure production build succeeds before adding auth.

## 7. Next.js rule

Use Server Components by default for ordinary dashboard pages where appropriate.

CAD/3D workspaces will necessarily use Client Components around interactive browser engines.

Do not make the entire application client-rendered simply because the editor is interactive.
