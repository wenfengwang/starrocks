---
displayed_sidebar: Chinese
---

# schema_privileges

`schema_privileges` はデータベース権限に関する情報を提供します。

`schema_privileges` は以下のフィールドを提供します：

| フィールド       | 説明                                                         |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限を付与されたユーザーの名前です。                         |
| TABLE_CATALOG  | スキーマが属するカタログの名前です。この値は常に def です。  |
| TABLE_SCHEMA   | スキーマの名前です。                                         |
| PRIVILEGE_TYPE | 付与された権限です。各行に一つの権限がリストされるため、権限を付与されたユーザーが持つスキーマ権限ごとに一行があります。 |
| IS_GRANTABLE   | ユーザーが GRANT OPTION 権限を持っている場合は YES、そうでない場合は NO です。GRANT OPTION は個別の行としては出力されず、PRIVILEGE_TYPE='GRANT OPTION' を使用します。 |
