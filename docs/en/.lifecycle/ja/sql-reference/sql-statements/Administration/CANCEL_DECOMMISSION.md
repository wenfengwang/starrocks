---
displayed_sidebar: "Japanese"
---

# デコミッションのキャンセル

## 説明

この文は、ノードのデコミッションを取り消すために使用されます。（管理者のみ！）

構文：

```sql
CANCEL DECOMMISSION BACKEND "<host>:<heartbeat_port>"[,"<host>:<heartbeat_port>"...]
```

## 例

1. 2つのノードのデコミッションをキャンセルする。

    ```sql
    CANCEL DECOMMISSION BACKEND "host1:port", "host2:port";
    ```
