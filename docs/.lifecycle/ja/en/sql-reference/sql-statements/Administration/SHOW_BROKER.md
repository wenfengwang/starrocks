---
displayed_sidebar: English
---

# SHOW BROKER

## 説明

このステートメントは、現在存在するブローカーを表示するために使用されます。

:::tip

この操作は、SYSTEMレベルのOPERATE権限を持つユーザー、または`cluster_admin`ロールを持つユーザーのみが実行できます。

:::

## 構文

```sql
SHOW BROKER
```

注記：

1. LastStartTimeは、最新のBE起動時刻を表します。
2. LastHeartbeatは、最新のハートビートを表します。
3. Aliveは、ノードが生存しているかどうかを示します。
4. ErrMsgは、ハートビートに失敗した際のエラーメッセージを表示するために使用されます。
