# Rust + WebAssembly Geometry Setup

Rust/WASM is optional until TypeScript geometry becomes a measured bottleneck or a complex algorithm clearly benefits from it.

Target:

```text
native/geometry-wasm
```

## 1. Install Rust

Recommended rustup installer:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Restart shell.

Verify:

```bash
rustc --version
cargo --version
rustup --version
```

## 2. Install wasm-pack

Use the current wasm-pack installation method for your platform.

Common Cargo option:

```bash
cargo install wasm-pack
```

Verify:

```bash
wasm-pack --version
```

## 3. Create library

```bash
mkdir -p native
cd native
cargo new --lib geometry-wasm
```

## 4. Cargo.toml

Add:

```toml
[lib]
crate-type = ["cdylib"]

[dependencies]
wasm-bindgen = "0.2"
```

Do not pin a stale patch version copied from documentation; let Cargo lock the compatible release.

## 5. Add wasm target if required

```bash
rustup target add wasm32-unknown-unknown
```

## 6. Build for web/bundler integration

Example:

```bash
cd native/geometry-wasm
wasm-pack build --target bundler
```

or use the target most appropriate to the current Next.js bundling integration.

## 7. First function

Start with a trivial deterministic geometry operation, not a full polygon engine.

Example candidates:

- distance between 2D points
- line intersection

## 8. Test Rust separately

```bash
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test
```

## 9. Integration rule

Expose a narrow wrapper in TypeScript.

Do not leak raw WASM memory/API details across the entire application.

## 10. When not to use WASM

Keep TypeScript if:

- operation is fast enough,
- implementation is simpler,
- there is no user-visible performance issue.

Profile before migrating code.
