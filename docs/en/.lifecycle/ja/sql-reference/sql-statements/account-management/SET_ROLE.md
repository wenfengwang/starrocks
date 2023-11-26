---
displayed_sidebar: "Japanese"
---

# SET ROLE（ロールの設定）

## 説明

現在のセッションでロールをアクティブ化します。ロールがアクティブ化されると、ユーザーはこのロールを使用して操作を実行することができます。

このコマンドを実行した後、`select is_role_in_session("<role_name>");` を実行して、このロールが現在のセッションでアクティブ化されているかどうかを確認できます。

このコマンドはv3.0からサポートされています。

## 構文

```SQL
-- 特定のロールをアクティブ化し、このロールで操作を実行します。
SET ROLE <role_name>[,<role_name>,..];
-- 特定のロールを除いて、ユーザーのすべてのロールをアクティブ化します。
SET ROLE ALL EXCEPT <role_name>[,<role_name>,..]; 
-- ユーザーのすべてのロールをアクティブ化します。
SET ROLE ALL;
```

## パラメーター

`role_name`: ロール名

## 使用上の注意

ユーザーは、自分に割り当てられたロールのみをアクティブ化できます。

[SHOW GRANTS](./SHOW_GRANTS.md) を使用して、ユーザーのロールをクエリできます。

`SELECT CURRENT_ROLE()` を使用して、現在のユーザーのアクティブなロールをクエリできます。詳細については、[current_role](../../sql-functions/utility-functions/current_role.md) を参照してください。

## 例

現在のユーザーのすべてのロールをクエリします。

```SQL
SHOW GRANTS;
+--------------+---------+----------------------------------------------+
| UserIdentity | Catalog | Grants                                       |
+--------------+---------+----------------------------------------------+
| 'test'@'%'   | NULL    | GRANT 'db_admin', 'user_admin' TO 'test'@'%' |
+--------------+---------+----------------------------------------------+
```

`db_admin` ロールをアクティブ化します。

```SQL
SET ROLE db_admin;
```

現在のユーザーのアクティブなロールをクエリします。

```SQL
SELECT CURRENT_ROLE();
+--------------------+
| CURRENT_ROLE()     |
+--------------------+
| db_admin           |
+--------------------+
```

## 参照

- [CREATE ROLE](CREATE_ROLE.md): ロールを作成します。
- [GRANT](GRANT.md): ユーザーまたは他のロールにロールを割り当てます。
- [ALTER USER](ALTER_USER.md): ロールを変更します。
- [SHOW ROLES](SHOW_ROLES.md): システム内のすべてのロールを表示します。
- [current_role](../../sql-functions/utility-functions/current_role.md): 現在のユーザーのロールを表示します。
- [is_role_in_session](../../sql-functions/utility-functions/is_role_in_session.md): ロール（またはネストされたロール）が現在のセッションでアクティブ化されているかどうかを確認します。
- [DROP ROLE](DROP_ROLE.md): ロールを削除します。
