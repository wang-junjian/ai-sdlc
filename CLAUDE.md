# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

这是一个**研究型项目**（非代码项目），目标是研究 AI 原生软件开发生命周期（AI-Native SDLC），通过对比分析多个 SDD（Spec-Driven Development，规范驱动开发）开源项目的 Spec 定义模式与 Workflows，结合核心参考文档，总结出一套可落地的 AI 原生开发方法论。

仓库内没有构建、测试或代码产物；工作产出是对比分析文档与方法论沉淀（Markdown）。

## 语言约定

所有分析与输出内容使用**中文**。

## 仓库结构

- `references/the-ai-native-sdlc-playbook/` — 核心参考文档：Anthropic 官方博客《The AI-Native SDLC playbook》的 Markdown 存档及配图。
- `analysis/` — 研究产出：对比分析与方法论文档，**一律输出为 HTML 格式（亮/暗双主题）**。

## 本地参考项目（仓库外部）

以下三个 SDD 开源项目位于本仓库之外，分析时直接读取，**不要修改**：

| 项目 | 路径 | 核心模式 |
|---|---|---|
| GitHub Spec Kit | `/Users/junjian/GitHub/github/spec-kit` | 规格即源码（spec 是 source of truth，代码是其生成物）。命令链：`/speckit.constitution` → `/speckit.specify` → `/speckit.plan` → `/speckit.tasks` → `/speckit.implement` → `/speckit.converge`。核心理念见 `spec-driven.md`，模板见 `templates/` |
| Fission-AI OpenSpec | `/Users/junjian/GitHub/Fission-AI/OpenSpec` | 面向 brownfield 的轻量增量模式：`openspec/changes/<change>/` 下 proposal.md + specs/（ADDED Requirements + WHEN/THEN 场景）+ design.md + tasks.md，验收后 archive 回 `openspec/specs/`。命令链：`/opsx:explore` → `/opsx:propose` → `/opsx:apply` → `/opsx:archive`。哲学：fluid / iterative / easy / brownfield-first |
| mattpocock/skills | `/Users/junjian/GitHub/mattpocock/skills` | 反流程框架立场：不拥有流程，只提供小而可组合的 skills（`skills/engineering/`：tdd、to-spec、to-tickets、research、diagnosing-bugs 等），强调工程师保留控制权 |

## 核心参考文档要点（the-ai-native-sdlc-playbook.md）

对比分析时以该文档为理论基准，其关键主张：

1. **代码不再是瓶颈**，瓶颈转移到构建阶段两侧的 plan、review/test、deploy 等仍以人速运转的环节。
2. **工件链（artifact chain）即审计轨迹**：`intent.md` → `spec.md` → `plan.md` → diff+tests → PR+review findings → incident record。每个阶段提交工件、下一阶段读取工件，形成循环而非线性流程。
3. **控制分层**：skill 是建议性控制（advisory），hook 是确定性强制（deterministic），人类审批保留在 production gate。
4. **闭环**：生产环境的 control-band 违规由确定性脚本检测，触发 Claude 写回新的 `intent.md`，循环自启动。

## 分析方法建议

做对比分析时，围绕以下维度展开（与 playbook 的阶段对应）：

- Spec 的定义模式：模板结构、粒度、人类可读 vs 机器可执行的平衡
- 工作流编排：阶段划分、阶段间触发方式、人在回路中的位置（gate 在哪）
- 治理与落地：如何保证 spec 与代码同步、回滚与归档机制、brownfield 适配能力
- 与 playbook 工件链的映射：各自的产物对应 intent/spec/plan/review 中的哪一环
