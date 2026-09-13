# Three.js 3D Engine Setup

Target:

```text
packages/engine-3d
```

## 1. Create package

```bash
mkdir -p packages/engine-3d/src
cd packages/engine-3d
pnpm init
```

Set name:

```json
{
  "name": "@buildora/engine-3d",
  "private": true,
  "version": "0.0.0"
}
```

## 2. Install Three.js

```bash
pnpm add three
```

If your current Three.js release requires external TypeScript declarations in your toolchain, add the current compatible `@types/three`; otherwise use the package-provided/current recommended typings.

## 3. Internal structure

```text
src/
├── scene/
├── camera/
├── rendering/
├── adapters/
├── selection/
├── materials/
├── clipping/
└── index.ts
```

## 4. Renderer adapter

Create:

```text
BuildingModelWall
        ↓
WallMeshAdapter
        ↓
THREE.Mesh
```

Do not let Three.js objects become persistent domain entities.

## 5. First fixture

One wall:

```text
length = 5000 mm
height = 2700 mm
thickness = 200 mm
```

Render it at the correct scale.

Choose a scene-unit mapping and document it. A simple starting convention is:

```text
1000 model mm = 1 Three.js world unit
```

while preserving canonical domain values in millimetres.

## 6. Camera/navigation

Add in order:

1. orbit
2. fit-to-model
3. top/isometric presets
4. selection
5. floor isolation
6. walkthrough

## 7. WebGPU

Do not require WebGPU for MVP.

Start with a stable renderer path and add WebGPU behind capability detection when the engine is mature.

## 8. Test

Test:

- mesh dimensions,
- transform,
- disposal,
- update after wall command,
- object-to-domain ID mapping.

Memory/resource disposal is mandatory when regenerating scenes.
