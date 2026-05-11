# <Feature> Implementation Plan

**Goal:** <本次实现的目标>
**OpenSpec 边界：** `openspec/changes/<name>/`
**Architecture:** <架构描述>
**Tech Stack:** <技术栈>

---

### Task 1: <任务名称>

**depends_on:** none
**Files:**
- Create/Modify: `<文件路径>`
- Test: `<测试文件路径>`

- [ ] Step 1: 写失败测试（RED）
- [ ] Step 2: 运行测试确认失败
- [ ] Step 3: 写最小实现（GREEN）
- [ ] Step 4: 运行测试确认通过
- [ ] Step 5: 重构（REFACTOR）
- [ ] Step 6: 提交

---

### Task 2: <任务名称>

**depends_on:** [Task 1]
**Files:**
- Create/Modify: `<文件路径>`
- Test: `<测试文件路径>`

- [ ] Step 1: 写失败测试（RED）
- [ ] Step 2: 运行测试确认失败
- [ ] Step 3: 写最小实现（GREEN）
- [ ] Step 4: 运行测试确认通过
- [ ] Step 5: 重构（REFACTOR）
- [ ] Step 6: 提交

---

<!--
depends_on 填写规则：
- 无前置依赖 → none（可作为 swarm 起始节点）
- 有依赖 → [Task N, Task M]（填编号）

判断是否有依赖（满足任一即是）：
- 修改了同一文件
- 依赖前序 Task 产出的接口或类型
- 共享状态 / 数据库 schema 须前序先建立
-->