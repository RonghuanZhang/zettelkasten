# Unprocessed Fleet Notes

```dataview
TABLE tags AS "Tags", created AS "创建时间"
FROM ""
WHERE type = "fleet-note"
  AND processed = false
SORT created ASC
```
# Unprocessed Literature Notes

```dataview
TABLE tags AS "Tags", created AS "创建时间"
FROM ""
WHERE type = "literature-note"
  AND processed = false
SORT created ASC
```

