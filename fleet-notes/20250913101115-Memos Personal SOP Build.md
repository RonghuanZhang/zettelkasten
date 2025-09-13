---
"type:": fleet-note
"title:": 20250913101115-Memos Personal SOP Build
id:: 20250913101136  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-09-13T10:11:36  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
# Deploy by Docker

```yaml
version: '3.8'

services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    ports:
      - "5230:5230"
    volumes:
      - ./memos-backup:/var/opt/memos
    environment:
      - MEMOS_MODE=prod
      - MEMOS_PORT=5230
    restart: unless-stopped
```


# Reference
* [Memos - Open Source, Self-hosted Note Taking - Memos](https://www.usememos.com/)