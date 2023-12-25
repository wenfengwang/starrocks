---
displayed_sidebar: Chinese
---

# コンピュートノードの表示

## 機能

このステートメントは、クラスタ内の CN ノードを表示するために使用されます。

> **注意**
>
> この操作には SYSTEM レベルの OPERATE 権限または `cluster_admin` ロールが必要です。

## 文法

```sql
SHOW COMPUTE NODES
```

コマンドの戻り値の説明：

1. **LastStartTime**：最近の CN ノードの起動時間を示します。
2. **LastHeartbeat**：最近の CN ノードのハートビートを示します。
3. **Alive**：CN ノードが生存しているかどうかを示します。
4. **SystemDecommissioned** が true の場合、CN ノードが安全にシャットダウン中であることを示します。
5. **ErrMsg**：ハートビートに失敗した際のエラーメッセージを表示します。
6. **Status**：CN ノードの状態情報を JSON 形式で表示します。現在は、CN ノードが最後にその状態情報を報告した時間情報を含んでいます。
