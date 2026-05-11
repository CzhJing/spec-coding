# Claude Code Agent Team 模式

**触发条件：** 人工确认节点用户选择"Agent Team"，且当前在 Claude Code 环境中。

**核心机制：** AI 读取 `openspec/changes/<name>/tasks.md`，根据任务描述和依赖关系，自动从 `~/.claude/agents/` 中选择最合适的 subagent 执行每个 Task，无需人工指定；如果`~/.claude/agents/` 中没有合适的 subagent，则根据实际工作内容创建合适的subagent。

---

## 执行前置条件

```bash
# 确认可用的 subagents
ls ~/.claude/agents/
```

AI 在分配任务前必须先列出可用 subagents，理解每个 agent 的能力范围，再做匹配决策。

---

## AI 调度规则

### 1. 能力匹配

AI 根据以下信息自动选择 subagent：

- Task 的**文件类型**（`.java`、`.ts`、`*.sql` 等）
- Task 的**操作性质**（实现、测试、重构、数据库变更等）
- subagent 的**名称和描述**中体现的专长

匹配优先级：**专用 agent > 通用 agent**。如果`~/.claude/agents/` 中没有合适的 subagent，则根据实际工作内容创建合适的subagent。

### 2. 依赖调度

读取 tasks.md 中的 `depends_on` 字段决定启动顺序：

```markdown
### Task N: <任务名称>
**depends_on:** [Task ID, ...]   # 无依赖填 none
```

- `depends_on: none` → 同时启动，分配给对应 agent
- 有依赖 → 前序 Task 完成后才启动

### 3. 文件冲突预检（并行安全保障）

调度计划生成后、执行前，AI 必须检查所有并行 Task 的操作文件是否有交集：

**处理规则：**
- 操作文件**无交集** → 允许并行，保持原调度计划
- 操作文件**有交集且涉及写操作** → 强制降级为顺序执行，在调度摘要中注明原因
- 操作文件**有交集但均为只读** → 允许并行（读操作无并发风险）

**tasks.md 写入规则：**
- 各 agent 完成任务后向主进程上报结果，由主进程统一勾选 `tasks.md` 中的 checkbox
- agent 不得直接修改 `tasks.md`，避免并发写入冲突

### 4. 调度摘要输出

分配开始前，AI 必须输出调度计划供用户确认：

```
## Agent Team 调度计划

Task 1: <任务名> → 分配给 <agent-name>（理由：<一句话说明>）
Task 2: <任务名> → 分配给 <agent-name>（理由：<一句话说明>）
Task 3: <任务名> → 主进程直接执行（无匹配 agent，不创建新 subagent）
Task 4: <任务名> → ⚠️ 降级顺序执行（与 Task 3 操作文件有交集）
...

并行组：[Task 1, Task 2]（depends_on: none）
顺序组：Task 3 在 Task 1 完成后启动

确认后开始执行，输入"继续"即可。
```

**未得到用户确认，不得开始执行。**

---

## 每个 Agent 的执行规范

每个 subagent 接到任务后，内部仍遵循 TDD 循环：

```
RED      → 写失败测试，运行确认失败
GREEN    → 写最小实现，运行确认通过
REFACTOR → 清理代码，测试仍通过
COMMIT   → feat/fix/refactor: <description>
```

---

## 三级审查

所有 agent 完成后统一审查：

```
所有 agent 完成
  ↓
规约审查：对照 openspec/changes/<name>/specs/ 中的 spec 描述
  → 不通过：定位到具体 Task，retry_count += 1
             retry_count < 3：重新分配给原 agent 修复
             retry_count = 3：⚠️ 触发升级机制
  ↓
代码质量审查：风格、命名、冗余、测试质量
  → 不通过：同上
  ↓
全部通过 → 进入分支收尾
```

每个 Task 独立计 `retry_count`。

---

## 升级机制（retry_count = 3 时触发）

```
⚠️ 任务 [Task N: <任务名称>] 审查连续失败 3 次，已暂停。

执行 agent：<agent-name>
失败原因（最近一次）：
- 审查类型：规约审查 / 代码质量审查
- 具体问题：<描述>

已尝试的修改方向：
1. <第1次修改思路>
2. <第2次修改思路>
3. <第3次修改思路>

请指示：
A. 提供额外上下文或指引方向，继续尝试（可指定换用其他 agent）
B. 降低该任务的审查标准（需说明原因）
C. 跳过该任务，稍后人工处理
```

---

## 行为约束

- AI 必须先列出可用 subagents 再做分配，不得凭空假设 agent 能力
- 调度计划必须输出并等待用户确认，不得静默开始
- 如果`~/.claude/agents/` 中没有合适的 subagent，则根据实际工作内容创建合适的subagent
- 并行 Task 操作文件有写冲突时，强制降级为顺序执行，不得忽略
- tasks.md 的 checkbox 由主进程统一勾选，agent 不得直接写入 tasks.md
- 规约审查对照 `openspec/changes/<name>/specs/` 目录，不使用 plan.md
- 前后端如需并行，各自开独立会话，不在同一会话中混合执行