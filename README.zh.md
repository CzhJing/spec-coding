# SpecCoding

> AI 辅助开发工作流 Skill。先用 OpenSpec 锁定边界，再用 Superpowers 在边界内执行。

---

## 概述

SpecCoding 是一个结构化的 AI 辅助开发工作流，适用于任何新功能开发、需求变更或 Bug 修复。

**核心理念：**

- **设计靠人决策** — 需求与边界由人工确认
- **执行尽量自主** — AI 在锁定边界内自主执行，尽量少中断

**工作流：**

```
代码库探索 → /opsx:explore → /opsx:propose → 创建分支 → 执行 → PR → /opsx:archive
```

---

## 前置要求

### 1. OpenSpec

> 需求探索 + 规格固化 + 变更记忆。

```bash
# 安装
pip install openspec-cli

# 或手动克隆安装
git clone https://github.com/Fission-AI/OpenSpec.git
cd OpenSpec
pip install -e .

# 验证
openspec --version
```

### 2. Superpowers

> 执行框架：计划拆分 + 代码实现 + TDD + 三级审查。

```bash
# 安装
pip install superpowers

# 或手动克隆安装
git clone https://github.com/obra/superpowers.git
cd superpowers
pip install -e .
```

### 3. Ruflo（可选 — 用于并行 Swarm 执行）

> 基于任务依赖关系的多智能体 Swarm 编排。

```bash
# 安装
pip install ruflo

# 或手动克隆安装
git clone https://github.com/ruvnet/ruflo.git
cd ruflo
pip install -e .

# 验证
ruflo --version
```

### 4. Claude Code Agent Teams 模式（可选 — 用于子代理调度）

> Claude Code 内置的多智能体团队协作。

**在 Claude Code 中启用：**

在 Claude Code 配置文件（如 `~/.claude/settings.json`）中添加环境变量：

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**验证可用的子代理：**

```bash
ls ~/.claude/agents/
```

---

## 安装

### 方式一：直接复制（推荐）

```bash
# 克隆此 Skill 仓库
git clone https://github.com/CzhJing/spec-coding.git ~/.claude/skills/speccoding

# 或复制到你的项目中
cp -r speccoding/ <your-project>/.claude/skills/
```

### 方式二：Claude Code Skill 注册

```bash
# 注册到 Claude Code
claude skill add speccoding /path/to/speccoding/SKILL.md
```

---

## 快速开始

### 触发关键词

提及以下任一关键词即可激活 SpecCoding：

- "开发新功能"
- "修改需求"
- "实现 XXX"
- "加一个 XXX 接口"
- "重构 XXX"
- "修复 Bug"

### 8 步工作流

```
① 代码库探索      → 读代码、出摘要、等确认
② /opsx:explore   → 需求探索（一次一问、提供 2-3 个方案）
③ /opsx:propose   → 生成 proposal、design、specs、tasks
    ⚑ 人工审核    → 审查 proposal.md，确认需求方向和边界
④ 创建分支        → git checkout -b feature/<name>
    ⚑ 选择模式    → 选择执行模式：

    并行执行（任务多、依赖关系清晰）：
    ⑤-A Ruflo Swarm   → 基于 depends_on 的多代理 Swarm
    ⑤-B Agent Team    → Claude Code 从 ~/.claude/agents/ 调度子代理

    顺序执行（任务少、依赖复杂）：
    ⑤-C Superpowers   → writing-plans → TDD → 三级审查

⑥ 分支收尾        → git push + 创建 PR
⑦ /opsx:archive   → 合并 Delta Spec，清理分支
```

---

## 执行模式

| 模式 | 适用场景 | 前置要求 |
|------|----------|----------|
| **Ruflo Swarm** | 任务多、依赖清晰 | 已安装 Ruflo |
| **Agent Team** | Claude Code 环境、有可用子代理 | Claude Code + ~/.claude/agents/ |
| **Superpowers** | 任务少、依赖复杂 | 已安装 Superpowers |

### 模式选择指南

```
任务多 + 依赖清晰  →  Ruflo Swarm
Claude Code + 有代理 →  Agent Team
任务少 + 依赖复杂  →  Superpowers
```

---

## 文件结构

```
speccoding/
├── SKILL.md                          # Skill 定义
├── README.md                         # 英文文档
├── README.zh.md                      # 本文档
├── references/
│   ├── EXPLORE.md                    # 代码库探索规则
│   ├── OPENSPEC.md                   # OpenSpec 命令（/opsx:explore、/opsx:propose、/opsx:archive）
│   ├── BRANCH.md                     # 分支创建、PR、合并
│   ├── CODING-PRINCIPLES.md          # 四条编码原则
│   ├── STANDARD.md                   # Superpowers 标准模式
│   ├── SWARM.md                      # Ruflo Swarm 模式
│   ├── AGENT-TEAM.md                 # Claude Code Agent Team 模式
│   ├── CHECKLIST.md                  # 各阶段检查清单
│   └── ANTI-PATTERNS.md              # 禁止行为 + 正确做法
└── templates/
    ├── plan.md                       # plan.md 模板（含 depends_on）
    ├── proposal.md                   # proposal.md 模板
    └── pr-body.md                    # PR body 模板
```

---

## 核心行为准则

### 编码原则

1. **编码前思考** — 明确假设、呈现权衡、困惑时停下来问
2. **简洁优先** — 最少代码解决问题，不过度推测
3. **精准修改** — 只碰必须碰的，只清理自己造成的混乱
4. **目标驱动执行** — 定义成功标准，循环验证直到达成

### TDD 循环（强制）

```
RED     → 写失败测试，运行确认失败
GREEN   → 写最小实现，运行确认通过
REFACTOR→ 清理代码，测试仍通过
COMMIT  → feat/fix/refactor: <description>
```

### 三级审查

```
规约审查 → 对照 openspec/changes/<name>/specs/
代码质量审查 → 风格、命名、冗余、测试质量
全部通过 → 标记完成 ✅
```

---

## 变更分级

| 变更类型 | 示例 | 是否走完整流程 |
|----------|------|----------------|
| 小改动 | 文案、日志、小重构 | 可简化，影响行为时补 delta spec |
| 常规功能 | 新增接口、流程调整 | 必须完整走 8 步 |
| 复杂重构 | 拆模块、改鉴权、换库 | 必须完整走，verify 包含回滚检查 |

---

## 何时停下来请示

AI 仅在以下场景暂停并请求人工输入：

1. 实际情况与已确认方案出现冲突（方案里没覆盖的关键分叉）
2. 不可逆 / 高破坏性操作（`git push --force`、`git reset --hard`、删共享分支、改 `main`）
3. 反复尝试同一思路仍失败，需要换方向
4. 规格文件明确要求人工确认的节点

---

## 参考文档

| 文件 | 内容 | 何时阅读 |
|------|------|----------|
| `references/EXPLORE.md` | 代码库探索 6 步规则和摘要格式 | 步骤 ① |
| `references/OPENSPEC.md` | explore / propose / archive 全规范 | 步骤 ②③⑦ |
| `references/BRANCH.md` | 分支创建、父分支记录、PR、合并 | 步骤 ④⑥ |
| `references/CODING-PRINCIPLES.md` | 编码四原则（思考/简洁/精准/目标驱动） | 任何编码任务开始前 |
| `references/SWARM.md` | Ruflo Swarm 模式完整规范 | 步骤 ⑤-A |
| `references/AGENT-TEAM.md` | Claude Code Agent Team 模式（AI 自动调度子代理） | 步骤 ⑤-B |
| `references/STANDARD.md` | Superpowers 标准模式（writing-plans + TDD + 三级审查） | 步骤 ⑤-C |
| `references/CHECKLIST.md` | 各阶段执行前的自检清单 | 每个步骤开始前 |
| `references/ANTI-PATTERNS.md` | 禁止行为 + 正确做法 | 任何时候遇到决策分叉 |

---

## 许可证

MIT
