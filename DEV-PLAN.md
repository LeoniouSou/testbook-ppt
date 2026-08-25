# Development Plan — TCSL 课件生产系统 · 工程化改造

> 本文件记录改造的阶段划分、当前进度和剩余工作。
> 新 session 启动时应首先阅读 Product-Spec.md 与本文件，了解状态后再继续。
> 铁律：任何 Phase 均不得改变已交付 PPTX 的内容与 SHA-256（Spec §0）。

---

## Phase 1: 状态三方核对与合并（P0 · REQ-001）

**交付内容**：
- 产出三方核对报告：`.tcsl-courseware/` × `.courseware/teacher-hsk1/` × 实际 PPTX（页数、SHA-256）逐项对照，差异逐条列出并解释；
- 确定合并方向（默认按 ASM-001：以 teacher-model.json 3.2 为新、ledger 事实以实物为准），确认 Q-001 的 teacher_id 命名；
- 将 `teacher-model.json` 迁入唯一状态根，统一全库 `teacher_id`；
- 归档 `.courseware/` 整目录至 `archive/2026-08-state-merge/` 后删除原目录。

**关键文件**：
- `archive/2026-08-state-merge/state-reconciliation-report.md` — 三方核对报告，合并的证据基础
- `.tcsl-courseware/teacher-model.json` — 迁入后的现行老师模型
- `archive/2026-08-state-merge/` — 旧状态归档

**验收标准**：
- 全库检索 `.courseware/` 除归档外 0 引用；
- 任一状态文件的 teacher_id/course_id/schema 全库一致（Spec AC）；
- 核对报告中每条差异都有解释或标记为待确认，无未解释差异被静默合并。

---

## Phase 2: 状态回写至事实（P0 · REQ-002）

**交付内容**：
- ledger 第 6–15 课改为 `machine_gate: pass` + `human_review: awaiting_teacher_review`，逐课登记最终文件、页数、SHA-256；
- 第 5 课登记 V3.3 40 页版为 accepted 基线，历史版本链（V3.2/主管稿/39页版）按现有说明文档归档记录；
- `production-state.json` 更新 phase、active_lesson、updated_at 至事实。

**关键文件**：
- `.tcsl-courseware/course-ledger.json` — 增加 machine_gate / human_review 双字段
- `.tcsl-courseware/production-state.json` — 现势状态

**验收标准**：
- ledger、HSK1-COURSEWARE-PROGRESS.md、`sha256sum` 实测三方完全一致；
- 冷启动演练：不带口头交代的新 session 判断"下一步 = 等待老师审阅 / 可接新教材"，而非重产第 6 课。

---

## Phase 3: SKILL 门禁增补（P0 · REQ-003/004）

**交付内容**：
- 在 SKILL.md §13 增补第四组"状态与账本门槛"（ledger 回写 + production-state 更新 + QA 摘要写入，三项缺一即该课未完成）；
- 将 §8 滚动放量升级为门禁：未审课次达批次上限即停成品交付、不停草稿准备；含所有者书面豁免通道，豁免记录进 ledger；
- 修订遵循"净规则量最小"：只加条款，不复述既有内容。

**关键文件**：
- `tcsl-integrated-courseware-generator/SKILL.md` — §8、§13 修订

**验收标准**：
- §13 检索到"状态与账本门槛"三项；
- 沙盘推演 Spec REQ-004 AC-001 场景（10 课待审 0 accepted，请求产第 16 课）：按修订后规则应被阻止。

---

## Phase 4: 目录重构（P1 · REQ-005）

**交付内容**：
- 建立 `courses/new_hsk_course_1/{deliverables,state,qa}`、`teachers/<统一teacher_id>/model`、`tools/`、`archive/`；
- `git mv` 迁移：10+1 份交付 PPTX → deliverables/；`.tcsl-courseware/*` → state/ 与 teachers/.../model/（按实体归属拆分）；PROGRESS.md、V3.3 说明 → courses 下；
- 同步修订 SKILL.md §5.5 路径规范；
- .gitignore 白名单从 `HSK1-[6-9]…` 文件名模式改为 `courses/*/deliverables/` 目录级规则。

**关键文件**：
- `courses/new_hsk_course_1/` — 教材生产单元
- `teachers/teacher_hsk1_set_a/model/` — 老师模型（命名以 Q-001 确认为准）
- `.gitignore` — 目录级白名单

**验收标准**：
- 全部已交付 PPTX 迁移后 SHA 与 ledger 一致（`git mv` 不改内容）；
- 模拟新建 `courses/hsk2/` 不需改动任何既有文件；
- SKILL.md 路径规范与实际目录一致。

---

## Phase 5: 可复现性（P1 · REQ-006/007）

**交付内容**：
- 清除全库 `D:/`、`D:\` 绝对路径（归档目录除外），改仓库相对路径；
- 创建 `courses/new_hsk_course_1/state/inputs-manifest.json`：教材 PDF、4 份样例 PPTX、各版本控制模板的逻辑名/期望路径/SHA/来源说明；
- 实现 `tools/verify-inputs.py`：对照 manifest 校验存在性与 SHA，输出缺失清单，缺失时退出码非 0；Windows 与 Linux 双平台验证；
- 创建 `DEPENDENCIES.md`：登记 Presentations skill 名称、来源、版本/commit、安装方式。

**关键文件**：
- `courses/new_hsk_course_1/state/inputs-manifest.json` — 外部输入登记
- `tools/verify-inputs.py` — 恢复校验脚本
- `DEPENDENCIES.md` — 外部 skill 依赖

**验收标准**：
- 全库检索 `D:/` 与 `D:\\`（排除 archive/）0 命中；
- 在本仓库（外部输入缺失状态）运行脚本：列出全部缺失项 + 期望 SHA + 获取说明，退出码非 0；
- 人为放置一个 SHA 正确的输入后重跑：该项转绿。

---

## Phase 6: QA 证据包与版本卫生（P2 · REQ-008/009）

**交付内容**：
- 实现 `tools/gen-qa-summary.py`：从最终 PPTX 生成 `qa/<lesson>-qa-summary.json`（SHA、页数、备注数、门槛结果、渲染图哈希清单、签核、时间）；
- 为已交付 HSK1 十课 + 第 5 课补写摘要，标注 `evidence_level: "recorded_post_hoc"`；旧的 lesson 5 fidelity check 移入 archive/；
- 落实 Q-002 决定：配置 Git LFS 追踪 `courses/*/deliverables/*.pptx`（或 tag+Release 方案）；
- 在 SKILL.md §14 增补一句版本规范：每课定稿一 commit + tag（`<course>-L<N>-final`），新交付物文件名不含版本尾缀。

**关键文件**：
- `tools/gen-qa-summary.py` — QA 摘要生成器
- `courses/new_hsk_course_1/qa/*.json` — 11 份摘要
- `.gitattributes` — LFS 追踪规则（如选 LFS）

**验收标准**：
- 任一课的 QA 摘要 SHA 与 `sha256sum` 实测一致；
- 摘要生成器对 SHA 不匹配的 PPTX 拒绝生成并报错；
- LFS/tag 方案在一次演练 commit 上走通。

---

## Phase 7: 进化 hook 豁免与总验收（P2 · REQ-010 + 完成定义）

**交付内容**：
- 修改 `.codex/hooks/detect-feedback-signal.sh`：命中课件修订特征（`P\d+` 页码引用、课次引用等）时豁免入队，非课件语境行为不变；
- 用 Spec REQ-010 两条 AC 的输入做黑盒测试；
- 执行完成定义总验收：冷启动演练、新机器演练（可用本云端环境模拟）、全部交付物 SHA 前后比对、SKILL 修订与 Spec 一致性检查；
- 更新本文件勾选状态，归档改造过程记录。

**关键文件**：
- `.codex/hooks/detect-feedback-signal.sh` — 豁免逻辑
- `DEV-PLAN.md` — 最终勾选

**验收标准**：
- REQ-010 AC-001/AC-002 黑盒测试通过；
- Spec §9 完成定义五项全部勾选。

---

## 技术栈

| 层级 | 技术 | 版本 | 说明 |
|------|------|------|------|
| 脚本 | Python 3 | ≥3.10 | 校验与 QA 摘要脚本，Windows/Linux 双平台 |
| PPTX 只读核对 | python-pptx | 1.x | 仅读页数/备注数，不写 PPTX |
| Hook | POSIX sh + jq | — | 沿用既有 hook 技术栈 |
| 版本管理 | git（+ Git LFS 可选） | — | LFS 与否按 Q-002 定 |

## 数据库表

无数据库。持久化实体均为文件（见 Product-Spec §6），Phase 1–2 建立唯一状态根，Phase 4 迁至 `courses/<id>/state/`。

## 开发规则

- Phase 顺序有依赖：1→2→3 必须顺序完成（P0 链）；4→5 依赖 1–2 的唯一状态根；6、7 依赖 4 的目录结构；
- 每个 Phase 完成执行：验收标准逐条核验 → 冷启动回归演练（Phase 2 起每次都跑）→ 交付物 SHA 全量比对未变 → commit；
- 本项目无编译环节，"四步走"中的编译验证替换为：`verify-inputs` + SHA 全量比对 + 相关脚本自测通过；
- Commit message 用 feat、fix、refactor、chore 前缀；改造类 commit 统一 `refactor(engineering):` 前缀，便于与课件生产 commit 区分；
- 涉及删除（`.courseware/` 旧状态）必须先归档、经所有者确认后执行（Spec §11.1）。
