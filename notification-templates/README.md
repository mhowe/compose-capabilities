# Notification Templates

Reusable notification templates for email, SMS, and Slack.

## What it contains

- **Kapp** `notification-templates` — hosts template records and related forms.
- **Task handler** `notification-send` — dispatches a rendered notification to the channel declared on the template.
- **Workflows** — hooks that resolve template references at runtime, merge values, and call the task handler.

## How other kapps use it

Other kapps reference a template by slug and pass merge values. The workflow looks up the template in this kapp, renders it, and dispatches. Installing this capability gives every other kapp on the space a single, consistent way to send notifications without each kapp rolling its own.

## Files

- `manifest.json` — capability manifest; read by the Compose Portal installer.
- `kapp.json` — kapp export (currently a stub; replace with a real export in Phase 4+).
