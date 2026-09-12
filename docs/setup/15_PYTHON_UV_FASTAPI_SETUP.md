# Python + uv + FastAPI Worker Setup

Python is used for computer vision, BIM processing, ML, and document pipelines.

Target:

```text
apps/ai-worker
apps/bim-worker
```

Do not create both immediately unless both are needed. Start with one Python worker project and split later.

## 1. Install uv

macOS:

```bash
brew install uv
```

or official installer:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Verify:

```bash
uv --version
```

## 2. Install/pin Python

For broad scientific compatibility, use a stable project Python version rather than automatically chasing the newest major/minor.

Example:

```bash
uv python install 3.13
```

In the Python project:

```bash
uv python pin 3.13
```

If a required ML/BIM package is not yet compatible with 3.13, intentionally pin 3.12 instead. Record the decision.

## 3. Create project

```bash
mkdir -p apps/ai-worker
cd apps/ai-worker
uv init --bare
```

## 4. Add FastAPI

```bash
uv add "fastapi[standard]"
```

Development packages:

```bash
uv add --dev pytest ruff pyright
```

## 5. Create app

Example:

```text
apps/ai-worker/
├── pyproject.toml
├── uv.lock
└── src/
    └── buildwise_ai/
        ├── __init__.py
        └── main.py
```

`main.py`:

```python
from fastapi import FastAPI

app = FastAPI(title="BuildWise AI Worker")

@app.get("/health")
def health():
    return {"status": "ok"}
```

## 6. Run

Depending on module path:

```bash
uv run fastapi dev src/buildwise_ai/main.py
```

## 7. Quality

```bash
uv run ruff check .
uv run pyright
uv run pytest
```

## 8. Add ML only when needed

Later:

```bash
uv add numpy opencv-python
```

Add PyTorch/ONNX following the platform-specific official install path when the plan-recognition milestone starts.

Do not install large GPU dependencies on day one.

## 9. Internal service security

Python worker endpoints should not be public internet APIs.

Production traffic should flow through controlled internal services/workflows.
