---
displayed_sidebar: English
---

# 删除文件

## 说明

您可以执行 DROP FILE 语句来删除一个文件。当您使用这个语句删除文件时，该文件将会同时从前端（FE）内存和 Berkeley DB Java 版本（BDBJE）中被删除。

:::提示

进行此操作需要系统级的 **FILE** 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 指令中的步骤来授予这项权限。

:::

## 语法

```SQL
DROP FILE "file_name" [FROM database]
[properties]
```

## 参数

|参数|必填|说明|
|---|---|---|
|file_name|是|文件的名称。|
|database|否|文件所属的数据库。|
|属性|是|文件的属性。下表描述了属性的配置项。|

**`properties`** 的配置项

|配置项|必填项|说明|
|---|---|---|
|catalog|是|文件所属的类别。|

## 示例

删除一个名为 **ca.pem** 的文件。

```SQL
DROP FILE "ca.pem" properties("catalog" = "kafka");
```
