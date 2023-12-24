---
displayed_sidebar: English
---

# 创建文件

您可以执行CREATE FILE语句来创建文件。文件创建完成后，文件会上传并持久化到 StarRocks 中。在数据库中，只有管理员用户可以创建和删除文件，并且所有有权访问数据库的用户都可以使用属于该数据库的文件。

:::提示

此操作需要SYSTEM级别的FILE权限。您可以按照[GRANT](../account-management/GRANT.md)中的说明授予此权限。

:::

## 基本概念

**文件**：指在 StarRocks 中创建并保存的文件。文件创建并存储在 StarRocks 中后，StarRocks 会为该文件分配一个唯一的ID。您可以根据数据库名称、目录和文件名查找文件。

## 语法

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## 参数

| **参数** | **必填** | **描述**                                              |
| ------------- | ------------ | ------------------------------------------------------------ |
| file_name     | 是          | 文件的名称。                                        |
| database      | 否           | 文件所属的数据库。如果不指定此参数，则默认为您在当前会话中访问的数据库的名称。 |
| properties    | 是          | 文件的属性。属性的配置项如下表所示。 |

**属性的配置项**

| **配置项** | **必填** | **描述**                                              |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| url                    | 是          | 可以下载文件的URL。仅支持未经身份验证的HTTP URL。文件存储在StarRocks中后，URL就不再需要。 |
| catalog                | 是          | 文件所属的类别。您可以根据业务需求指定目录。但是在某些情况下，必须将此参数设置为特定的目录。例如，如果从Kafka加载数据，StarRocks会从Kafka数据源中搜索目录中的文件。 |
| MD5                    | 否           | 用于检查文件的消息摘要算法。如果指定此参数，StarRocks会在文件下载后检查文件。 |

## 例子

- 在名为kafka的类别下创建名为test.pem的文件。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "url" = "https://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "catalog" = "kafka"
);
```

- 在名为my_catalog的类别下创建名为client.key的文件。

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
