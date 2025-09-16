![[Study SOP]]


# Tasks

## Due Today Tasks
```tasks
not done
due today
sort by description
```
## Expired Tasks
```tasks
not done
due before today
sort by due
```
## This Week
```tasks
not done
due after today
due before in 1 week
sort by due
```

## None
```tasks
not done
no due date
sort by path
```
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

# Unprocessed Source Notes

```dataview
TABLE tags AS "Tags", created AS "创建时间"
FROM ""
WHERE type = "source-note"
  AND processed = false
SORT created ASC
```