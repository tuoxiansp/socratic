# State Model

The learning coach depends on structured state. The agent may reason flexibly, but state shapes should stay stable.

Runtime persistence layout, filenames, and write order are defined in [PERSISTENCE.md](PERSISTENCE.md). Skill and state migration rules are defined in [UPDATE_POLICY.md](UPDATE_POLICY.md).

Persisted Socratic state should be versioned with `state_schema_version` in the runtime state manifest, separate from the skill package version.

## Learner Profile

Represents user-level preferences that apply across goals.

Key fields:

- `learner_key`: stable runtime learner key
- `locale`: preferred language or locale
- `timezone`: user's timezone for scheduling
- `preferred_pace`: overall learning pace
- `session_minutes`: preferred default session length
- `notification_preferences`: whether and how proactive lesson pushes are allowed

Example:

```json
{
  "learner_key": "user_opaque123",
  "locale": "zh-CN",
  "timezone": "Asia/Shanghai",
  "preferred_pace": "gentle",
  "session_minutes": 15,
  "notification_preferences": {
    "enabled": true,
    "frequency": "daily",
    "preferred_time_window": {
      "start": "20:00",
      "end": "21:30"
    },
    "quiet_hours": {
      "start": "22:30",
      "end": "09:00"
    },
    "quiet_days": [],
    "max_push_length": "one_question"
  }
}
```

## Learning Goal

Represents what the user wants to move toward.

Key fields:

- `goal_id`: stable identifier
- `title`: human-readable goal
- `user_motivation`: why the user wants this
- `desired_identity`: who the user wants to become through learning
- `preferred_style`: how the user wants to learn
- `target_domains`: domains involved
- `current_phase`: current learning phase
- `constraints`: time, emotional, and format constraints

Example:

```json
{
  "goal_id": "rebuild-scientific-intuition",
  "title": "重建数学和物理直觉",
  "user_motivation": "想重新学习数学和物理，但害怕从头上课",
  "desired_identity": "成为一个能用科学和数学理解世界的人",
  "preferred_style": "生活问题驱动，低计算，逐步形式化",
  "target_domains": ["physics", "math"],
  "current_phase": "calibration",
  "constraints": {
    "session_minutes": 15,
    "avoid_school_language": true,
    "calculation_intensity": "low"
  }
}
```

## Knowledge Node

Represents a concept or ability in the learning graph. Prefer a DAG over a strict tree because many concepts have multiple prerequisites.

Key fields:

- `node_id`
- `title`
- `domain`
- `level`
- `prerequisites`
- `unlocks`
- `learning_outcome`
- `abstraction_ladder`
- `example_questions`
- `common_misconceptions`
- `source_refs`

## Learner Node State

Represents this user's relationship with a knowledge node.

Key fields:

- `node_id`
- `status`: `unseen`, `learning`, `review`, `mastered`, or `blocked`
- `mastery`: 0 to 1
- `confidence`: 0 to 1
- `last_seen_at`
- `next_review_at`
- `exposure_count`
- `evidence`
- `active_misconceptions`

Do not mark a node as mastered only because the user said "I understand." Prefer evidence from restatement, transfer, or application.

## Learning Event

Append-only record of an interaction. Events are the audit trail for state updates.

Key fields:

- `event_id`
- `goal_id`: the learning goal this event belongs to
- `event_type`
- `created_at`
- `summary_for_future_agent`

Useful event types:

- `goal_created`
- `graph_created`
- `calibration_completed`
- `lesson_planned`
- `lesson_started`
- `lesson_completed`
- `lesson_abandoned`
- `misconception_found`
- `misconception_resolved`
- `review_scheduled`
- `state_redacted`

## Lesson Plan

Represents the next unit of interaction.

Key fields:

- `lesson_id`
- `mode`: `proactive` or `on_demand`
- `lesson_type`: `calibration`, `new_concept`, `review`, `misconception_repair`, `synthesis`, or `confidence_rebuild`
- `target_nodes`
- `question`
- `teaching_goal`
- `expected_intuitions`
- `socratic_prompt_sequence`
- `minimum_teaching_point`
- `transfer_check`
- `completion_criteria`
- `state_update_hints`

## Mastery Evidence

Use evidence rather than vibes. Good evidence types:

- `initial_intuition`: user's first guess
- `restatement`: user explains in their own words
- `transfer_question`: user applies the concept to a nearby scenario
- `counterexample_response`: user handles a challenging example
- `calculation`: user performs minimal formal work
- `self_reflection`: user describes what changed in their understanding

Evidence result values:

- `success`
- `partial`
- `failed`
- `skipped`

## Node Selection Policy

When choosing what to teach next:

1. If the user is new, run calibration with low-stakes questions and collect push preferences.
2. If a target node has unmet prerequisites, choose the weakest prerequisite.
3. If the user recently failed transfer, schedule transfer practice.
4. If a misconception is active, repair it before adding more abstraction.
5. If old nodes are due for review, insert a short review.
6. If multiple related nodes are strong, generate a synthesis problem.
7. If the user starts an on-demand session and performs well, keep advancing.
8. If the user shows fatigue or anxiety, lower abstraction and shorten the loop.

