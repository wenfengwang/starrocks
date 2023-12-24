---
displayed_sidebar: English
---

# Hive 目录

Hive 目录是 StarRocks 从 v2.4 开始支持的一种外部目录。在 Hive 目录中，您可以：

- 直接查询存储在 Hive 中的数据，无需手动创建表。
- 使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 或异步物化视图（从 v2.5 开始支持）处理存储在 Hive 中的数据，并将数据加载到 StarRocks 中。
- 在 StarRocks 上执行创建或删除 Hive 数据库和表的操作，或者使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 将 StarRocks 表中的数据下沉到 Parquet 格式的 Hive 表中（从 v3.2 开始支持此功能）。

为确保 Hive 集群上的 SQL 工作负载成功运行，StarRocks 集群需要集成两个重要组件：

- 分布式文件系统 （HDFS） 或对象存储，如 AWS S3、Microsoft Azure 存储、Google GCS 或其他与 S3 兼容的存储系统（例如 MinIO）

- 元存储，如 Hive 元存储或 AWS Glue

  > **注意**
  >
  > 如果您选择 AWS S3 作为存储，则可以使用 HMS 或 AWS Glue 作为元存储。如果选择其他存储系统，则只能使用 HMS 作为元存储。

## 使用说明

- StarRocks 支持的 Hive 文件格式包括 Parquet、ORC、CSV、Avro、RCFile 和 SequenceFile：

  - Parquet 文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP 和 NO_COMPRESSION。从 v3.1.5 开始，Parquet 文件还支持 LZO 压缩格式。
  - ORC 文件支持以下压缩格式：ZLIB、SNAPPY、LZO、LZ4、ZSTD 和 NO_COMPRESSION。
  - CSV 文件从 v3.1.5 开始支持 LZO 压缩格式。

- StarRocks 不支持的 Hive 数据类型有 INTERVAL、BINARY 和 UNION。此外，StarRocks 不支持 CSV 格式的 Hive 表的 MAP 和 STRUCT 数据类型。
- 只能使用 Hive 目录查询数据。不能使用 Hive 目录将数据拖放、删除或插入到 Hive 群集中。

## 集成准备

在创建 Hive 目录之前，请确保您的 StarRocks 集群可以与 Hive 集群的存储系统和元存储集成。

### AWS IAM

如果您的 Hive 集群使用 AWS S3 作为存储，或者使用 AWS Glue 作为元存储，请选择合适的身份验证方式并做好必要的准备工作，以确保您的 StarRocks 集群能够访问相关的 AWS 云资源。

建议使用以下身份验证方法：

- 实例配置文件
- 承担的角色
- IAM 用户

在上述三种身份验证方法中，实例配置文件是应用最广泛的。

有关更多信息，请参阅 [在 AWS IAM 中准备身份验证](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果您选择 HDFS 作为存储，请按如下方式配置 StarRocks 集群：

- （可选）设置用于访问 HDFS 集群和 Hive 元存储的用户名。StarRocks 默认使用 FE 和 BE 进程的用户名访问 HDFS 集群和 Hive 元存储。也可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件的开头和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。设置用户名后，重启每个 FE 和每个 BE，使参数设置生效。每个 StarRocks 集群只能设置一个用户名。
- 当您查询 Hive 数据时，StarRocks 集群的 FE 和 BE 会使用 HDFS 客户端访问 HDFS 集群。大多数情况下，您不需要配置 StarRocks 集群来达到这个目的，StarRocks 会使用默认配置来启动 HDFS 客户端。只有在以下情况下，您才需要配置 StarRocks 集群：

  - 开启 HDFS 集群高可用 （HA）：将 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径中。
  - 开启 HDFS 集群的 View 文件系统 （ViewFs）：将 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径中。

> **注意**
>
> 如果发送查询时返回未知主机的错误，则需要将 HDFS 集群节点的主机名和 IP 地址之间的映射添加到 **/etc/hosts** 路径中。

### Kerberos 身份验证

如果您的 HDFS 集群或 Hive 元存储开启了 Kerberos 认证，请按如下方式配置 StarRocks 集群：

- 在每个 FE 和每个 BE 上运行 `kinit -kt keytab_path principal` 命令，从密钥分发中心（KDC）获取 Ticket Granting Ticket（TGT）。若要运行此命令，您必须具有访问 HDFS 群集和 Hive 元存储的权限。请注意，使用此命令访问 KDC 是时间敏感的。因此，您需要使用 cron 定期运行此命令。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改路径。

## 创建 Hive 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "hive",
    GeneralParams,
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### 参数

#### catalog_name

Hive 目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### 评论

Hive 目录的说明。此参数是可选的。

#### 类型

数据源的类型。将值设置为 `hive`。

#### GeneralParams

一组常规参数。

下表描述了可以在 **GeneralParams** 中配置的参数。

| 参数                | 必填 | 描述                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| enable_recursive_listing | 不       | 指定 StarRocks 是否从表及其分区以及表及其分区的物理位置内的子目录中读取数据。有效值： `true` 和 `false`。默认值： `false`。`true` 值指定以递归方式列出子目录，`false` 值指定忽略子目录。 |

#### MetastoreParams

一组关于 StarRocks 如何与数据源的元存储集成的参数。

##### Hive 元存储

如果选择 Hive 元存储作为数据源的元存储，请按如下方式进行配置 **MetastoreParams** ：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询 Hive 数据之前，您需要将 Hive 元存储节点的主机名和 IP 地址之间的映射关系添加到路径中 `/etc/hosts` 。否则，StarRocks 可能会在您发起查询时无法访问您的 Hive 元存储。

下表描述了您需要在 **MetastoreParams** 中配置的参数。

| 参数           | 必填 | 描述                                                  |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | 是的      | 用于 Hive 群集的元存储类型。将值设置为 `hive`。 |
| hive.metastore.uris | 是的      | Hive 元存储的 URI。格式： `thrift://<metastore_IP_address>:<metastore_port>`。<br />如果为 Hive 元存储启用了高可用性 （HA），则可以指定多个元存储 URI，并用逗号 （,） 分隔它们，例如 `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS 胶水

如果选择 AWS Glue 作为数据源的元存储（仅当您选择 AWS S3 作为存储时才支持），请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法， **MetastoreParams** 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择假定的基于角色的身份验证方法， **MetastoreParams** 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择 IAM 基于用户的身份验证方法， **MetastoreParams** 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 **MetastoreParams** 中配置的参数。

| 参数                     | 必填 | 描述                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | 是的      | 用于 Hive 群集的元存储类型。将值设置为 `glue`。 |
| aws.glue.use_instance_profile | 是的      | 指定是否启用基于实例配置文件的身份验证方法和假定的基于角色的身份验证。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.glue.iam_role_arn         | 不       | 对您的 AWS Glue 数据目录具有权限的 IAM 角色的 ARN。如果您使用假定的基于角色的身份验证方法访问 AWS Glue，则必须指定此参数。 |
| aws.glue.区域               | 是的      | 您的 AWS Glue 数据目录所在的区域。示例： `us-west-1`。 |
| aws.glue.access_key           | 不       | 您的 AWS IAM 用户的访问密钥。如果您使用 IAM 基于用户的身份验证方法访问 AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | 不       | 您的 AWS IAM 用户的私有密钥。如果您使用 IAM 基于用户的身份验证方法访问 AWS Glue，则必须指定此参数。 |

有关如何选择用于访问 AWS Glue 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS Glue 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

一组关于 StarRocks 如何与您的存储系统集成的参数。此参数集是可选的。

如果您使用 HDFS 作为存储系统，则无需配置 `StorageCredentialParams`。

如果您使用 AWS S3、其他兼容 S3 的存储系统、Microsoft Azure 存储或 Google GCS 作为存储系统，则必须配置 `StorageCredentialParams`。

##### AWS S3

如果您选择将 AWS S3 作为 Hive 集群的存储系统，请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于假定角色的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于 IAM 用户的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.s3.iam_role_arn         | 否       | 具有对 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您使用基于假定角色的身份验证方法访问 AWS S3，则必须指定此参数。  |
| aws.s3.region               | 是      | 您的 AWS S3 存储桶所在的区域。示例： `us-west-1`. |
| aws.s3.access_key           | 否       | IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | IAM 用户的私有密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。 |

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储系统

从 v2.5 开始，Hive 目录支持与 S3 兼容的存储系统。

如果选择将兼容 S3 的存储系统（如 MinIO）作为 Hive 群集的存储系统，请按以下方式配置 `StorageCredentialParams` 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                        | 必填 | 描述                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | 是      | 指定是否启用 SSL 连接。<br />有效值： `true` 和 `false`。默认值： `true`。 |
| aws.s3.enable_path_style_access  | 是      | 指定是否启用路径样式访问。<br />有效值： `true` 和 `false`。默认值：`false`。对于 MinIO，必须将值设置为 `true`。<br />路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建了名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | 是      | 用于连接到与 S3 兼容的存储系统（而不是 AWS S3）的终端节点。 |
| aws.s3.access_key                | 是      | IAM 用户的访问密钥。 |
| aws.s3.secret_key                | 是      | IAM 用户的私有密钥。 |

##### Microsoft Azure 存储

Hive 目录从 v3.0 开始支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果选择 Blob 存储作为 Hive 群集的存储系统，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**              | **必填** | **描述**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | 是          | Blob 存储帐户的用户名。   |
  | azure.blob.shared_key      | 是          | Blob 存储帐户的共享密钥。 |

- 要选择 SAS 令牌身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**             | **必填** | **描述**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| 是          | Blob 存储帐户的用户名。                   |
  | azure.blob.container      | 是          | 存储数据的 Blob 容器的名称。        |
  | azure.blob.sas_token      | 是          | 用于访问 Blob 存储帐户的 SAS 令牌。 |

###### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为 Hive 群集的存储系统，请执行以下操作之一：

- 要选择托管服务标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                            | **必填** | **描述**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是          | 指定是否启用托管服务标识身份验证方法。将值设置为 `true`。 |

- 要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                 | **必填** | **描述**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是          | 服务主体的客户端（应用程序）ID。        |
  | azure.adls1.oauth2_credential | 是          | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls1.oauth2_endpoint   | 是          | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

###### Azure Data Lake Storage Gen2

如果选择 Data Lake Storage Gen2 作为 Hive 群集的存储系统，请执行以下操作之一：

- 要选择托管标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                           | **必填** | **描述**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是          | 指定是否启用托管标识身份验证方法。将值设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是          | 要访问其数据的租户的 ID。          |
  | azure.adls2.oauth2_client_id            | 是          | 托管标识的客户端（应用程序）ID。         |

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**               | **必填** | **描述**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是          | Data Lake Storage Gen2 存储帐户的用户名。 |
  | azure.adls2.shared_key      | 是          | Data Lake Storage Gen2 存储帐户的共享密钥。 |

- 要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                      | **必填** | **描述**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是          | 服务主体的客户端（应用程序）ID。        |
  | azure.adls2.oauth2_client_secret   | 是          | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls2.oauth2_client_endpoint | 是          | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

##### Google GCS

Hive 目录从 v3.0 开始支持 Google GCS。

如果选择 Google GCS 作为 Hive 群集的存储系统，请执行以下操作之一：

- 要选择基于 VM 的身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL

  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                              | **默认值** | **示例值** | **描述**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用绑定到您的 Compute Engine 的服务帐号。 |

- 要选择基于服务帐户的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                          | **默认值** | **示例值**                                        | **描述**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 创建服务帐户时生成的 JSON 文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 创建服务帐户时生成的 JSON 文件中的私钥 ID。 |
  | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 创建服务帐户时生成的 JSON 文件中的私钥。 |

- 要选择基于模拟的身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  - 使 VM 实例模拟服务帐户：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                              | **默认值** | **示例值** | **描述**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用绑定到您的 Compute Engine 的服务帐号。 |
    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 您想要模拟的服务帐户。            |

  - 使服务帐户（暂时命名为元服务帐户）模拟另一个服务帐户（暂时命名为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                          | **默认值** | **示例值**                                        | **描述**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 创建元服务帐户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 您想要模拟的数据服务帐户。       |

#### MetadataUpdateParams

关于 StarRocks 如何更新 Hive 缓存元数据的一组参数。此参数集是可选的。

StarRocks 默认实现了 [自动异步更新策略](#appendix-understand-metadata-automatic-asynchronous-update) 。

在大多数情况下，您可以忽略 `MetadataUpdateParams`，不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了开箱即用的性能。

但是，如果 Hive 中数据更新的频率较高，您可以调整这些参数，以进一步优化自动异步更新的性能。

> **注意**
>
> 在大多数情况下，如果 Hive 数据以 1 小时或更短的粒度更新，则认为数据更新频率较高。

| 参数                              | 必填 | 描述                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | 否       | 指定 StarRocks 是否缓存 Hive 表的元数据。有效值： `true` 和 `false`。默认值：`true`。该`true`值启用缓存，该值 `false` 禁用缓存。 |
| enable_remote_file_cache               | 否       | 指定 StarRocks 是否缓存 Hive 表或分区的底层数据文件的元数据。有效值： `true` 和 `false`。默认值：`true`。该`true`值启用缓存，该值 `false` 禁用缓存。 |
| metastore_cache_refresh_interval_sec   | 否       | StarRocks 异步更新自身缓存的 Hive 表或分区元数据的时间间隔。单位：秒。默认值： `7200`，即 2 小时。 |
| remote_file_cache_refresh_interval_sec | 否       | StarRocks 异步更新自身缓存的 Hive 表或分区底层数据文件的元数据的时间间隔。单位：秒。默认值： `60`。 |
| metastore_cache_ttl_sec                | 否       | StarRocks 自动丢弃自身缓存的 Hive 表或分区的元数据的时间间隔。单位：秒。默认值： `86400`，即 24 小时。 |
| remote_file_cache_ttl_sec              | 否       | StarRocks 自动丢弃自身缓存的 Hive 表或分区底层数据文件的元数据的时间间隔。单位：秒。默认值： `129600`，即 36 小时。 |
| enable_cache_list_names                | 否       | 指定 StarRocks 是否缓存 Hive 分区名称。有效值： `true` 和 `false`。默认值：`true`。该`true`值启用缓存，该值 `false` 禁用缓存。 |

### 示例

以下示例创建一个名为 `hive_catalog_hms` 或 `hive_catalog_glue` 的 Hive 目录，具体取决于您使用的元存储类型，用于查询 Hive 群集中的数据。

#### HDFS

如果您使用 HDFS 作为存储，运行以下命令：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### 基于实例配置文件的身份验证

- 如果您在 Hive 群集中使用 Hive 元存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 假定的基于角色的身份验证

- 如果您在 Hive 群集中使用 Hive 元存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### IAM 用户身份验证

- 如果您在 Hive 群集中使用 Hive 元存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hive 集群中使用 AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "false",
      "aws.glue.access_key" = "<iam_user_access_key>",
      "aws.glue.secret_key" = "<iam_user_secret_key>",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

#### 兼容 S3 的存储系统

以 MinIO 为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure 存储

##### Azure Blob 存储

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果选择 SAS 令牌身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- 如果选择托管服务标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- 如果选择托管标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果选择基于 VM 的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务帐户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果使 VM 实例模拟服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果使一个服务帐户模拟另一个服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 查看 Hive 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `hive_catalog_glue` 的 Hive 目录的创建语句：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## 切换到 Hive 目录和其中的数据库

您可以使用以下方法之一切换到 Hive 目录和其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 指定当前会话中的 Hive 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定活动数据库：

  ```SQL
  -- 切换到当前会话中指定的目录：
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到 Hive 目录和其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除 Hive 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `hive_catalog_glue` 的 Hive 目录：

```SQL
DROP Catalog hive_catalog_glue;
```

## 查看 Hive 表的架构

您可以使用以下语法之一查看 Hive 表的架构：

- 查看架构

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 从 CREATE 语句查看架构和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询 Hive 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Hive 群集中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到 Hive 目录和其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)。

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从 Hive 加载数据

假设您有一个名为 `olap_tbl` 的 OLAP 表，您可以像下面这样转换和加载数据：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## 授予对 Hive 表和视图的权限

您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 语句将 Hive 目录中所有表或视图的权限授予特定角色。

- 授予角色查询 Hive 目录中所有表的权限：

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- 授予角色查询 Hive 目录中所有视图的权限：

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

例如，使用以下命令创建名为 `hive_role_table` 的角色，切换到 Hive 目录 `hive_catalog`，然后授予该角色 `hive_role_table` 查询 Hive 目录中所有表和视图的权限：

```SQL
-- 创建名为 hive_role_table 的角色。
CREATE ROLE hive_role_table;

-- 切换到 Hive 目录 hive_catalog。
SET CATALOG hive_catalog;

-- 授予角色 hive_role_table 查询 Hive 目录 hive_catalog 中所有表的权限。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;

-- 授予角色 hive_role_table 查询 Hive 目录 hive_catalog 中所有视图的权限。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

## 创建 Hive 数据库

与 StarRocks 的内部目录类似，如果您拥有 Hive 目录的 CREATE DATABASE 权限，则可以使用 CREATE DATABASE 语句在该 Hive 目录下创建数据库。从 v3.2 开始支持此功能。

> **注意**
>
> 您可以使用 GRANT 和 REVOKE 授予和撤消权限。

切换到 Hive 目录，并使用以下语句在该目录中创建 Hive 数据库：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` 参数指定要在其中创建数据库的文件路径，该路径可以位于 HDFS 或云存储中。

- 使用 Hive 元存储作为 Hive 群集的元存储时，如果在创建数据库时未指定该参数，则该参数默认为 `<warehouse_location>/<database_name.db>`，Hive 元存储支持该参数。
- 当您使用 AWS Glue 作为 Hive 集群的元存储时，该参数没有默认值，因此您必须在创建数据库时指定该参数。

根据您使用的存储系统的不同，`prefix` 的值也会有所不同：

| 存储系统                                         | `Prefix` 价值                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS的                                                       | `hdfs`                                                       |
| 谷歌GCS                                                 | `gs`                                                         |
| Azure Blob 存储                                         | <ul><li>如果您的存储帐户允许通过 HTTP 访问，则 `prefix` 为 `wasb`。</li><li>如果您的存储帐户允许通过 HTTPS 访问，则 `prefix` 为 `wasbs`。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>如果您的存储帐户允许通过 HTTP 访问，则 `prefix` 为 `abfs`。</li><li>如果您的存储帐户允许通过 HTTPS 访问，则 `prefix` 为 `abfss`。</li></ul> |
| AWS S3 或其他与 S3 兼容的存储（例如，MinIO） | `s3`                                                         |

## 删除 Hive 数据库

与 StarRocks 的内部数据库类似，如果您拥有对 Hive 数据库的 [DROP](../../administration/privilege_item.md#database) 权限，则可以使用 [DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) 语句删除该 Hive 数据库。此功能从 v3.2 版本开始受支持。只能删除空数据库。

> **注意**
>
> 您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

删除 Hive 数据库时，数据库在 HDFS 集群或云存储中的文件路径不会随数据库一起删除。

[切换到 Hive 目录](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语句在该目录中删除 Hive 数据库：

```SQL
DROP DATABASE <database_name>
```

## 创建 Hive 表

与 StarRocks 的内部数据库类似，如果您对 Hive 数据库拥有 [CREATE TABLE](../../administration/privilege_item.md#database) 权限，则可以使用 [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 或 [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) 语句在该 Hive 数据库中创建托管表。此功能从 v3.2 版本开始受支持。

> **注意**
>
> 您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

[切换到 Hive 目录和其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语法在该数据库中创建 Hive 托管表。

### 语法

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### 参数

#### column_definition

`column_definition` 的语法如下：

```SQL
col_name col_type [COMMENT 'comment']
```

以下表描述了参数。

| 参数 | 描述                                                  |
| --------- | ------------------------------------------------------------ |
| col_name  | 列的名称。                                      |
| col_type  | 列的数据类型。支持以下数据类型：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR、[（长度）]ARRAY、MAP 和 STRUCT。不支持 LARGEINT、HLL 和 BITMAP 数据类型。 |

> **注意**
>
> 所有非分区列都必须使用 `NULL` 作为默认值。这意味着您必须在表创建语句中为每个非分区列指定 `DEFAULT "NULL"`。此外，分区列必须在非分区列之后定义，并且不能使用 `NULL` 作为默认值。

#### partition_desc

`partition_desc` 的语法如下：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

目前，StarRocks 仅支持身份转换，这意味着 StarRocks 为每个唯一的分区值创建一个分区。

> **注意**
>
> 分区列必须在非分区列之后定义。分区列支持除 FLOAT、DOUBLE、DECIMAL 和 DATETIME 之外的所有数据类型，并且不能使用 `NULL` 作为默认值。此外，在 `partition_desc` 中声明的分区列的顺序必须与在 `column_definition` 中定义的列的顺序一致。

#### PROPERTIES

您可以按照 `"key" = "value"` 的格式在 `PROPERTIES` 中指定表属性。

以下表描述了一些关键属性。

| **属性**      | **描述**                                              |
| ----------------- | ------------------------------------------------------------ |
| location          | 要在其中创建托管表的文件路径。使用 HMS 作为元存储时，无需指定 `location` 参数，因为 StarRocks 会在当前 Hive 目录的默认文件路径下创建表。当您使用 AWS Glue 作为元数据服务时：<ul><li>如果您为要在其中创建表的数据库指定了 `location` 参数，则无需为表指定 `location` 参数。因此，表默认为其所属数据库的文件路径。</li><li>如果您尚未为要在其中创建表的数据库指定 `location`，则必须为表指定 `location` 参数。</li></ul> |
| file_format       | 托管表的文件格式。仅支持 Parquet 格式。默认值： `parquet`。 |
| compression_codec | 用于托管表的压缩算法。支持的压缩算法包括 SNAPPY、GZIP、ZSTD 和 LZ4。默认值： `gzip`。 |

### 示例

1. 创建一个名为 `unpartition_tbl` 的非分区表。该表由两列 `id` 和 `score` 组成，如下所示：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. 创建名为 `partition_tbl_1` 的分区表。该表由三列 `action`、`id` 和 `dt` 组成，其中 `id` 和 `dt` 被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 查询名为 `partition_tbl_1` 的现有表，并根据查询结果创建名为 `partition_tbl_2` 的分区表。对于 `partition_tbl_2`，`id` 和 `dt` 被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * from partition_tbl_1;
   ```

## 将数据下沉到 Hive 表

与 StarRocks 的内部表类似，如果您对 Hive 表（可以是托管表或外部表）拥有 [INSERT](../../administration/privilege_item.md#table) 权限，则可以使用 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 语句将 StarRocks 表的数据下沉到该 Hive 表中（目前仅支持 Parquet 格式的 Hive 表）。此功能从 v3.2 版本开始受支持。默认情况下，将数据下沉到外部表处于禁用状态。要将数据下沉到外部表，必须将 [系统变量 `ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md) 设置为 `true`。

> **注意**
>
> 您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

[切换到 Hive 目录和其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后按照以下语法将 StarRocks 表的数据下沉到该数据库中的 Parquet 格式的 Hive 表中。

### 语法

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 如果要将数据下沉到指定的分区，请使用以下语法：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> 分区列不允许 `NULL` 值。因此，必须确保不会将空值加载到 Hive 表的分区列中。

### 参数

| 参数   | 描述                                                  |
| ----------- | ------------------------------------------------------------ |
| INTO        | 将 StarRocks 表的数据附加到 Hive 表中。 |
| OVERWRITE   | 将 Hive 表的现有数据覆盖为 StarRocks 表的数据。 |
| column_name | 要将数据加载到的目标列的名称。您可以指定一列或多列。如果指定多列，请用逗号（`,`）分隔它们。您只能指定 Hive 表中实际存在的列，并且您指定的目标列必须包含 Hive 表的分区列。无论目标列名称如何，您指定的目标列都会依次映射到 StarRocks 表的列中。如果未指定目标列，则数据将加载到 Hive 表的所有列中。如果 StarRocks 表的非分区列无法映射到 Hive 表的任何列，StarRocks 会将默认值 `NULL` 写入 Hive 表列。如果 INSERT 语句中包含的查询语句返回的列类型与目标列的数据类型不同，StarRocks 会对不匹配的列进行隐式转换。如果转换失败，将返回语法解析错误。 |
| expression  | 将值赋给目标列的表达式。    |
| DEFAULT     | 为目标列分配默认值。           |
| query       | 查询语句，其结果将被加载到 Hive 表中。它可以是 StarRocks 支持的任何 SQL 语句。 |
| PARTITION   | 要将数据加载到的分区。您必须在此属性中指定 Hive 表的所有分区列。在此属性中指定的分区列的顺序可以与在表创建语句中定义的分区列的顺序不同。如果指定此属性，则无法指定 `column_name` 属性。 |

### 示例

1. 将三个数据行插入到 `partition_tbl_1` 表中：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 将包含简单计算的 SELECT 查询的结果插入到 `partition_tbl_1` 表中：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. 将从 `partition_tbl_1` 表中读取数据的 SELECT 查询的结果插入到同一表中：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 将 SELECT 查询的结果插入到满足两个条件 `dt='2023-09-01'` 和 `id=1` 的分区的 `partition_tbl_2` 表中：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   或

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```
5.  用以下命令覆盖 `partition_tbl_1` 表中满足 `dt='2023-09-01'` 和 `id=1` 两个条件的分区的所有 `action` 列的值为 `close`：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   或

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## 删除 Hive 表

与 StarRocks 的内部表类似，如果您拥有对 Hive 表的 [DROP](../../administration/privilege_item.md#table) 权限，您可以使用 [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 语句删除该 Hive 表。从 v3.1 开始支持此功能。需要注意的是，目前 StarRocks 仅支持删除 Hive 的托管表。

> **注意**
>
> 您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 授予和撤消权限。

删除 Hive 表时，必须在 `DROP TABLE` 语句中指定 `FORCE` 关键字。操作完成后，将保留表的文件路径，但 HDFS 集群或云存储上的表数据会随表一起删除。执行此操作以删除 Hive 表时，请谨慎操作。

[切换到 Hive 目录和其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语句删除该数据库中的 Hive 表。

```SQL
DROP TABLE <table_name> FORCE
```

## 手动或自动更新元数据缓存

### 手动更新

StarRocks 默认缓存 Hive 的元数据，并以异步模式自动更新元数据，以提供更好的性能。此外，在对 Hive 表进行一些 Schema 更改或表更新后，您还可以使用 [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) 手动更新其元数据，从而确保 StarRocks 能够第一时间获取到最新的元数据，并生成合适的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

在以下情况下，您需要手动更新元数据：

- 更改现有分区中的数据文件，例如通过运行 `INSERT OVERWRITE ... PARTITION ...` 命令。
- 对 Hive 表进行架构更改。
- 使用 `DROP` 语句删除已有的 Hive 表，并新建一个与已删除的 Hive 表同名的 Hive 表。
- 在创建 Hive 目录时，已指定 `"enable_cache_list_names" = "true"`，并且想要查询刚刚在 Hive 群集上创建的新分区。

  > **注意**
  >
  > 从 v2.5.5 开始，StarRocks 提供了 Hive 元数据缓存周期性刷新功能。有关详细信息，请参阅本主题的以下“[定期刷新元数据缓存](#periodically-refresh-metadata-cache)”部分。开启该功能后，StarRocks 默认每 10 分钟刷新一次 Hive 元数据缓存。因此，在大多数情况下不需要手动更新。仅当希望在 Hive 群集上创建新分区后立即查询新分区时，才需要执行手动更新。

请注意，REFRESH EXTERNAL TABLE 仅刷新 FE 中缓存的表和分区。

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略允许 StarRocks 集群中的 FE 从 Hive 元存储中读取事件，例如添加列、删除分区和更新数据。StarRocks 可以根据这些事件自动更新 FE 中缓存的元数据。这意味着您无需手动更新 Hive 表的元数据。

此功能可能会对 HMS 造成很大的压力，使用此功能时请谨慎使用。我们建议您使用 [定期刷新元数据缓存](#periodically-refresh-metadata-cache)。

若要启用自动增量更新，请按照下列步骤操作：

#### 步骤 1：为 Hive 元存储配置事件侦听器

Hive 元存储 v2.x 和 v3.x 都支持配置事件侦听器。此步骤以 Hive 元存储 v3.1.2 的事件侦听器配置为例。将以下配置项添加到 **$HiveMetastore/conf/hive-site.xml** 文件，然后重启 Hive 元存储：

```XML
<property>
    <name>hive.metastore.event.db.notification.api.auth</name>
    <value>false</value>
</property>
<property>
    <name>hive.metastore.notifications.add.thrift.objects</name>
    <value>true</value>
</property>
<property>
    <name>hive.metastore.alter.notifications.basic</name>
    <value>false</value>
</property>
<property>
    <name>hive.metastore.dml.events</name>
    <value>true</value>
</property>
<property>
    <name>hive.metastore.transactional.event.listeners</name>
    <value>org.apache.hive.hcatalog.listener.DbNotificationListener</value>
</property>
<property>
    <name>hive.metastore.event.db.listener.timetolive</name>
    <value>172800s</value>
</property>
<property>
    <name>hive.metastore.server.max.message.size</name>
    <value>858993459</value>
</property>
```

您可以在 FE 日志文件中搜索 `event id`，以检查事件侦听器是否配置成功。如果配置失败，`event id` 的值将为 `0`。

#### 第 2 步：开启 StarRocks 自动增量更新

您可以为 StarRocks 集群中的单个 Hive 目录或所有 Hive 目录启用自动增量更新。

- 若要为单个 Hive 目录启用自动增量更新，请在创建 Hive 目录时，在 `PROPERTIES` 中将 `enable_hms_events_incremental_sync` 参数设置为 `true`，如下所示：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- 如需开启所有 Hive 目录的自动增量更新，请在每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中添加 `"enable_hms_events_incremental_sync" = "true"`，然后重启每个 FE 以使参数设置生效。

您也可以根据业务需要，在每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中调整以下参数，然后重启每个 FE 以使参数设置生效。

| 参数                         | 描述                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks 从 Hive 元存储读取事件的时间间隔。默认值： `5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks 一次可读取的最大事件数。默认值： `500`。 |
| enable_hms_parallel_process_evens | 指定 StarRocks 在读取事件时是否并行处理事件。有效值： `true` 和 `false`。默认值：`true`。该值`true`启用并行性，该值 `false` 禁用并行性。 |
| hms_process_events_parallel_num   | StarRocks 可并行处理的最大事件数。默认值： `4`。 |

## 定期刷新元数据缓存

从 v2.5.5 开始，StarRocks 可以定期刷新频繁访问的 Hive 目录的缓存元数据，以感知数据变化。您可以通过以下 [FE 参数](../../administration/FE_configuration.md) 配置 Hive 元数据缓存刷新：

| 配置项                                           | 默认                              | 描述                          |
| ------------------------------------------------------------ | ------------------------------------ | ------------------------------------ |
| enable_background_refresh_connector_metadata                 | `true` 在 v3.0 中<br />`false` 在 v2.5 中  | 是否开启定期 Hive 元数据缓存刷新。开启后，StarRocks 会轮询 Hive 集群的元存储（Hive Metastore 或 AWS Glue），并刷新频繁访问的 Hive 目录的缓存元数据，以感知数据变化。`true` 指示启用 Hive 元数据缓存刷新，并 `false` 指示禁用它。此项为[有限元动态参数](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令对其进行修改。|
| background_refresh_metadata_interval_millis                  | `600000` （10分钟）                | 两次连续 Hive 元数据缓存刷新之间的时间间隔。单位：毫秒。此项为[有限元动态参数](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令对其进行修改。|
| background_refresh_metadata_time_secs_since_last_access_secs | `86400` （24小时）                   | Hive 元数据缓存刷新任务的过期时间。对于已访问的 Hive 目录，如果超过指定时间未访问，StarRocks 将停止刷新其缓存的元数据。对于尚未访问的 Hive 目录，StarRocks 不会刷新其缓存的元数据。单位：秒。此项为[有限元动态参数](../../administration/FE_configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令对其进行修改。|

结合使用定期 Hive 元数据缓存刷新功能和元数据自动异步更新策略，可显著加快数据访问速度，减少外部数据源的读取负载，并提高查询性能。

## 附录：了解元数据自动异步更新

自动异步更新是 StarRocks 用于更新 Hive 目录下元数据的默认策略。

默认情况下（即 `enable_metastore_cache` 和 `enable_remote_file_cache` 参数都设置为 `true`时），如果查询命中 Hive 表的某个分区，StarRocks 会自动缓存该分区的元数据和该分区底层数据文件的元数据。缓存的元数据使用延迟更新策略进行更新。

例如，有一个名为 `table2` 的 Hive 表，该表有四个分区：`p1`、 `p2`、 `p3` 和 `p4`。查询命中 `p1`，StarRocks 会缓存 `p1` 的元数据和底层数据文件的元数据。假设更新和丢弃缓存元数据的默认时间间隔如下：

- 异步更新缓存元数据 `metastore_cache_refresh_interval_sec` 的时间间隔为 2 小时。
- 异步更新基础数据文件的缓存元数据 `remote_file_cache_refresh_interval_sec` 的时间间隔为 60 秒。
- 自动丢弃缓存元数据 `metastore_cache_ttl_sec` 的时间间隔为 24 小时。
- 自动丢弃基础数据文件的缓存元数据 `remote_file_cache_ttl_sec` 的时间间隔为 36 小时。
下图显示了时间轴上的时间间隔，以便更容易理解。

![更新和丢弃缓存的元数据时间线](../../assets/catalog_timeline.png)

然后，StarRocks 根据以下规则更新或丢弃元数据：

- 如果另一个查询再次命中`p1`，并且距离上次更新的当前时间小于 60 秒，则 StarRocks 不会更新`p1`的缓存元数据，也不会更新`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中`p1`，并且距离上次更新的当前时间超过 60 秒，StarRocks 会更新`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中`p1`，并且距离上次更新的当前时间超过 2 小时，StarRocks 会更新`p1`的缓存元数据。
- 如果在上次更新后`p1`在 24 小时内未被访问，StarRocks 会丢弃`p1`的缓存元数据。元数据将在下一次查询时缓存。
- 如果在上次更新后`p1`在 36 小时内未被访问，StarRocks 会丢弃`p1`的底层数据文件的缓存元数据。元数据将在下一次查询时缓存。