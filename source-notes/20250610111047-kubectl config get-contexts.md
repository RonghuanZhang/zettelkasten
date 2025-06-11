---
"type:": source-note
"title:": 20250610111047-kubectl config get-contexts
"id:": 20250610111048
"created:": 2025-06-10T11:10:48
source:
  - AI
url: 
tags:
  - source-note
  - golang
"processed:": false
"archived:": false
---
`go.sum` 文件是 Go Modules 机制的一部分，它的作用是 **确保依赖项的完整性和安全性**，用来 **校验模块下载后是否被篡改**。它配合 `go.mod` 文件一起使用，在项目构建、下载依赖时提供安全保障。

---

### 一、作用详解

1. **记录依赖模块的哈希值**

   * `go.sum` 记录的是依赖模块的版本和对应文件的哈希值（通常是 zip 包的 SHA-256 哈希）。
   * 格式如下：

     ```
     github.com/some/module v1.2.3 h1:abc123...
     github.com/some/module v1.2.3/go.mod h1:def456...
     ```

2. **防止依赖被篡改**

   * 当你执行 `go build`, `go test`, `go mod tidy` 等命令时，Go 会校验当前下载的模块 zip 和 `go.sum` 中记录的哈希值是否一致。
   * 如果不一致，就会报错，提示模块内容可能被篡改。

3. **提高缓存一致性**

   * 在多人协作或 CI 构建环境中，`go.sum` 确保大家使用的是完全相同的依赖内容。

---

### 二、为什么不能忽略？

* 虽然项目运行时只依赖 `go.mod` 确定版本，但缺少 `go.sum`：

  * 会导致首次拉取依赖时无法校验完整性。
  * Go 在没有 `go.sum` 时也能工作，但会提示缺失校验信息。

**官方推荐把 `go.sum` 加入版本控制（如 Git）中**，防止依赖变化或安全问题。

---

### 三、常见命令行为

* `go mod tidy`：会自动添加缺失的校验值到 `go.sum`，并删除不再使用的模块。
* `go clean -modcache`：清理模块缓存，不会删除 `go.sum`。

---

### 四、补充说明

* `go.mod` 指定**依赖的版本**
* `go.sum` 保证**依赖的内容安全**

你可以把 `go.mod` 看作依赖的“清单”，而 `go.sum` 是每个依赖“快照的指纹”。

---

需要我帮你演示一个具体例子，或者你项目中对 `go.sum` 的改动分析吗？
