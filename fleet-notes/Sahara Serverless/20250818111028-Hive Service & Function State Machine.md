---
"type:": fleet-note
"title:": 20250818111028-Hive Service & Function State Machine
id:: 20250818111111  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-08-18T11:11:11  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

## State Machine

```mermaid
---
title: Service & Function Business State
---
stateDiagram-v2
    [*] --> APPLIED: User apply the configuration.
    APPLIED --> DEPLOYING: User trigger the deployment.
    DEPLOYING --> READY: Deploy successfully.
    DEPLOYING --> NOT_READY: Deploy unsuccessfully.
    READY --> [*]
    NOT_READY --> [*]

```

# Reference