---
"type:": fleet-note
"title:": 20250520104631-Sync upstream with origin
"id:": 20250520104646
"created:": 2025-05-20T10:46:46
url: 
tags:
  - fleet-note
  - git
"processed:": false
"archived:": false
---
# Upstream 有分支，Origin 没有

首先，简要概括一下整个流程：你需要在本地为上游仓库（即原始仓库）添加一个名为 `upstream` 的远程，随后从该远程拉取所有分支，包括新建的 `dev` 分支，然后在本地检出该分支并将其推送到你自己的 `origin`（Fork） 仓库。下面分步说明。

## 1. 配置上游（upstream）远程

1. 切换到本地项目根目录，查看已有的远程：

   ```bash
   git remote -v
   ```
2. 如果列表中没有 `upstream`，则添加指向原始仓库的远程：

   ```bash
   git remote add upstream https://github.com/ORIGINAL-OWNER/REPO.git
   ```

   这样就把原始仓库标记为 `upstream` 远程([Stack Overflow][1])。

## 2. 从上游拉取并检出 `dev` 分支

1. 拉取上游仓库的所有更新，包括新创建的分支：

   ```bash
   git fetch upstream
   ```

   这会将 `upstream` 上的分支更新到本地的 `upstream/*` 命名空间([Coded Ocean][2])。
2. 查看所有本地和远程分支，确认已获取到 `upstream/dev`：

   ```bash
   git branch -a
   ```
3. 在本地新建并切换到与上游 `dev` 分支对应的分支：

   ```bash
   git checkout -b dev upstream/dev
   ```

   这里 `-b dev` 表示创建一个名为 `dev` 的本地分支，并从 `upstream/dev` 检出([Coded Ocean][2])。

## 3. 推送到你的 Fork （origin）

1. 将新建的 `dev` 分支推送到你自己的远程 `origin` 并建立追踪关系：

   ```bash
   git push -u origin dev
   ```
2. 这样，你的 GitHub Fork 仓库就会出现 `dev` 分支，且本地 `dev` 分支会默认与 `origin/dev` 同步([Coded Ocean][2])。

**-u / --set-upstream**：在第一次推送时，设置本地分支 `dev` 的上游（upstream）分支为 `origin/dev`，后续直接运行 `git push` 或 `git pull` 时就不需要再显式指定远程和分支名称了

---

完成以上操作后，你本地和你的 Fork 仓库都会包含最新的 `dev` 分支。如果上游后续还有其他新分支，只需重复第二步（`git fetch upstream` + `git checkout -b <branch> upstream/<branch>`）即可。

[1]: https://stackoverflow.com/questions/50220114/how-do-i-sync-all-new-upstream-branches-with-my-fork?utm_source=chatgpt.com "git - How do I sync all new upstream branches with my fork?"
[2]: https://www.clarkrichards.org/2021/11/28/forking-and-syncing-branches-with-git-and-github/?utm_source=chatgpt.com "Forking and syncing branches with git and Github - Clark Richards"


# Upstream 分支进行了更新，需要同步到 Origin

```shell
# 从上游仓库拉取最新更新，该命令会把上游仓库所有分支的新提交下载到本地，但不会修改当前工作分支
git fetch upstream

# 切换到本地 dev 分支
git checkout dev

# 将上游 dev 的更新合并到本地
git merge upstream/dev
git rebase upstream/dev

# 推送
git push origin dev

# 关联推送
git push -u origin dev
# 以后就可以使用
git push

# 强制推送
git push --force-with-lease origin dev
```

# Reference