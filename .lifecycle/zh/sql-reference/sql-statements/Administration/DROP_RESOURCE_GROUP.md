---
displayed_sidebar: English
---

# 删除资源组

## 描述

删除指定的资源组。

:::提示

此操作需要在目标资源组上拥有 **DROP** 权限。您可以根据 [GRANT](../account-management/GRANT.md) 中的指南来授予相应权限。

:::

## 语法

```SQL
DROP RESOURCE GROUP <resource_group_name>
```

## 参数

|参数|说明|
|---|---|
|resource_group_name|要删除的资源组的名称。|

## 示例

示例 1：删除名为 rg1 的资源组。

```SQL
DROP RESOURCE GROUP rg1;
```
