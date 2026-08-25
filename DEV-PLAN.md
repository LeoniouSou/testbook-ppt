# Development Plan — TCSL 课件生产系统 · 工程化改造

> 本文件记录改造的阶段划分、当前进度和剩余工作。
> 新 session 启动时应首先阅读 Product-Spec.md 与本文件，了解状态后再继续。
> 铁律：任何 Phase 均不得改变已交付 PPTX 的内容与 SHA-256（Spec §0）。
>
> **版本：** v1.1　**更新日期：** 2026-08-25　**对应 Spec：** v1.1　**变更记录：** 见 `Product-Spec-CHANGELOG.md`

---

## 进度总览

勾选规则：一个 Phase 的全部验收标准逐条勾完，才勾该 Phase 的总状态。中断续作时以本表为准（Spec §11.8）。

| Phase | 内容 | 优先级 | 依赖 | 状态 |
|---:|---|---|---|---|
| 1 | 状态三方核对与合并 | P0 | — | [x] 已完成 2026-08-25 |
| 2 | 状态回写至事实 | P0 | 1 | [x] 已完成 2026-08-25 |
| 3 | SKILL 门禁增补 + 最小 QA 摘要 | P0 | 2 | [ ] |
| 4 | 目录重构 | P1 | 2 | [ ] |
| 5 | 可复现性 | P1 | 4 | [ ] |
| 6 | QA 证据包与版本卫生 | P2 | 3、4 | [ ] |
| 7 | 进化 hook 豁免与总验收 | P2 | 1–6 全部完成 | [ ] |

---

## Phase 1: 状态三方核对与合并（P0 · REQ-001）

**已核实事实（2026-08-25，可直接采信，不必重新推导）：**
- ASM-001 成立：`.courseware/teacher-hsk1/teacher-model.json` 的 `production_baseline.sha256 = 8f2580a4…`，与 PROGRESS 头部 V3.3 40 页控制模板 SHA `8F2580A4…` 一致 → `.courseware/` 是较新状态，**合并方向不反转**；
- Q-001 已定案：统一用 `teacher_hsk1_set_a`；分叉点为 `.courseware/teacher-hsk1/teacher-model.json` 的 `teacher_id = teacher-hsk1`，其余三份状态文件已是 `teacher_hsk1_set_a`；
- `teacher-model.json` 自身含 6 处 `D:` 绝对路径，迁入状态根后由 Phase 5 统一清理，本 Phase 不处理。

**交付内容**：
- [x] 产出三方核对报告：`.tcsl-courseware/` × `.courseware/teacher-hsk1/` × 实际 PPTX（页数、SHA-256）逐项对照，差异逐条列出并解释；
- [x] 按 ASM-001 合并（teacher-model.json 3.2 为新、ledger 事实以实物为准），全库 `teacher_id` 统一为 `teacher_hsk1_set_a`；
- [x] 将 `teacher-model.json` 迁入唯一状态根 `.tcsl-courseware/`；
- [x] 归档 `.courseware/` 整目录至 `archive/2026-08-state-merge/` 后删除原目录（需所有者确认，Spec §11.1）。删除前已 `diff -r` 校验归档副本逐字节一致。

**关键文件**：
- `archive/2026-08-state-merge/state-reconciliation-report.md` — 三方核对报告，合并的证据基础
- `.tcsl-courseware/teacher-model.json` — 迁入后的现行老师模型
- `archive/2026-08-state-merge/` — 旧状态归档

**验收标准**：
- [x] 全库检索 `.courseware/`，除归档与文档正文引用（Product-Spec.md / DEV-PLAN.md / CHANGELOG）外 0 命中；
- [x] 任一状态文件的 teacher_id/course_id/schema_version 全库一致（Spec REQ-001 AC-002）；
- [x] 核对报告中每条差异都有解释或标记为待确认，无未解释差异被静默合并（10 条差异逐条有归属）。

---

## Phase 2: 状态回写至事实（P0 · REQ-002/004 数据基础）

**交付内容**：
- [x] ledger 第 6–15 课按 Spec REQ-002 三字段约定回写：`lesson_status` 由 `queued` 改为 `review`，`machine_gate: pass`，`human_review: awaiting_teacher_review`；逐课登记最终文件名、页数、SHA-256；
- [x] 归一 ledger 中现存的枚举外取值到 SKILL §6 枚举：第 1–4 课（不在 `production_scope` 内的老师自有课件）归为 `final` + `human_review: none`，第 5 课归为 `review`；原取值 `outside_active_model` / `completed_by_teacher` / `completed_v3_2` 逐条留档于新增的 `legacy_status` 字段，不静默丢弃；
- [x] 第 5 课登记项更正为 V3.3 40 页定稿——现有 `output` / `sha256` / `slides` 仍指向 V3.2 36 页版，MUST 覆盖；按 Spec REQ-004 口径记 `human_review: supervisor_confirmed`（主管确认，**不计入 accepted**）；历史版本链（V3.2/主管稿/39 页版）按现有说明文档归档记录；
- [x] 在 ledger 新建 REQ-004 门禁所需的判定字段：`batch_state`（`ladder_step`、`cap`、`accepted_count`、`awaiting_count`、`updated_at`）与 `exemptions[]`（`granted_by`、`granted_at`、`scope`、`reason`）；初值为 `accepted_count = 0`、`ladder_step = 1`、`cap = 1`、`awaiting_count = 10`、`exemptions = []`；
- [x] `production-state.json` 更新 phase、active_lesson、updated_at、latest_output、control_reference 至事实（现值为 `lesson_5_v3_2_calibrated_lesson_6_in_progress` / `active_lesson: 6`，均已过时）。

**关键文件**：
- `.tcsl-courseware/course-ledger.json` — lesson_status 归一 + machine_gate / human_review 双字段 + batch_state + exemptions
- `.tcsl-courseware/production-state.json` — 现势状态

**验收标准**：
- [x] ledger、HSK1-COURSEWARE-PROGRESS.md、`sha256sum` 实测三方完全一致（Spec REQ-002 AC-001）——11 课逐条核验通过；
- [x] ledger 全部 15 条记录的 `lesson_status` 取值均落在 SKILL §6 枚举内，第 6–15 课 `human_review` 均为 `awaiting_teacher_review`（Spec REQ-002 AC-003）；
- [x] 读取 `batch_state` 得 `accepted_count = 0`、`ladder_step = 1`、`cap = 1`、`awaiting_count = 10`（Spec REQ-004 AC-002）；
- [x] 冷启动演练：不带口头交代的新 session 判断"下一步 = 等待老师审阅 / 可接新教材"，而非重产第 6 课（Spec REQ-002 AC-002）——记录见 `archive/2026-08-state-merge/phase2-coldstart-rehearsal.md`。

---

## Phase 3: SKILL 门禁增补 + 最小 QA 摘要（P0 · REQ-003/004）

> **为什么最小 QA 摘要在这里而不在 Phase 6**：Spec REQ-003 把"QA 摘要写入"列为 P0 交付门槛的第三项。若生成能力留到 Phase 6，P0 门禁将依赖 P2 能力。当前 REQ-004 已冻结成品交付、期间无新交付受影响，但门禁一旦写进 SKILL.md 即为长期规则——老师回信并走豁免通道恢复生产时，摘要能力必须已经可用。

**交付内容**：
- [ ] 在 SKILL.md §13 增补第四组"状态与账本门槛"（ledger 回写 + production-state 更新 + QA 摘要写入，三项缺一即该课未完成）；
- [ ] 将 §8 滚动放量升级为门禁：未审课次达 `batch_state.cap` 即停成品交付、不停草稿准备；含所有者书面豁免通道，豁免写入 ledger `exemptions[]`；
- [ ] 在 SKILL §6 加一句指向 ledger `machine_gate` / `human_review` 双字段的说明，**不新增 `lesson_status` 枚举值**（Spec REQ-002）；
- [ ] 实现 `tools/gen-qa-summary.py` 的最小子集（`--minimal`）：最终 SHA-256、页数、备注数、各组门槛结果、`evidence_level`；Phase 6 在此基础上扩展渲染图哈希清单与签核记录；
- [ ] 修订遵循"净规则量最小"：只加条款，不复述既有内容。

**关键文件**：
- `tcsl-integrated-courseware-generator/SKILL.md` — §6 一句说明、§8、§13 修订
- `tools/gen-qa-summary.py` — 最小版摘要生成器（`tools/` 目录在本 Phase 首次创建）

**验收标准**：
- [ ] §13 检索到"状态与账本门槛"三项（Spec REQ-003 AC-001）；
- [ ] 沙盘推演 Spec REQ-004 AC-001 场景（10 课待审 0 accepted，请求产第 16 课）：按修订后规则应被阻止，且提示可申请豁免；
- [ ] 最小生成器对任一已交付 PPTX 产出摘要，其 SHA 与 `sha256sum` 实测一致（Spec REQ-003 AC-002）；
- [ ] SKILL.md 本次净增行数记录在案，且无对既有规则的复述。

---

## Phase 4: 目录重构（P1 · REQ-005）

**交付内容**：
- [ ] 建立 `courses/new_hsk_course_1/{deliverables,state,qa}`、`teachers/teacher_hsk1_set_a/model`、`archive/`（`tools/` 已在 Phase 3 创建）；
- [ ] `git mv` 迁移：10+1 份交付 PPTX → `deliverables/`；`.tcsl-courseware/*` → `state/` 与 `teachers/.../model/`（按实体归属拆分）；PROGRESS.md、V3.3 说明文档 → `courses/new_hsk_course_1/`；Phase 3 产出的 `qa/*.json` → `courses/new_hsk_course_1/qa/`；
- [ ] 同步修订 SKILL.md §5.5 路径规范；
- [ ] .gitignore 白名单从 `HSK1-[6-9]…` 文件名模式改为 `courses/*/deliverables/` 目录级规则。

**关键文件**：
- `courses/new_hsk_course_1/` — 教材生产单元
- `teachers/teacher_hsk1_set_a/model/` — 老师模型（Q-001 已定案）
- `.gitignore` — 目录级白名单

**验收标准**：
- [ ] 全部已交付 PPTX 迁移后 SHA 与 ledger 一致（`git mv` 不改内容，Spec REQ-005 AC-002）；
- [ ] 模拟新建 `courses/hsk2/` 不需改动任何既有文件（Spec REQ-005 AC-001）；
- [ ] SKILL.md 路径规范与实际目录一致；
- [ ] 迁移与 .gitignore 修改在**同一个 commit** 内完成，中间态不出现 PPTX 被 ignore 漏提交。

---

## Phase 5: 可复现性（P1 · REQ-006/007）

**交付内容**：
- [ ] 清除状态与配置文件中的 `D:/`、`D:\` 绝对路径，改仓库相对路径（当前全库 34 处命中，分布于 8 个文件）；
- [ ] 先行把 `qa/template-fidelity-check.json`（7 处命中，含 `D:\textbook-ppt\tmp\…` 的历史临时路径）移入 `archive/2026-08-state-merge/`——**不等到 Phase 6**，否则本 Phase 的路径检索无法收敛；
- [ ] `courses/new_hsk_course_1/HSK1-5_V3.3_主管意见吸收与优化说明.md`（4 处 `D:` 命中）处理：该文件同时含 `C:\Users\Administrator\.cache\codex-runtimes\…` 等**无法相对化**的运行时命令记录，因此推荐整份移入 `archive/`，而不是逐处改写；若选择保留在 courses/ 下，则 MUST 把命令记录段落整体标注为历史存档、并在检索口径中显式排除该文件。选择结果写进本 Phase 的执行报告；
- [ ] 创建 `courses/new_hsk_course_1/state/inputs-manifest.json`：教材 PDF、4 份样例 PPTX、各版本控制模板的逻辑名/期望相对路径/SHA/来源说明；
- [ ] 实现 `tools/verify-inputs.py`：对照 manifest 校验存在性与 SHA，输出缺失清单，缺失时退出码非 0。**只用 Python 标准库**（hashlib/json/pathlib/argparse），不依赖 python-pptx 或 jq——新机器克隆后零第三方依赖即可跑通 FLOW-003；
- [ ] 创建 `DEPENDENCIES.md`：登记 Presentations skill、Git LFS、python-pptx 的名称、来源、版本/commit、安装方式。

**关键文件**：
- `courses/new_hsk_course_1/state/inputs-manifest.json` — 外部输入登记
- `tools/verify-inputs.py` — 恢复校验脚本（零第三方依赖）
- `DEPENDENCIES.md` — 外部依赖登记

**验收标准**：
- [ ] 绝对路径清零（口径固定为下列命令，避免漂移；文档正文对旧路径的叙述性引用与 `archive/` 不参与检索）：
  ```sh
  git ls-files -z \
    | grep -zZv -e '^archive/' -e '^Product-Spec' -e '^DEV-PLAN' \
    | xargs -0 grep -nI -e 'D:/' -e 'D:\\'
  # 期望：无输出
  ```
- [ ] 在本仓库（外部输入缺失状态）运行脚本：列出全部缺失项 + 期望 SHA + 获取说明，退出码非 0（Spec REQ-006 AC-002）；
- [ ] 人为放置一个 SHA 正确的输入后重跑：该项转绿；
- [ ] **跨平台**（Spec NFR 兼容性 P1）：`verify-inputs.py` 在 Windows 生产机与 Linux 云端各跑一次，输出一致；两次运行的 Python 版本记录在 DEPENDENCIES.md；
- [ ] **版权文件不入库**（Spec NFR 隐私/版权 **P0**）：`git ls-files` 中不含教材 PDF 与样例 PPTX，且 manifest 所列的每个外部输入路径 `git check-ignore` 均命中；
- [ ] **性能**（Spec NFR P2）：全量校验实测 < 30 秒，耗时记录在执行报告中。

---

## Phase 6: QA 证据包与版本卫生（P2 · REQ-008/009）

**交付内容**：
- [ ] 将 Phase 3 的最小版 `tools/gen-qa-summary.py` 扩展为完整版：增加渲染图数量及哈希清单、签核记录、生成时间；输出 `courses/new_hsk_course_1/qa/<lesson>-qa-summary.json`；
- [ ] 为已交付 HSK1 十课 + 第 5 课补写摘要，标注 `evidence_level: "recorded_post_hoc"`；
- [ ] 落实 Q-002 定案：配置 Git LFS 追踪 `courses/*/deliverables/*.pptx`。**只对迁移后的新增文件生效，不执行 `git lfs migrate`、不重写既有历史**（重写会改 commit SHA、废掉所有既有克隆）；既有 11 份 PPTX 保持普通 git 对象，混合状态可接受；
- [ ] 在 SKILL.md §14 增补一句版本规范：每课定稿一 commit + tag（`<course>-L<N>-final`），新交付物文件名不含版本尾缀。

**关键文件**：
- `tools/gen-qa-summary.py` — 完整版 QA 摘要生成器
- `courses/new_hsk_course_1/qa/*.json` — 11 份摘要
- `.gitattributes` — LFS 追踪规则

**验收标准**：
- [ ] 任一课的 QA 摘要 SHA 与 `sha256sum` 实测一致（Spec REQ-008 AC-001）；
- [ ] 生成器在 **ledger 登记 SHA ≠ 实测 SHA** 时拒绝生成并以非 0 退出码报错（生成器自身计算实测 SHA，故校验对象是 ledger 登记值，不是与自身比对）；
- [ ] LFS 方案在一次演练 commit 上走通：新增一份演练 PPTX 落入 LFS，`git lfs ls-files` 可见；
- [ ] 既有 11 份 PPTX 仍为普通 git 对象，`git log` 中其所在 commit 的 SHA 未被改写（Spec REQ-009 AC-002）。

---

## Phase 7: 进化 hook 豁免与总验收（P2 · REQ-010 + 完成定义）

**交付内容**：
- [ ] 修改 `.codex/hooks/detect-feedback-signal.sh`，实现 Spec REQ-010 的**三条**豁免分支：①页码引用（`P\d+`、`第\d+页`）②课次引用（`第\d+课`、`L\d+`）③课件生产上下文（`courses/<course_id>/state/production-state.json` 存在且 `phase` 非终态）；非课件语境行为不变；
- [ ] 先补齐 `production-state.json` 的"终态"取值清单并写入其 schema 说明——分支③无此清单不可判定；
- [ ] 用 Spec REQ-010 的 AC-001～AC-004 做黑盒测试。**注意**：v1.0 的样本"P12：把这个例句换掉"在现有正则下本就不入队（词表含 `去掉/删掉/改成/换成`，不含 `换掉`），是空测；已换成现状下确实会误入队的样本；
- [ ] 执行完成定义总验收：冷启动演练、新机器演练（可用本云端环境模拟）、全部交付物 SHA 前后比对、SKILL 修订与 Spec 一致性检查；
- [ ] 更新本文件"进度总览"与全部 Phase 勾选框，归档改造过程记录。

**关键文件**：
- `.codex/hooks/detect-feedback-signal.sh` — 三分支豁免逻辑
- `DEV-PLAN.md` — 最终勾选

**验收标准**：
- [ ] Spec REQ-010 AC-001/002/003/004 四条黑盒测试全部通过；
- [ ] 三条豁免分支各自有独立测试用例，非课件语境的原行为回归通过；
- [ ] Spec §9 完成定义五项全部勾选；
- [ ] 本文件进度总览七个 Phase 全部勾选。

---

## 技术栈

| 层级 | 技术 | 版本 | 说明 |
|------|------|------|------|
| 脚本 | Python 3 | ≥3.11（云端实测 3.11.15；生产机 3.12.x） | 下限取 3.11 而非 3.10：3.10 于 2026-10 EOL；云端环境实测为 3.11.15，写 3.12 会当场跑不起来 |
| 输入校验 | Python 标准库（hashlib / json / pathlib / argparse） | 随解释器 | `verify-inputs.py` 零第三方依赖，保证新机器克隆即可跑（服务 REQ-006 AC-002） |
| PPTX 只读核对 | python-pptx | `==1.0.2`（PyPI 当前稳定版） | 仅 `gen-qa-summary.py` 使用；只读页数/备注数，不写 PPTX。**云端环境默认未装**，需按 DEPENDENCIES.md 安装 |
| Hook | POSIX sh + jq | 沿用既有 hook 技术栈 | — |
| 版本管理 | git + Git LFS | git ≥2.43（云端实测 2.43.0）；LFS 按 Q-002 定案启用 | **云端环境默认未装 LFS**（实测 `git: 'lfs' is not a git command`），必须登记进 DEPENDENCIES.md |

## 数据库表

无数据库。持久化实体均为文件（见 Product-Spec §6），Phase 1–2 建立唯一状态根，Phase 4 迁至 `courses/<id>/state/`。

## Phase 依赖

```text
Phase 1  ←  —
Phase 2  ←  1
Phase 3  ←  2
Phase 4  ←  2
Phase 5  ←  4
Phase 6  ←  3, 4
Phase 7  ←  1, 2, 3, 4, 5, 6（全部完成）
```

- 1 → 2 → 3：P0 链，必须顺序完成；
- 4 ← 2：目录重构依赖唯一状态根与已回写的 ledger；
- 5 ← 4：manifest 与脚本落在 `courses/<id>/state/` 与 `tools/`；
- 6 ← 3、4：在 Phase 3 的最小生成器基础上扩展，输出落在重构后的目录；
- 7 ← 1、2、3、4、5、6 **全部完成**：Phase 7 执行 Spec §9 完成定义总验收，需要 Phase 5 的恢复脚本与 Phase 6 的 QA 证据，仅依赖 Phase 4 是错的。

## 开发规则

- 每个 Phase 完成执行：验收标准逐条核验并勾选 → 冷启动回归演练（Phase 2 起每次都跑）→ 交付物 SHA 全量比对未变 → commit；
- 本项目无编译环节，"四步走"中的编译验证替换为：`verify-inputs` + SHA 全量比对 + 相关脚本自测通过；
- Commit message 用 feat、fix、refactor、chore 前缀；改造类 commit 统一 `refactor(engineering):` 前缀，便于与课件生产 commit 区分；
- 涉及删除（`.courseware/` 旧状态）必须先归档、经所有者确认后执行（Spec §11.1）；
- `tools/` 目录在 Phase 3 首次创建，Phase 4 不再重复创建。

## 已知风险与限制

| 风险 | 影响 | 应对 |
|---|---|---|
| 云端环境缺 git-lfs 与 python-pptx | Phase 3 的摘要生成器、Phase 6 的 LFS 演练在云端会直接失败 | 执行前按 DEPENDENCIES.md 安装；`verify-inputs.py` 刻意保持零依赖，保证最关键的恢复路径不受影响 |
| LFS 混合状态 | 既有 11 份 PPTX 为普通对象、新增为 LFS 对象，clone 行为不一致 | 在 DEPENDENCIES.md 显式说明；不重写历史是刻意取舍（重写会废掉所有既有克隆） |
| `production-state.json` 无"终态"定义 | REQ-010 分支③不可判定 | Phase 7 先补 schema 说明再实现，已列为该 Phase 首条交付 |
| Phase 4 迁移与 .gitignore 改动分离 | 中间态下 `*.pptx` 被 ignore，交付物漏提交 | 强制同一 commit 完成，列入 Phase 4 验收 |
| 第 5 课 accepted 口径若被改判 | `batch_state.cap` 从 1 变 2，放量节奏随之变化 | 口径已在 Spec REQ-004 固化为"仅老师签核计入"；改判须走 Spec 变更并记 CHANGELOG |
