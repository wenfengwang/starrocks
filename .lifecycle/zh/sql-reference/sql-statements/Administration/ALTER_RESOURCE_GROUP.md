---
displayed_sidebar: English
---

# 修改资源组配置

## 描述

调整资源组的配置设置。

:::提示

执行此操作需要对目标资源组有 **ALTER** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 指令中的说明来授予这项权限。

:::

## 语法

```SQL
ALTER RESOURCE GROUP resource_group_name
{  ADD CLASSIFIER1, CLASSIFIER2, ...
 | DROP (CLASSIFIER_ID_1, CLASSIFIER_ID_2, ...)
 | DROP ALL
 | WITH resource_limit 
};
```

## 参数

|参数|说明|
|---|---|
|resource_group_name|要更改的资源组的名称。|
|ADD|将分类器添加到资源组。有关如何定义分类器的详细信息，请参阅创建资源组 - 参数。|
|DROP|通过分类器 ID 从资源组中删除分类器。您可以通过 SHOW RESOURCE GROUP 语句检查分类器的 ID。|
|DROP ALL|从资源组中删除所有分类器。|
|WITH|修改资源组的资源限制。有关如何设置资源限制的详细信息，请参阅创建资源组 - 参数。|

## 示例

示例 1：为资源组 rg1 添加一个新的分类器。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

示例 2：从资源组 rg1 中移除 ID 为 300040、300041 和 300042 的分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

示例 3：移除资源组 rg1 中的所有分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

示例 4：修改资源组 rg1 的资源限额。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```
