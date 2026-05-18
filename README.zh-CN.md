# Socratic

![Skill Version](https://img.shields.io/badge/skill-v0.2.0-2ea44f)
![Status](https://img.shields.io/badge/status-early%20skill%20spec-blue)
![Runtime](https://img.shields.io/badge/runtime-platform%20neutral-lightgrey)

**一个平台无关的 AI 学习教练 Skill：每天一个真正有分量的问题，重建数学、物理和科学直觉。**

[English](README.md)

Socratic 不是课程平台，也不是“直接给答案”的聊天机器人。它把用户模糊的学习愿望转成可持续推进的学习路径：先抛出一个值得认真思考的问题，让用户表达直觉，再通过追问、提示、最小讲解和迁移题，帮助用户真正理解一个概念。

这个仓库当前发布的是可复用的 Agent Skill，核心产物位于 [`skill/`](skill/)。

## 为什么做这个项目

AI 会继续自动化确定性任务，但人仍然需要理解复杂世界、迁移知识、判断模型是否可靠。很多成年人想重新学习数学、物理或科学思维，却很难再从教材第一章开始。

Socratic 的假设是：成年人缺的往往不是更多内容，而是一个低压力、低摩擦、但足够有智识密度的入口。

```text
一个复杂系统问题：不是加资源就一定变好。

如果一个城市道路越修越宽，为什么拥堵有时反而没有缓解，甚至更严重？

这个问题会带你理解反馈回路、诱导需求和系统建模：当我们改变一个系统时，系统里的人也会改变行为。
```

## 核心能力

- **苏格拉底式引导**：用户第一次尝试前不直接给标准答案，而是用追问、反例和分层提示帮助用户自己推进。
- **个性化学习路径**：从用户动机、恐惧、目标身份和当前能力出发，生成下一步学习计划。
- **知识图谱驱动**：用概念节点、前置关系、常见误解和示例问题组织学习内容。
- **结构化学习状态**：记录掌握证据、误解、复习时机和学习事件，而不是只依赖聊天历史。
- **主动与按需两种入口**：支持每日轻量推送，也支持用户主动要求继续学习。
- **平台无关 Skill 包**：教学策略、状态模型和运行手册与具体系统解耦，可接入 OpenClaw 等主动式 agent。

## 适合谁

Socratic 首先面向那些想重新开始学习，但一想到“系统学习”就感到压力的人。

- 你可能离开数学、物理和科学训练很久了，但仍然希望重新理解这个世界。
- 你不想再被考试、排名、公式堆砌和“从第一章开始”劝退。
- 你希望 AI 不只是告诉你答案，而是陪你把自己的直觉训练得更清楚。
- 你每天只有十几分钟，但想长期积累真正可迁移的思考能力。
- 你正在做学习产品或教育工具，想参考一种更克制、更重视长期学习状态的交互方式。

当前仓库提供的是 Socratic 的核心教学能力包，不是一个完整 App。它可以接入 OpenClaw 这类主动式 agent，也适合被产品团队改造成自己的学习体验。

## 快速开始

如果你已经有一个支持 Skills 的 OpenClaw workspace，最快的方式是把下面这段 prompt 交给具备 workspace 权限的 agent：

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

更完整的安装和更新说明见 [`INSTALL.md`](INSTALL.md)。

## 它如何工作

每次学习都尽量形成一个短闭环：

1. 读取学习目标、用户画像、近期学习事件和相关知识节点。
2. 选择下一步动作：校准、引入新概念、修复误解、复习、综合练习或降低抽象度。
3. 生成一个具体问题、一条提示阶梯、一个最小教学点和一个迁移检查。
4. 与用户进行苏格拉底式对话，先观察直觉，再逐步释放提示。
5. 根据复述、迁移题、反例回应等证据更新学习状态。
6. 追加学习事件，给未来 agent 留下可检索、可迁移的状态。

更完整的编排流程见 [`skill/RUNBOOK.md`](skill/RUNBOOK.md)。

## 示例：下一课计划

```json
{
  "mode": "proactive",
  "lesson_type": "new_concept",
  "target_nodes": ["complex-systems", "feedback-loops", "induced-demand"],
  "question": "一个复杂系统问题：不是加资源就一定变好。如果一个城市道路越修越宽，为什么拥堵有时反而没有缓解，甚至更严重？",
  "teaching_goal": "帮助用户理解系统中的行为反馈：当道路容量改变，人的出行选择也会改变，最终结果可能抵消原本的干预效果。",
  "socratic_prompt_sequence": [
    "如果道路变宽后通勤时间短了，会不会有一部分原本不开车的人开始开车？",
    "当更多人因为开车更方便而上路，原本新增的道路容量会发生什么变化？",
    "如果每个人的选择都是理性的，为什么整个系统的结果仍然可能变差？"
  ],
  "minimum_teaching_point": "复杂系统里，干预不会只改变一个变量。道路变宽会降低出行成本，出行成本下降会诱导更多驾驶需求，新增需求又会反过来吞掉新增容量。",
  "transfer_check": "如果一个 App 为了提升活跃度不断增加推送，为什么用户反而可能更快关闭通知？这个问题和修路缓解拥堵有什么相似之处？"
}
```

更多输入、计划和学习事件样例见 [`skill/EXAMPLES.md`](skill/EXAMPLES.md)。

## 愿景

Socratic 想探索一种新的学习关系：AI 不再只是更快的答案机器，而是一个能长期陪伴、能记住你的误解、能保护你的好奇心、也能把你推向更深处的学习伙伴。

我们希望它最终帮助更多成年人重新获得一种感觉：数学、物理和科学不是学校里留下的压力，而是一套仍然可以被重新点亮的理解世界的工具。
