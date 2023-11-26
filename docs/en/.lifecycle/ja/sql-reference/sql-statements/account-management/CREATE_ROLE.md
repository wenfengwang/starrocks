---
displayed_sidebar: "Japanese"
---

# ロールの作成

## 説明

ロールを作成します。ロールが作成されると、そのロールに特権を付与し、その後、このロールをユーザーまたは他のロールに割り当てることができます。これにより、このロールに関連付けられた特権がユーザーまたはロールに引き継がれます。

`user_admin`ロールまたは`GRANT`特権を持つユーザーのみがロールを作成できます。

## 構文

```sql
CREATE ROLE <role_name>
```

## パラメーター

`role_name`: ロールの名前。命名規則:

- 数字（0-9）、文字、またはアンダースコア（_）のみを含むことができ、文字で始まる必要があります。
- 長さは64文字を超えることはできません。

作成されたロール名は、[システム定義のロール](../../../administration/privilege_overview.md#system-defined-roles)と同じにすることはできません。

## 例

ロールを作成します。

```sql
CREATE ROLE role1;
```

## 参照

- [GRANT](GRANT.md)
- [SHOW ROLES](SHOW_ROLES.md)
- [DROP ROLE](DROP_ROLE.md)
