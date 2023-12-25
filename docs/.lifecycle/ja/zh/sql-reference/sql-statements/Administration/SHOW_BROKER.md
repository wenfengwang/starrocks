---
displayed_sidebar: Chinese
---

# SHOW BROKER

## 機能

このステートメントは、現在存在するブローカーを表示するために使用されます。

> **注意**
>
> この操作には SYSTEM レベルの OPERATE 権限または cluster_admin ロールが必要です。

## 文法

```sql
SHOW BROKER
```

コマンドの戻り値についての説明：

1. LastStartTime は最後に BE が起動した時間を示します。
2. LastHeartbeat は最後のハートビートを示します。
3. Alive はノードが生きているかどうかを示します。
4. ErrMsg はハートビートが失敗した時のエラーメッセージを表示します。
