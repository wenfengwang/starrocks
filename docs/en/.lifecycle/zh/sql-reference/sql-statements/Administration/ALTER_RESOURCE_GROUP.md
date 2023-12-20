---
displayed_sidebar: English
---

# 更改资源组

## 描述

更改资源组的配置。

:::tip

此操作需要对目标资源组的 **ALTER** 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

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

|**参数**|**说明**|
|---|---|
|resource_group_name|要更改的资源组的名称。|
|ADD|向资源组添加分类器。有关如何定义分类器的更多信息，请参见 [CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)。|
|DROP|通过分类器 ID 从资源组中移除分类器。您可以通过 [SHOW RESOURCE GROUP](../Administration/SHOW_RESOURCE_GROUP.md) 语句来检查分类器的 ID。|
|DROP ALL|从资源组中移除所有分类器。|
|WITH|修改资源组的资源限制。有关如何设置资源限制的更多信息，请参见 [CREATE RESOURCE GROUP - Parameters](../Administration/CREATE_RESOURCE_GROUP.md)。|

## 示例

示例 1：向资源组 `rg1` 添加新分类器。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

示例 2：从资源组 `rg1` 中移除 ID 为 `300040`、`300041` 和 `300042` 的分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300042);
```

示例 3：移除资源组 `rg1` 中的所有分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

示例 4：修改资源组 `rg1` 的资源限制。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```