# First QS + BOQ + Cost Vertical Slice

Do only after the first 2D/3D Building Model slice is stable.

## Fixture

```text
4 walls
1 door
1 window
1 floor slab
```

## Step 1 — Geometry-derived quantity

For each wall:

```text
gross area = length × height
opening area = sum(door/window opening areas)
net area = gross area - opening area
```

## Step 2 — Unit test

Use known numbers.

Example:

```text
wall length = 5000 mm
wall height = 2700 mm
opening = 1000 × 2100 mm

gross = 13.5 m²
opening = 2.1 m²
net = 11.4 m²
```

## Step 3 — Material

Create tiny material table:

- concrete block
- cement
- sand
- plaster
- primer
- paint.

## Step 4 — Assembly

Create:

```text
EXT-BLOCK-200
```

with per-m² recipe quantities.

Use deliberately simple verified rules for the first fixture.

## Step 5 — QS output

Return:

```text
element
quantity type
quantity
unit
calculation trace
assembly source
```

## Step 6 — BOQ

Create a minimal BOQ section:

```text
Masonry
Finishes
```

## Step 7 — Rates

Use a static test rate provider.

Do not integrate commercial pricing yet.

## Step 8 — Cost

Use decimal-safe arithmetic.

Store money with currency and exact decimal values.

## Step 9 — UI

Create:

- Quantities page
- BOQ page
- Cost page

All should link back to the contributing elements.

## Step 10 — Change propagation

Move a wall.

Expected:

```text
model version changes
 ↓
quantity invalidated
 ↓
quantity recalculated
 ↓
BOQ recalculated
 ↓
cost recalculated
```

## Acceptance

A golden fixture must always calculate the same quantity and cost given the same model/rates.
