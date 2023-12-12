```yaml
displayed_sidebar: "Japanese"
```

# user_privileges

`user_privileges` はユーザー権限に関する情報を提供します。

`user_privileges` には次のフィールドが提供されています。

| **Field**      | **Description**                                              |
| -------------- | ------------------------------------------------------------ |
| GRANTEE        | 権限が付与されるユーザーの名前。                                |
| TABLE_CATALOG  | カタログの名前。この値は常に `def` です。                    |
| PRIVILEGE_TYPE | 付与される権限。この値はグローバルレベルで付与できる権限であることがあります。  |
| IS_GRANTABLE   | ユーザーが `GRANT OPTION` 権限を持っている場合は `YES`、それ以外の場合は `NO` です。出力は`PRIVILEGE_TYPE='GRANT OPTION'` として別の行に `GRANT OPTION` をリスト表示しません。 |
```