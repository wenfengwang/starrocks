---
displayed_sidebar: "Japanese"
---

# ブローカーの表示

## 説明

このステートメントは現在存在するブローカーを表示するために使用されます。

構文:

```sql
SHOW BROKER
```

注意:

1. LastStartTimeは最新のBE起動時刻を表します。
2. LastHeartbeatは最新のハートビートを表します。
3. Aliveはノードが生存しているかどうかを示します。
4. ErrMsgはハートビートが失敗した場合にエラーメッセージを表示するために使用されます。
