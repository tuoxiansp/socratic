# Persistence Contract

This skill does not own the storage engine. The agent runtime owns memory, permissions, backups, locks, sync, and encryption.

The skill does define a stable persistence layout so different runtimes can store Socratic learning state consistently and migrate it safely.

## Storage Boundary

Do not write learner state into the installed skill directory, such as `skills/socratic/`.

The runtime should choose a writable memory root and create a Socratic namespace under it:

```text
<runtime_memory_root>/
└── socratic/
```

The runtime may map `<runtime_memory_root>` to a database, object store, local folder, or encrypted vault. If the backing store is not a filesystem, preserve the same logical paths as keys.

## Namespace Layout

Required logical layout for writable runtimes:

A writable runtime must store Socratic learner state using this logical layout. If it finds legacy Socratic learning state without a matching `state_schema_version`, it must back up and migrate that state into this layout before continuing normal writes.

```text
socratic/
├── manifest.json
├── users/
│   └── <learner_key>/
│       ├── profile.json
│       ├── goals/
│       │   └── <goal_id>.json
│       ├── graphs/
│       │   └── <graph_id>/
│       │       ├── graph.json
│       │       └── nodes/
│       │           └── <node_id>.json
│       ├── learner_state/
│       │   └── <goal_id>/
│       │       └── <node_id>.json
│       ├── lesson_plans/
│       │   ├── active.json
│       │   └── archive/
│       │       └── <lesson_id>.json
│       ├── events/
│       │   └── <goal_id>/
│       │       └── YYYY-MM.jsonl
│       ├── reviews/
│       │   └── due.json
│       └── migrations/
│           └── applied.jsonl
├── shared/
│   ├── knowledge_overrides/
│   │   └── <graph_id>/
│   │       └── nodes/
│   │           └── <node_id>.json
│   └── sources/
│       └── <source_id>.json
└── backups/
    └── <timestamp>-<reason>/
```

## Learner Keys

`<learner_key>` must be stable, path-safe, and non-reversible when possible.

Preferred forms:

- `user_<opaque_runtime_id>` if the runtime already uses opaque user ids
- `sha256_<first_16_hex_chars>` if the runtime only has email, phone, or another personal identifier

Do not use raw email addresses, phone numbers, or display names in paths.

## File Responsibilities

`manifest.json` describes the whole Socratic state namespace. It should include:

```json
{
  "skill_name": "socratic",
  "skill_version": "0.1.0",
  "state_schema_version": "0.1.0",
  "created_at": "2026-05-14T00:00:00Z",
  "updated_at": "2026-05-14T00:00:00Z"
}
```

`profile.json` stores learner-level preferences that are not tied to one goal, such as preferred pace, emotional constraints, locale, and notification preferences.

`goals/<goal_id>.json` stores one `learning_goal`.

`graphs/<graph_id>/nodes/<node_id>.json` stores user-specific or expanded `knowledge_node` objects. Built-in starter nodes may be read from `knowledge/starter-knowledge-map.json` instead of copied, unless the runtime customizes them.

`learner_state/<goal_id>/<node_id>.json` stores one `learner_node_state` for one goal and one node.

`lesson_plans/active.json` stores the next active `lesson_plan`. Once used or replaced, move it to `lesson_plans/archive/<lesson_id>.json`.

`events/<goal_id>/YYYY-MM.jsonl` stores append-only `learning_event` records for one learning goal, one JSON object per line, partitioned by UTC month. Each event must also include the same `goal_id` inside the JSON object so exported events remain self-describing outside their original path.

`reviews/due.json` stores runtime-computed review queues. It is derived state and may be rebuilt from learner node states and events.

`migrations/applied.jsonl` stores one append-only record per successful state migration.

`shared/knowledge_overrides/` stores graph additions that are not private to a single learner. Avoid putting learner evidence or personal motivation here.

`backups/` stores runtime-created snapshots before migrations or risky maintenance.

## Naming Rules

Use these rules for ids and filenames:

- ids are lowercase kebab-case where humans create them, for example `heat-transfer`
- runtime-generated ids may use opaque prefixes, for example `lesson_01hx...`
- timestamps are UTC ISO 8601 strings in file contents
- backup folder timestamps use compact UTC form: `YYYYMMDDTHHMMSSZ`
- JSON files contain one object
- JSONL files contain one object per line and are append-only

## Write Rules

The runtime should make state updates in this order:

1. Append the `learning_event`.
2. Write small `learner_state` patches or replacements.
3. Update `lesson_plans/active.json`.
4. Update `manifest.json.updated_at`.

Append events to `events/<goal_id>/YYYY-MM.jsonl`, where `<goal_id>` matches the event payload. Use atomic writes for JSON files: write to a temporary key or file, validate, then rename or commit. Do not partially rewrite JSONL event files.

## Read Rules

Before planning a lesson, read:

1. `manifest.json`
2. the active or selected `learning_goal`
3. relevant `knowledge_node` files
4. relevant `learner_state` files
5. recent `events/<goal_id>/YYYY-MM.jsonl`
6. `reviews/due.json` if present

If the state schema version is newer than the skill supports, the agent should stop and ask the runtime to update the skill instead of guessing.

## Privacy And Retention

Learner motivation, misconceptions, and evidence are personal data. Runtimes should encrypt at rest when available, avoid raw personal identifiers in paths, and allow user-level export or deletion.

Events are append-only for learning quality, but the runtime may implement retention, redaction, or deletion for privacy requirements. If an event is removed, append a `state_redacted` event rather than silently changing downstream mastery state.
