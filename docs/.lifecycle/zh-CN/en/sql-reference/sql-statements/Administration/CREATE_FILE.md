---
displayed_sidebar: "Chinese"
---

# 创建文件

您可以执行创建文件语句来创建文件。文件创建后，文件将被上传并在StarRocks中持久化。在数据库中，只有管理员用户可以创建和删除文件，所有有权限访问数据库的用户都可以使用属于数据库的文件。

## 基本概念

**文件**：指在StarRocks中创建和保存的文件。文件创建并存储在StarRocks中后，StarRocks会为文件分配一个唯一的ID。您可以根据数据库名称、类别和文件名称找到文件。

## 语法

```SQL
CREATE FILE "file_name" [IN database]
[properties]
```

## 参数

| **参数**         | **是否必须**  | **描述**                                                  |
| ------------- | ------------ | ------------------------------------------------------------ |
| file_name     | 是           | 文件名称。                                                      |
| database      | 否           | 文件所属的数据库。如果不指定此参数，此参数默认为您在当前会话中访问的数据库名称。  |
| properties    | 是           | 文件的属性。以下表格描述了属性的配置项。                         |

**`properties` 的配置项**

| **配置项**            | **是否必须**  | **描述**                                                  |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| URL                    | 是           | 您可以下载文件的URL。只支持未经身份验证的HTTP URL。文件存储在StarRocks后，将不再需要该URL。 |
| 类别                   | 是           | 文件所属的类别。根据您的业务需求，您可以指定一个类别。不过，在某些情况下，您必须将此参数设置为特定的类别。例如，如果从Kafka加载数据，StarRocks将从Kafka数据源的类别中搜索文件。 |
| MD5                    | 否           | 用于检查文件的消息摘要算法。如果指定此参数，StarRocks会在文件下载后检查文件。   |

## 示例

- 创建一个名为 **test.pem** 的文件，属于名为 kafka 的类别。

```SQL
CREATE FILE "test.pem"
PROPERTIES
(
    "URL" = "https://starrocks-public.oss-cn-xxxx.aliyuncs.com/key/test.pem",
    "类别" = "kafka"
);
```

- 创建一个名为 **client.key** 的文件，属于名为 my_catalog 的类别。

```SQL
CREATE FILE "client.key"
IN my_database
PROPERTIES
(
    "URL" = "http://test.bj.bcebos.com/kafka-key/client.key",
    "类别" = "my_catalog",
    "MD5" = "b5bb901bf10f99205b39a46ac3557dd9"
);
```