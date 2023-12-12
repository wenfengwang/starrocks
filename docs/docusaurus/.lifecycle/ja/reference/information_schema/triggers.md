---
displayed_sidebar: "Japanese"
---

# トリガー

`triggers` はトリガーに関する情報を提供します。

`triggers` には次のフィールドが提供されます:

| **フィールド**        | **説明**                                                     |
| --------------------- | ------------------------------------------------------------ |
| TRIGGER_CATALOG       | トリガーが属するカタログの名前。この値は常に `def` です。    |
| TRIGGER_SCHEMA        | トリガーが属するデータベースの名前。                         |
| TRIGGER_NAME          | トリガーの名前。                                             |
| EVENT_MANIPULATION    | トリガーイベント。これはトリガーがアクティブになる関連テーブル上の操作のタイプです。値は `INSERT`（行が挿入されました）、`DELETE`（行が削除されました）、または `UPDATE`（行が変更されました）。 |
| EVENT_OBJECT_CATALOG  | すべてのトリガーは正確に1つのテーブルに関連付けられています。このテーブルが存在するカタログ。 |
| EVENT_OBJECT_SCHEMA   | すべてのトリガーは正確に1つのテーブルに関連付けられています。このテーブルが存在するデータベース。 |
| EVENT_OBJECT_TABLE    | トリガーが関連付けられているテーブルの名前。               |
| ACTION_ORDER          | 同じ `EVENT_MANIPULATION` および `ACTION_TIMING` の値を持つ同じテーブル上のトリガーのリスト内でのトリガーのアクションの序数位置。 |
| ACTION_CONDITION      | この値は常に `NULL` です。                                   |
| ACTION_STATEMENT      | トリガーボディー、つまりトリガーがアクティブになったときに実行されるステートメント。このテキストは UTF-8 エンコーディングを使用しています。 |
| ACTION_ORIENTATION    | この値は常に `ROW` です。                                   |
| ACTION_TIMING         | トリガーがトリガーイベントの前または後にアクティブになるかどうか。値は `BEFORE` または `AFTER` です。 |
| ACTION_REFERENCE_OLD_TABLE | この値は常に `NULL` です。                               |
| ACTION_REFERENCE_NEW_TABLE | この値は常に `NULL` です。                               |
| ACTION_REFERENCE_OLD_ROW   | 古い列識別子。値は常に `OLD` です。                       |
| ACTION_REFERENCE_NEW_ROW   | 新しい列識別子。値は常に `NEW` です。                       |
| CREATED               | トリガーが作成された日時。これはトリガーの `DATETIME(2)` 値（秒の百分の2の精度を持つ）です。 |
| SQL_MODE              | トリガーが作成されたときに有効な SQL モード、およびトリガーが実行される条件。 |
| DEFINER               | `DEFINER` 句で指定されたユーザー（しばしばトリガーを作成したユーザー）。 |
| CHARACTER_SET_CLIENT  |                                                              |
| COLLATION_CONNECTION  |                                                              |
| DATABASE_COLLATION    | トリガーが関連付けられているデータベースの照合順序。       |