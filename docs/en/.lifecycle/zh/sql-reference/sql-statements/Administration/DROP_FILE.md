---
displayed_sidebar: English
---

# 删除文件

## 描述

您可以执行 DROP FILE 语句来删除文件。当您使用此语句删除文件时，文件将同时从前端（FE）内存和 Berkeley DB Java Edition（BDBJE）中删除。

:::tip

此操作需要 SYSTEM 级别的 FILE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明授予此权限。

:::

## 语法

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## 参数

|**参数**|**是否必填**|**描述**|
|---|---|---|
|file_name|是|文件的名称。|
|database|否|文件所属的数据库。|
|properties|是|文件的属性。下表描述了 properties 的配置项。|

**properties 的配置项**

|**配置项**|**是否必填**|**描述**|
|---|---|---|
|catalog|是|文件所属的分类。|

## 示例

删除名为 **ca.pem** 的文件。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");
```