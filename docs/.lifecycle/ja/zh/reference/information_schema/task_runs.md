---
displayed_sidebar: Chinese
---

# task_runs

`task_runs` は非同期タスクの実行に関する情報を提供します。

`task_runs` は以下のフィールドを提供します：

| フィールド    | 説明                                                         |
| ------------- | ------------------------------------------------------------ |
| QUERY_ID      | クエリの ID。                                                |
| TASK_NAME     | タスクの名前。                                               |
| CREATE_TIME   | タスクが作成された時間。                                     |
| FINISH_TIME   | タスクが完了した時間。                                       |
| STATE         | タスクの状態。`PENDING`（保留中）、`RUNNING`（実行中）、`FAILED`（失敗）、`SUCCESS`（成功）を含む。 |
| DATABASE      | タスクが属するデータベース。                                 |
| DEFINITION    | タスクの SQL 定義。                                          |
| EXPIRE_TIME   | タスクの有効期限が切れる時間。                               |
| ERROR_CODE    | タスクのエラーコード。                                       |
| ERROR_MESSAGE | タスクのエラーメッセージ。                                   |
| PROGRESS      | タスクの進行状況。                                           |
| EXTRA_MESSAGE | タスクに関する追加メッセージ。例えば、非同期マテリアライズドビュー作成タスクでのパーティション情報など。 |
