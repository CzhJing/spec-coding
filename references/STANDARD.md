# Superpowers 标准模式

**触发条件：** 人工确认节点用户选择"Superpowers"。

流程：writing-plans → executing-plans（TDD + 三级审查）。

---

## § 制定计划（writing-plans）

参考模板：`templates/plan.md`

**Agent 行为约束：**

- 每个步骤必须是 2-5 分钟可完成的原子操作
- 每个任务必须包含测试文件路径
- 计划内容不得超出 OpenSpec 提案的边界（`tasks.md` 是上限）
- 产出路径固定：`openspec/changes/<name>/plan.md`，不散落到其他位置
- 计划生成后必须做自查：spec 覆盖度、placeholder、类型一致性

---

## § TDD 循环

每个 Task 严格按此顺序，不可跳步：

```
RED      → 写失败测试，运行确认失败
GREEN    → 写最小实现，运行确认通过
REFACTOR → 清理代码，测试仍通过
COMMIT   → feat/fix/refactor: <description>
```

---

## § 三级审查循环

```
任务开始（retry_count = 0）
  ↓
实现子代理完成 TDD 循环
  ↓
规约审查子代理
  → 对照 openspec/changes/<name>/plan.md 中的 spec 描述
  → 不通过 → retry_count += 1
             retry_count < 3：附失败原因，打回实现子代理
             retry_count = 3：⚠️ 触发升级机制
  ↓
代码质量审查子代理
  → 检查风格、命名、冗余、测试质量
  → 不通过 → retry_count += 1
             retry_count < 3：附失败原因，打回实现子代理
             retry_count = 3：⚠️ 触发升级机制
  ↓
全部通过 → 任务标记完成 ✅ → 在 tasks.md 勾选 → 下一个任务
```

**子代理优先级：** 优先从 `~/.codex/agents/` 和 `~/.claude/agents/` 中查找已安装 subAgent。

---

## § 升级机制（retry_count = 3 时触发）

Agent 必须立即停止，输出以下格式，等待人工介入，不得自行继续：

```
⚠️ 任务 [Task N: <任务名称>] 审查连续失败 3 次，已暂停。

失败原因（最近一次）：
- 审查类型：规约审查 / 代码质量审查
- 具体问题：<审查子代理给出的失败描述>

已尝试的修改方向：
1. <第1次修改思路>
2. <第2次修改思路>
3. <第3次修改思路>

请指示：
A. 提供额外上下文或指引方向，Agent 继续尝试
B. 降低该任务的审查标准（需说明原因）
C. 跳过该任务，稍后人工处理
```

---

## § 行为约束

- 不允许跳过任何一级审查
- 实现必须严格按 TDD 循环执行
- 规约审查对照 **plan.md 中的 spec**，不直接对照 `openspec/specs/` 主规约
- 前后端如需并行，各自开独立会话，不在同一会话中混合执行