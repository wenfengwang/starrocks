---
displayed_sidebar: "Chinese"
---

# 修改资源组

## 描述

修改资源组的配置。

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

| **参数**             | **描述**                                                      |
| ------------------- | ------------------------------------------------------------ |
| resource_group_name | 要修改的资源组的名称。                                         |
| ADD                 | 将分类器添加到资源组。有关如何定义分类器的更多信息，请参阅[CREATE RESOURCE GROUP-参数](../Administration/CREATE_RESOURCE_GROUP.md)。 |
| DROP                | 通过分类器ID从资源组中删除分类器。您可以通过[SHOW RESOURCE GROUP](../Administration/SHOW_RESOURCE_GROUP.md)语句来检查分类器的ID。 |
| DROP ALL            | 从资源组中删除所有分类器。                                       |
| WITH                | 修改资源组的资源限制。有关如何设置资源限制的更多信息，请参阅[CREATE RESOURCE GROUP-参数](../Administration/CREATE_RESOURCE_GROUP.md)。 |

## 示例

示例1：将一个新的分类器添加到资源组`rg1`。

```SQL
ALTER RESOURCE GROUP rg1 ADD (user='root', query_type in ('select'));
```

示例2：从资源组`rg1`中删除ID为`300040`，`300041`和`300041`的分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP (300040, 300041, 300041);
```

示例3：从资源组`rg1`中删除所有分类器。

```SQL
ALTER RESOURCE GROUP rg1 DROP ALL;
```

示例4：修改资源组`rg1`的资源限制。

```SQL
ALTER RESOURCE GROUP rg1 WITH (
    'cpu_core_limit' = '20'
);
```