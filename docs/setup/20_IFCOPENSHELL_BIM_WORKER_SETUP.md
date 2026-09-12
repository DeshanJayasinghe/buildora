# IfcOpenShell BIM Worker Setup

IfcOpenShell belongs in the Python/BIM worker, not in the browser domain model.

## 1. Create BIM worker

If it is time to split the Python worker:

```bash
mkdir -p apps/bim-worker
cd apps/bim-worker
uv init --bare
```

## 2. Pin compatible Python

IfcOpenShell wheel/package availability can vary by Python version/platform.

Before choosing Python 3.13 or 3.12, verify current IfcOpenShell package compatibility.

Record the chosen version in:

```text
.python-version
pyproject.toml
```

## 3. Install

Use the current official IfcOpenShell installation method/package for your platform.

Do not force source compilation unless needed.

## 4. Initial worker responsibilities

- inspect IFC metadata
- list storeys
- list supported elements
- extract properties
- validate import
- produce mapping payloads
- export supported BuildWise elements to IFC later

## 5. API boundary

Return BuildWise-neutral DTOs.

Bad:

```text
UI receives arbitrary IfcOpenShell Python objects
```

Good:

```json
{
  "externalId": "IFC_GUID",
  "ifcClass": "IfcWall",
  "name": "External Wall",
  "levelExternalId": "..."
}
```

## 6. Tests

Keep small IFC fixtures in the fixture catalogue.

Test:

- valid simple file
- malformed file
- missing storey
- unsupported entities
- multiple levels
- unit conversions.

## 7. Do not make IFC the source of truth

Once imported, authoritative supported elements live in the BuildWise model.
