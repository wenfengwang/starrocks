---
displayed_sidebar: English
---

# is_role_in_session

## 描述

验证当前会话中是否激活了角色（或嵌套角色）。

此功能从 v3.1.4 版本开始支持。

## 语法

```Haskell
BOOLEAN is_role_in_session(VARCHAR role_name);
```

## 参数

`role_name`：要验证的角色（也可以是嵌套角色）。支持的数据类型为 VARCHAR。

## 返回值

返回 BOOLEAN 值。`1` 表示角色在当前会话中是激活的。`0` 表示相反。

## 例子

1. 创建角色和用户。

   ```sql
   -- 创建三个角色。
   create role r1;
   create role r2;
   create role r3;

   -- 创建用户 u1。
   create user u1;

   -- 将角色 r2 和 r3 传递给 r1，并授予 r1 给用户 u1。这样，用户 u1 就拥有了三个角色：r1、r2 和 r3。
   grant r3 to role r2;
   grant r2 to role r1;
   grant r1 to user u1;

   -- 切换到用户 u1 并作为 u1 执行操作。
   execute as u1 with no revert;
   ```

2. 验证 `r1` 是否激活。结果显示该角色未激活。

   ```plaintext
   select is_role_in_session("r1");
   +--------------------------+
   | is_role_in_session('r1') |
   +--------------------------+
   |                        0 |
   +--------------------------+
   ```

3. 运行 [SET ROLE](../../sql-statements/account-management/SET_ROLE.md) 命令激活 `r1`，并使用 `is_role_in_session` 验证角色是否激活。结果显示 `r1` 是激活的，嵌套在 `r1` 中的角色 `r2` 和 `r3` 也是激活的。

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
