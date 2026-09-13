# Browser BIM Setup — That Open, Fragments and IFC

This is not required for the first Building Model → 2D → 3D slice.

Add it when IFC/BIM viewing/import is scheduled.

## 1. Packages

In the web/BIM integration package, install the current compatible releases:

```bash
pnpm add @thatopen/components @thatopen/components-front @thatopen/fragments
```

The ecosystem also relies on Three.js and IFC tooling. Follow the current That Open compatibility guidance when selecting matching versions.

## 2. Keep BIM behind an adapter

Suggested package:

```text
packages/bim-web/
```

Do not mix That Open-specific classes into the Buildora AI domain model.

## 3. First capability

Start with:

```text
Load IFC
 ↓
View IFC
 ↓
Select element
 ↓
Read IFC properties
```

Do not start with editable IFC authoring.

## 4. Second capability

Map a small supported IFC subset into Buildora AI:

- IfcBuildingStorey → Level
- IfcWall / IfcWallStandardCase → Wall
- IfcDoor → Door
- IfcWindow → Window
- IfcSlab → Slab
- IfcSpace → Room/Space

## 5. Fragments

Use Fragments as an optimized browser BIM representation where it provides measurable value.

Fragments are not the authoritative Buildora AI project database.

## 6. Version compatibility

That Open packages evolve quickly.

Pin compatible versions in the lockfile and upgrade intentionally.

Before upgrading:

```text
- read release notes
- run IFC fixtures
- run selection/property tests
- run browser regression
```

## 7. Security

Treat IFC files as untrusted input.

Perform file-size limits and server-side validation where appropriate.
