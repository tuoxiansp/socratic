# Runbook

This runbook describes how a proactive agent runtime should use the skill.

## Runtime Responsibilities

The runtime should provide:

- schedule or wake-up trigger
- user session channel
- memory read and write using the layout in [PERSISTENCE.md](PERSISTENCE.md)
- retrieval tools
- notification delivery
- timestamp and locale
- a way to enable, disable, or update proactive push configuration after user consent

The skill provides:

- teaching strategy
- state interpretation
- next lesson planning
- Socratic interaction policy
- memory update shape

## Entry: Initial Calibration

Use when the user first installs the skill or has no reliable learner state.

Process:

1. Ask why the user wants to learn now.
2. Ask what they are afraid of or tired of in past learning.
3. Ask for preferred pace and session length.
4. Ask whether they want proactive lesson pushes.
5. If they want pushes, ask for push frequency, preferred time window, quiet days or hours, and maximum interruption level.
6. If the runtime can configure scheduling, create or update the schedule before ending calibration. If it cannot, record a clear setup action for the runtime owner.
7. Give one low-stakes question.
8. Observe how the user reasons, not only whether they are correct.
9. Create an initial learner `profile`, `learning_goal`, starter graph scope, and first node states.

Avoid asking the user to fill a long form. Calibrate through conversation.

The push decision is part of calibration, not an optional afterthought. Persist it in `profile.json` even when the user declines pushes so future agents do not keep asking.

Minimum push preference fields:

- `enabled`
- `frequency`
- `preferred_time_window`
- `quiet_hours`
- `quiet_days`
- `timezone`
- `max_push_length`

If the user has not explicitly opted in, default `enabled` to `false`.

## Entry: Proactive Lesson Push

Use when the agent wakes up daily or periodically.

Process:

1. Read `profile.json` and stop if proactive pushes are disabled.
2. Confirm the current wake-up is inside the user's allowed push window.
3. Read recent learning events and due reviews.
4. Choose one next action.
5. Generate one question and one short opening message.
6. Push only the question and a gentle invitation.
7. If the user responds, run the Socratic flow.
8. If the user ignores it, do not mark failure; record a skipped opportunity only if useful.

Preferred push style:

```text
一个小问题：为什么冬天摸铁门把手会比摸木头桌子更冷？它们明明在同一个房间里。
先不用算，凭直觉说说你觉得原因是什么。
```

Avoid:

```text
今天我们学习热传导第一课，请完成以下任务。
```

## Entry: On-Demand Learning

Use when the user says they want to learn more, continue, go deeper, or start a longer session.

Process:

1. Confirm desired intensity if unclear.
2. Reuse the same planner as proactive mode.
3. Continue lesson by lesson, not as a long lecture.
4. After each lesson, decide whether to advance, review, synthesize, or stop.
5. Offer a natural stopping point after visible effort.

Stop or slow down when:

- the user gives repeated shallow agreement
- the user becomes anxious or self-critical
- the user fails the same transfer twice
- the lesson requires too many new concepts at once

## Lesson Planning Procedure

When creating a lesson plan:

1. Identify target node and prerequisites.
2. Select lesson type.
3. Choose abstraction level.
4. Draft a concrete question.
5. Write expected user intuitions, including wrong but reasonable answers.
6. Write hint ladder.
7. Write the minimum teaching point.
8. Write one transfer check.
9. Define what evidence will update state.

## Socratic Interaction Procedure

1. Ask the question.
2. Wait for user's intuition.
3. Ask why they think that.
4. If useful, ask a contrast question.
5. Provide one hint at a time.
6. Teach only after the user has explored.
7. Ask for restatement.
8. Ask transfer check.
9. Summarize what changed.
10. Update state through event and learner-node patches.

## State Update Procedure

After a lesson, write:

- one short summary for future agents
- evidence items
- node state changes
- misconceptions found or resolved
- next recommended action
- review timing hint

Prefer append-only learning events plus small node-state patches. Do not rewrite history.

Persist the update using this order:

1. Append the learning event.
2. Write learner node state updates.
3. Replace or archive the active lesson plan.
4. Update the runtime state manifest timestamp.

If any write fails, keep the previous state readable and retry from the append-only event when possible.

## Retrieval Procedure

When the graph lacks a domain:

1. Retrieve authoritative sources or textbooks.
2. Extract concepts, prerequisites, misconceptions, and examples.
3. Build a small graph slice first.
4. Use the graph slice to plan the next lesson.
5. Save source references on nodes.

Do not retrieve huge content dumps unless needed. The lesson should stay small.

