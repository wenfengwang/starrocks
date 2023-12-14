---
displayed_sidebar: "中文"
---

# 删除角色（DROP ROLE）

## 功能

删除一个角色。如果一个角色已经被某个用户拥有，那么即便删除了该角色，该用户还是会保留该角色所拥有的相关权限。

> **注意**
>
> - 只有 `user_admin` 用户可以删除角色。
> - StarRocks 系统中预置的角色不能被删除。

## 语法

```SQL
DROP ROLE <role_name>
```

## 参数说明

`role_name`：要删除的角色的名称。

## 示例

删除名为`analyst`的角色。

```SQL
  DROP ROLE analyst;
```