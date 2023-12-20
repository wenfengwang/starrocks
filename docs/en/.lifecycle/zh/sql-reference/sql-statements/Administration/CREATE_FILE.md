---
displayed_sidebar: English
---

# 创建文件

您可以执行 CREATE FILE 语句来创建文件。文件创建后，将被上传并持久化保存在 StarRocks 中。在数据库中，只有管理员用户可以创建和删除文件，所有有权限访问该数据库的用户都可以使用属于该数据库的文件。

:::tip

此操作需要 SYSTEM 级别的 FILE 权限。您可以按照 [GRANT](../account-management/GRANT.md) 中的说明来授予此权限。

:::

## 基本概念

**文件**：指在 StarRocks 中创建并保存的文件。文件在 StarRocks 中创建并存储后，StarRocks 会为该文件分配一个唯一 ID。您可以根据数据库名称、catalog 和文件名来查找文件。

## 语法

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## 参数

|**参数**|**必填**|**说明**|
|---|---|---|
|file_name|是|文件的名称。|
|database|否|文件所属的数据库。如果您不指定此参数，它将默认为您在当前会话中访问的数据库名称。|
|properties|是|文件的属性。下表描述了 properties 的配置项。|

**`properties` 的配置项**

|**配置项**|**必填**|**说明**|
|---|---|---|
|url|是|您可以从该 URL 下载文件。仅支持无需认证的 HTTP URL。文件存储在 StarRocks 中后，URL 将不再需要。|
|catalog|是|文件所属的分类。您可以根据业务需求指定 catalog。但在某些情况下，您必须将此参数设置为特定的 catalog。例如，如果您从 Kafka 加载数据，StarRocks 会在 Kafka 数据源的 catalog 中查找文件。|
|MD5|否|用于校验文件的消息摘要算法。如果您指定了此参数，StarRocks 会在文件下载后进行校验。|

## 示例

- 在名为 kafka 的 catalog 下创建名为 **test.pem** 的文件。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "url" = "https://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "catalog" = "kafka"
);
```

- 在名为 my_catalog 的 catalog 下创建名为 **client.key** 的文件。

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