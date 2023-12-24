---
displayed_sidebar: English
---

# 删除文件

## 描述

您可以执行DROP FILE语句来删除文件。当您使用此语句删除文件时，文件将在前端（FE）内存和Berkeley DB Java Edition（BDBJE）中均被删除。

:::提示

此操作需要SYSTEM级别的FILE权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明来授予此权限。

:::

## 语法

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| file_name     | 是          | 文件的名称。                                        |
| database      | 否           | 文件所属的数据库。                        |
| properties    | 是          | 文件的属性。属性的配置项如下表所示。 |

**属性的配置项**

| **配置项** | **必填** | **描述**                       |
| ----------------------- | ------------ | ------------------------------------- |
| catalog                 | 是          | 文件所属的类别。 |

## 例子

删除名为**ca.pem**的文件。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");