---
displayed_sidebar: Chinese
---

# トリガー

`triggers` はトリガーに関する情報を提供します。

`triggers` は以下のフィールドを提供します：

| フィールド                 | 説明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| TRIGGER_CATALOG            | トリガーが属するカタログの名前。この値は常に def です。     |
| TRIGGER_SCHEMA             | トリガーが属するデータベースの名前。                         |
| TRIGGER_NAME               | トリガーの名前。                                             |
| EVENT_MANIPULATION         | トリガーイベント。これはトリガーがアクティブになる関連テーブル上の操作の種類です。値は INSERT（行が挿入された）、DELETE（行が削除された）、または UPDATE（行が更新された）のいずれかです。 |
| EVENT_OBJECT_CATALOG       | 各トリガーは正確に一つのテーブルに関連付けられています。そのテーブルがあるカタログ。 |
| EVENT_OBJECT_SCHEMA        | 各トリガーは正確に一つのテーブルに関連付けられています。そのテーブルがあるデータベース。 |
| EVENT_OBJECT_TABLE         | トリガーが関連付けられているテーブルの名前。                 |
| ACTION_ORDER               | トリガーアクションは、同じ EVENT_MANIPULATION と ACTION_TIMING 値を持つ同一テーブル上のトリガーリスト内での順序位置です。 |
| ACTION_CONDITION           | この値は常に NULL です。                                      |
| ACTION_STATEMENT           | トリガー本体；つまり、トリガーがアクティブになった時に実行されるステートメント。このテキストは UTF-8 エンコーディングを使用します。 |
| ACTION_ORIENTATION         | この値は常に ROW です。                                       |
| ACTION_TIMING              | トリガーがイベントの前か後かでアクティブになるか。この値は BEFORE または AFTER です。 |
| ACTION_REFERENCE_OLD_TABLE | この値は常に NULL です。                                      |
| ACTION_REFERENCE_NEW_TABLE | この値は常に NULL です。                                      |
| ACTION_REFERENCE_OLD_ROW   | 古い行の識別子。この値は常に OLD です。                       |
| ACTION_REFERENCE_NEW_ROW   | 新しい行の識別子。この値は常に NEW です。                     |
| CREATED                    | トリガーが作成された日付と時間。トリガーについては、これは DATETIME(2) 値です（秒の小数部分は百分の一秒）。 |
| SQL_MODE                   | トリガーが作成された時の有効な SQL モードと、トリガーが実行される SQL モード。 |
| DEFINER                    | DEFINER 句で指定されたユーザー（通常はトリガーを作成したユーザー）。 |
| CHARACTER_SET_CLIENT       | クライアントの文字セット。                                   |
| COLLATION_CONNECTION       | 接続の照合順序。                                             |
| DATABASE_COLLATION         | トリガーに関連付けられたデータベースの照合順序。             |
