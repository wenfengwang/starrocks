---
displayed_sidebar: Chinese
---

# table_privileges

`table_privileges` はテーブルの権限に関する情報を提供します。

`table_privileges` は以下のフィールドを提供します：

| フィールド       | 説明                                                         |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限を付与されたユーザーの名前です。                         |
| TABLE_CATALOG  | テーブルが属するカタログの名前です。この値は常に def です。 |
| TABLE_SCHEMA   | テーブルが属するデータベースの名前です。                     |
| TABLE_NAME     | テーブルの名前です。                                         |
| PRIVILEGE_TYPE | 付与された権限です。テーブルレベルで付与できる任意の権限がこの値になり得ます。 |
| IS_GRANTABLE   | ユーザーが GRANT OPTION 権限を持っている場合は YES、そうでなければ NO です。GRANT OPTION は個別の行としては出力されず、PRIVILEGE_TYPE='GRANT OPTION' を使用して表されます。 |

