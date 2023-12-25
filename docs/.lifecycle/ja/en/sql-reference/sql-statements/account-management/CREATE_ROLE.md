---
displayed_sidebar: English
---

# CREATE ROLE

## 説明

ロールを作成します。ロールが作成された後、そのロールに権限を付与し、そのロールをユーザーまたは他のロールに割り当てることができます。そうすることで、このロールに関連する権限がユーザーやロールに引き継がれます。

`user_admin` ロールを持つユーザー、または `GRANT` 権限を持つユーザーのみがロールを作成できます。

## 構文

```sql
CREATE ROLE <role_name>
```

## パラメーター

`role_name`: ロールの名前。命名規則は以下の通りです:

- 数字 (0-9)、英字、またはアンダースコア (_) のみを含むことができ、英字で始まる必要があります。
- 長さは 64 文字を超えてはなりません。

作成されたロール名は[システム定義のロール](../../../administration/privilege_overview.md#system-defined-roles)と同じであってはなりません。

## 例

 ロールを作成します。

  ```sql
  CREATE ROLE role1;
  ```

## 参照

- [GRANT](GRANT.md)
- [SHOW ROLES](SHOW_ROLES.md)
- [DROP ROLE](DROP_ROLE.md)
