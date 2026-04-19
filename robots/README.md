# Robots

Time-based workflow execution — the scheduling/automation layer for the space.

## What it contains

- **Kapp** `robots` — hosts robot definitions (name, schedule, workflow to run).
- **Task handler** — cron-like tick that fires scheduled workflows.
- **Default robots** — common patterns (daily digest, SLA warning) that admins can enable or disable out of the box.

## Files

- `manifest.json` — capability manifest.
- `kapp.json` — kapp export (stub).
