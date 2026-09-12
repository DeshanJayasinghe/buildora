# Optional OpenCascade Setup

OpenCascade is a later-stage native CAD/geometry capability.

Do not add it while basic wall/room/slab geometry can be reliably handled by TypeScript/Rust.

## Appropriate use cases

- B-Rep solids
- advanced Boolean operations
- sweeps
- complex roof geometry
- robust CAD conversions
- section solids
- geometry repair.

## Integration boundary

Preferred:

```text
BuildWise geometry service interface
        ↓
OpenCascade adapter/service
```

Do not spread OpenCascade-specific types across the application.

## Platform setup

Use OpenCascade's current official packages/source distribution for the deployment platform.

Native dependency setup changes over time, so do not hardcode stale Homebrew/CMake instructions into the core repository without testing them.

## Service language

Likely:

- C++ service/library
- or carefully wrapped native/WASM integration.

Choose only after a real geometry requirement exists.

## Testing

Create geometry fixtures before integrating.

Validate:

- tolerance
- unit conversion
- Boolean robustness
- determinism
- serialization.
