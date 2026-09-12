# Design System, Tailwind, Radix and Storybook Setup

Target package:

```text
packages/design-system
```

## 1. Use the Next.js Tailwind setup

Current Next.js scaffolding can create Tailwind automatically. Keep the generated configuration compatible with current Tailwind conventions.

## 2. Create design-system package

```bash
mkdir -p packages/design-system/src
cd packages/design-system
pnpm init
```

Set package name:

```json
{
  "name": "@buildwise/design-system",
  "private": true,
  "version": "0.0.0"
}
```

## 3. Add UI primitives selectively

Do not install every Radix package up front.

Example:

```bash
pnpm add @radix-ui/react-dialog
pnpm add @radix-ui/react-dropdown-menu
pnpm add @radix-ui/react-tooltip
pnpm add @radix-ui/react-tabs
```

Add others only when a real component needs them.

## 4. Storybook

Run from the web/design-system environment after the basic web app exists:

```bash
pnpm create storybook@latest
```

Choose the Next.js/React integration detected by the CLI.

If you prefer Storybook to live with the web app initially, that is acceptable for a solo MVP. Move stories into a dedicated package only when shared components justify it.

## 5. Initial stories

Create stories for:

- Button
- Input
- Select
- Checkbox
- Tabs
- Badge
- Card
- Table
- Dialog
- Drawer
- Tooltip
- AI action card
- Confidence badge
- CAD toolbar button
- Property row
- Empty state
- Error state
- Loading state

## 6. Tokens

Define tokens for:

- background
- surface
- foreground
- muted
- border
- primary
- success
- warning
- destructive
- AI/accent
- spacing
- radius
- typography
- elevation

Do not create page-specific arbitrary colors when a token should exist.

## 7. Verification

```bash
pnpm storybook
```

and:

```bash
pnpm build-storybook
```

The design system is considered ready for application work once core components can be reviewed independently.
