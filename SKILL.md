---
name: speccoding
description: >
  AI 辅助开发工作流 skill，适用于任何新功能开发、需求变更、Bug 修复。
  基于 OpenSpec → Superpowers 范式：先用 OpenSpec 探索需求并锁定边界，
  再用 Superpowers 在边界内执行。触发关键词：开发新功能、修改需求、
  实现 XXX、加一个 XXX 接口、重构 XXX。
---

# SpecCoding 开发工作流

> 核心理念：**先用 OpenSpec 探索需求并锁定边界，再用 Superpowers 在边界内执行。**
> 设计靠人决策，执行尽量自主。

---

## 工具职责

| 工具 | 职责 |
|------|------|
| **OpenSpec** (`/opsx:explore` → `/opsx:propose`) | 需求探索 + 规格固化 + 变更记忆 |
| **Superpowers 执行工作流** | 计划拆分 + 代码实现 |

---

## 流程骨架（8 步）

```
① 代码库探索      → 读代码、出摘要、等确认
                     详见 references/EXPLORE.md

② 需求探索        → /opsx:explore
                     详见 references/OPENSPEC.md § 需求探索

③ OpenSpec 提案   → /opsx:propose <change-name>
                     详见 references/OPENSPEC.md § 规格固化

⚑ 人工审核        → 审查 proposal.md，确认需求方向和边界
                     【通过】继续 ④ | 【不通过】返回 ② 修订

④ 创建分支        → git checkout -b feature/<name>
                     详见 references/BRANCH.md

⚑ 人工确认        → 选择执行模式（三选一）

  并行执行（任务多、依赖关系清晰）：
  ⑤-A Ruflo swarm  → 已安装 Ruflo，基于 depends_on 字段调度
                       详见 references/SWARM.md
  ⑤-B Agent Team   → Claude Code 环境，优先从 ~/.claude/agents/ 调度
                       详见 references/AGENT-TEAM.md

  顺序执行（任务少、依赖复杂）：
  ⑤-C Superpowers  → writing-plans → executing-plans
                       详见 references/STANDARD.md

⑥ 分支收尾        → git push + PR
                     详见 references/BRANCH.md § 收尾

⑦ 归档            → /opsx:archive <change-name>
                     详见 references/OPENSPEC.md § 归档
```

---

## 执行阶段必须停下来请示的场景（仅限）

1. 实际情况与已确认方案出现冲突（方案里没覆盖的关键分叉）
2. 不可逆 / 高破坏性操作（`git push --force`、`git reset --hard`、删共享分支、改 `main`）
3. 反复尝试同一思路仍失败，需要换方向
4. 规格文件明确要求人工确认的节点

---

## 编码行为准则

详见 `references/CODING-PRINCIPLES.md`，覆盖四条核心原则：编码前思考、简洁优先、精准修改、目标驱动执行。

**任何编码任务开始前必须读取此文件。**

---

## 变更分级

| 变更类型 | 示例 | 是否走完整流程 |
|----------|------|----------------|
| **小改动** | 文案、日志、小重构 | 可简化，影响行为时补 delta spec |
| **常规功能** | 新增接口、流程调整 | 必须完整走 8 步 |
| **复杂重构** | 拆模块、改鉴权、换库 | 必须完整走，verify 包含回滚检查 |

---

## 参考文档索引

| 文件 | 内容 | 何时读 |
|------|------|--------|
| `references/EXPLORE.md` | 代码库探索的 6 步规则和摘要格式 | 步骤 ① |
| `references/OPENSPEC.md` | explore / propose / archive 全规范 | 步骤 ②③⑦ |
| `references/BRANCH.md` | 分支创建、父分支记录、PR、合并 | 步骤 ④⑥ |
| `references/CODING-PRINCIPLES.md` | 编码四原则（思考/简洁/精准/目标驱动） | 任何编码任务开始前 |
| `references/SWARM.md` | Ruflo Swarm 模式完整规范 | 步骤 ⑤-A |
| `references/AGENT-TEAM.md` | Claude Code Agent Team 模式（AI 自动调度 subagents） | 步骤 ⑤-B |
| `references/STANDARD.md` | Superpowers 标准模式（writing-plans + TDD + 三级审查） | 步骤 ⑤-C |
| `references/CHECKLIST.md` | 各阶段执行前的自检清单 | 每个步骤开始前 |
| `references/ANTI-PATTERNS.md` | 禁止行为 + 正确做法 | 任何时候遇到决策分叉 |
| `templates/plan.md` | plan.md 标准格式（含 depends_on） | 制定计划时 |
| `templates/proposal.md` | proposal.md 标准格式 | 生成提案时 |
| `templates/pr-body.md` | PR body 标准格式 | 创建 PR 时 |