---
displayed_sidebar: "英語"
---

# table_privileges

`table_privileges`には、テーブル特権に関する情報が提供されます。

以下のフィールドが`table_privileges`に提供されています。

| **フィールド**  | **説明**                                                      |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 特権が付与されるユーザーの名前です。                                  |
| TABLE_CATALOG  | テーブルが属するカタログの名前です。この値は常に`def`です。       |
| TABLE_SCHEMA   | テーブルが属するデータベースの名前です。                             |
| TABLE_NAME     | テーブルの名前です。                                           |
| PRIVILEGE_TYPE | 付与された特権です。テーブルレベルで付与できる任意の特権の値になります。       |
| IS_GRANTABLE   | ユーザーが`GRANT OPTION`特権を持っている場合は`YES`と表示され、そうでない場合は`NO`と表示されます。出力には`PRIVILEGE_TYPE='GRANT OPTION'`として別行で`GRANT OPTION`はリストされません。 |