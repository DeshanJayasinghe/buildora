# Optional Blender Render Worker Setup

This is not an MVP blocker.

Use it only when users need server-generated photoreal images/video.

## Architecture

```text
Render Request
 ↓
Credit reservation
 ↓
Temporal workflow
 ↓
Render worker
 ↓
Blender
 ↓
Upload output
 ↓
Settle credits
```

## Local setup

Install Blender using the official macOS package or Homebrew cask if appropriate.

Verify headless execution using Blender's current CLI.

## Worker input

Do not pass arbitrary executable scripts from users.

Pass a validated render specification:

```text
model asset
camera
lighting preset
quality
resolution
material references
```

## Storage

Input/output through object storage.

## Security

Run render workers in isolated containers/VMs.

Treat imported 3D files as untrusted.

## Cost control

Photoreal rendering consumes render credits.

Interactive Three.js viewing consumes no render credit.

## Start small

First supported render:

```text
one exterior still
fixed quality preset
```

Do not begin with 4K video walkthroughs.
