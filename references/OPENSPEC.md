# OpenSpec 规范

覆盖三个命令：`/opsx:explore`（需求探索）、`/opsx:propose`（规格固化）、`/opsx:archive`（归档）。

---

## § 需求探索（`/opsx:explore`）

**触发时机：** 代码库探索完成、用户确认摘要后进入。

### 执行顺序（6 步）

1. 基于探索摘要，**一次只问一个**澄清问题，逐个确认细节
2. 提出 **2-3 个方案**，带权衡分析和推荐
3. 分段展示设计，每段确认后继续
4. 写设计备忘（暂存，后续由 `/opsx:propose` 正式固化）
5. 设计自查（检查 placeholder、矛盾、歧义、范围蔓延）
6. 请人工审阅，确认后自动衔接 `/opsx:propose`

### 行为约束

- 不能同时问多个问题
- 必须提供方案对比，不能只给一个选项
- 不能在需求边界确认前开始写代码
- 不能跳过代码库探索直接进入此步骤
- 不能跳过此步骤直接进入 `/opsx:propose`

---

## § 规格固化（`/opsx:propose`）

**触发时机：** `/opsx:explore` 需求方向经人工确认后自动衔接。

### 初始化命令

```bash
# 首次初始化项目（仅执行一次）
openspec init

# 创建变更脚手架（只建目录，不填内容）
openspec-cn new change "<name>"

# 基于 /opsx:explore 结论生成完整提案
/opsx:propose <change-name>
```

### 产出物结构

所有文件**统一存放**在 `openspec/changes/<name>/`，不散落到其他位置：

```
openspec/changes/<name>/
├── proposal.md     # 意图和范围（为什么做、影响范围、回滚计划）
├── design.md       # 技术方案（怎么做、架构决策、风险权衡）
├── specs/
│   └── <feature>/spec.md   # 行为规约（Given/When/Then）
├── tasks.md        # 实现任务清单（checkable）
└── plan.md         # 由 writing-plans 生成（执行阶段产出）
```

参考模板：`templates/proposal.md`

### 行为规约格式（Given/When/Then）

```markdown
- GIVEN <前置条件>
- WHEN <触发动作>
- THEN <期望结果>
```

### Delta Spec 变更类型

| 类型 | 含义 |
|------|------|
| `ADDED Requirements` | 新增行为 |
| `MODIFIED Requirements` | 修改行为 |
| `REMOVED Requirements` | 废弃行为 |

### 制品依赖关系（DAG）

```
proposal（根节点）
    ├── specs（依赖 proposal）
    ├── design（依赖 proposal）
    └── tasks（依赖 specs + design）
```

> 不能跳过 proposal 直接写 tasks。

### 行为约束

- `proposal.md` 必须包含：意图、影响范围、回滚计划
- `specs` 使用 Given/When/Then 格式，不使用自由格式
- 变更使用增量描述，不覆盖 `openspec/specs/` 主规约
- **人工确认 `proposal.md` 后，OpenSpec 提案即为执行边界，后续不得随意扩展**

---

## § 归档（`/opsx:archive`）

**触发时机：** 分支收尾、PR 创建后执行。

```bash
/opsx:archive <change-name>
```

### 归档时 AI 自动执行

1. Delta Spec 合并到 `openspec/specs/`
2. 扫描 `openspec/changes/<name>/design.md`：若含跨模块影响 / 新增外部依赖 / 数据模型变更，**提请人工确认**是否需要单独记录架构决策；人工点头才写入

### 合并代码

```bash
# 读取父分支
parent=$(git config --get branch.feature/<name>.parent)

# 合并回父分支
git checkout $parent
git merge feature/<name>

# 删除 feature 分支
git branch -d feature/<name>
```