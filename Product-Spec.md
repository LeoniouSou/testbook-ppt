# 产品需求规范：TCSL 课件生产系统 · 工程化改造（testbook-ppt）

> **版本：** v1.1　**更新日期：** 2026-08-25　**变更记录：** 见 `Product-Spec-CHANGELOG.md`

## 0. AI 使用说明

- 本文档是本次改造的功能、范围、行为和验收标准的事实来源。
- AI MUST 优先实现 P0。
- AI MUST NOT 实现"不在本版本范围"中明确排除的内容。
- AI MUST 根据"验收标准"判断功能是否完成。
- 如果信息不明确，AI MUST 使用"假设"中的假设；仍无法判断的记录到"待确认问题"，不自行扩展需求。
- 改造期间 MUST NOT 修改任何已交付课件 PPTX 的内容与 SHA-256。

---

## 1. 产品上下文

### 1.1 产品摘要

把 testbook-ppt 从"HSK1 单次任务的工作区快照"改造成可持续运转的课件生产系统：状态唯一可信、跨机器可校验重建、目录结构支持多教材/多老师扩展。改造对象是仓库的工程层（状态、结构、证据、门禁），不是课件生产方法论本身——`tcsl-integrated-courseware-generator` SKILL 的教学与质检设计保持不变，仅按本 Spec 增补执行纪律条款。

### 1.2 用户问题

2026-08 的 HSK1 生产中，Agent 在运行时另建了第二套状态目录（`.courseware/teacher-hsk1/`），与 SKILL 规定的 `.tcsl-courseware/` 分叉：旧 ledger 停在"第 6 课进行中"，而实际 10 课已交付。下一个冷启动的 session 按 SKILL 读旧状态，会重复生产已交付课次。同时所有状态引用 `D:/textbook-ppt` 绝对路径、验收证据全部留在被 gitignore 的 tmp 中，换机器或换 Agent 无法接续，也无法复验"终验通过"的声明。HSK2 及更多老师即将接入，现有根目录平铺结构无法承载。

### 1.3 目标用户

| 用户类型 | 描述 | 核心需求 |
|---|---|---|
| 生产 Agent（Codex/Claude 等） | 冷启动读仓库状态后接续生产的 AI | 一次读取即得到唯一、准确的进度真相 |
| 项目所有者（Leoniou） | 调度生产、对接老师与主管的人 | 随时看清每课真实状态；换机器可继续 |
| 老师 / 教学主管 | 审阅并修改交付课件的人 | 交付节奏与其审阅能力匹配，反馈可追溯 |

### 1.4 核心价值

任何 Agent 在任何机器上克隆仓库后，不需要口头交代就能准确接续生产；每一份"已通过"的声明在仓库内可复验。

### 1.5 成功标准

| 判断标准 | 目标 / 信号 |
|---|---|
| 冷启动准确性 | 新 session 仅凭仓库文件判断出的"下一个待生产课次"与实际一致，0 次重复生产 |
| 可移植性 | 在一台全新机器克隆后，校验脚本可跑通并明确报告缺哪些外部输入 |
| 扩展就绪 | 接入 HSK2 或第二位老师时，不需要修改任何既有目录/gitignore 规则 |
| 证据可复验 | 每个标记为已交付的课次，仓库内有与最终 SHA 绑定的 QA 摘要 |

---

## 2. 范围

### 2.1 本版本范围

| 编号 | 内容 | 优先级 | 备注 |
|---|---|---|---|
| SCOPE-001 | 状态收敛：合并两套状态库为唯一状态根，回写 10 课真实状态 | P0 | 详见 REQ-001/002 |
| SCOPE-002 | SKILL.md 增补两条门禁：状态回写是交付必要条件；滚动放量硬约束 | P0 | 详见 REQ-003/004 |
| SCOPE-003 | 目录重构：`courses/<course_id>/` + `teachers/<teacher_id>/`，对齐既有 id 字段 | P1 | 详见 REQ-005 |
| SCOPE-004 | 可复现性：路径相对化 + inputs-manifest + 校验脚本 + 外部依赖记录 | P1 | 详见 REQ-006/007 |
| SCOPE-005 | QA 证据包：每课一份机器可读 QA 摘要入库 | P2 | 详见 REQ-008 |
| SCOPE-006 | 版本卫生：交付物 Git LFS/tag 管理、每课定稿一 commit、文件名去版本尾缀 | P2 | 详见 REQ-009 |
| SCOPE-007 | 进化 hook 课件语境豁免，防止内容修订指令污染信号队列 | P2 | 详见 REQ-010 |

### 2.2 不在本版本范围

| 编号 | 内容 | 原因 |
|---|---|---|
| OUT-001 | 修改课件生产方法论（证据分级、二元门槛、控制性参考机制等） | 已验证有效，本次只补执行纪律 |
| OUT-002 | 重构「废才」编程框架（AGENTS.md 主流程、11 个通用 skill） | 所有者明确：编程框架不动 |
| OUT-003 | 重新生产或修改任何已交付的 HSK1 课件内容 | 已过独立终验，改动需走老师反馈流程 |
| OUT-004 | 将教材 PDF / 老师样例 PPTX 纳入 git | 版权敏感，以 manifest + SHA 校验替代 |
| OUT-005 | 建设跨教师全局规则库、仪表盘、数据库等平台设施 | SKILL 禁止事项，规模未到 |

---

## 3. 用户任务

| 编号 | 用户任务 | 用户类型 | 优先级 |
|---|---|---|---|
| TASK-001 | 冷启动后准确接续生产，不重复、不遗漏 | 生产 Agent | P0 |
| TASK-002 | 交付一课后，让仓库状态立即反映事实 | 生产 Agent | P0 |
| TASK-003 | 在新机器上恢复完整生产环境 | 项目所有者 | P1 |
| TASK-004 | 接入新教材或新老师开始生产 | 项目所有者 | P1 |
| TASK-005 | 复验任一已交付课次的验收证据 | 项目所有者 / 主管 | P2 |

---

## 4. 用户流程

### FLOW-001: 冷启动接续生产

**关联任务：** TASK-001　**优先级：** P0
**目标：** Agent 凭仓库文件独立判断进度并接续。

**入口：** 新 session 在仓库根启动。

**主路径：**
1. 读唯一状态根 `.tcsl-courseware/` 的 `production-state.json` 与 `course-ledger.json`；
2. 校验 `schema_version` 与 `teacher_id`/`course_id` 一致性；
3. 按 ledger 的 `lesson_status` 确定下一个待生产课次；
4. 校验控制模板文件存在且 SHA 匹配后开始生产。

**边界情况：**
- schema 不兼容或状态文件缺失 → 按 SKILL 规定停下询问，不猜测、不静默重建；
- 状态与交付物 SHA 对不上 → 报告差异并暂停该课次。

**完成状态：** Agent 报告的进度与 PROGRESS 文档、交付物实际一致。

### FLOW-002: 交付即回写

**关联任务：** TASK-002　**优先级：** P0
**目标：** 每次交付后状态零漂移。

**主路径：**
1. 课件通过全部质检门槛；
2. 更新 ledger 该课 `lesson_status`、最终 SHA、页数；
3. 更新 `production-state.json` 的 `active_lesson` 与 `updated_at`；
4. 写入该课 QA 摘要（REQ-008）；
5. 以上完成后，才允许宣布该课"交付完成"。

**边界情况：** 回写失败 → 交付视为未完成，禁止开始下一课。

### FLOW-003: 新机器恢复

**关联任务：** TASK-003　**优先级：** P1
**主路径：** 克隆仓库 → 运行 `tools/verify-inputs` 校验脚本 → 脚本列出缺失的外部输入（教材 PDF、样例 PPTX、Presentations skill）及其 SHA 与存放路径 → 用户补齐 → 重跑通过 → 可开始生产。

**完成状态：** 校验脚本输出全绿，或明确列出缺什么、放哪里。

### FLOW-004: 接入新教材/新老师

**关联任务：** TASK-004　**优先级：** P1
**主路径：** 新建 `courses/<新course_id>/` 与（如需）`teachers/<新teacher_id>/` → 按 SKILL 首次建模路径运行 → 状态、交付物、QA 全部落在新目录内。

**边界情况：** 不同老师/教材的模型 MUST NOT 交叉引用（SKILL §3 既有红线）。

---

## 5. 功能需求

### REQ-001: 状态库合并

**优先级：** P0　**关联：** TASK-001 / FLOW-001

**用途：** 消除 `.tcsl-courseware/`（schema 3.0）与 `.courseware/teacher-hsk1/`（contract 3.2）的分叉。

**行为：** 以实际交付物为准核对两套状态；将 `teacher-model.json` 迁入 `.tcsl-courseware/`，统一 `teacher_id` 命名（二选一，全库一致）；`.courseware/` 整目录归档后删除。

**规则：**
- MUST 迁移前对两套状态与实际 PPTX（页数、SHA）做三方核对，差异逐条记录；
- MUST 保留归档副本于 `archive/`（或 git 历史 tag），不静默销毁；
- MUST NOT 在合并中修改 teacher model 的规则内容本身。

**验收标准：**
- [ ] AC-001: Given 合并完成, when 全库搜索 `.courseware/`, then 除归档外无任何引用。
- [ ] AC-002: Given 合并完成, when 读取任一状态文件, then `teacher_id`/`course_id`/`schema_version` 全库唯一一致。

### REQ-002: 状态回写至事实

**优先级：** P0　**关联：** TASK-001 / FLOW-001

**行为：** ledger 中第 6–15 课状态由 `queued` 改为反映事实的状态；第 5 课登记项由已被取代的 V3.2 36 页版更正为 V3.3 40 页定稿；`production-state.json` 的 phase/active_lesson 同步。

**状态字段约定（与 SKILL §6 枚举对齐）：** SKILL §6 已定义 `lesson_status` 枚举为 `queued / drafting / review / accepted / final`，其中**不含** `awaiting_teacher_review`。为同时满足本 Spec 的"状态反映事实"与 NFR"SKILL 净增量最小"，本版**不扩展该枚举**，改为三字段分工：

| 字段 | 取值 | 含义 |
|---|---|---|
| `lesson_status` | SKILL §6 既有枚举 | 冷启动判断下一待产课次的唯一依据（FLOW-001 步骤 3） |
| `machine_gate` | `pass` / `fail` / `not_run` | 机器门禁结果 |
| `human_review` | `awaiting_teacher_review` / `teacher_accepted` / `supervisor_confirmed` / `none` | 人工签核细分状态 |

**规则：**
- MUST 将第 6–15 课的 `lesson_status` 由 `queued` 改为 `review`（机器门禁已过、等待老师审阅），不得标 `final`，不得留在 `queued`；
- MUST 在 ledger 增加 `machine_gate` 与 `human_review` 两个独立字段承载细分语义（本版由 SHOULD 升为 MUST：既然不扩展枚举，细分语义没有别的落点）；
- MUST 记录每课最终文件名、页数、SHA-256，与 PROGRESS 文档及实际文件三方一致；第 5 课既有的 `output` / `sha256` / `slides` 三个字段指向 V3.2 36 页版，MUST 覆盖为 V3.3 40 页定稿；
- MUST 把 ledger 中现存的枚举外取值（`completed_by_teacher`、`outside_active_model`、`completed_v3_2`）一并归一到 SKILL §6 枚举，原始语义转记到 `human_review`；
- MUST NOT 在 SKILL §6 中新增枚举值；SKILL 侧只允许加一句指向 ledger 双字段的说明。

**验收标准：**
- [ ] AC-001: Given 回写完成, when 对比 ledger、PROGRESS.md、`sha256sum` 实测值, then 三者完全一致。
- [ ] AC-002: Given 新 session 冷启动, when 询问"下一步做什么", then 回答是"等待老师审阅第 6–15 课 / 可接 HSK2"而非"生产第 6 课"。
- [ ] AC-003: Given 回写完成, when 读取 ledger 全部 15 条课次记录, then 每条 `lesson_status` 取值均落在 SKILL §6 枚举内，且第 6–15 课的 `human_review` 均为 `awaiting_teacher_review`。

### REQ-003: 交付门禁——状态回写为必要条件

**优先级：** P0　**关联：** TASK-002 / FLOW-002

**行为：** 在 SKILL.md §13 质检节增补一组"状态门槛"：ledger 回写、production-state 更新、QA 摘要写入三项全部完成，才允许把课次报告为交付完成。

**规则：**
- MUST 写入 SKILL.md，成为与来源/教学/视觉门槛并列的第四组二元门槛；
- MUST NOT 依赖 Agent 自觉——门槛缺一即判定该课未完成；
- MUST 在门禁生效的同时具备可用的 QA 摘要写入手段。REQ-008 的完整生成器为 P2，但其**最小子集**（最终 SHA-256、页数、备注数、各组门槛结果、`evidence_level`）随本需求一起落地为 P0，避免 P0 门禁依赖 P2 能力；
- 门禁自写入 SKILL.md 起即时生效，不设过渡期。生效期间若因 REQ-004 暂停成品交付，本门禁不产生额外阻塞；一旦通过豁免通道恢复交付，最小摘要能力必须已经可用。

**验收标准：**
- [ ] AC-001: Given SKILL.md 修订后, when 检索 §13, then 存在"状态与账本门槛"且含上述三项。
- [ ] AC-002: Given 门禁生效后任一课次被报告为交付完成, when 检查该课, then ledger 回写、production-state 更新、QA 摘要文件三者均存在，且摘要可由仓库内脚本重新生成并与实测 SHA 一致。

### REQ-004: 滚动放量硬约束

**优先级：** P0　**关联：** TASK-001

**用途：** 防止再次出现"老师只审过 1 课、系统一次产 10 课"的返工风险敞口。

**行为：** SKILL.md §8 的滚动放量从建议升级为门禁：ledger 中处于 `awaiting_teacher_review` 且未获任何回应的课次数达到当前批次上限时，暂停生产后续新课，转入可重算草稿准备（不停解析，只停成品交付）。

**`accepted` 的判定口径：** 沿用 SKILL §6 定义——`accepted` 仅指**老师**的签核动作（老师回复"继续"，或老师返回修改稿）。教学主管确认、内部独立终验、机器严格门禁 PASS 均**不**计入 `accepted`。第 5 课现有记录为教学主管最终确认（PROGRESS 2026-08-22），因此在本口径下记为 `lesson_status = review` + `human_review = supervisor_confirmed`，**不计入阶梯计数**；当前 `accepted_count = 0`，阶梯处于第 1 级、批次上限 = 1。

**规则：**
- MUST 批次上限沿用既有阶梯（1→2→4→6–8），当前阶梯由 ledger 中 `human_review = teacher_accepted` 的课次数决定；
- MUST 把门禁判定所需状态显式落到 ledger，不依赖运行时推算：新增 `batch_state` 对象（`ladder_step`、`cap`、`accepted_count`、`awaiting_count`、`updated_at`）与 `exemptions` 数组（`granted_by`、`granted_at`、`scope`、`reason`）；
- MUST 把"未获任何回应"定义为可判定条件：该课 `human_review = awaiting_teacher_review` 且 ledger 中无该课的老师反馈记录；
- MUST 允许所有者显式书面豁免单次批产，豁免写入 `exemptions`（本版由 SHOULD 升为 MUST：没有豁免通道时门禁无法解除，会把生产线永久锁死）。

**验收标准：**
- [ ] AC-001: Given 10 课待审 0 课 accepted, when Agent 计划生产第 16 课成品, then 门禁阻止并提示等待审阅或申请豁免。
- [ ] AC-002: Given 回写后的 ledger, when 读取 `batch_state`, then `accepted_count = 0`、`ladder_step = 1`、`cap = 1`、`awaiting_count = 10`。

### REQ-005: 多教材/多老师目录结构

**优先级：** P1　**关联：** TASK-004 / FLOW-004

**行为：** 重构为：

```text
courses/<course_id>/          # 如 new_hsk_course_1
├── deliverables/             # 交付 PPTX
├── state/                    # 该教材的 ledger、production-state、manifest
├── qa/                       # 每课 QA 摘要
└── PROGRESS.md
teachers/<teacher_id>/        # 如 teacher_hsk1_set_a
└── model/                    # teacher-model、profile、preferences
tools/                        # 校验与 QA 摘要脚本
archive/                      # 历史版本与归档状态
```

**规则：**
- MUST 目录名直接使用状态文件中既有的 `course_id`/`teacher_id` 值；
- MUST 同步更新 SKILL.md §5.5 的路径规范与 .gitignore（白名单从文件名模式改为 `courses/*/deliverables/` 目录级）；
- MUST NOT 破坏已交付 PPTX 的 SHA（纯移动，`git mv`）。

**验收标准：**
- [ ] AC-001: Given 重构完成, when 模拟新建 `courses/hsk2/`, then 无需修改根目录任何既有文件即可开始。
- [ ] AC-002: Given 重构完成, when 校验全部已交付 PPTX 的 SHA, then 与 ledger 记录一致。

### REQ-006: 路径相对化 + inputs-manifest

**优先级：** P1　**关联：** TASK-003 / FLOW-003

**行为：** 所有状态文件中的 `D:/textbook-ppt/...` 绝对路径改为仓库相对路径；新增 `courses/<id>/state/inputs-manifest.json`，记录每个不入库外部输入的：逻辑名、期望相对路径、SHA-256、来源说明（从哪获取/谁持有）。

**规则：**
- MUST 覆盖教材 PDF、四份样例 PPTX、各版本控制模板；
- MUST 提供 `tools/verify-inputs`（POSIX sh 或 Python，双平台可跑），对照 manifest 逐项校验存在性与 SHA，输出缺失清单；
- MUST NOT 把版权文件本体加入 git。

**验收标准：**
- [ ] AC-001: Given 全库检索 `D:/` 与 `D:\\`, then 0 命中（归档目录除外）。
- [ ] AC-002: Given 缺少教材 PDF 的全新克隆, when 运行校验脚本, then 明确列出缺失项、期望 SHA 与获取说明，退出码非 0。

### REQ-007: 外部依赖记录

**优先级：** P1　**关联：** TASK-003

**行为：** 在 manifest 或独立 `DEPENDENCIES.md` 中记录 `Presentations` skill 的名称、来源、版本/commit 与安装方式；后续新增外部 skill 依赖时同步登记。

**验收标准：**
- [ ] AC-001: Given 新机器恢复流程, when 按记录安装依赖, then 生产路径可启动（或明确列出无法自动安装的项）。

### REQ-008: 每课 QA 证据摘要

**优先级：** P2　**关联：** TASK-005

**行为：** 每课定稿时生成 `courses/<id>/qa/<lesson>-qa-summary.json`：最终 SHA、页数、备注数、各组门槛结果（含状态门槛）、渲染图数量及其哈希清单、签核记录、生成时间。渲染图本体不入库。

**规则：**
- MUST 摘要与最终 PPTX 的 SHA 绑定，PPTX 变则摘要必须重生成；
- SHOULD 对已交付的 HSK1 十课凭现有 PROGRESS 记录补写摘要，并标注 `evidence_level: "recorded_post_hoc"`，不冒充完整证据。

**验收标准：**
- [ ] AC-001: Given 任一已交付课次, when 读取其 QA 摘要并 `sha256sum` 实测 PPTX, then 二者一致。

### REQ-009: 版本卫生

**优先级：** P2　**关联：** TASK-005

**行为：** 交付 PPTX 改用 Git LFS 追踪（Q-002 已定案：选 Git LFS，不选 tag+Release）；确立"每课定稿一个 commit + tag（如 `hsk1-L6-final`）"规范写入 SKILL.md；新交付物文件名去掉版本尾缀（版本由 ledger SHA + git tag 承担），既有文件名不追溯改名。

**规则：**
- MUST 只对迁移后的 `courses/*/deliverables/*.pptx` 启用 LFS 追踪；MUST NOT 用 `git lfs migrate` 重写既有历史（会改写 commit SHA、废掉所有既有克隆）。既有 11 份 PPTX 保持普通 git 对象，普通对象与 LFS 对象混合的状态可接受；
- MUST 把 Git LFS 登记进 REQ-007 的 `DEPENDENCIES.md`，并标注为 Windows/Linux 双平台前置——云端 Agent 环境默认未安装 LFS（2026-08-25 实测：`git: 'lfs' is not a git command`）。

**验收标准：**
- [ ] AC-001: Given 规范生效后的下一次定稿, when 查看 git 历史, then 该课有独立 commit 与 tag，且 ledger SHA 可在该 commit 复验。
- [ ] AC-002: Given LFS 启用后, when 检查既有 11 份 PPTX 的 blob, then 仍为普通 git 对象，git 历史的 commit SHA 未被改写。

### REQ-010: 进化 hook 课件语境豁免

**优先级：** P2　**关联：** TASK-002

**用途：** "P12：把这个例句换掉"类正常修订指令不应进入进化信号队列。

**行为：** `detect-feedback-signal.sh` 增加豁免：消息匹配课件修订特征时不入队；豁免逻辑不影响编程项目场景。三条触发分支全部必须实现：

| 分支 | 判定方式 |
|---|---|
| 页码引用 | 消息匹配 `P\d+` 或 `第\d+页` |
| 课次引用 | 消息匹配 `第\d+课` 或 `L\d+` |
| 课件生产上下文 | 仓库内存在 `courses/<course_id>/state/production-state.json` 且其 `phase` 不是终态。hook 为 UserPromptSubmit，只能读 stdin 的 `.prompt`，拿不到 session 内存，故语境标记 MUST 落成仓库内可读文件 |

**规则：**
- MUST 保持原 hook 在非课件语境的行为不变；
- MUST 三条分支各自具备独立的黑盒测试用例；
- SHOULD 宁可漏抓由主 Agent 补记（既有机制），不可误抓污染队列。

**验收标准：**
> 说明：v1.0 使用的样本"P12：把这个例句换掉"在现有正则下**本来就不会入队**（词表含 `去掉/删掉/改成/换成`，不含 `换掉`），属于空测，不能证明豁免生效。本版换成现状下确实会误入队的样本。

- [ ] AC-001: Given 输入"P8：这一页的图换成清晰的"（现状会误入队）, when hook 执行, then signals.jsonl 不新增记录。
- [ ] AC-002: Given 输入"第9课第7页的学生指令改成口语一点"（现状会误入队）, when hook 执行, then signals.jsonl 不新增记录。
- [ ] AC-003: Given 输入"你又搞错了，我说过规则要写进 SKILL", when hook 执行, then 正常入队。
- [ ] AC-004: Given production-state.json 的 phase 为终态或该文件不存在, when 输入一条含 `P8` 的编程类修正消息, then 语境分支不生效，按原规则正常入队。

---

## 6. 数据模型

### 6.1 核心实体

| 实体 | 描述 | 关键字段 |
|---|---|---|
| Course | 一本教材的整册生产单元 | course_id、textbook SHA、lesson_count |
| Teacher Model | 一位老师×一套教材的风格模型 | teacher_id、contract_version、stable/conditional 规则 |
| Lesson | 单课生产记录（ledger 条目） | lesson、lesson_status（SKILL §6 枚举）、machine_gate、human_review、final SHA、页数 |
| Batch State | 滚动放量门禁的判定状态（ledger 内） | ladder_step、cap、accepted_count、awaiting_count、exemptions[] |
| Deliverable | 定稿 PPTX | 文件、SHA-256、对应 tag |
| QA Summary | 单课验收证据摘要 | 绑定 SHA、各门槛结果、evidence_level |
| Inputs Manifest | 外部输入登记 | 逻辑名、相对路径、SHA、来源说明 |

### 6.2 实体关系

| 关系 | 描述 |
|---|---|
| Course has many Lessons | ledger 内逐课记录 |
| Course references one Teacher Model | 通过 teacher_id；不同教材/老师不得交叉 |
| Lesson has one Deliverable + one QA Summary | 三者 SHA 互相绑定 |

### 6.3 数据规则

- 状态创建/更新只发生在 `.tcsl-courseware/`（迁移后为 `courses/<id>/state/`）唯一根下；
- 任何交付物 SHA 变更 MUST 级联更新 ledger 与 QA 摘要；
- `lesson_status` 取值 MUST 限定在 SKILL §6 枚举内，人工签核细分一律走 `human_review`（REQ-002）；
- 归档只移动不删除；旧版本可回退（沿用 SKILL §11.7）；
- 老师样例与教材本体永不入库，只入 manifest。

---

## 7. 外部依赖

| 编号 | 依赖 | 用途 | 是否必需 | 备注 |
|---|---|---|---:|---|
| DEP-001 | Presentations skill | PPTX 读写、模板跟随、渲染质检 | Yes | 版本待登记（REQ-007） |
| DEP-002 | 教材 PDF（HSK1 等） | 事实源 | Yes | 不入库，manifest 校验 |
| DEP-003 | 老师样例 PPTX | 风格建模证据 | Yes | 同上 |
| DEP-004 | Git LFS | 交付物版本管理 | Yes | Q-002 已定案选用；云端环境默认未装，安装方式登记于 DEPENDENCIES.md |
| DEP-005 | jq / Python 3 | 校验与摘要脚本 | Yes | 双平台（Windows/Linux）可用 |

---

## 8. 非功能需求

| 类别 | 要求 | 优先级 |
|---|---|---|
| 可靠性 | 冷启动进度判断 0 误判；状态回写失败即交付失败 | P0 |
| 兼容性 | 脚本在 Windows（当前生产机）与 Linux（云端 Agent）均可运行 | P1 |
| 可移植性 | 全库无绝对路径；克隆+补输入即可生产 | P1 |
| 隐私/版权 | 教材与样例本体不入 git；仓库默认 Private | P0 |
| 可维护性 | SKILL.md 净增量最小化——只加门禁条款，不复述既有规则 | P1 |
| 性能 | 校验脚本全量运行 < 30 秒 | P2 |

---

## 9. 完成定义

- [ ] 所有 P0 requirements 已实现，AC 全部通过
- [ ] FLOW-001 冷启动演练：新 session 判断结果与事实一致
- [ ] FLOW-003 新机器演练：校验脚本行为符合 AC
- [ ] 已交付 10 课 PPTX 的 SHA 在改造前后完全不变
- [ ] SKILL.md 修订与本 Spec P0 内容一致

---

## 10. 假设与待确认问题

### 10.1 假设

| 编号 | 假设 | 假设依据 | 错误风险 |
|---|---|---|---|
| ASM-001 | `.courseware/teacher-hsk1/teacher-model.json`（contract 3.2，绑定 V3.3 模板）是较新的有效状态，`.tcsl-courseware/` 为过时残留 | 其绑定的 V3.3 模板与 PROGRESS.md 8-22 记录吻合 | **已核实（2026-08-25）**：该文件 `production_baseline.sha256 = 8f2580a4…` 与 PROGRESS 头部 V3.3 40 页控制模板 SHA 一致，假设成立，合并方向不反转 |
| ASM-002 | 第 6–15 课老师均未实际审阅 | ledger 无 accepted 记录，PROGRESS 只提主管确认第 5 课 | **已核实（2026-08-25）**：ledger 全库无 `accepted` 取值，`accepted_count = 0`；若后续部分已审，改该课 `human_review = teacher_accepted` 并重算 `batch_state` 即可 |
| ASM-003 | 生产暂在原 Windows 机器继续，云端/他机为增量目标 | 所有者选择"跨机器可重现 + 扩展"，未说迁移 | 无重大风险，脚本双平台已覆盖 |

### 10.2 待确认问题

| 编号 | 问题 | 是否阻塞 | 备注 |
|---|---|---:|---|
| Q-001 | 统一 teacher_id 用 `teacher_hsk1_set_a` 还是 `teacher-hsk1`？ | No | **已定案（2026-08-25）**：用 `teacher_hsk1_set_a`。`.tcsl-courseware/` 三份状态文件已用该值，仅 `.courseware/teacher-hsk1/teacher-model.json` 用 `teacher-hsk1`，按信息量与改动量取前者 |
| Q-002 | REQ-009 选 Git LFS 还是 tag+Release？ | No | **已定案（2026-08-25）**：选 Git LFS，只追踪迁移后的新路径、不重写历史。理由见 REQ-009 规则 |
| Q-003 | HSK2 教材与样例是否已到位？ | No | 只影响 FLOW-004 演练能否用真数据 |

---

## 11. Agent 系统规格

### 11.1 自主性与人在回路

| 动作类别 | 自主级别 | 审批 / 回滚 |
|---|---|---|
| 状态核对与差异报告 | 自动 | 只读，无需审批 |
| 状态合并/回写、目录重构 | 建议确认 | git 可回滚；归档保留 |
| 删除 `.courseware/` 旧状态 | 需人确认 | 先归档后删 |
| 修改 SKILL.md 门禁条款 | 建议确认 | 逐条给 diff 供确认 |
| 修改已交付 PPTX | 禁止 | — |

### 11.2 工具与能力集

| 工具 / 能力 | 用途 | 权限级别 |
|---|---|---|
| 文件读写 + git | 状态迁移、目录重构 | 读/写 |
| sha256sum / Python | 三方核对、校验脚本 | 执行 |
| python-pptx | 只读核对页数/备注数 | 读 |

### 11.3–11.8 摘要

- 上下文：改造任务单 session 可完成；状态核对结果写入中间文件，不依赖记忆。
- 编排：单 Agent 顺序执行即可，无需子 Agent；Phase 间有依赖（见 DEV-PLAN）。
- Eval：完成定义清单即回归集；每 Phase 结束跑 FLOW-001 冷启动演练。
- 成本：纯文件操作，可忽略。
- 失败兜底：任何核对差异无法解释时停下报告，不强行合并。
- 状态：改造本身的进度记录在 DEV-PLAN 勾选项中，可中断续作。
