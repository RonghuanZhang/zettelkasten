---
type:: literature-note         # 笔记类型，区分闪念/文献/永久等类别
title:: Lec 1
id:: 20250902211707  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-09-02T21:17:07  # 创建时间（ISO 格式）
source:
- coursera
url:
tags: [literature-note]         # 便于快速筛选文献笔记
processed:: false         # 是否已处理（由闪念/文献转永久）
related-notes: []         # 如果已处理，关联到对应的永久笔记
archived:: false          # 是否已归档（处理完成后置为 true）
---
# Introduction
* fault tolerance 
	* High Availability
	* Replicated Servers
* consistency
	* read(x) yields the value from the most recent write(x)
	* Scaling gets harder as N grows
		* Load imbalance
		* Slowest-of-N latency
* performance
* tradeoffs
	* Fault-tolerance, consistency, and performance are enemies.
