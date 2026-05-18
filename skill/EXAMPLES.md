# Examples

## Example: Initial Calibration Profile

After initial calibration, persist the learner-level profile before planning proactive pushes:

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
    "max_push_length": "one_question",
    "runtime_schedule_ref": "cron:socratic-daily-learning"
  }
}
```

If the user has not opted in, set `notification_preferences.enabled` to `false` and do not create a schedule.

## Example: Plan Next Lesson

Input context:

```json
{
  "mode": "proactive",
  "learning_goal": {
    "goal_id": "rebuild-scientific-intuition",
    "title": "重建数学和物理直觉",
    "preferred_style": "生活问题驱动，低计算，逐步形式化",
    "current_phase": "build_intuition"
  },
  "weak_nodes": ["temperature-vs-heat"],
  "active_misconceptions": ["cold-as-substance"],
  "due_reviews": []
}
```

Expected lesson plan:

```json
{
  "mode": "proactive",
  "lesson_type": "misconception_repair",
  "target_nodes": ["heat-transfer"],
  "question": "为什么冬天摸铁门把手会比摸木头桌子更冷？它们明明在同一个房间里。",
  "teaching_goal": "帮助用户区分温度、冷热感觉和热量流动速度。",
  "expected_intuitions": [
    "铁本身更冷",
    "铁会吸走手上的热",
    "木头比较保暖"
  ],
  "socratic_prompt_sequence": [
    "如果铁和木头在同一个房间放了一整晚，你觉得它们温度真的不同吗？",
    "你的手觉得冷，可能是手上的热量发生了什么变化？",
    "如果一种材料更快把热从手带走，你的感觉会怎样？"
  ],
  "minimum_teaching_point": "温度相同不代表触感相同。铁更容易把热从手传走，所以手降温更快，感觉更冷。",
  "transfer_check": "为什么瓷砖地板通常比地毯摸起来更凉？",
  "completion_criteria": [
    "用户不再说铁本身温度更低",
    "用户能用热量从手流走更快解释类似现象"
  ],
  "state_update_hints": {
    "if_transfer_success": "increase mastery for heat-transfer and temperature-vs-heat",
    "if_transfer_partial": "schedule another material-comparison transfer question"
  }
}
```

## Example: End Lesson Event

```json
{
  "event_id": "event_20260514_heat_transfer_001",
  "goal_id": "rebuild-scientific-intuition",
  "event_type": "lesson_completed",
  "created_at": "2026-05-14T00:00:00Z",
  "mode": "proactive",
  "target_nodes": ["heat-transfer"],
  "lesson_result": "partial_mastery",
  "summary_for_future_agent": "用户已经意识到铁不一定温度更低，并能说出热从手流走更快，但对热传导率这个命名还不熟。",
  "evidence": [
    {
      "type": "transfer_question",
      "result": "partial",
      "summary": "用户能解释瓷砖比地毯凉，但仍使用'吸冷'表达。"
    }
  ],
  "new_misconceptions": [],
  "resolved_misconceptions": ["metal-is-colder"],
  "recommended_next_action": "transfer_practice"
}
```

## Example: On-Demand Continuation

If the user asks "我今天还想继续学", do not dump a syllabus.

Good response pattern:

```text
可以，我们继续沿着刚才的概念走一步。刚才你已经抓到了“热从手流走的速度”这个关键点。

下一个小问题：为什么同样是金属勺子，放进热汤里很快烫手，但木勺子不会那么快？
```

Then use the same Socratic flow and update state again after the interaction.

