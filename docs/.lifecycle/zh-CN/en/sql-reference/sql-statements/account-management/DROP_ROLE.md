---
displayed_sidebar: "Chinese"
---

# 删除角色

## 描述

此语句允许用户删除一个角色。

## 语法

```sql
DROP ROLE <role_name>
```

删除一个角色不会影响以前属于该角色的用户的权限。它只是将角色与用户分离，而不会改变用户已经从角色获得的权限。

## 示例

删除一个角色。

  ```sql
  DROP ROLE role1;
  ```