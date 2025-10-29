# Git 应用

> 本文档用于回答大作业中“Git 应用”部分的问题。所有图片已经放在本文件夹下的 `images/` 目录。

---

## 1. 已修改部分文件并将其中一部分加入暂存区，如何回退这些修改，恢复到修改前最后一次提交的状态？（至少两种方式）

### 方法一：使用 `git restore`

- 对于**工作区**未暂存的修改：
  ```bash
  git restore <file>
  ```
- 对于**暂存区**的修改：
  ```bash
  git restore --staged <file>
  ```
- 若要同时恢复工作区和暂存区：
  ```bash
  git restore --staged <file>
  git restore <file>
  ```

![git-restore 示例](images/git-restore.png)

> （此处插入你操作 `git restore` 前后的截图）

### 方法二：使用 `git checkout`（旧版命令，兼容性好）

- 恢复工作区文件：
  ```bash
  git checkout -- <file>
  ```
- 恢复暂存区文件：
  ```bash
  git reset HEAD <file>
  git checkout -- <file>
  ```

![git-checkout 示例](images/git-checkout.png)

> （此处插入你操作 `git checkout` 前后的截图）

---

## 2. 已经提交了一个新版本，需要回退该版本，如何操作？（分别给出不修改历史或修改历史的至少两种方式）

### 不修改历史（保留历史记录）

> 这类方法会在历史上新增一个“反向操作”的提交，原来错误的提交仍然可见，但仓库状态被恢复到目标版本。适用于已推送到远程或多人协作的场景。

#### 方法一（推荐）：使用 `git revert` 来生成一个反向提交

步骤演示：

1. 查看提交历史，找到要回退的提交 ID（用短 id 即可）：
```bash
git log --oneline -n 6
# 示例输出（假设要回退的是 abc1234）
# abc1234  修错：引入了有问题的变更
# def5678  新功能
# ...
```
2. 使用 `git revert` 撤销该提交：
```bash
git revert abc1234
```
这会启动一次新的提交，提交信息默认为“Revert "<原提交信息>"”。如果是合并提交，需要用 `-m` 指定主分支（慎用）：
```bash
git revert -m 1 <merge_commit_id>
```
3. 查看历史确认（会看到新产生的一条 revert 提交）：
```bash
git log --oneline -n 5
# ghi8901  Revert "修错：引入了有问题的变更"
# abc1234  修错：引入了有问题的变更
# def5678  新功能
```
4. 推送到远程（普通推送即可）：
```bash
git push origin HEAD
```

注意：`git revert` 是最安全的回退方式，尤其当错误提交已经被其他人拉取时。

#### 方法二：用旧提交的文件状态恢复并提交（手动恢复）

当你只想恢复某些文件到旧版本并作为新提交保留历史时，可按下面操作：

1. 找到目标提交 ID（例如 abc1234），然后把当时的文件取回到工作区：
```bash
git checkout abc1234 -- path/to/file1 path/to/dir/
```
这会把这些文件恢复为指定提交时的内容（写入工作区，但不修改 HEAD）。
2. 检查变更并提交：
```bash
git status
git add path/to/file1
git commit -m "Restore file1 to state from abc1234"
```
3. 推送到远程：
```bash
git push origin HEAD
```

注意：这种方法适合只需恢复部分文件的情况，并且不会重写历史。


### 修改历史（重写历史）

> 这些方法会改变分支的提交历史（移除或修改提交），如果提交已经推送到了公共远程仓库，强制推送会影响其他协作者。使用前请先与团队沟通并备份分支。

#### 方法一：使用 `git reset` 并强制推送（快速但危险）

场景：你在本地提交了错误的提交，并且确定要把历史回退到某一旧提交（比如回退到 def5678），且你可以强制覆盖远程分支。

步骤：
1. 查看历史并确认目标提交：
```bash
git log --oneline -n 6
# def5678  正确的基线
# abc1234  错误的提交
# ghi8901  另外一个提交
```
2. 将当前分支重置到目标提交（彻底回退并把工作树同步）：
```bash
git reset --hard def5678
```
3. 强制推送到远程（推荐使用带保护的方式）：
```bash
git push --force-with-lease origin HEAD:branch-name
```
说明：`--force-with-lease` 比 `--force` 更安全，它会在远程分支自你上次拉取后被别人改动时拒绝推送。

恢复策略：如果误操作可以用 `git reflog` 找回被丢弃的提交：
```bash
git reflog
git checkout <lost_commit_id>
```

#### 方法二：使用交互式变基（`git rebase -i`）编辑/删除特定提交，然后强制推送

适用场景：你想清理多次提交（比如合并多个修复，或删除某个中间提交），并把分支线性化。

步骤：
1. 找到要变基的上游点（例如想修改最近 4 次提交）：
```bash
git log --oneline -n 8
git rebase -i HEAD~4
```
2. 在打开的编辑器中：
 - 把 `pick` 改为 `drop` （或删除该行）以删除某次提交；
 - 或者把 `pick` 改为 `edit` 来修改某次提交内容；
 - 或者用 `squash` 将多个提交合并。
3. 保存退出，按照提示完成变基流程（解决冲突并使用 `git rebase --continue`）。
4. 变基完成后，强制推送到远程（使用 `--force-with-lease`）：
```bash
git push --force-with-lease origin HEAD:branch-name
```

注意与建议：
- 在对公共分支（如 `master`、`main`、`release`）执行重写历史操作前，先创建备份分支：
  ```bash
  git branch backup-before-reset
  git push origin backup-before-reset
  ```
- 如果不确定，优先使用 `git revert`。
- 强制推送会使其他开发者的本地分支出现“与远程脱节”的情况，他们需要用 `git fetch` + `git reset --hard` 或重新克隆来同步。

### 示例场景汇总（简短参考）

- 错误已经推送到远程，且多人协作：使用 `git revert`（非破坏性）。
- 错误仅在你本地，且你确认要彻底删掉：`git reset --hard <good>` + `git push --force-with-lease`。
- 需要清理多个提交或合并提交：`git rebase -i`，然后强制推送（仅当你能协调好协作影响时）。

![git-revert 示例](images/git-revert.png)

> （此处插入你操作 `git revert` / `git reset` / `git rebase` 的截图）

---

## 3. 合并分支的方式（至少两种）

### 方法一：使用 `git merge`

- 合并分支：
  ```bash
  git merge <branch>
  ```

![git-merge 示例](images/git-merge.png)

> （此处插入你操作 `git merge` 的截图）

### 方法二：使用 `git rebase`

- 变基合并分支：
  ```bash
  git rebase <branch>
  ```

![git-rebase 示例](images/git-rebase.png)

> （此处插入你操作 `git rebase` 的截图）

### 方法三：使用 `git cherry-pick`

- 选择性合并某个提交：
  ```bash
  git cherry-pick <commit_id>
  ```

![git-cherry-pick 示例](images/git-cherry-pick.png)

> （此处插入你操作 `git cherry-pick` 的截图）

---

> **注意：** 请将所有图片放在 `2025-finalproject/images/` 目录下，并在上述预留位置插入相对路径引用的图片。
