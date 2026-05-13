# Publishing

This project publishes the reusable skill folder at `skill/`.

The skill is intentionally platform-neutral, but it is compatible with OpenClaw and ClawHub because it follows the Agent Skills folder pattern.

## Artifact

Publish this folder:

```text
skill/
├── SKILL.md
├── STATE_MODEL.md
├── RUNBOOK.md
├── EXAMPLES.md
├── schemas/
└── knowledge/
```

Do not publish the repository root as the skill package. The root contains project documentation; `skill/` is the actual skill artifact.

## Pre-Publish Checklist

- `skill/SKILL.md` exists.
- `skill/SKILL.md` frontmatter has `name` and `description`.
- The skill name is `socratic`.
- All JSON files under `skill/schemas/` and `skill/knowledge/` parse successfully.
- No secrets, private user data, API keys, or local machine paths are included.
- Supporting files are text-based.
- The install prompt in `INSTALL.md` points to the correct public repository URL.
- Version and changelog are decided before publishing.

## Local Validation

Run from the repository root:

```bash
python3 -m json.tool skill/schemas/learning-goal.schema.json >/dev/null
python3 -m json.tool skill/schemas/knowledge-node.schema.json >/dev/null
python3 -m json.tool skill/schemas/learner-state.schema.json >/dev/null
python3 -m json.tool skill/schemas/lesson-plan.schema.json >/dev/null
python3 -m json.tool skill/schemas/learning-event.schema.json >/dev/null
python3 -m json.tool skill/knowledge/starter-knowledge-map.json >/dev/null
python3 -m json.tool skill/knowledge/starter-lessons.json >/dev/null
python3 -m json.tool skill/knowledge/intent-patterns.json >/dev/null
```

Optional OpenClaw local test:

```bash
mkdir -p skills/socratic
cp -R skill/. skills/socratic/
openclaw skills list
openclaw skills info socratic
```

## Publish To ClawHub

Install and sign in:

```bash
npm i -g clawhub
clawhub login
clawhub whoami
```

Publish:

```bash
clawhub skill publish ./skill \
  --slug socratic \
  --name "Socratic" \
  --version 0.1.0 \
  --changelog "Initial release"
```

Inspect after publishing:

```bash
clawhub inspect socratic
```

Users can install with:

```bash
openclaw skills install socratic
```

## Versioning

Use semantic versions:

- `0.1.0`: first public skill draft
- `0.x.y`: early state model and teaching strategy changes
- `1.0.0`: stable state schema and runtime contract

Create a new version when:

- `SKILL.md` changes behavior materially
- state schemas change
- starter knowledge changes significantly
- OpenClaw metadata or install requirements change

## Security Notes

This skill currently contains instructions, schemas, and starter knowledge only. It does not require API keys, custom binaries, or executable scripts.

If future versions add tools, scripts, or third-party services:

- declare required binaries or environment variables in `SKILL.md` metadata
- document setup in `INSTALL.md`
- keep secrets out of the repository
- test in a sandboxed agent environment before publishing

