---
"type:": permanent-note
"title:": commit过的文件重新加入gitignore
"id:": 20250607165019
"created:": 2025-06-07T16:50:19
tags:
  - permanent-note
  - git
related-context: 
related-notes:
---
我们有的时候会遇到以下这种常见，已经将某个文件 commit 了，但我们发现又不需要 track 这个文件，将其加入 `.ignore` 文件 `commit` 发现并不会生效，这是因为 git 已经 track 了该文件了。

**第一步**：加入到 `.ignore` 文件中

**第二步**：停止 track 该文件

```shell
git rm --cached filename
```

只会删除 git 索引而不会删除文件。

**第三步**：commit
**第四步**：push

如果需要删除某个文件夹的索引，则可以使用：
```shell
git rm -r --cached dir
```