---
displayed_sidebar: "Japanese"
---

# ノードの運用停止を取り消す

## 説明

この文は、ノードの運用停止を取り消すために使用されます。 (管理者のみ!)

構文:

```sql
CANCEL DECOMMISSION BACKEND "<host>:<heartbeat_port>"[,"<host>:<heartbeat_port>"...]
```

## 例

1. 2つのノードの運用停止を取り消す。

    ```sql
    CANCEL DECOMMISSION BACKEND "ホスト1:ポート", "ホスト2:ポート";
    ```