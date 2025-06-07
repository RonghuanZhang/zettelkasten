---
"type:": permanent-note
"title:": 同步Upstream的变更到Origin
"id:": 20250607174609
"created:": 2025-06-07T17:46:09
tags:
  - permanent-note
  - git
related-context: 
related-notes:
---
当 Upstream 仓库有了其他开发者的 commit，该如何同步到自己的仓库或者本地？

第一种：可以直接使用 Github 的同步按钮
第二种：使用命令

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