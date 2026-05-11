# Ruflo Swarm 模式

**触发条件：** 人工确认节点用户选择"Ruflo swarm"。

**输入：** 直接读取 `openspec/changes/<name>/tasks.md`，不需要生成 plan.md。

---

## tasks.md 中的依赖声明格式

```markdown
### Task N: <任务名称>
**depends_on:** [Task ID, ...]   # 无依赖填 none
```

## 依赖判断依据（满足任一即为有依赖）

- 修改了同一文件
- 依赖前序 Task 产出的接口或类型
- 共享状态 / 数据库 schema 须前序先建立

---

## tasks.md 与状态管理

**tasks.md 在 Ruflo 模式下只是输入配置，不是运行时状态文件。**

Ruflo 有自己的状态管理机制，不依赖 tasks.md 的 checkbox 追踪进度：

- **共享内存 namespace**：所有 worker 通过 `memory store/retrieve` 共享任务状态，运行时状态存在 Ruflo 内存里
- **Coordinator agent**：hierarchical 拓扑下有专属 Coordinator 负责协调所有 worker，维护权威状态（raft consensus），worker 向 Coordinator 上报结果而不是直接改文件
- **依赖层级执行**：Ruflo 按 `depends_on` 字段自动划分执行层级（Level 0 → Level 1 → Level 2），同层并行，跨层顺序，框架层保证执行顺序

因此：
- worker **不直接写 tasks.md**，无并发写入冲突
- 任务完成后由 Ruflo 统一归档状态，tasks.md 的 checkbox 可在所有任务完成后由主进程批量更新

---

## 执行规则

- `depends_on: none` 的 Task 作为起始节点**同时启动**
- 每个 swarm worker 负责独立任务单元，不互相干扰
- 每个 worker 内部遵循 TDD 循环：RED → GREEN → REFACTOR
- 所有 worker 完成后**统一**进行三级审查

---

## 三级审查

```
所有 worker 完成
  ↓
规约审查：对照 openspec/changes/<name>/specs/ 中的 spec 描述
  → 不通过：定位到具体 worker，retry_count += 1
             retry_count < 3：附失败原因，该 worker 重新实现
             retry_count = 3：⚠️ 触发升级机制（格式见下）
  ↓
代码质量审查：风格、命名、冗余、测试质量
  → 不通过：同上
  ↓
全部通过 → 进入分支收尾
```

每个 worker 独立计 `retry_count`，互不影响。

---

## 升级机制（retry_count = 3 时触发）

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

## 行为约束

- 不允许跳过任何一级审查
- 规约审查对照 `openspec/changes/<name>/specs/` 目录，不使用 plan.md
- 前后端如需并行，各自开独立会话，不在同一会话中混合执行