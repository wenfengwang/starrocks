---
displayed_sidebar: English
---

# トリガー

`triggers` はトリガーに関する情報を提供します。

`triggers` で提供されるフィールドは以下の通りです。

| **フィールド**                  | **説明**                                              |
| -------------------------- | ------------------------------------------------------------ |
| TRIGGER_CATALOG            | トリガーが属するカタログの名前。この値は常に `def` です。 |
| TRIGGER_SCHEMA             | トリガーが属するデータベースの名前。       |
| TRIGGER_NAME               | トリガーの名前。                                     |
| EVENT_MANIPULATION         | トリガーイベント。これは、トリガーがアクティブになる関連テーブルに対する操作のタイプです。値は `INSERT`（行が挿入された）、`DELETE`（行が削除された）、または `UPDATE`（行が変更された）です。 |
| EVENT_OBJECT_CATALOG       | すべてのトリガーは、ただ一つのテーブルに関連付けられます。このテーブルが存在するカタログ。 |
| EVENT_OBJECT_SCHEMA        | すべてのトリガーは、ただ一つのテーブルに関連付けられます。このテーブルが存在するデータベース。 |
| EVENT_OBJECT_TABLE         | トリガーが関連付けられているテーブルの名前。   |
| ACTION_ORDER               | `EVENT_MANIPULATION` と `ACTION_TIMING` の値が同じである同じテーブル上のトリガーのリスト内でのトリガーのアクションの順序。 |
| ACTION_CONDITION           | この値は常に `NULL` です。                                 |
| ACTION_STATEMENT           | トリガー本体。つまり、トリガーがアクティブになったときに実行されるステートメントです。このテキストはUTF-8エンコーディングを使用します。 |
| ACTION_ORIENTATION         | この値は常に `ROW` です。                                  |
| ACTION_TIMING              | トリガーがトリガーイベントの前または後にアクティブになるかどうか。値は `BEFORE` または `AFTER` です。 |
| ACTION_REFERENCE_OLD_TABLE | この値は常に `NULL` です。                                 |
| ACTION_REFERENCE_NEW_TABLE | この値は常に `NULL` です。                                 |
| ACTION_REFERENCE_OLD_ROW   | 古い行識別子。値は常に `OLD` です。       |
| ACTION_REFERENCE_NEW_ROW   | 新しい行識別子。値は常に `NEW` です。       |
| CREATED                    | トリガーが作成された日時。これは `DATETIME(2)` 値（秒の百分の一の小数部を含む）です。 |
| SQL_MODE                   | トリガーが作成されたときのSQLモードで、トリガーが実行される際にも適用されます。 |
| DEFINER                    | `DEFINER` 句で指定されたユーザー（多くの場合、トリガーを作成したユーザー）。 |
| CHARACTER_SET_CLIENT       |                                                              |
| COLLATION_CONNECTION       |                                                              |
| DATABASE_COLLATION         | トリガーが関連付けられているデータベースの照合順序。 |
