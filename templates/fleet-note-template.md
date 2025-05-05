---
type:: fleet-note           # 笔记类型，区分闪念/文献/永久等类别
title:: <% tp.file.title %>
id:: {{date:YYYYMMDDHHmmss}}  # 唯一 ID，基于创建时间确保全局唯一
created:: {{date:YYYY-MM-DDTHH:mm:ss}}  # 创建时间（ISO 格式）
source:
- web
- AI
- wechat
- book
- video
url:
tags: [fleet-note]         # 便于快速筛选闪念笔记
processed:: false         # 是否已处理（由闪念/文献转永久）
archived:: false          # 是否已归档（处理完成后置为 true）
---
