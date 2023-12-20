---
displayed_sidebar: English
---

# 当前会话中的角色状态

## 描述

验证某个角色（或嵌套角色）在当前会话中是否是激活状态。

该函数从 v3.1.4 版本开始支持。

## 语法

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## 参数

role_name：需要验证的角色（也可以是嵌套角色）。支持的数据类型为 VARCHAR。

## 返回值

返回一个布尔值。1 表示该角色在当前会话中是激活的。0 表示不是。

## 示例

1. 创建角色和一个用户。

   ```sql
   -- Create three roles.
   create role r1;
   create role r2;
   create role r3;
   
   -- Create user u1.
   create user u1;
   
   -- Pass roles r2 and r3 to r1, and grant r1 to user u1. This way, user u1 has three roles: r1, r2, and r3.
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;
   
   -- Switch to user u1 and perform operations as u1.
   execute as u1 with no revert;
   ```

2. 验证角色 r1 是否激活。结果显示这个角色没有激活。

   ```plaintext
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. 运行[SET ROLE](../../sql-statements/account-management/SET_ROLE.md)命令以激活 `r1`，并使用`is_role_in_session`函数来验证角色是否激活。结果表明`r1`已激活，并且角色`r2`和`r3`嵌套在`r1`里也已激活。

   ```sql
   set role "r1";
   
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   
   select is_role_in_session("r2");
   +--------------------------+
   | is_role_in_session('r2') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   
   select is_role_in_session("r3");
   +--------------------------+
   | is_role_in_session('r3') |
   +--------------------------+
   |                        1 |
   +--------------------------+
   ```
