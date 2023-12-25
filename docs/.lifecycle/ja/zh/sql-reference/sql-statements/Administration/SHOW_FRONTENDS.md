---
displayed_sidebar: Chinese
---

# SHOW FRONTENDS

## 機能

このコマンドはFEノードを表示するために使用されます。FEノードの追加方法と高可用性デプロイについては、[クラスタデプロイ](../../../administration/Deployment.md#部署-fe-高可用集群)のセクションを参照してください。

> **注意**
>
> この操作にはSYSTEMレベルのOPERATE権限またはcluster_adminロールが必要です。

## 文法

```sql
SHOW FRONTENDS
```

コマンドの戻り値の説明：

1. name はbdbje内のそのFEノードの名前を示します。
2. Join がtrueの場合、そのノードが以前にクラスタに参加したことを示しますが、現在もクラスタ内にいることを意味するわけではありません（接続が失われている可能性があります）。
3. Alive はノードが生きているかどうかを示します。
4. ReplayedJournalId はそのノードが現在再生した最大のメタデータログIDを示します。
5. LastHeartbeat は最後のハートビートです。
6. IsHelper はそのノードがbdbjeのヘルパーノードであるかどうかを示します。
7. ErrMsg はハートビートに失敗した時のエラーメッセージを表示します。
