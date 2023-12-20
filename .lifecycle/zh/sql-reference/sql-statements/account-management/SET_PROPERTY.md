---
displayed_sidebar: English
---

# 设置属性

## 描述

### 语法

```SQL
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

设置用户属性，包括为用户分配的资源等。这里所说的用户属性指的是针对用户的属性，而非user_identity的属性。也就是说，如果通过CREATE USER语句创建了两个用户'jack'@'%'和'jack'@'192.%'，那么SET PROPERTY语句只适用于用户名为jack的用户，不适用于'jack'@'%'或'jack'@'192.%'。

关键字：

超级用户权限：

```plain
max_user_connections: Maximum number of connections
resource.cpu_share: cpu resource assignment
```

普通用户权限：

```plain
quota.normal: Resource allocation at the normal level
quota.high: Resource allocation at the high level 
quota.low: Resource allocation at the low level
```

## 示例

1. 为用户jack修改最大连接数至1000

   ```SQL
   SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
   ```

2. 为用户jack将cpu_share修改为1000

   ```SQL
   SET PROPERTY FOR 'jack' 'resource.cpu_share' = '1000';
   ```

3. 为用户jack修改其普通级别的权重

   ```SQL
   SET PROPERTY FOR 'jack' 'quota.normal' = '400';
   ```
