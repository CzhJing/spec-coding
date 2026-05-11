# 分支管理

覆盖：创建分支、记录父分支、验证基线、创建 PR、合并收尾。

---

## § 创建分支

**触发时机：** OpenSpec 提案经人工确认后执行。

```bash
# 1. 记录当前分支（父分支，通常为 main）
parent=$(git rev-parse --abbrev-ref HEAD)

# 2. 创建 feature 分支
git checkout -b feature/<name>

# 3. 显式记录父分支元数据（合并时依赖此配置）
git config branch.feature/<name>.parent "$parent"

# 4. 验证测试基线
./mvnw test        # Spring Boot 项目
# npm test         # Node 项目（按项目实际命令替换）
```

### 分支模型

```
main ── feature/<需求A>
      └── feature/<需求B>
```

- feature 分支从 `main`（或当前所在分支）拉出
- 合并时回到**被拉出时的那条分支**（读父分支配置）
- `main` 的合并由人工处理，AI 默认不碰 `main`

### 父分支识别规则

- **创建时**：用 `git config branch.<name>.parent` 显式记录
- **合并时**：用 `git config --get branch.<current>.parent` 读取
- **读不到配置时**：必须向用户确认父分支，不得靠 reflog 猜测

### 行为约束

- 测试基线必须通过（0 failures）；失败须上报，不能直接开始实现
- 所有实现工作在 feature 分支内进行，不在 main 上直接修改

---

## § 分支收尾

**触发时机：** 所有任务三级审查通过后。

> 测试已在各 Task 执行阶段（TDD 循环）和三级审查中完成，此阶段不重复跑测试。

```bash
# 推送分支
git push -u origin feature/<name>

# 创建 PR（使用 templates/pr-body.md 填写 body）
gh pr create \
  --title "feat: <description>" \
  --body "$(cat templates/pr-body.md)"
```

参考模板：`templates/pr-body.md`