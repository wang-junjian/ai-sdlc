# SDD 工具与 AI-Native SDLC 环节映射分析

> 基于《The AI-Native SDLC Playbook》的两张核心图示（线 vs 环、Plays 采用顺序依赖图），
> 对比 Spec Kit / OpenSpec / mattpocock-skills 三个 SDD 工具在各环节的覆盖方式。

## 1. 两张图的核心信息

### 图 1：线 vs 环

传统 SDLC 是 Plan→…→Maintain 的单向流水线，回流一次就是一个新发布周期；AI 原生模式下六阶段变成以 Claude 为中心的**环**，周期从周压缩到小时，人从"环内执行者"变为"环上的发起、指挥与治理者"。

### 图 2：Plays 采用顺序依赖图

图中箭头是**采纳顺序**（先建什么能力），不是阶段顺序。五个橙色起点 play 无前置依赖，可任意开始：

```text
第1层(起点): Capture intent │ CLAUDE.md │ Feedback loop │ Hooks │ Plan mode
第2层:       Skills / Subagents / Evals        ← 依赖 CLAUDE.md、Feedback loop、Hooks
第3层:       Requirements & design / PR review  ← 依赖 intent、Skills、Evals
第4层:       CI/CD                              ← 依赖 Plan mode、PR review
第5层:       Closing the loop                   ← 依赖 Capture intent、CI/CD，闭环回写
```

## 2. 逐环节映射

### 2.1 Plan（Capture intent → intent.md）

Playbook 主张：提出者用自己的语言与 AI 头脑风暴，产出**版本化的 proto-spec**。

| 工具 | 工作流 | 对应产物 |
|---|---|---|
| **OpenSpec** | `/opsx:explore`（先澄清想法、调研代码库）→ `/opsx:propose` | `changes/<name>/proposal.md` —— 最接近 intent.md 的定位：why + what changes |
| **Spec Kit** | `/speckit-specify`（自动编号 001/002…、自动建分支、生成 `specs/[branch]/` 结构） | `spec.md` 直接进入正式规格，跳过了 proto-spec 这一层 |
| **mattpocock** | `research`（查一手资料存成 Markdown）→ 对话收敛后 `to-spec`（把对话发布到 issue tracker） | issue 即 intent，工件留在 tracker 而非 repo |

**差异要点**：OpenSpec 的 explore→propose 与 playbook 的"头脑风暴→intent.md"几乎一一对应；Spec Kit 把 Plan 和 Design 压缩成一步，适合意图已清晰的新项目；mattpocock 不定义 intent 格式，把载体交给团队已有的 issue tracker。

### 2.2 Design（Requirements & design → spec.md）

Playbook 主张：需求与设计在一次会话内完成，组织政策以 skills 形式**在写 spec 时**生效，而非数周后评审时发现。

| 工具 | 工作流 | Spec 定义模式 |
|---|---|---|
| **OpenSpec** | propose 时生成 `specs/` + `design.md`；`/opsx:update` 保持各工件一致 | 纯 Markdown 的**增量 delta spec**：`## ADDED Requirements`，每条 Requirement 下挂 `WHEN/THEN` 场景；无特殊语法，brownfield 友好 |
| **Spec Kit** | `/speckit-constitution`（一次性确立原则，相当于 playbook 的"政策 skills"）→ specify → `/speckit-plan` | **全量 spec**：用户故事 + 验收标准 + 强制澄清标记 `[NEEDS CLARIFICATION]`；constitution 是全局不可违抗约束 |
| **mattpocock** | `domain-modeling`（统一领域词汇）→ `grill-with-docs`（拷问式访谈 sharpen 设计，顺带产出 ADR 和术语表）→ 存疑先 `prototype`（一次性原型验证设计问题） | 无固定模板，产出是 ADR + glossary + issue，**人是设计的主导者**，skill 只负责逼问 |

**差异要点**：这是三者分化最大的环节。OpenSpec 的 delta spec + archive 机制天然支持"spec 随代码演进"（brownfield）；Spec Kit 的 constitution 最接近 playbook 说的"政策编码为 skills"；mattpocock 的 grill-with-docs 对应 playbook 里"像分析师一样提问"的步骤，但拒绝模板化。

### 2.3 Build（CLAUDE.md / Plan mode / Skills / Subagents → plan.md + diff）

Playbook 主张：plan mode 起步、提交 `plan.md`、工件可被一个没看过对话的人执行。

| 工具 | 工作流 |
|---|---|
| **Spec Kit** | `/speckit-plan`（读 spec + constitution，产出 plan + data-model + contracts + research）→ `/speckit-tasks`（拆成可执行任务清单）→ `/speckit-implement`。**链路最完整**，plan.md 是一等公民 |
| **OpenSpec** | 快路径 `/opsx:propose` 一步生成全部规划工件；展开路径 `/opsx:new` → `/opsx:continue`（逐工件生成）→ `/opsx:ff`；执行用 `/opsx:apply` 按 `tasks.md` checklist 逐项打勾，**允许实现中回改工件**（fluid） |
| **mattpocock** | `to-tickets`（把 plan/spec 拆成 tracer-bullet tickets，声明依赖）→ `implement`（按 spec/ticket 实现）。超大块工作用 `wayfinder` 拆成跨会话的决策 ticket 地图 |

**差异要点**：Spec Kit 是"计划先行、工件驱动实现"的重量级流程；OpenSpec 的 apply 允许边做边改工件，对应 playbook 的"实现偏离计划时在同一 commit 更新 plan.md"；mattpocock 用 ticket 替代 plan.md，把"计划"外置到 tracker。

### 2.4 Test（Feedback loop / Evals）

Playbook 主张：会话自我验证（跑测试/构建/截图），验证先于人看；配置变更要像代码一样过 eval 回归。

| 工具 | 能力 |
|---|---|
| **mattpocock** | **最强**：`tdd`（red-green-refactor，正是 playbook"先写失败测试再修"的玩法）；`diagnosing-bugs`（疑难 bug 的诊断循环） |
| **Spec Kit** | `/speckit-converge`：实现完毕后对照 spec、plan、tasks **收敛校验**，反复直到报告 Converged |
| **OpenSpec** | `/opsx:verify`：验证实现与工件是否一致 |

**差异要点**：三者都覆盖"对照工件验证"，但只有 mattpocock 覆盖 playbook 强调的**会话内即时反馈环**（TDD）；Evals 这一层（对 agent 配置本身的回归测试）三者都没有，需按 playbook 自建 CI eval 套件。

### 2.5 Deploy（Hooks / PR review / CI/CD）

Playbook 主张：hook 做确定性审批门，PR review 双向（AI 评审人、AI 响应评审），agent 止步于 production gate。

三个工具在这一环节**都基本没有内建能力**——它们刻意不碰部署治理。能用的是：

- **mattpocock**：`code-review`（沿 Standards 与 Spec 符合度两条轴审查 diff，最接近 playbook 的 REVIEW.md 分 pass 思路）、`triage`（issue/PR 状态机分拣）、`resolving-merge-conflicts`、`wizard`（生成 bash 向导，引导人执行只有人能做的步骤——恰好对应"production gate 留给人"）
- **OpenSpec / Spec Kit**：依赖 git 分支 + PR 的常规流程，review 交给 agent 平台自身能力（如 Claude Code Review）

这一环节是 playbook 相对三个工具最大的增量：**hooks as approval gates + 分层评审**需要按 playbook 第 5 章自建。

### 2.6 Maintain（Closing the loop）

Playbook 主张：确定性检测脚本发现 control-band 违规 → 无人工触发 Claude → 诊断写回为新的 intent.md，环自启动。

| 工具 | 相关机制 |
|---|---|
| **OpenSpec** | `/opsx:archive` 把完成的 change 合并回 `openspec/specs/` 主规格——**唯一内建"知识回写"机制**的工具，但方向是"实现→规格"，不是"生产→intent" |
| **mattpocock** | `diagnosing-bugs` + `triage` 覆盖人工响应侧；`wizard` 可固化 runbook |
| **Spec Kit** | 无对应机制（其 spec-driven.md 提到生产反馈更新规格，但工具未实现） |

**结论**：闭环的检测层（确定性脚本、bands.yaml 分层响应）三个工具都不提供，是自建空间；但 OpenSpec 的 change→archive 循环可以复用为"新 intent 落地"的承载结构。

## 3. 整体覆盖度与组合建议

把三者放到 Plays 依赖图上，覆盖度一览：

```text
Playbook 环节        OpenSpec      Spec Kit      mattpocock    需自建
Capture intent       ●●●          ●●            ●●
Requirements/design  ●●●          ●●●           ●●
Plan/Build           ●●●          ●●●           ●●
Feedback loop/Test   ●            ●●            ●●●
PR review/Deploy     ○            ○             ●●            hooks/gates
Closing the loop     ●(archive)   ○             ●             检测脚本+触发器
```

**可落地的组合策略**：

1. **以 OpenSpec 为工件骨架**：`changes/` + delta spec + archive 机制最贴合 playbook 的"工件链即审计轨迹"，且 brownfield 友好，可从任意存量项目起步。
2. **以 mattpocock skills 填充工程实践层**：TDD（feedback loop）、code-review（PR 双向评审）、grill-with-docs（设计澄清）——它们不拥有流程，恰好嵌入 OpenSpec 的各个环节而不冲突。
3. **Spec Kit 作为 greenfield 参照系**：其 constitution→specify→plan→tasks→converge 全链路展示了"规格即源码"的极限形态，适合新项目或需要强审计的受监管场景借鉴其 constitution 机制。
4. **自建治理层**：hooks（审批门）、evals（配置回归）、生产闭环（检测脚本→intent）按 playbook 落地——这是三个工具共同的空白，也是方法论的最大增值点。
