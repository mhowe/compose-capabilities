# Datastore

Shared reference data the whole space can draw from.

## What it contains

- **Kapp** `datastore` — hosts datastore forms.
- **Seeded datastores** — common reference lists (e.g., US states, ISO countries, priority values) installed by default, each an optional add.

## When to use it

If the same list of values appears in more than one kapp (50 US states, 100 departments, severity levels), move it into a datastore here. Other kapps query by name instead of maintaining their own copy.

## Files

- `manifest.json` — capability manifest.
- `kapp.json` — kapp export (stub).
