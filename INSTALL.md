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
3. Confirm the skill contains schemas/ and knowledge/.
4. Tell me whether I need to start a new OpenClaw session or restart the gateway for the skill to load.
```

This prompt uses the public repository under the `tuoxiansp` GitHub account.

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

