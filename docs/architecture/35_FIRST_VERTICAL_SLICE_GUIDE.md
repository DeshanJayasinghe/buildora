# First BuildWise Vertical Slice

This should be the first meaningful product milestone.

## Goal

```text
Create Project
 ↓
Create Ground Floor
 ↓
Draw 4 Walls
 ↓
Create Room
 ↓
Add Door
 ↓
Add Window
 ↓
Switch to 3D
 ↓
Save
 ↓
Reload
 ↓
Same result
```

No AI.

No QS.

No IFC.

## Step 1 — Project

Implement:

- organization
- project
- project permission
- create/open project.

## Step 2 — Units package

Implement canonical:

```text
LengthMm
```

with conversion helpers.

Do not allow arbitrary metres/feet deep inside geometry.

## Step 3 — Building Model

Entities:

- Project model root
- Level
- Wall
- Door
- Window
- Space later if room detection not ready

Commands:

- CreateLevel
- CreateWall
- UpdateWall
- DeleteWall
- CreateDoor
- CreateWindow

## Step 4 — Version

Every committed command changes model version.

## Step 5 — Persistence

Save parametric model data.

Reload from API and reconstruct identical model.

## Step 6 — 2D

Render wall.

Then add:

- select
- pan
- zoom
- create wall.

## Step 7 — 3D

Render wall mesh from same Wall object.

## Step 8 — Openings

Add door/window visuals.

Keep hole geometry simple initially if needed, but do not falsify the Building Model.

## Step 9 — Synchronization

Move wall in 2D:

```text
command
 ↓
Building Model
 ↓
2D updates
 ↓
3D updates
```

## Step 10 — Test

Automate:

- create fixture
- serialize
- deserialize
- 2D adapter
- 3D adapter
- wall update
- reload.

## Acceptance

The slice is complete when closing/reopening the browser does not change geometry and there is only one authoritative copy of the model.
