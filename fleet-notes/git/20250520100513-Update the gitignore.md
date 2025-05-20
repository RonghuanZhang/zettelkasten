---
"type:": fleet-note
"title:": 20250520100513-Update the gitignore
"id:": 20250520100526
"created:": 2025-05-20T10:05:26
url: 
tags:
  - fleet-note
  - git
"processed:": false
"archived:": false
---

```shell
# Remove the cache
git rm -r --cached .

# Trace again.
git add . 

# Commit
git commit -m "update .gitignore"


git push origin master #可选，如果需要同步到remote上的话
```

# Reference