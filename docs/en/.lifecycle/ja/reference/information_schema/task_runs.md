---
displayed_sidebar: "Japanese"
---

# task_runs

`task_runs`は非同期タスクの実行に関する情報を提供します。

`task_runs`には以下のフィールドが提供されます：

| **フィールド**  | **説明**                                                     |
| -------------- | ------------------------------------------------------------ |
| QUERY_ID       | クエリのID。                                                 |
| TASK_NAME      | タスクの名前。                                                |
| CREATE_TIME    | タスクが作成された時刻。                                       |
| FINISH_TIME    | タスクが終了した時刻。                                         |
| STATE          | タスクの状態。有効な値は`PENDING`、`RUNNING`、`FAILED`、`SUCCESS`です。 |
| DATABASE       | タスクが所属するデータベース。                                 |
| DEFINITION     | タスクのSQL定義。                                             |
| EXPIRE_TIME    | タスクの有効期限。                                             |
| ERROR_CODE     | タスクのエラーコード。                                         |
| ERROR_MESSAGE  | タスクのエラーメッセージ。                                     |
| PROGRESS       | タスクの進捗状況。                                            |
| EXTRA_MESSAGE  | タスクの追加メッセージ。例えば、非同期マテリアライズドビュー作成タスクのパーティション情報など。 |
