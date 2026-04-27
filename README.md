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
├── index.json                          # Registry index — list of capabilities
├── README.md                           # You are here
├── notifications/
│   ├── manifest.json                   # Capability manifest (schema below)
│   ├── kapp.json                       # Kapp export
│   ├── <form>.json                     # Form exports
│   ├── <form>-submissions.csv          # Optional seed data for a form
│   ├── <handler>.zip                   # Task handler packages
│   ├── tree_<Name>.xml                 # Workflow tree exports
│   ├── routine_<Name>.xml              # Workflow routine exports
│   └── integrations.json               # Integration / connection export
├── robots/
│   └── ...                             # Same structure as above
└── datastore/
    └── ...                             # A minimal capability: kapp + a form
```

Folder names are convention, not required — `manifestUrl` in `index.json` can point anywhere. Include only the asset files a capability actually needs; omit manifest keys for assets you don't have rather than leaving empty arrays.

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
  "id": "notifications",
  "name": "Notifications",
  "description": "What this capability does, in one paragraph.",
  "version": "0.1.0",
  "author": "Kinetic Data",
  "tags": ["optional", "labels"],

  "kapp": {
    "definition": "./kapp.json"
  },

  "forms": [
    {
      "definition": "./form-slug.json",
      "data": ["./form-slug-submissions.csv"]
    }
  ],

  "taskHandlers": [
    {
      "definition": "./handler_name_v1.zip",
      "configuration_properties": {
        "kapp_slug": "notification-templates"
      }
    }
  ],

  "workflows": {
    "trees": [
      { "definition": "./tree_Name.xml" }
    ],
    "routines": [
      "./routine_Name.xml"
    ]
  },

  "integrations": [
    { "definition": "./integrations.json" }
  ],

  "spaceConfiguration": {
    "userProfileAttributes": [
      {
        "name": "<attribute name>",
        "allowsMultiple": false,
        "description": "Human-readable description of what this attribute controls."
      }
    ],
    "teamAttributes": [
      {
        "name": "<attribute name>",
        "allowsMultiple": false,
        "description": "Human-readable description of what this attribute controls."
      }
    ]
  },

  "notes": {
    "manual_steps": [
      {
        "id": "1",
        "text": "Human-readable instruction for a step an operator must complete after install.",
        "link": "/app/console/#/space/plugins/connections"
      }
    ]
  },

  "requires": []
}
```

### Field reference

**Top-level metadata**

- `id` / `name` / `description` / `version` / `author` / `tags` — user-visible. `id` must match the id in `index.json` and becomes the kapp's slug on install. `version` should be semver.

**`kapp`** *(required)* — the kapp to install.

- `definition` — URL to a kapp export JSON.

The installed kapp's slug and name come from kapp.json's slug and name fields. Compose Portal detects the capability as installed when it finds any kapp carrying a `Capability Metadata` attribute referencing this manifest's id.

**`forms`** *(optional)* — datastore or kapp forms to install. Array of:

- `definition` — URL to the form export JSON.
- `data` *(optional)* — array of CSV URLs whose rows are seeded as submissions after the form is created.

**`taskHandlers`** *(optional)* — handler packages to register. Array of:

- `definition` — URL to the handler `.zip`.
- `configuration_properties` *(optional)* — keys/values written to the handler's configuration (e.g. `kapp_slug`, form slugs the handler should target). Use this when the handler needs to know about slugs the capability itself is creating.

  **All-or-nothing rule:** if you set *any* value in `configuration_properties`, you must include *every* required field the handler defines — including sensitive ones like passwords or API keys. The handler update API rejects the call if any required field is blank, so omitting a required field causes the entire install to fail. For required fields you don't have a real value for, supply a placeholder string (e.g. `"placeholder"`) and add a `notes.manual_steps` entry telling the operator to open the handler and replace the placeholders with real values before the capability will work.

**`workflows`** *(optional)* — workflow assets. An object with:

- `trees` — array of `{ "definition": "<url>.xml" }`. Trees bind to specific forms or events.
- `routines` — array of URL strings pointing to reusable workflow routines.

**`integrations`** *(optional)* — external system connections. Array of `{ "definition": "<url>.json" }`.

**`spaceConfiguration`** *(optional)* — space-level settings the capability needs in place to function. The installer ensures these exist on the space; values are typically populated per-user/per-team after install. An object with:

- `userProfileAttributes` — array of user profile attribute definitions the capability reads or writes. Each entry: `name` (attribute name), `allowsMultiple` (boolean — whether the attribute supports multiple values), `description` (human-readable explanation, including any expected values).
- `teamAttributes` — array of team attribute definitions, same shape as above.

Use `spaceConfiguration` for attributes the capability *requires* to work (e.g. a notifications capability that reads a user's `Notification Delivery Method`). Don't list attributes a capability merely reads opportunistically.

**`notes.manual_steps`** *(optional)* — post-install steps an operator must complete manually (plugin credentials, handler config, etc.). Each entry:

- `id` — stable identifier for the step.
- `text` — instruction shown to the operator.
- `link` — relative path inside the admin console that opens the right screen.

**`requires`** *(optional, future)* — array of capability ids this capability depends on. Honored in Phase 6+.

### Omit what doesn't apply

Every top-level key except `id`, `name`, `description`, `version`, and `kapp` is optional. Omit keys that don't apply rather than including empty arrays or placeholder objects — an empty `"forms": []` or a `"notes": {}` reads as unfinished work, not a deliberate "no forms here." `datastore/manifest.json` is a minimal example: a kapp and one form, nothing else.

### Asset URLs

Any URL in the manifest (`kapp.definition`, `forms[].definition`, `forms[].data[]`, `taskHandlers[].definition`, `workflows.trees[].definition`, `workflows.routines[]`, `integrations[].definition`) may be **relative to the manifest** — as in the example above — or an **absolute HTTPS URL** pointing anywhere reachable from the portal.

## Adding a new capability

1. Create a folder named after the capability id (kebab-case). The folder name is convention, not required — `manifestUrl` in `index.json` can point anywhere.
2. Add a `manifest.json` following the schema above.
3. Add a `kapp.json` (stub is fine for now).
4. Add an entry to `index.json`'s `capabilities` array pointing at your new manifest.
5. Commit + push. GitHub Pages republishes in about a minute.

## Versioning

Bump `version` in the capability's `manifest.json` whenever you change it. Compose Portal compares the manifest's version against the value recorded in each installed kapp's `Capability Metadata` attribute; a mismatch surfaces an "Upgrade available" badge in Space Settings → Capabilities.

## Status

This schema is v1 and will extend as Compose Portal's installer grows. The asset arrays (`forms`, `taskHandlers`, `workflows`, `integrations`, `notes`) are populated in this registry ahead of the installer work so later phases have the data ready to consume.

- Phase 3 (portal side): fetch + merge registries, display capabilities, detect installed status — **current focus**. Reads top-level metadata only.
- Phase 4: install action for kapp + forms + kapp attribute definitions.
- Phase 5: install support for task handlers, workflows, integrations; surface `notes.manual_steps` in the install UI.
- Phase 6: upgrade flow + customization detection via SHA-256 checksums on asset URLs; honor `requires`.
