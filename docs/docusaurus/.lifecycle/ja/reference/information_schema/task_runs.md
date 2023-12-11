---
displayed_sidebar: "Japanese"
---

# task_runs（タスクの実行）

`task_runs`は非同期タスクの実行に関する情報を提供します。

`task_runs`には次のフィールドが提供されます：

| **フィールド**  | **説明**                                                                                   |
| ------------- | ------------------------------------------------------------ |
| QUERY_ID      | クエリのID。                                               |
| TASK_NAME     | タスクの名前。                                             |
| CREATE_TIME   | タスクが作成された時刻。                                     |
| FINISH_TIME   | タスクが終了した時刻。                                       |
| STATE         | タスクの状態。有効な値：`PENDING`、`RUNNING`、`FAILED`、および`SUCCESS`。      |
| DATABASE      | タスクが属するデータベース。                                   |
| DEFINITION    | タスクのSQL定義。                                           |
| EXPIRE_TIME   | タスクの有効期限時刻。                                       |
| ERROR_CODE    | タスクのエラーコード。                                       |
| ERROR_MESSAGE | タスクのエラーメッセージ。                                    |
| PROGRESS      | タスクの進捗状況。                                           |
| EXTRA_MESSAGE | タスクの追加メッセージ。例えば、非同期マテリアライズドビュー作成タスクのパーティション情報など。 |