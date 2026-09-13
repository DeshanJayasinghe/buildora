# PixiJS 2D Editor Setup

Target package:

```text
packages/cad-2d
```

PixiJS is the renderer, not the Building Model.

## 1. Create package

```bash
mkdir -p packages/cad-2d/src
cd packages/cad-2d
pnpm init
```

Set:

```json
{
  "name": "@buildora/cad-2d",
  "private": true,
  "version": "0.0.0"
}
```

## 2. Install PixiJS

```bash
pnpm add pixi.js
```

## 3. Dependencies

The package may depend on:

```text
@buildora/building-model
@buildora/units
```

It must not become the owner of persistent Building Model state.

## 4. Initial internal structure

```text
src/
├── editor/
├── rendering/
├── camera/
├── selection/
├── tools/
├── snapping/
└── index.ts
```

## 5. First milestone

Render a static Wall entity from a fixture.

Do not implement drawing tools before the render adapter can render domain elements deterministically.

## 6. Second milestone

Add:

- pan
- zoom
- select

## 7. Third milestone

Add Wall drawing command.

The tool should generate a domain command such as:

```text
CreateWall
```

It should not directly append arbitrary graphics into persistent editor state.

## 8. Coordinate systems

Define explicitly:

- model coordinates: millimetres
- screen coordinates: pixels
- camera transform: model ↔ screen

Never store screen pixels as construction geometry.

## 9. Verification

Use a fixed fixture and test:

- camera conversion
- rendered bounds
- selection hit
- wall command output.

Keep screenshot regression for later.
