# Optional Open Design Alliance SDK Setup

ODA is intended for professional DWG/DXF and potentially other commercial CAD/BIM interoperability.

This requires licensing review.

## Do not install for MVP unless required

Initial BuildWise import can begin with:

- PDF
- images
- DXF through simpler tooling
- IFC.

## Before implementation

1. Confirm exact ODA product required.
2. Review commercial licensing.
3. Confirm server/web redistribution terms.
4. Confirm supported OS/architectures.
5. Obtain SDK through official ODA access.
6. Create a spike branch.

## Adapter boundary

Create:

```text
CadImportProvider
```

Possible implementations:

```text
BasicDxfImportProvider
OdaCadImportProvider
```

## Output

ODA/importer should return BuildWise-neutral intermediate geometry/metadata.

It must not become the domain model.

## Test corpus

Keep legally usable test drawings:

- basic lines
- walls/layers
- blocks
- dimensions
- multiple units
- malformed file.

## Security

Run complex CAD parsing in isolated workers.

Never trust uploaded DWG/DXF input.
