---
displayed_sidebar: English
---

# 创建文件

您可以执行 CREATE FILE 语句来创建文件。文件创建后，会被上传并持久化存储在 StarRocks 中。在数据库中，只有管理员用户可以创建和删除文件，而所有被授权访问该数据库的用户均可以使用属于该数据库的文件。

:::tip

此操作需要系统级别的 FILE 权限。您可以按照[GRANT](../account-management/GRANT.md)指令中的说明来授予此权限。

:::

## 基本概念

**文件**：指的是在 StarRocks 中创建并保存的文件。文件在 StarRocks 中创建并存储后，StarRocks 会为其分配一个唯一 ID。您可以基于数据库名称、目录和文件名来查找文件。

## 语法

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## 参数

|参数|必填|说明|
|---|---|---|
|file_name|是|文件的名称。|
|database|否|文件所属的数据库。如果不指定此参数，则此参数默认为您在当前会话中访问的数据库的名称。|
|属性|是|文件的属性。下表描述了属性的配置项。|

**`properties`** 的配置项

|配置项|必填|说明|
|---|---|---|
|url|是|您可以从中下载文件的 URL。仅支持未经身份验证的 HTTP URL。文件存储在 StarRocks 中后，不再需要 URL。|
|catalog|是|文件所属的类别。您可以根据您的业务需求指定目录。但是，在某些情况下，您必须将此参数设置为特定目录。例如，如果您从 Kafka 加载数据，StarRocks 将在 Kafka 数据源的目录中搜索文件。|
|MD5|否|用于检查文件的消息摘要算法。如果指定此参数，StarRocks 将在下载文件后检查该文件。|

## 示例

- 在名为 **kafka** 的分类下创建一个名为 **test.pem** 的文件。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "url" = "https://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "catalog" = "kafka"
);
```

- 在名为 my_catalog 的分类下创建一个名为 **client.key** 的文件。

```SQL
CREATE FILE "client.key"
IN my_database
PROPERTIES
(
    "url" = "http://test.bj.bcebos.com/kafka-key/client.key",
    "catalog" = "my_catalog",
    "md5" = "b5bb901bf10f99205b39a46ac3557dd9"
);
```
