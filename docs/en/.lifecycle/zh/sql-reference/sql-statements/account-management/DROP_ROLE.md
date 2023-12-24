---
displayed_sidebar: English
---

# 删除角色

## 描述

此语句允许用户删除角色。

## 语法

```sql
DROP ROLE <role_name>
```

 删除角色不会影响以前属于该角色的用户的权限。它只是将角色与用户解绑，而不会改变用户已从该角色获得的权限。

## 例子

删除一个角色。

  ```sql
  DROP ROLE role1;
  ```
