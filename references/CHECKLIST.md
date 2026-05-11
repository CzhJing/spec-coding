# 执行检查清单

每个阶段**开始前**过一遍对应清单，完成后再进入下一阶段。

---

## ① 代码库探索

- [ ] 已定位相关模块，阅读了接口 / 类型 / 数据模型
- [ ] 探索摘要已按格式输出
- [ ] 探索摘要已呈现给用户，用户已确认
- [ ] 探索阶段未写任何代码

## ② 需求探索（`/opsx:explore`）

- [ ] 基于代码库探索摘要提问，未脱离现有代码约束
- [ ] 每次只问了一个问题
- [ ] 提供了 2-3 个方案对比（不是只给一个选项）
- [ ] 需求边界清晰，人工已确认

## ③ OpenSpec 提案

- [ ] `proposal.md` 包含：意图、范围、回滚计划
- [ ] `specs` 使用 Given/When/Then 格式
- [ ] 制品依赖顺序正确：proposal → specs/design → tasks
- [ ] 人工已确认 `proposal.md`（边界已锁定）

## ④ 创建分支

- [ ] feature 分支已创建
- [ ] 父分支元数据已记录（`git config branch.<name>.parent`）
- [ ] 测试基线通过（0 failures）

## ⑤ 执行（所有模式公共项）

- [ ] 已向用户确认执行模式（Ruflo swarm / Agent Team / Superpowers），得到明确回复

**Ruflo swarm 模式追加：**
- [ ] `tasks.md` 中各 Task 已声明 `depends_on` 字段
- [ ] 起始节点（`depends_on: none`）已识别

**Agent Team 模式追加：**
- [ ] 已列出 `~/.claude/agents/` 下可用 subagents
- [ ] 调度计划已输出（Task → agent 映射 + 并行/顺序分组）
- [ ] 用户已确认调度计划
- [ ] `tasks.md` 中各 Task 已声明 `depends_on` 字段

**Superpowers 模式追加：**
- [ ] `plan.md` 写入 `openspec/changes/<name>/plan.md`
- [ ] 每个 Task 已填写 `depends_on` 字段（无依赖填 `none`）
- [ ] 计划内容未超出 OpenSpec 边界（`tasks.md` 是上限）
- [ ] 计划自查通过：无 placeholder、无矛盾、类型一致

## ⑤ 实现阶段

- [ ] 每个任务严格按 TDD：RED → GREEN → REFACTOR
- [ ] 三级审查全部通过才标记任务完成
- [ ] 每步提交信息清晰（`feat/fix/refactor: <description>`）
- [ ] 未出现 retry_count = 3 的悬停任务（或已人工处理）

## ⑥ 分支收尾

- [ ] PR 已推送并创建
- [ ] PR body 包含变更说明和 OpenSpec 边界路径

## ⑦ 归档

- [ ] `/opsx:archive` 已执行，Delta Spec 已合并到 `openspec/specs/`
- [ ] 跨模块影响 / 外部依赖变更已提请人工确认
- [ ] feature 分支已合并并删除