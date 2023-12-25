---
displayed_sidebar: Chinese
---

# user_privileges

`user_privileges` はユーザー権限に関する情報を提供します。

`user_privileges` は以下のフィールドを提供します：

| フィールド     | 説明                                                         |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されたユーザーの名前です。                         |
| TABLE_CATALOG  | カタログの名前です。この値は常に def です。                  |
| PRIVILEGE_TYPE | 付与された権限です。この値はグローバルレベルで付与可能な任意の権限になり得ます。 |
| IS_GRANTABLE   | ユーザーが GRANT OPTION 権限を持っている場合は YES、そうでなければ NO です。GRANT OPTION は PRIVILEGE_TYPE='GRANT OPTION' として別行には表示されません。 |
