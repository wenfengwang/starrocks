---
displayed_sidebar: Chinese
---

# デコミッションのキャンセル

## 機能

このステートメントは、ノードのデコミッションをキャンセルするために使用されます。

> **注意**
>
> `cluster_admin` ロールを持つユーザーのみがクラスタ管理関連の操作を実行できます。

## 文法

```sql
CANCEL DECOMMISSION BACKEND "host:heartbeat_port"[,"host:heartbeat_port"...]
```

## 例

二つのノードのデコミッションをキャンセルする:

```sql
CANCEL DECOMMISSION BACKEND "host1:port", "host2:port";
```
