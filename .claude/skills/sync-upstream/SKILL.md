---
name: sync-upstream
description: "从上游（母项目）仓库同步最新更新到当前 fork 项目，自动 fetch、merge 并 push。当用户提到 同步上游、sync upstream、同步母项目、拉取上游更新、更新 fork、sync fork、从原始项目同步、同步原始仓库 时触发此 skill。"
---

# Sync Upstream — 同步上游仓库

将 fork 的母项目（upstream）最新提交同步到当前项目，自动完成 fetch → merge → push 全流程。

## 执行流程

### 1. 检查前提条件

先确认工作区状态，确保可以安全合并：

```bash
# 检查 upstream 远程是否已配置
git remote -v

# 检查工作区是否干净（有未提交的改动会导致合并失败）
git status --short
```

- 如果没有 `upstream` 远程仓库，提示用户先添加：`git remote add upstream <母项目URL>`
- 如果工作区有未提交的改动，提示用户先 commit 或 stash，不要自动处理

### 2. 拉取上游更新

```bash
git fetch upstream
```

拉取完成后，检查 upstream/main 是否有新提交：

```bash
git log --oneline HEAD..upstream/main
```

- 如果没有新提交，告知用户"上游没有新的更新"，结束
- 如果有新提交，显示提交数量和摘要，继续下一步

### 3. 合并上游更新

```bash
git merge upstream/main --no-edit
```

- 合并成功：显示合并结果摘要（新增/修改/删除的文件统计）
- 合并冲突：**不要自动解决**，列出冲突文件，告知用户需要手动处理冲突后再继续

### 4. 推送到 origin

合并成功后，推送到自己的远程仓库：

```bash
git push origin
```

推送前确认当前分支有 upstream 跟踪。如果没有，使用 `git push -u origin <当前分支名>`。

### 5. 汇报结果

最终输出包含：
- 同步了多少个新提交
- 主要变更摘要（新功能、修复、文档等）
- push 是否成功

## 注意事项

- 始终合并到当前所在分支（通常是 main）
- 如果当前不在 main 分支，先提醒用户确认是否要在当前分支上合并
- 遇到冲突时停下来让用户决定，不要自动解决冲突
- 不要使用 `git pull`（它会同时 fetch+merge，不够透明）
- 不要使用 rebase（保持 merge commit 更安全，便于回退）
