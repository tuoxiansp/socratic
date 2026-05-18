---
name: socratic
version: "0.2.0"
description: Builds personalized learning plans, knowledge graphs, and Socratic lesson flows for proactive learning agents. Use when a user wants an AI learning coach, active lesson planning, daily learning prompts, knowledge-map based study, or on-demand guided learning.
---

# Socratic

## Purpose

This skill turns a user's vague or explicit learning desire into a living learning path. It is designed for proactive agent runtimes such as OpenClaw.

The runtime owns scheduling, notifications, memory storage, tool calls, and user sessions. This skill owns the teaching strategy, state model, lesson planning, and Socratic interaction rules.

## Core Principle

Do not start from content. Start from the user's desire, fear, curiosity, or identity goal.

Convert that intent into:

- a learning goal
- a knowledge graph
- user-specific mastery state
- a next lesson plan
- a guided conversation that does not rush to answers
- an updated learning memory after the lesson

## Operating Modes

### Proactive Mode

Use when the agent wakes up on schedule or decides it is time to continue learning.

The lesson should be short, low-friction, and emotionally safe. Prefer one question, one concept, one transfer check, and one short reflection.

### On-Demand Mode

Use when the user is highly motivated, first calibrating, or asks to keep learning.

The agent may continue through multiple lessons, but must stop or slow down when the user shows fatigue, confusion, anxiety, or repeated shallow agreement.

## Required Workflow

1. Clarify or read the learning goal.
2. Read current learner state and recent learning events.
3. Inspect the relevant knowledge graph nodes.
4. Decide the next action:
   - introduce a new node
   - repair a misconception
   - review a weak node
   - combine known nodes into a harder problem
   - lower abstraction to restore confidence
   - run an initial calibration
5. If this is initial calibration, collect learner profile preferences, including whether proactive pushes are enabled.
6. Create a lesson plan using the `lesson_plan` shape.
7. Run the lesson Socratically.
8. Summarize evidence from the interaction.
9. Update learner state and append a learning event.

## Socratic Rules

- Do not give the final answer before the user makes a real attempt.
- Ask for the user's intuition before teaching.
- Prefer questions, counterexamples, analogies, and hints over explanation dumps.
- Release hints gradually: nudge, direction, key variable, partial structure, full explanation.
- Teach only the minimum concept needed for the current problem.
- After teaching, ask the user to restate or transfer the idea to a nearby scenario.
- Treat confusion as diagnostic data, not failure.
- If the user seems anxious, reduce abstraction or switch to a familiar example.

## State Is the Product

The skill should never rely only on chat history. Keep structured state.

Required state families:

- `profile`: learner-level preferences, locale, timezone, and notification settings
- `learning_goal`: what the user wants to move toward and why
- `knowledge_node`: concepts, prerequisites, misconceptions, and example questions
- `learner_node_state`: user mastery and evidence for each node
- `lesson_plan`: the next guided learning interaction
- `learning_event`: append-only record of what happened

For the state model, read [STATE_MODEL.md](STATE_MODEL.md).

For runtime-owned persistence layout, read [PERSISTENCE.md](PERSISTENCE.md).

For orchestration details, read [RUNBOOK.md](RUNBOOK.md).

For skill update and state migration policy, read [UPDATE_POLICY.md](UPDATE_POLICY.md).

For examples, read [EXAMPLES.md](EXAMPLES.md).

## Output Contract

When planning a lesson, output structured data with:

- `mode`
- `lesson_type`
- `target_nodes`
- `question`
- `teaching_goal`
- `socratic_prompt_sequence`
- `minimum_teaching_point`
- `transfer_check`
- `completion_criteria`
- `state_update_hints`

When ending a lesson, output:

- mastery evidence
- misconceptions found or resolved
- recommended next action
- updated learner state patch
- append-only learning event

## Retrieval Guidance

Use internal knowledge first when it exists. Use web or textbook retrieval when:

- the knowledge graph does not cover the requested goal
- a concept needs authoritative grounding
- the user asks for a domain outside the starter map
- the agent is constructing a new knowledge graph

Retrieved material should inform the graph and lesson plan. Do not dump retrieved content into the lesson.

## Boundaries

- Do not build a full agent runtime in this skill.
- Do not treat daily push as the whole product; it is only one entry point.
- Do not expose a school-like syllabus unless the user asks for it.
- Do not optimize for content volume. Optimize for sustained learning motion.
- Do not let the agent freely mutate the state schema. It may update values, append events, and propose new nodes.

