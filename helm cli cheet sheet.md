---
"type:": cheet-sheet
"title:": helm cli cheet sheet
"id:": 20250717104942
"created:": 2025-07-17T10:49:42
url: 
tags:
  - cheet-sheet
  - helm
---
## 核心概念

### Chart

定义一个 Kubernetes 应用的包，本质是一个文件夹，包含了：
* `Chart.yaml`，元数据，定义版本、依赖、图标等等
* `values.yaml`，默认配置值
* `templates/`，渲染模板文件，通常是 Kubernetes 资源定义的 Go template 文件
* `charts/`，依赖的子 Chart
* `README`
### Values

用户通过 `values.yaml` 默认**配置值**，用于渲染 templates，替换变量，可通过 `helm install -f my-values.yaml` 使用自定义配置。

### Templates

使用 **Go Template 语法** 编写的 YAML 文件，通过 values 渲染为 Kubernetes 资源对象。

### Release

Helm 安装 Chart 后的实例，称为 **Release**，一个 Chart 可以安装多次，每次安装生成一个独立的 Release，且可配置不同的 values，`helm install <release-name> <chart>`

### Repository (Repo)

存放 Charts 的 **仓库**，常见的仓库有：


## 添加 repo

```shell
helm repo add <repo-name> <repo-url>
```

## 更新 repo

```shell
helm repo update
```

## 拉取