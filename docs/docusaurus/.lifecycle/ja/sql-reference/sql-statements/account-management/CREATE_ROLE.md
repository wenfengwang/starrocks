---
displayed_sidebar: "英語"
---

# CREATE ROLE

## 説明

ロールを作成します。ロールを作成した後、そのロールに権限を付与し、その後ユーザーまたは別のロールにこのロールを割り当てることができます。このようにして、このロールに関連する権限はユーザーやロールに伝達されます。

`user_admin` ロールまたは `GRANT` 権限を持つユーザーのみがロールを作成できます。

## 構文

```sql
CREATE ROLE <role_name>
```

## パラメータ

`role_name`: ロールの名前。命名規則：

- 数字 (0-9)、文字、またはアンダースコア (_) のみを含むことができ、文字で始まらなければなりません。
- 長さは 64 文字を超えてはなりません。

作成されたロール名は[システム定義ロール](../../../administration/privilege_overview.md#system-defined-roles)と同じであってはなりません。

## 例

 ロールを作成。

  ```sql
  CREATE ROLE role1;
  ```

## 参照

- [GRANT](GRANT.md)
- [SHOW ROLES](SHOW_ROLES.md)
- [DROP ROLE](DROP_ROLE.md)