# Update And Migration Policy

This document defines how the Socratic skill should be updated and how runtime-owned learning state should be migrated safely.

## What Other Skill Systems Do

Current skill ecosystems tend to separate three concerns:

- AgentSync keeps installed skills in a canonical `.agents/skills/` directory, tracks installed skills in `registry.json`, and requires a valid `version` in `SKILL.md` for update commands.
- The broader Agent Skills spec discussions focus on preventing mixed-version reads: agents should not read instructions from one version while executing or loading resources from another.
- Common package-manager practice is to keep package updates separate from user data, then run explicit migrations for state that must change.

Useful patterns to adopt:

- keep `SKILL.md` lightweight but include a skill `version`
- keep runtime state outside the skill install directory
- record a separate `state_schema_version` in persisted learner state
- validate state after updates instead of relying on chat history
- use backups and atomic replacement for migrations
- treat scripts or generated migration tools as versioned resources if they are introduced later

This skill is currently prompt, schema, and knowledge only. It does not ship executable migration scripts.

## Version Fields

Use two independent versions:

- `skill_version`: semantic version of the skill package, read from `SKILL.md`
- `state_schema_version`: semantic version of persisted Socratic state, stored in `socratic/manifest.json`

They may move together, but they are not the same thing. A teaching prompt change may update `skill_version` without changing `state_schema_version`.

Current versions:

```json
{
  "skill_version": "0.2.0",
  "state_schema_version": "0.2.0"
}
```

## Migration Notes

### `0.1.0` to `0.2.0`

This update makes learner profile and proactive push preferences explicit.

Migration procedure:

1. Create `users/<learner_key>/profile.json` if it does not already exist.
2. Set `learner_key` from the runtime's stable learner key.
3. Preserve existing locale, timezone, pace, session length, and emotional constraints if the runtime already stores them.
4. If an old `learning_goal.constraints.preferred_push_frequency` exists, copy it to `profile.notification_preferences.frequency`.
5. Set `profile.notification_preferences.enabled` to `false` unless there is explicit evidence that the user opted in to proactive pushes.
6. Do not create a schedule during migration solely from inferred preferences. Scheduling requires user consent or an existing runtime schedule.
7. Validate the created profile against `profile.schema.json`.

## Migration Required Conditions

Migration decisions must be based only on `state_schema_version`.

Determine the current state schema version this way:

1. If `socratic/manifest.json` exists and contains `state_schema_version`, use that value.
2. If Socratic learner state exists but no `state_schema_version` can be found, treat it as `legacy-unversioned`.
3. If no Socratic learner state exists, no state migration is needed.

Known legacy Socratic state includes older files such as `memory/learning/*.json` or `memory/learning/linear-algebra-*.json`, and any Socratic learning files that do not belong to the required `socratic/` layout with a manifest.

Migration is required when the current state schema version is `legacy-unversioned` or differs from the target `state_schema_version` supported by the new skill.

Do not skip migration because legacy files appear compatible at runtime. Do not use behavioral compatibility, file naming similarity, or successful ad hoc reads as reasons to leave old state in place.

If migration is required and the runtime has write permission, the agent or runtime must create a backup and migrate automatically. It must not merely report the mismatch to the user or ask whether migration should happen.

The agent may stop and ask the user only when one of these blockers occurs:

- the runtime does not have write permission
- the learner identity or memory root cannot be determined
- the legacy data cannot be mapped without likely data loss
- backup creation fails
- migrated data fails validation

## Compatibility Rules

Patch skill updates may clarify prompts, examples, or documentation. They must not require state migration.

Minor skill updates may add optional fields, starter knowledge, lesson types, or validation guidance. They may include a backward-compatible state migration.

Major skill updates may rename fields, remove fields, change required meanings, or reorganize persisted state. They must include a migration plan.

During `0.x`, any state model change may still be breaking. Treat changes to required schema fields, enum values, directory layout, or event semantics as migration-requiring changes even if the skill version only changes minor.

## Update Procedure For Runtimes

When installing a new Socratic skill version over an existing one:

1. Read the old installed skill version and the new skill version.
2. Determine the current state schema version using the rules above.
3. Check whether the new skill supports that `state_schema_version`.
4. Lock the learner namespace or pause writes for the affected user.
5. Create a backup under `socratic/backups/<timestamp>-pre-update-<old>-to-<new>/`.
6. When migration is required, migrate into a temporary namespace, not in place.
7. Validate migrated files against the schemas in the new skill.
8. Run integrity checks described below.
9. Atomically promote the migrated namespace.
10. Append a migration record to `migrations/applied.jsonl`.
11. Unlock the namespace and resume lessons.

If any step fails, keep the old state and old skill active. Do not continue with partial migrated state.

## Migration Record

Each successful migration should append one JSON object to the learner's `migrations/applied.jsonl`:

```json
{
  "migration_id": "state-0.1.0-to-0.2.0",
  "from_skill_version": "0.1.0",
  "to_skill_version": "0.2.0",
  "from_state_schema_version": "0.1.0",
  "to_state_schema_version": "0.2.0",
  "started_at": "2026-05-14T00:00:00Z",
  "completed_at": "2026-05-14T00:00:03Z",
  "backup_path": "socratic/backups/20260514T000000Z-pre-update-0.1.0-to-0.2.0",
  "summary": "Created learner profile with explicit notification preferences; no event rewrite needed."
}
```

## Data Correctness Checks

After any migration, validate:

- every JSON file parses
- every JSONL line parses independently
- each learner `profile.json` matches `profile.schema.json`
- each `learning_goal` matches `learning-goal.schema.json`
- each `knowledge_node` matches `knowledge-node.schema.json`
- each `learner_node_state` matches `learner-state.schema.json`
- each archived or active `lesson_plan` matches `lesson-plan.schema.json`
- each `learning_event` matches `learning-event.schema.json`
- each `learning_event.goal_id` references an existing `learning_goal`
- each event stored under `events/<goal_id>/` has a matching payload `goal_id`
- every learner node state references an existing or built-in knowledge node
- every lesson target node exists
- event ids are unique within a learner namespace
- event timestamps are non-empty UTC timestamps
- `manifest.json.state_schema_version` equals the migrated target version

Do not infer missing mastery evidence during migration. If old state lacks evidence, preserve the uncertainty and schedule review.

## State Schema Change Policy

When changing persisted state:

1. Update the relevant schema file.
2. Update `STATE_MODEL.md` if the meaning changes.
3. Update `PERSISTENCE.md` if layout, filename, or write order changes.
4. Update this file with migration notes.
5. Add or update examples in `EXAMPLES.md`.
6. Update publishing checks so changed schemas are validated.

Prefer additive changes:

- add optional fields before making them required
- keep old event history append-only
- preserve original evidence summaries
- write derived data into new files instead of rewriting history

Avoid:

- renaming ids without an alias map
- deleting old events
- changing the meaning of existing enum values
- mixing learner-private data into shared graph files

## Rollback

Rollback is supported only if the migration has not promoted the new namespace. After promotion, the runtime may restore from the backup snapshot, but the skill should not try to downgrade state automatically unless a specific downgrade plan exists.

## Future Migration Scripts

If future versions ship migration scripts, they must be treated as update-sensitive resources:

- include the script version in the filename or manifest metadata
- document inputs and outputs
- run only against a backup or temporary namespace first
- validate output before promotion
- keep scripts deterministic and idempotent when possible

Until scripts exist, migrations should be expressed as human-readable plans that an agent runtime can execute conservatively.
