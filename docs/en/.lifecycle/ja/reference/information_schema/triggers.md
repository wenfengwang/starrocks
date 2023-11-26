---
displayed_sidebar: "Japanese"
---

# トリガー

`triggers`はトリガーに関する情報を提供します。

`triggers`には以下のフィールドが提供されます：

| **フィールド**                  | **説明**                                              |
| -------------------------- | ------------------------------------------------------------ |
| TRIGGER_CATALOG            | トリガーが所属するカタログの名前です。この値は常に `def` です。 |
| TRIGGER_SCHEMA             | トリガーが所属するデータベースの名前です。       |
| TRIGGER_NAME               | トリガーの名前です。                                     |
| EVENT_MANIPULATION         | トリガーのイベントです。これはトリガーがアクティブになる関連テーブルの操作の種類です。値は `INSERT`（行が挿入された）、`DELETE`（行が削除された）、または `UPDATE`（行が変更された）です。 |
| EVENT_OBJECT_CATALOG       | すべてのトリガーは必ず1つのテーブルに関連付けられています。このテーブルが存在するカタログです。 |
| EVENT_OBJECT_SCHEMA        | すべてのトリガーは必ず1つのテーブルに関連付けられています。このテーブルが存在するデータベースです。 |
| EVENT_OBJECT_TABLE         | トリガーが関連付けられているテーブルの名前です。   |
| ACTION_ORDER               | トリガーのアクションが、同じ`EVENT_MANIPULATION`および`ACTION_TIMING`の値を持つ同じテーブルのトリガーのリスト内での順序位置です。 |
| ACTION_CONDITION           | この値は常に `NULL` です。                                 |
| ACTION_STATEMENT           | トリガーがアクティブになったときに実行されるステートメント、つまりトリガーの本体です。このテキストはUTF-8エンコーディングを使用しています。 |
| ACTION_ORIENTATION         | この値は常に `ROW` です。                                  |
| ACTION_TIMING              | トリガーがトリガーイベントの前または後にアクティブになるかどうかを示します。値は `BEFORE` または `AFTER` です。 |
| ACTION_REFERENCE_OLD_TABLE | この値は常に `NULL` です。                                 |
| ACTION_REFERENCE_NEW_TABLE | この値は常に `NULL` です。                                 |
| ACTION_REFERENCE_OLD_ROW   | 古い列の識別子です。値は常に `OLD` です。       |
| ACTION_REFERENCE_NEW_ROW   | 新しい列の識別子です。値は常に `NEW` です。       |
| CREATED                    | トリガーが作成された日時です。トリガーの場合、これは `DATETIME(2)` 値（秒の百分の一の小数部を持つ）です。 |
| SQL_MODE                   | トリガーが作成されたときに有効なSQLモード、およびトリガーが実行されるSQLモードです。 |
| DEFINER                    | `DEFINER`句で指定されたユーザー（通常はトリガーを作成したユーザー）です。 |
| CHARACTER_SET_CLIENT       |                                                              |
| COLLATION_CONNECTION       |                                                              |
| DATABASE_COLLATION         | トリガーが関連付けられているデータベースの照合順序です。 |
