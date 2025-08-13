---
"type:": fleet-note
"title:": 20250808172404-DDD Engineering Practice
"id:": 20250808172452
"created:": 2025-08-08T17:24:52
url: 
tags:
  - fleet-note
  - ddd
"processed:": false
"archived:": false
---

## 怎么处理 CQRS？

给 Adapter 层最终展示使用：

* Client 层定义查询接口 `XXXQueryI`
* Application 层实现接口，注入 Mapper 或其他直接查询
* Application 层通过 `QueryExe` 调用 ` XXXQueryI ` 查询

Domain 层需要的查询：

* Domain 层定义 Gateway 接口
* Infrastructure 层实现Gateway 接口
* DomainService 调用Gateway 接口
参考： [[20250808173336-领域层的CQRS的Q]]


# Reference