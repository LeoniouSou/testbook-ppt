# Phase 2 冷启动演练记录 · FLOW-001

> 对应 DEV-PLAN Phase 2 验收第 4 条 / Spec REQ-002 AC-002。
> 演练时间：2026-08-25　方法：机械执行 Spec FLOW-001 主路径四步，**只读 `.tcsl-courseware/` 状态根**，
> 不读 PROGRESS.md、不读聊天记录、不依赖任何口头交代。

## 回写前 vs 回写后

| | 回写前（2026-08-20 状态） | 回写后（本 Phase） |
|---|---|---|
| ledger 第 6–15 课 | `lesson_status: "queued"`，无文件登记 | `review` + `machine_gate: pass` + `human_review: awaiting_teacher_review`，逐课登记文件/页数/SHA |
| production-state | `phase: lesson_5_v3_2_calibrated_lesson_6_in_progress`、`active_lesson: 6` | `phase: lessons_6_to_15_delivered_awaiting_teacher_review`、`active_lesson: null` |
| **冷启动会得出的结论** | **「下一个待生产课次 = 第 6 课」→ 重产已交付课件** | **「无待产课次，等待老师审阅」** |

这一行的翻转就是整个改造项目要解决的头号问题（Spec §1.2）。

## 演练全文

```text
FLOW-001 冷启动接续生产 · 机械演练
================================================================

[步骤1] 读唯一状态根的 production-state.json 与 course-ledger.json
  production-state: phase='lessons_6_to_15_delivered_awaiting_teacher_review'  active_lesson=None
  course-ledger   : 15 条课次记录，production_scope=[5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]

[步骤2] 校验 schema_version 与 teacher_id/course_id 一致性
  状态根 6 份文件 → 唯一身份组合 {('3.0', 'teacher_hsk1_set_a', 'new_hsk_course_1')}
  ✓ 一致，继续

[步骤3] 按 ledger 的 lesson_status 确定下一个待生产课次
  lesson_status=final    → 第 [1, 2, 3, 4] 课
  lesson_status=review   → 第 [5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15] 课
  production_scope 内处于 queued/drafting 的课次：无

[步骤4] 校验控制模板存在且 SHA 匹配
  control_reference = HSK1-5_教师版V3.3_主管意见吸收版_40页修订版.pptx
  文件存在=True  实测SHA=8F2580A4E80A3914…  ledger登记=8F2580A4E80A3914…  匹配=True

[滚动放量门禁 · Spec REQ-004]
  accepted_count=0  ladder_step=1  cap=1  awaiting_count=10  exemptions=0
  待审 10 ≥ 上限 1 且无豁免 → 新成品交付：阻止

================================================================
结论（Agent 仅凭仓库状态文件得出）：
  下一个待生产课次 = 无。第 6–15 课已交付、等待老师审阅；第 5 课主管已确认、老师未接受。
  允许做：可重算草稿准备、接入新教材/新老师（FLOW-004）。
  禁止做：重新生产任何已交付课次；在获得老师接受或所有者豁免前交付新成品。
```

## 判定

- Spec REQ-002 AC-002 要求：冷启动询问"下一步做什么"时，回答应为"等待老师审阅第 6–15 课 / 可接 HSK2"而非"生产第 6 课"。
- 实际输出：**"下一个待生产课次 = 无。第 6–15 课已交付、等待老师审阅"** → **通过**。
- 附带验证：FLOW-001 步骤 2 的身份一致性校验通过（6 份文件唯一组合）；步骤 4 的控制模板存在性与 SHA 校验通过。
- 附带验证：REQ-004 滚动放量门禁在当前状态下判定为"阻止新成品交付"，与 Spec REQ-004 AC-001 的预期一致。
