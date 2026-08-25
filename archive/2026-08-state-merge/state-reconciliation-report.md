# 状态三方核对报告 · 2026-08 状态合并

> 对应 DEV-PLAN Phase 1（P0 · Spec REQ-001）。本报告是合并动作的证据基础。
> 生成时间：2026-08-25　执行环境：Linux 云端（Python 3.11.15，git 2.43.0）

---

## 1. 核对范围与方法

三方为：

| 代号 | 对象 | 说明 |
|---|---|---|
| **A** | 实际交付物 | 仓库根目录 11 份 PPTX，逐份实测 SHA-256、幻灯片数、备注页数 |
| **B** | `HSK1-COURSEWARE-PROGRESS.md` | 人工维护的进度文档 |
| **C** | `.tcsl-courseware/`（schema 3.0） | SKILL 规定的状态根，5 份文件 |
| **D** | `.courseware/teacher-hsk1/`（contract 3.2） | 运行时另建的第二套状态，1 份文件 |

方法：SHA-256 用 `hashlib` 全文件计算；页数与备注数用 `zipfile` 统计 `ppt/slides/slideN.xml`
与 `ppt/notesSlides/notesSlideN.xml` 条目数（不依赖 python-pptx，云端环境未安装）。

---

## 2. A 组：实际交付物实测清单

| 文件 | 页 | 备注 | MB | SHA-256（实测） |
|---|---:|---:|---:|---|
| HSK1-5_教师版V3.3_主管意见吸收版_40页修订版.pptx | 40 | 40 | 7.65 | `8F2580A4E80A39149A4D7E57C4617A3194D4A433A017E0BA17BC0DB679F5A95C` |
| HSK1-6_教师版.pptx | 30 | 30 | 6.02 | `D071FD76822DBC8A677EF0596456E1B837B96BA1A0E4612F20198AC9577F575A` |
| HSK1-7_教师版.pptx | 30 | 30 | 0.90 | `E98952ABE09030AAC7103058CFE8E2709F122A37AD6BD64FE5AE929580B4A576` |
| HSK1-8_教师版.pptx | 30 | 30 | 6.45 | `622F30D3E68E900F2A1D82B7B0CAE9B4F6D5D59305C1687A9218B03D7029255D` |
| HSK1-9_教师版.pptx | 30 | 30 | 2.24 | `DA509F0057645F5CAC707852A98E05E8F4ACE314BF15F8B2BBDC2CC6D2FE1DFD` |
| HSK1-10_教师版.pptx | 30 | 30 | 2.04 | `5D06E446D23B1038741A65B8E5298CF9FF639F142853BCDAC80BEA133E5DD551` |
| HSK1-11_教师版.pptx | 30 | 30 | 2.31 | `060B291AA0F95578A67B0A44B8D02CEC2B256D85E9E37EFAFFA34C93D5C7B1CF` |
| HSK1-12_教师版.pptx | 30 | 30 | 2.69 | `512C0FAB8134AD3A11AF48347D672323B00E040F552D08AA3C63840427B59076` |
| HSK1-13_教师版.pptx | 30 | 30 | 2.35 | `E9753378A84FAF7DEA18C84BC818B35E6F23047D528F00DD16F4D72FA9641E07` |
| HSK1-14_教师版.pptx | 30 | 30 | 2.98 | `671C3B8C9370FAF99745A4927120CBDB7683A121A25A6A49DC2297F22AC31035` |
| HSK1-15_教师版.pptx | 30 | 30 | 2.55 | `2D3CA360563F7068438F645C9127DEB8A4B4B63A77B675F600D221132EFA07AC` |

每份 PPTX 的备注页数与幻灯片数相等，与 PROGRESS 记录的"逐页备注完整"一致。

---

## 3. B 组：PROGRESS.md 与实物比对

| 课次 | PROGRESS 记的页数 / SHA | 实测 | 结论 |
|---:|---|---|---|
| 6–15 | 各 30 页，逐课 SHA-256 | 完全相同 | **10/10 一致** |
| 5（控制模板） | 40 页，`8F2580A4…` | 完全相同 | **一致** |

> **PROGRESS.md 是当前唯一与事实 100% 吻合的记录。** 它没有任何一处需要更正——
> 需要更正的是两套机器状态库。

---

## 4. 差异清单

每条差异给出：现象 → 判定依据 → 处理决定 → 归属 Phase。无一条被静默合并。

### D-01 teacher_id 分叉（已处理）
- **现象**：C 组 5 份文件全部为 `teacher_hsk1_set_a`；D 组为 `teacher-hsk1`。
- **依据**：Spec Q-001 已定案取 `teacher_hsk1_set_a`（信息量更大，且改动面 5:1）。
- **决定**：迁入时改为 `teacher_hsk1_set_a`。**本 Phase 已执行。**

### D-02 版本号维度不同，非冲突（无需处理）
- **现象**：C 组 `schema_version: "3.0"`；D 组 `model_version: 3` + `courseware_contract_version: "3.2"`。
- **依据**：前者是状态文件 schema 版本，后者是课件生产契约版本，属不同维度。
- **决定**：并存。迁入的 teacher-model.json 补 `schema_version: "3.0"` 以满足 REQ-001 AC-002，
  原有两个版本号原样保留。**本 Phase 已执行。**

### D-03 ledger 第 5 课登记指向已被取代的 V3.2，且该文件不在仓库
- **现象**：`course-ledger.json` 第 5 课 `output` 指向 `HSK1-5_教师版V3.2备注优化版.pptx`、
  `sha256: 3F6B89C5…`、`slides: 36`。仓库中**不存在**该文件（未入库）。
- **依据**：D 组 `production_baseline` 指向 V3.3 40 页版（`8f2580a4…`），与 A 组实物、B 组
  PROGRESS 三方一致。**这是 ASM-001 成立的直接证据**，合并方向不反转。
- **决定**：以 V3.3 为准，ledger 第 5 课 `output`/`sha256`/`slides` 覆盖。→ **Phase 2**

### D-04 ledger 第 6–15 课停在 `queued`、零文件登记
- **现象**：10 条记录均为 `lesson_status: "queued"`，无 `output` / `sha256` / `slides`。
- **依据**：A 组 10 份实物存在，SHA 与页数与 B 组 PROGRESS 完全一致。
- **决定**：ledger 落后事实 10 课，按 Spec REQ-002 三字段约定回写
  （`lesson_status: review` + `machine_gate: pass` + `human_review: awaiting_teacher_review`）。→ **Phase 2**
- **影响**：这正是"新机器冷启动会重产第 6 课"的根因。

### D-05 ledger `lesson_status` 使用 SKILL §6 枚举外的取值
- **现象**：第 1 课 `outside_active_model`、第 2–4 课 `completed_by_teacher`、第 5 课 `completed_v3_2`。
  SKILL §6 定义的枚举为 `queued / drafting / review / accepted / final`。
- **依据**：Spec v1.1 REQ-002 明确不扩展枚举，细分语义走 `human_review`。
- **决定**：归一到枚举内，原语义转记 `human_review`。→ **Phase 2**

### D-06 production-state.json 整体停在"第 6 课进行中"
- **现象**：`phase: "lesson_5_v3_2_calibrated_lesson_6_in_progress"`、`active_lesson: 6`、
  `latest_output` 与 `control_reference` 均指 V3.2、`updated_at: 2026-08-20`。
- **依据**：A/B 两组均证明第 6–15 课已交付，最后更新应在 2026-08-22 之后。
- **决定**：整体更新至事实。→ **Phase 2**

### D-07 两套库对"当前控制模板"的记录互相矛盾
- **现象**：C 组 `control_reference` = V3.2；D 组 `control_template` = V3.3。
- **依据**：A 组仓库内只有 V3.3（40 页，`8F2580A4…`），V3.2 文件根本不存在；B 组 PROGRESS
  头部写明"最新控制模板：V3.3 40 页"。
- **决定**：取 V3.3。C 组该字段在 Phase 2 更正。→ **Phase 2**

### D-08 两套库共 14 处 Windows 绝对路径
- **现象**：teacher-model 6 处、course-ledger 2 处、production-state 2 处、sample-manifest 4 处。
- **决定**：本 Phase 不动（迁移只改身份字段，避免与内容修改混在一次操作里）。→ **Phase 5**

### D-09 teacher-model 的规则内容在 C 组无对应文件（净新增，无冲突）
- **现象**：`stable_rules` / `visual` / `pedagogy` / `notes` / `archetypes` /
  `supervisor_feedback_loop` / `exceptions` 七个键，C 组侧不存在同类文件。
- **决定**：整体迁入，逐键校验未改动。**本 Phase 已执行。**

### D-10 样例与教材的 SHA 跨库比对（无差异，记录备查）
- **现象**：C 组 `sample-manifest.json` 与 D 组 `teacher-model.sources` 各自独立记录了
  第 2/3/4 课样例的 SHA-256。
- **结果**：三条**逐字节一致**（仅大小写不同）。第 5 课控制模板 SHA 亦一致。
- **决定**：无需处理。两套库在外部输入这一层从未分叉，分叉只发生在"进度"这一层。

---

## 5. 无差异确认项

以下项目经核对确认一致，记录在案以证明核对是完整的、而非只挑差异：

- [x] `course_id` 全库一致：`new_hsk_course_1`（6 份状态文件 + 报告）；
- [x] 第 2/3/4 课样例 SHA-256 两套库一致；
- [x] 教材 PDF SHA-256（`84FDDB0D…`，142 页）在 C 组内部一致，D 组未独立记录，无冲突；
- [x] 第 5 课控制模板 SHA-256 在 A/B/D 三方一致（C 组不一致，见 D-03/D-07）；
- [x] 10 份第 6–15 课 PPTX 的 SHA-256 与页数在 A/B 两方一致（C 组未登记，见 D-04）；
- [x] 每份 PPTX 备注页数 = 幻灯片数。

---

## 6. 本 Phase 已执行的合并动作

| 动作 | 结果 |
|---|---|
| 归档 `.courseware/` 整目录 | → `archive/2026-08-state-merge/courseware-legacy/`（1 份文件，原样保留） |
| 归档副本原文件 SHA-256 | `1EC1550E7424E28EB164C2A8A326C8E7451425B91C775FF30775CF5331D06689` |
| 迁入 teacher-model.json | → `.tcsl-courseware/teacher-model.json`，SHA-256 `2526E415EC238BC3CCBCBF58931C1CCE929C06C3FE6849691C7C5D4D4697D582` |
| 迁入时的字段改动 | 仅三处身份字段：`teacher_id` 由 `teacher-hsk1` 改为 `teacher_hsk1_set_a`；新增 `schema_version: "3.0"`；新增 `course_id: "new_hsk_course_1"` |
| 规则内容校验 | 13 个顶层键（model_version、courseware_contract_version、textbook_series、control_template、sources、stable_rules、visual、pedagogy、notes、archetypes、production_baseline、supervisor_feedback_loop、exceptions）逐键 JSON 比对**未变**，符合 Spec REQ-001「MUST NOT 修改 teacher model 规则内容」 |

> 迁入文件 SHA 与归档副本不同是**预期结果**：改了三个身份字段并统一为 2 空格缩进。
> 规则内容未变已由上表的逐键比对证明；如需字节级回退，归档副本即原件。

---

## 7. 遗留待确认

| 项 | 状态 |
|---|---|
| 删除 `.courseware/` 原目录 | **待所有者确认**（Spec §11.1：先归档、经确认后执行）。归档副本已就位，删除后可从 `archive/` 或 git 历史完整恢复 |
| D-03～D-08 共 6 条差异 | 已定处理方向，执行归属 Phase 2 / Phase 5，本 Phase 不动 |
