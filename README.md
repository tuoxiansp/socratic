# Socratic

![Skill Version](https://img.shields.io/badge/skill-v0.2.0-2ea44f)
![Status](https://img.shields.io/badge/status-early%20skill%20spec-blue)
![Runtime](https://img.shields.io/badge/runtime-platform%20neutral-lightgrey)

**A platform-neutral AI learning coach skill: one meaningful question a day to rebuild mathematical, physical, and scientific intuition.**

[简体中文](README.zh-CN.md)

Socratic is not a course platform, and it is not a chatbot that simply gives answers. It turns a user's vague learning intention into a sustainable learning path: start with a question worth thinking about, ask the user to express their intuition, then use follow-up questions, hints, minimal explanations, and transfer checks to help them truly understand a concept.

This repository currently publishes a reusable Agent Skill. The core artifact lives in [`skill/`](skill/).

## Why This Exists

AI will keep automating deterministic work, but people still need to understand complex systems, transfer knowledge, and judge whether models are reliable. Many adults want to relearn math, physics, or scientific thinking, but restarting from chapter one of a textbook is a high-friction path.

Socratic starts from a simple assumption: adults often do not need more content. They need a low-pressure, low-friction entry point with enough intellectual density to be worth returning to.

```text
A complex systems question: adding resources does not always make things better.

If a city keeps widening its roads, why might congestion fail to improve, or even get worse?

This question introduces feedback loops, induced demand, and systems modeling: when we change a system, the people inside the system also change their behavior.
```

## Core Capabilities

- **Socratic guidance**: before the user's first attempt, the skill avoids giving the standard answer directly and instead uses questions, counterexamples, and layered hints to help the user move forward.
- **Personalized learning paths**: generates the next learning step from the user's motivation, fears, target identity, and current ability.
- **Knowledge graph driven**: organizes learning content with concept nodes, prerequisites, common misconceptions, and example questions.
- **Structured learning state**: records evidence of mastery, misconceptions, review timing, and learning events instead of relying only on chat history.
- **Proactive and on-demand entry points**: supports lightweight daily prompts as well as user-initiated learning sessions.
- **Platform-neutral skill package**: decouples teaching strategy, state models, and runbooks from any specific runtime, so it can plug into proactive agents such as OpenClaw.

## Who It Is For

Socratic is primarily for people who want to start learning again, but feel pressure as soon as learning becomes "systematic."

- You may have been away from math, physics, or scientific training for a long time, but still want to understand the world more clearly.
- You do not want exams, rankings, formula dumps, or "start from chapter one" to block you again.
- You want AI to do more than tell you the answer. You want it to help you sharpen your own intuition.
- You only have ten or twenty minutes a day, but want to build thinking skills that transfer.
- You are building a learning product or education tool and want a restrained interaction model that takes long-term learning state seriously.

This repository provides Socratic's core teaching capability package, not a complete app. It can be connected to proactive agents such as OpenClaw, or adapted by product teams into their own learning experience.

## Quick Start

If you already have an OpenClaw workspace that supports Skills, the fastest path is to give the following prompt to an agent with workspace access:

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

For fuller installation and update instructions, see [`INSTALL.md`](INSTALL.md).

## How It Works

Each learning session tries to form a short loop:

1. Read the learning goals, learner profile, recent learning events, and relevant knowledge nodes.
2. Choose the next action: calibrate, introduce a new concept, repair a misconception, review, synthesize, or lower the abstraction level.
3. Generate a concrete question, a hint ladder, a minimal teaching point, and a transfer check.
4. Run a Socratic dialogue with the user: observe intuition first, then release hints step by step.
5. Update learning state from evidence such as restatements, transfer answers, and responses to counterexamples.
6. Append a learning event so future agents can retrieve and transfer state.

For the fuller orchestration flow, see [`skill/RUNBOOK.md`](skill/RUNBOOK.md).

## Example: Next Lesson Plan

```json
{
  "mode": "proactive",
  "lesson_type": "new_concept",
  "target_nodes": ["complex-systems", "feedback-loops", "induced-demand"],
  "question": "A complex systems question: adding resources does not always make things better. If a city keeps widening its roads, why might congestion fail to improve, or even get worse?",
  "teaching_goal": "Help the learner understand behavioral feedback in systems: when road capacity changes, people's travel choices also change, and the final result may offset the original intervention.",
  "socratic_prompt_sequence": [
    "If wider roads make commute times shorter, might some people who previously did not drive start driving?",
    "When more people drive because driving becomes more convenient, what happens to the newly added road capacity?",
    "If each person's choice is individually rational, why can the overall system still get worse?"
  ],
  "minimum_teaching_point": "In a complex system, an intervention does not change only one variable. Widening roads lowers the cost of driving; lower travel cost induces more driving demand; that new demand can consume the added capacity.",
  "transfer_check": "If an app keeps adding notifications to increase engagement, why might users become more likely to turn notifications off? How is that similar to widening roads to reduce congestion?"
}
```

For more input, plan, and learning event examples, see [`skill/EXAMPLES.md`](skill/EXAMPLES.md).

## Vision

Socratic explores a different learning relationship: AI is not just a faster answer machine, but a learning partner that can stay with you over time, remember your misconceptions, protect your curiosity, and push you toward deeper understanding.

The hope is to help more adults recover a feeling that may have gone missing: math, physics, and science are not just sources of pressure from school. They are tools for understanding the world, and they can be lit up again.
