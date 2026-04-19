# Compose Capabilities

A capability registry for [Compose Portal](https://github.com/your-org/compose-portal). This repo hosts the list of installable capabilities (kapps + their supporting forms, task handlers, workflows, and datastores) that Compose Portal can install into a Kinetic Platform space.

This particular repo is a **sample / template** — clone it, push to your own GitHub org, enable Pages, and you have a working registry. Add or remove capability folders as your bundle grows.

## Hosting with GitHub Pages

1. Create a new public repo on GitHub (private repos can't serve Pages on the free tier).
2. Copy everything in this directory into the repo and push to `main`.
3. In the repo's **Settings → Pages**:
   - **Source**: *Deploy from a branch*
   - **Branch**: `main`, folder: `/ (root)`
   - Save.
4. Wait ~1 minute for the first build. GitHub will show a banner with your URL once it's live — something like `https://<your-github-user>.github.io/<repo-name>/`.
5. Verify by fetching the registry index:
   ```
   curl https://<your-github-user>.github.io/<repo-name>/index.json
   ```
   If you get the JSON back, you're done. Give Compose Portal that `index.json` URL as your registry (see Configuration below).

GitHub Pages serves static files with `Access-Control-Allow-Origin: *` by default, so Compose Portal can fetch from any origin — including `http://localhost:3000` in dev.

## Configuring Compose Portal to use this registry

Compose Portal reads registry URLs from a space attribute named **`Capability Registry URLs`**. The attribute allows multiple values; the portal fetches each, merges, and de-duplicates.

1. In Compose Portal → **Space Settings → Bundle Setup**, make sure `Capability Registry URLs` shows **Defined**. If it's missing, click **Deploy**.
2. In the Kinetic admin console, open the space attribute and add your registry URL, e.g. `https://your-user.github.io/compose-capabilities/index.json`.
3. Reload Compose Portal → **Space Settings → Capabilities**. The list now reflects this registry.

## Repo layout

```
compose-capabilities/
├── index.json                       # Registry index — list of capabilities
├── notification-templates/
│   ├── manifest.json                # Capability manifest (schema below)
│   ├── kapp.json                    # Kapp export (stub today)
│   └── README.md                    # Human docs for this capability
├── robots/
│   ├── manifest.json
│   ├── kapp.json
│   └── README.md
├── datastore/
│   ├── manifest.json
│   ├── kapp.json
│   └── README.md
└── README.md                        # You are here
```

## `index.json` schema

The registry index enumerates the capabilities hosted in this registry.

```json
{
  "registry": "your-registry-name",
  "schemaVersion": "1",
  "description": "Optional human-readable description.",
  "capabilities": [
    { "id": "<capability-id>", "manifestUrl": "./<folder>/manifest.json" }
  ]
}
```

- `id` — matches the capability's `manifest.id` (and becomes the kapp's slug on install).
- `manifestUrl` — relative to `index.json`, or absolute. Compose Portal resolves with standard URL semantics.

## `manifest.json` schema

```json
{
  "id": "notification-templates",
  "name": "Notification Templates",
  "description": "What this capability does, in one paragraph.",
  "version": "0.1.0",
  "author": "Optional author string.",
  "tags": ["optional", "labels"],
  "kapp": {
    "slug": "notification-templates",
    "name": "Notification Templates",
    "export": "./kapp.json"
  },
  "forms": [],
  "taskHandlers": [],
  "workflows": [],
  "datastores": [],
  "requires": []
}
```

- `id` / `name` / `description` / `version` are user-visible. `version` should be semver.
- `kapp` is required. Its `slug` becomes the installed kapp's slug on the space — that's also how Compose Portal detects the capability as installed (kapp with this slug + a `Capability Metadata` kapp attribute set to this manifest's id and version).
- `forms`, `taskHandlers`, `workflows`, `datastores` are arrays of install items. Today they're empty — the Phase 3 portal only reads the top-level metadata. Phase 4+ will define shapes for each array entry and actually import them.
- `requires` is an array of other capability ids this capability depends on. Phase 6+.

Asset URLs (the kapp `export`, and eventually entries in the other arrays) may be **relative to the manifest** (as shown here) or absolute URLs pointing anywhere reachable over HTTPS.

## Adding a new capability

1. Create a folder named after the capability id (kebab-case). The folder name is convention, not required — `manifestUrl` in `index.json` can point anywhere.
2. Add a `manifest.json` following the schema above.
3. Add a `kapp.json` (stub is fine for now).
4. Add an entry to `index.json`'s `capabilities` array pointing at your new manifest.
5. Commit + push. GitHub Pages republishes in about a minute.

## Versioning

Bump `version` in the capability's `manifest.json` whenever you change it. Compose Portal compares the manifest's version against the value recorded in each installed kapp's `Capability Metadata` attribute; a mismatch surfaces an "Upgrade available" badge in Space Settings → Capabilities.

## Status

This schema is v1 — deliberately minimal. We will extend it over time as Compose Portal's installer grows:

- Phase 3 (portal side): fetch + merge registries, display capabilities, detect installed status — **current focus**
- Phase 4: install action for kapp + forms + kapp attribute definitions
- Phase 5: install support for task handlers, workflows, datastores, integrations
- Phase 6: upgrade flow + customization detection via SHA-256 checksums on asset URLs
