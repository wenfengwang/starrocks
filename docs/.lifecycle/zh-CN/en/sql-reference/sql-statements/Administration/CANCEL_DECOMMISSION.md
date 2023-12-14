---
displayed_sidebar: "Chinese"
---

# 取消撤销

## 描述

此语句用于撤销节点的撤销。 （仅限管理员！）

语法：

```sql
取消撤销后端 "<host>:<heartbeat_port>"[，"<host>:<heartbeat_port>"...]
```

## 例子

1. 取消两个节点的撤销。

    ```sql
    取消撤销后端 "host1:port"，"host2:port"；
    ```