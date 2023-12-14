---
displayed_sidebar: "Chinese"
---

# 恢复例行加载

## 功能

恢复已暂停的例行加载任务，可以通过[PASUE](../data-manipulation/PAUSE_ROUTINE_LOAD.md)命令来暂停加载任务，并修改例行加载任务属性，详细操作请参考[ALTER ROUTINE LOAD](./ALTER_ROUTINE_LOAD.md)。

## 示例

1. 恢复名称为 test1 的例行加载作业。

```sql
    RESUME ROUTINE LOAD FOR test1;
```