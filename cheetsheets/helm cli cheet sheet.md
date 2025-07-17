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

存放 Charts 的 **仓库**，常见的公开仓库有：
* Artifact Hub
* Bitnami

### Upgrade / Rollback

- **Upgrade**：升级 Release 到新版本 Chart 或修改 values
- **Rollback**：回滚 Release 到之前的版本

### Dependency (依赖)

- Chart 可以依赖其他 Charts（子 Chart），通过 `Chart.yaml` 的 `dependencies` 字段声明
- Helm 会自动下载依赖 Charts 到 `charts/` 目录

### Hook

- 特殊的 template，定义在资源 annotations 中，允许在 Release 生命周期的不同阶段执行（如 pre-install, post-install）
- 用于初始化任务、DB migration 等

# Commands

### 🏷️ **1. 仓库操作**

| 命令                           | 作用                      |
| ---------------------------- | ----------------------- |
| `helm repo add <name> <url>` | 添加 Chart 仓库             |
| `helm repo update`           | 更新仓库索引                  |
| `helm repo list`             | 查看已添加的仓库                |
| `helm search repo <keyword>` | 在已添加仓库中搜索 Chart         |
| `helm search hub <keyword>`  | 在 Artifact Hub 搜索 Chart |

---

### 📦 **2. 安装/卸载**

| 命令                                                    | 作用                   |
| ----------------------------------------------------- | -------------------- |
| `helm install <release-name> <chart> [flags]`         | 安装 Chart 为一个 Release |
| `helm uninstall <release-name>`                       | 卸载 Release           |
| `helm install <release-name> <chart> -f values.yaml`  | 使用指定配置文件安装           |
| `helm install <release-name> <chart> --set key=value` | 使用命令行覆盖配置安装          |

---

### 🔄 **3. 升级/回滚**

| 命令                                              | 作用                |
| ----------------------------------------------- | ----------------- |
| `helm upgrade <release-name> <chart> [flags]`   | 升级 Release        |
| `helm upgrade --install <release-name> <chart>` | 如果不存在则安装，存在则升级    |
| `helm rollback <release-name> [revision]`       | 回滚到指定版本（默认为上一个版本） |

---

### 🔍 **4. 查询/查看**

| 命令                            | 作用                |
| ----------------------------- | ----------------- |
| `helm list`                   | 查看所有 Release      |
| `helm list -A`                | 查看所有命名空间的 Release |
| `helm status <release-name>`  | 查看 Release 状态     |
| `helm history <release-name>` | 查看 Release 历史版本   |

---

### 📝 **5. 模板渲染/调试**

| 命令                                                      | 作用                                   |
| ------------------------------------------------------- | ------------------------------------ |
| `helm template <chart>`                                 | 将 Chart 渲染为 Kubernetes YAML 输出（不会安装） |
| `helm install --dry-run --debug <release-name> <chart>` | 预览安装，检查模板渲染结果                        |
| `helm upgrade --dry-run --debug <release-name> <chart>` | 预览升级，检查差异                            |

---

### 🔧 **6. 创建/打包 Chart**

| 命令                         | 作用                 |
| -------------------------- | ------------------ |
| `helm create <chart-name>` | 创建一个新的 Chart 模板    |
| `helm package <chart-dir>` | 将 Chart 打包成 tgz 文件 |
| `helm lint <chart>`        | 检查 Chart 的语法与规范问题  |

---

### 🚀 **7. 依赖管理**

| 命令                       | 作用                   |
| ------------------------ | -------------------- |
| `helm dependency update` | 更新 Chart 的依赖         |
| `helm dependency build`  | 根据 `Chart.yaml` 构建依赖 |
| `helm dependency list`   | 查看 Chart 的依赖列表       |

---

### 🔑 **8. 插件管理**

| 命令                             | 作用         |
| ------------------------------ | ---------- |
| `helm plugin list`             | 查看已安装插件    |
| `helm plugin install <url>`    | 安装 Helm 插件 |
| `helm plugin uninstall <name>` | 卸载 Helm 插件 |

