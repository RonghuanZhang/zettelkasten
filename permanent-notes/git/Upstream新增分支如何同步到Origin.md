---
"type:": permanent-note
"title:": Upstream新增分支如何同步到Origin
"id:": 20250607172805
"created:": 2025-06-07T17:28:05
tags:
  - permanent-note
  - git
related-context: 
related-notes:
---

Step 1: 查看当前是否配置 Upstream

```shell
git remote -v
```

Step 2: 配置 Upstream 仓库

```shell
git remote add upstream https://github.com/UPSTREAM-OWNER/REPO.git
```

Step 3: 同步 Upstream 仓库信息并拉取 Upstream 新增分支

```shell
// 同步信息
git fetch upstream

// 查看分支
git branch -a

// Checkout 新分支
git checkout -b dev upstream/dev
```

Step 4: Push 分支到 Origin 仓库

```shell
git push -u origin dev
```

使用 `-u` 可以绑定关联，以后可以直接使用 `git push` 而不需要指定分支。