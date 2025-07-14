---
"type:": literature-note
"title:": Hash Pointers and Data Structures
"id:": 20250623204609
"created:": 2025-06-23T20:46:09
source:
  - coursera
url: https://www.coursera.org/learn/cryptocurrency/lecture/EYEAo/hash-pointers-and-data-structures
tags:
  - literature-note
"processed:": false
related-notes: 
"archived:": false
---
## What is hash pointer?

Hash pointer is:
* Pointer to where some info is stored, and hash of the info
If we have a hash pointer, we can ask to get the info back and verify that hasn't changed.

![image.png](https://images.hnzhrh.com/note/20250623210732471.png)


![image.png](https://images.hnzhrh.com/note/20250623211029767.png)


![image.png](https://images.hnzhrh.com/note/20250623211302451.png)

## Mine

What is a hash pointer?  The pointer points to the preblock and records the hash. So if I want to tamper with the data, the chain will detect the error.