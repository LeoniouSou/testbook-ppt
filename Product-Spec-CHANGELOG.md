# 变更记录

本文件记录 `Product-Spec.md` 的迭代变更历史。同一次修订中 `DEV-PLAN.md` 的联动改动一并附记，便于冷启动时对照。

---

## [v1.1] - 2026-08-25

本次修订源于一次对 v1.0 Spec 与 DEV-PLAN 的外部校审，加上逐条回仓库核验。核验推翻了「两份文档未纳入 Git、命名错误」这一条——实际 `Product-Spec.md` 与 `DEV-PLAN.md` 均已跟踪于 commit `5f3f0cd`，命名符合 AGENTS.md 约定；确认了其余问题；并补出四个校审未发现的缺陷（枚举冲突、门禁无数据落点、REQ-010 空测、Phase 5 验收不可达）。

### 新增

- REQ-002 新增「状态字段约定」小节：明确 `lesson_status` / `machine_gate` / `human_review` 三字段分工表。起因是 SKILL §6 已把 `lesson_status` 枚举定死为 `queued / drafting / review / accepted / final`，其中不含 v1.0 要求写入的 `awaiting_teacher_review`，与 NFR「SKILL 净增量最小」直接冲突。
- REQ-002 新增 AC-003：ledger 全部 15 条记录的 `lesson_status` 必须落在 SKILL §6 枚举内，第 6–15 课 `human_review` 均为 `awaiting_teacher_review`。
- REQ-003 新增 AC-002：交付完成时三项状态门槛均存在，且摘要可由仓库内脚本重新生成。
- REQ-004 新增「`accepted` 的判定口径」小节：`accepted` 仅指老师签核动作，教学主管确认、内部终验、机器门禁 PASS 均不计入。第 5 课记为 `human_review = supervisor_confirmed`、不计入阶梯，故当前 `accepted_count = 0`、批次上限 = 1。口径不清会让第 5 课被误算成 accepted，把放量阶梯从 1 课直接放到 2 课。
- REQ-004 新增 ledger 判定字段 `batch_state`（`ladder_step`/`cap`/`accepted_count`/`awaiting_count`/`updated_at`）与 `exemptions[]`，并新增 AC-002。v1.0 只写了门禁规则，没有任何字段承载阶梯计数、批次上限与豁免记录，Phase 3 无法实现为确定性检查。
- REQ-004 新增「未获任何回应」的可判定定义。
- REQ-009 新增「规则」小节：LFS 只追踪迁移后的新路径，禁止 `git lfs migrate` 重写既有历史；新增 AC-002 校验既有 11 份 PPTX 仍为普通 git 对象、commit SHA 未被改写。
- REQ-010 新增三条触发分支的判定方式表，并把「课件生产上下文」定义为可判定条件（读仓库内 `production-state.json` 的 `phase`）。hook 为 UserPromptSubmit、只能读 stdin 的 `.prompt`，v1.0 的措辞不可实现。
- REQ-010 新增 AC-004：语境分支不生效时的回归用例。
- §6.1 新增实体 Batch State；§6.3 新增一条 `lesson_status` 取值限制规则。
- 文档头新增版本行（v1.1 / 2026-08-25）。

### 修改

- REQ-002 行为：「改为反映事实的终态」改为「改为反映事实的状态」，并补入第 5 课登记项由 V3.2 36 页版更正为 V3.3 40 页定稿——现有 `output` / `sha256` / `slides` 三字段仍指向已被取代的旧版。
- REQ-002 规则：`lesson_status` 目标值由 `awaiting_teacher_review` 改为 `review`；`machine_gate` / `human_review` 双字段由 SHOULD 升为 MUST；新增枚举外取值归一要求（`completed_by_teacher`、`outside_active_model`、`completed_v3_2`）。
- REQ-003 规则：新增「QA 摘要最小子集随本需求落地为 P0」，解除 v1.0 中 P0 门禁依赖 P2 能力的倒置；并写明门禁即时生效、无过渡期。
- REQ-004 规则：阶梯计数依据由「已 accepted 课次数」改为「`human_review = teacher_accepted` 的课次数」；书面豁免通道由 SHOULD 升为 MUST（没有豁免通道时门禁无法解除，会把生产线永久锁死）。
- REQ-009 行为：Git LFS 与 tag+Release 的「二选一」改为定案选 Git LFS。
- REQ-010 验收标准：AC-001 样本由「P12：把这个例句换掉」换为「P8：这一页的图换成清晰的」，AC-002 换为「第9课第7页的学生指令改成口语一点」。原样本在现有正则下本就不入队（词表含 `去掉/删掉/改成/换成`，不含 `换掉`），属空测，改不改代码都能通过。原 AC-002 保留为 AC-003。
- DEP-004 Git LFS：「是否必需」由 No 改为 Yes，备注补充云端环境默认未安装。
- §10.1 ASM-001 / ASM-002：补记已核实结论——`.courseware/teacher-hsk1/teacher-model.json` 的 `production_baseline.sha256 = 8f2580a4…` 与 PROGRESS 头部 V3.3 40 页控制模板 SHA 一致，假设成立、合并方向不反转；ledger 全库无 `accepted` 取值。
- §10.2 Q-001：定案统一用 `teacher_hsk1_set_a`。Q-002：定案 Git LFS，且不重写历史。

### 删除

- 删除 REQ-009 中 tag+Release 备选方案的表述（Q-002 定案后不再保留二选一占位，符合 dev-planner「无占位符」要求）。
- 删除 REQ-010 中「或当前 session 处于课件生产上下文标记」这一不可实现的模糊措辞，改为可判定的文件条件。

### DEV-PLAN.md 联动修订（同日，v1.1）

- 新增「进度总览」表与全文 62 个勾选框。此前全文 0 个勾选框，而 Spec §11.8 声称改造进度记录在 DEV-PLAN 勾选项中、可中断续作。
- Phase 3 更名为「SKILL 门禁增补 + 最小 QA 摘要」，把最小版 `tools/gen-qa-summary.py` 从 Phase 6 前移至此；`tools/` 目录改为 Phase 3 首次创建。
- Phase 2 补入 `lesson_status` 回写、枚举归一、第 5 课登记更正、`batch_state` / `exemptions` 建立，并补三条对应验收。
- Phase 5 重写「`D:` 路径 0 命中」验收：固定检索命令、显式排除文档正文与 `archive/`、把 `qa/template-fidelity-check.json`（7 处命中）的归档从 Phase 6 提前到 Phase 5。原验收在 Phase 5 时点不可能通过。
- Phase 5 补入三条此前缺失的 NFR 验收：跨平台、版权文件不入库（P0）、全量校验 < 30 秒；并规定 `verify-inputs.py` 只用 Python 标准库，保证新机器零依赖可跑。
- Phase 6 验收改写：由「对 SHA 不匹配的 PPTX 拒绝生成」改为「ledger 登记 SHA ≠ 实测 SHA 时拒绝生成」。原措辞逻辑不自洽——生成器自己计算 SHA，永远不会与自身不匹配，照原文实现会得到一个恒真的检查。
- Phase 7 补入 REQ-010 第三条分支，以及 `production-state.json` 终态清单的前置定义。
- Phase 依赖由「6、7 依赖 4 的目录结构」改为显式依赖图，Phase 7 依赖 1–6 全部完成（总验收需要 Phase 5 的恢复脚本与 Phase 6 的 QA 证据）。
- 技术栈改为实测值：Python 下限 ≥3.11（云端实测 3.11.15；3.10 于 2026-10 EOL）、python-pptx `==1.0.2`、git 2.43.0 且云端无 `git lfs` 子命令。
- 新增「已知风险与限制」表 5 条。
- Phase 1 补入已核实事实（ASM-001 成立、Q-001 定案、teacher-model.json 含 6 处 `D:` 路径），减少 Phase 1 的重复推导。

---

## [v1.0] - 2026-08-25

- 初始版本：10 条功能需求（REQ-001～REQ-010）、7 项范围、4 条用户流程、5 条完成定义；配套 DEV-PLAN 划分 7 个 Phase。
