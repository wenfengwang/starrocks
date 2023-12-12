---
displayed_sidebar: "Japanese"
---

# トランザクションの表示

## 説明

この構文は、指定されたトランザクションIDのトランザクション詳細を表示するために使用されます。

構文:

```sql
SHOW TRANSACTION
[FROM <db_name>]
WHERE id = transaction_id
```

返される結果の例:

```plain text
TransactionId: 4005
Label: insert_8d807d5d-bcdd-46eb-be6d-3fa87aa4952d
Coordinator: FE: 10.74.167.16
TransactionStatus: VISIBLE
LoadJobSourceType: INSERT_STREAMING
PrepareTime: 2020-01-09 14:59:07
CommitTime: 2020-01-09 14:59:09
FinishTime: 2020-01-09 14:59:09
Reason:
ErrorReplicasCount: 0
ListenerId: -1
TimeoutMs: 300000
```

* TransactionId: トランザクションID
* Label: タスクに対応するラベルをインポート
* Coordinator: トランザクション調整を担当するノード
* TransactionStatus: トランザクションのステータス
* PREPARE: 準備段階
* COMMITTED: トランザクションは成功したが、データは表示されていない
* VISIBLE: トランザクションは成功し、データが表示されている
* ABORTED: トランザクションが失敗した
* LoadJobSourceType: インポートタスクのタイプ
* PrepareTime: トランザクション開始時刻
* CommitTime: トランザクションが正常にコミットされた時刻
* FinishTime: データが表示された時刻
* Reason: エラーメッセージ
* ErrorReplicasCount: エラーのあるレプリカの数
* ListenerId: 関連するインポートジョブのID
* TimeoutMs: トランザクションのタイムアウト（ミリ秒単位）

## 例

1. IDが4005のトランザクションを表示する場合:

    ```sql
    SHOW TRANSACTION WHERE ID=4005;
    ```

2. 指定されたデータベースで、IDが4005のトランザクションを表示する場合:

    ```sql
    SHOW TRANSACTION FROM db WHERE ID=4005;
    ```