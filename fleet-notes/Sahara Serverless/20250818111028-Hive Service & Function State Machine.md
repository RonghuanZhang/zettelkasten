---
"type:": fleet-note
"title:": 20250818111028-Hive Service & Function State Machine
"id:": 20250818111111
"created:": 2025-08-18T11:11:11
url:
tags:
  - fleet-note
  - project/hive
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