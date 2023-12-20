---
displayed_sidebar: English
---

# 删除角色

## 说明

此语句允许用户删除一个角色。

## 语法

```sql
DROP ROLE <role_name>
```

删除角色不会影响先前属于该角色的用户的权限。它仅仅是将角色与用户解除关联，而不改变用户已经由该角色获得的权限。

## 示例

删除一个角色。

```sql
DROP ROLE role1;
```
