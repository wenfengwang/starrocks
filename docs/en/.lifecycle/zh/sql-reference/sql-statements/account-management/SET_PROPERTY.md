---
displayed_sidebar: English
---

# 设置属性

## 描述

### 语法

```SQL
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

设置用户属性，包括分配给用户的资源等。这里的用户属性指的是用户的属性，而不是user_identity的属性。也就是说，如果通过CREATE USER语句创建了两个用户'jack'@'%'和'jack'@'192.%'，则SET PROPERTY语句只能用于用户jack，而不能用于'jack'@'%'或'jack'@'192.%'。

键：

超级用户权限：

```plain text
max_user_connections: 最大连接数
resource.cpu_share: CPU资源分配
```

普通用户权限：

```plain text
quota.normal: 普通级别的资源分配
quota.high: 高级别的资源分配
quota.low: 低级别的资源分配
```

## 例子

1. 修改用户jack的最大连接数为1000

    ```SQL
    SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
    ```

2. 修改用户jack的cpu_share为1000

    ```SQL
    SET PROPERTY FOR 'jack' 'resource.cpu_share' = '1000';
    ```

3. 修改用户jack的普通级别权重

    ```SQL
    SET PROPERTY FOR 'jack' 'quota.normal' = '400';
    ```
