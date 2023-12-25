---
displayed_sidebar: Chinese
---

# SET ROLE

## 機能

現在のセッションで、現在のユーザーが持つ権限のあるロールをアクティブにします。アクティブにした後、ユーザーはそのロールを使用して関連する操作を行うことができます。

`select is_role_in_session("<role_name>")` を使用して、指定されたロールが現在のセッションで既にアクティブになっているかどうかを確認できます。

このコマンドはバージョン 3.0 からサポートされています。

## 注意事項

- ユーザーは自分が持つ権限のあるロールのみをアクティブにできます。

- ユーザーは [SHOW GRANTS](SHOW_GRANTS.md) で自分が持つロールを確認でき、[SELECT CURRENT_ROLE](../../sql-functions/utility-functions/current_role.md) で現在アクティブなロールを確認できます。

## 文法

```SQL
-- 特定のロールをアクティブにし、そのロールを使用して操作を行います。
SET ROLE <role_name>[,<role_name>,..];
-- 指定されたロールを除く、ユーザーが持つすべてのロールをアクティブにします。
SET ROLE ALL EXCEPT <role_name>[,<role_name>,..]; 
-- ユーザーが持つすべてのロールをアクティブにします。
SET ROLE ALL;
```

## パラメータ説明

`role_name`: ユーザーが持つロール名。

## 例

1. 現在のユーザーが持つロールを確認します。

    ```SQL
    SHOW GRANTS;
    +--------------+---------+----------------------------------------------+
    | UserIdentity | Catalog | Grants                                       |
    +--------------+---------+----------------------------------------------+
    | 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
    +--------------+---------+----------------------------------------------+
    ```

2. `db_admin` ロールをアクティブにします。

    ```plain
    SET ROLE db_admin;
    ```

3. 現在アクティブなロールを確認します。

    ```SQL
    SELECT CURRENT_ROLE();
    +--------------------+
    | CURRENT_ROLE()     |
    +--------------------+
    | db_admin           |
    +--------------------+
    ```

## 関連文書

- [CREATE ROLE](CREATE_ROLE.md): ロールを作成します。
- [GRANT](GRANT.md): ロールをユーザーや他のロールに割り当てます。
- [ALTER USER](ALTER_USER.md): ユーザーのロールを変更します。
- [SHOW ROLES](SHOW_ROLES.md): システム上のすべてのロールを確認します。
- [current_role](../../sql-functions/utility-functions/current_role.md): 現在のユーザーが持つロールを確認します。
- [is_role_in_session](../../sql-functions/utility-functions/is_role_in_session.md)
- [DROP ROLE](DROP_ROLE.md): ロールを削除します。
