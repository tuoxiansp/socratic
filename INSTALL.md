# Install

This repository ships a platform-neutral Agent Skill in `skill/`.

The skill can be installed manually, through an agent-assisted prompt, or later through a public registry such as ClawHub.

## Agent-Assisted Install Prompt

Copy this prompt into an agent that has access to your OpenClaw workspace:

```text
Install the skill "Socratic" from this repository:
https://github.com/tuoxiansp/socratic

The skill folder is:
skill/

Copy the skill into the active OpenClaw workspace as:
skills/socratic/

Keep the work scoped to this skill only.

After installing:
1. Inspect skills/socratic/SKILL.md and supporting files.
2. Verify the skill metadata name is "socratic".
3. Confirm the skill contains schemas/, knowledge/, PERSISTENCE.md, and UPDATE_POLICY.md.
4. Tell me whether I need to start a new OpenClaw session or restart the gateway for the skill to load.
```

This prompt uses the public repository under the `tuoxiansp` GitHub account.

## Agent-Assisted Update Prompt

Copy this prompt into an agent that has access to your OpenClaw workspace:

```text
Update the installed skill "Socratic" from this repository:
https://github.com/tuoxiansp/socratic

The source skill folder is:
skill/

The installed skill folder in the active OpenClaw workspace should be:
skills/socratic/

Before updating:
1. Inspect the currently installed skills/socratic/SKILL.md and record its version.
2. Inspect the new skill/SKILL.md and record its version.
3. Read skill/PERSISTENCE.md and skill/UPDATE_POLICY.md.
4. Confirm that learner state is not stored inside skills/socratic/.
5. If Socratic learner state exists in the runtime memory root, read socratic/manifest.json and check whether state migration is required.

During update:
1. Keep learner state outside the skill install directory.
2. If state migration is required, follow UPDATE_POLICY.md: lock writes, create a backup, migrate into a temporary namespace, validate, then promote atomically.
3. Replace only the installed skill package at skills/socratic/ with the new skill/ contents.
4. Verify the updated skill metadata name is "socratic" and the version is the expected new version.
5. Confirm the skill contains schemas/, knowledge/, PERSISTENCE.md, and UPDATE_POLICY.md.

After updating:
1. Validate all JSON files under skills/socratic/schemas/ and skills/socratic/knowledge/.
2. Report the old skill version, new skill version, whether state migration was needed, backup location if any, and whether I need to start a new OpenClaw session or restart the gateway.
```

## Manual OpenClaw Install

From an OpenClaw workspace root:

```bash
mkdir -p skills/socratic
cp -R /path/to/socratic/skill/. skills/socratic/
```

Then start a new session so OpenClaw refreshes the available skills:

```text
/new
```

Or restart the gateway if your setup requires it:

```bash
openclaw gateway restart
```

Verify:

```bash
openclaw skills list
openclaw skills info socratic
```

## Install From ClawHub

After the skill is published to ClawHub, users should be able to install it with:

```bash
openclaw skills install socratic
```

The ClawHub install path should place the skill into the active OpenClaw workspace `skills/` directory.

## Manual OpenClaw Update

If OpenClaw provides a skill update command, prefer that command so the runtime can preserve registry metadata and run any built-in checks.

If updating manually from an OpenClaw workspace root, first review `skill/UPDATE_POLICY.md`, then replace only the installed skill package:

```bash
cp -R /path/to/socratic/skill/. skills/socratic/
```

Then start a new session or restart the gateway if your setup requires it, and verify:

```bash
openclaw skills info socratic
```

