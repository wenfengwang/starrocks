---
displayed_sidebar: "英語"
---

# schema_privileges

`schema_privileges`は、データベース権限についての情報を提供します。

`schema_privileges`には以下のフィールドが含まれています：

| **フィールド**      | **説明**                                                     |
| ---------------- | ------------------------------------------------------------ |
| GRANTEE          | 権限が付与されるユーザーの名前です。                             |
| TABLE_CATALOG    | スキーマが属しているカタログの名前です。この値は常に`def`です。         |
| TABLE_SCHEMA     | スキーマの名前です。                                           |
| PRIVILEGE_TYPE   | 付与されている権限です。各行は一つの権限をリストしているため、付与者によって保持されるスキーマ権限ごとに一行があります。 |
| IS_GRANTABLE     | ユーザーが`GRANT OPTION`権限を持っている場合は`YES`、そうでない場合は`NO`です。出力には`GRANT OPTION`を`PRIVILEGE_TYPE='GRANT OPTION'`として別行でリストすることはありません。 |