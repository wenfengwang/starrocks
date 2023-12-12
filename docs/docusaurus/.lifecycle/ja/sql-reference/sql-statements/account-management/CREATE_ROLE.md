---
displayed_sidebar: "Japanese"
---

# ロールの作成

## 説明

ロールを作成します。ロールを作成した後は、そのロールに特権を付与し、そのロールをユーザーまたは他のロールに割り当てることができます。これにより、このロールに関連付けられた特権がユーザーやロールに引き継がれます。

`user_admin`ロールまたは`GRANT`特権を持つユーザーのみがロールを作成できます。

## 構文

```sql
CREATE ROLE <role_name>
```

## パラメータ

`role_name`: ロールの名前。命名規則:

- 数値（0-9）、英字、またはアンダースコア（_）のみを含むことができ、英字で始まる必要があります。
- 64文字を超えないこと。

作成したロール名は[システム定義ロール](../../../administration/privilege_overview.md#system-defined-roles)と同じにはできません。

## 例

ロールを作成します。

  ```sql
  CREATE ROLE role1;
  ```

## 参照

- [GRANT](GRANT.md)
- [SHOW ROLES](SHOW_ROLES.md)
- [DROP ROLE](DROP_ROLE.md)