---
displayed_sidebar: English
---

# task_runs

`task_runs` は非同期タスクの実行に関する情報を提供します。

`task_runs` には、以下のフィールドが提供されています。

| **フィールド** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| QUERY_ID      | クエリのIDです。                                             |
| TASK_NAME     | タスクの名前です。                                            |
| CREATE_TIME   | タスクが作成された時刻です。                               |
| FINISH_TIME   | タスクが終了した時刻です。                                 |
| STATE         | タスクの状態です。有効な値は `PENDING`、`RUNNING`、`FAILED`、`SUCCESS` です。 |
| DATABASE      | タスクが属するデータベースです。                             |
| DEFINITION    | タスクのSQL定義です。                                  |
| EXPIRE_TIME   | タスクの有効期限が切れる時刻です。                                  |
| ERROR_CODE    | タスクのエラーコードです。                                      |
| ERROR_MESSAGE | タスクのエラーメッセージです。                                   |
| PROGRESS      | タスクの進行状況です。                                    |
| EXTRA_MESSAGE | タスクに関する追加メッセージです。例えば、非同期マテリアライズドビュー作成タスクのパーティション情報など。 |
