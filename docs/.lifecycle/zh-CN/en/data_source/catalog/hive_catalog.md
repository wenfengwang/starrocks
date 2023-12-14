---
displayed_sidebar: "Chinese"
---

# Hive 目录

Hive 目录是 StarRocks 从 v2.4 版本开始支持的一种外部目录。在 Hive 目录中，您可以：

- 直接查询存储在 Hive 中的数据，无需手动创建表。
- 使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 或异步物化视图（从 v2.5 开始支持）处理存储在 Hive 中的数据，并将数据加载到 StarRocks 中。
- 在 StarRocks 上执行操作，创建或删除 Hive 数据库和表，或使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 将 StarRocks 表的数据下沉到 Parquet 格式的 Hive 表（从 v3.2 版本开始支持此功能）。

为了确保在 Hive 集群上成功执行 SQL 工作负载，您的 StarRocks 集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或像 AWS S3、Microsoft Azure 存储、Google GCS 或其他兼容S3的对象存储等

- Hive metastore 或 AWS Glue 等元数据存储

  > **注意**
  >
  > 如果选择 AWS S3 作为存储，可以使用 HMS 或 AWS Glue 作为元数据存储。如果选择任何其他存储系统，则只能使用 HMS 作为元数据存储。

## 使用说明

- StarRocks 支持的 Hive 文件格式包括 Parquet、ORC、CSV、Avro、RCFile 和 SequenceFile：

  - Parquet 文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP 和 NO_COMPRESSION。从 v3.1.5 版本开始，Parquet 文件还支持 LZO 压缩格式。
  - ORC 文件支持以下压缩格式：ZLIB、SNAPPY、LZO、LZ4、ZSTD 和 NO_COMPRESSION。
  - CSV 文件从 v3.1.5 版本开始支持 LZO 压缩格式。

- StarRocks 不支持的 Hive 数据类型包括 INTERVAL、BINARY 和 UNION。此外，对于 CSV 格式的 Hive 表，StarRocks 还不支持 MAP 和 STRUCT 数据类型。
- 只能使用 Hive 目录来查询数据。不能使用 Hive 目录在 Hive 集群中删除、清除或插入数据。

## 集成准备

在创建 Hive 目录之前，请确保您的 StarRocks 集群可以与 Hive 集群的存储系统和元数据存储进行集成。

### AWS IAM

如果您的 Hive 集群使用 AWS S3 作为存储或 AWS Glue 作为元数据存储，请选择适合的身份验证方法，并做好必要准备以确保您的 StarRocks 集群可以访问相关的 AWS 云资源。

建议使用以下身份验证方法：

- 实例配置文件
- 假定的角色
- IAM 用户

在上述三种身份验证方法中，实例配置文件是最常用的。

有关更多信息，请参见 [AWS IAM 身份验证准备](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果选择 HDFS 作为存储，请按照以下步骤配置您的 StarRocks 集群：

- （可选）设置用于访问 HDFS 集群和 Hive metastore 的用户名。默认情况下，StarRocks 使用 FE 和 BE 进程的用户名来访问您的 HDFS 集群和 Hive metastore。您也可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<用户名>"` 来设置用户名。设置这些文件后，重新启动每个 FE 和每个 BE 使参数设置生效。每个 StarRocks 集群只能设置一个用户名。
- 当查询 Hive 数据时，您的 StarRocks 集群的 FE 和 BE 使用 HDFS 客户端访问您的 HDFS 集群。大多数情况下，您无需配置您的 StarRocks 集群即可实现该目的，因为 StarRocks 使用默认配置启动 HDFS 客户端。您只需要在以下情况下配置您的 StarRocks 集群：

  - 您的 HDFS 集群启用了高可用 (HA)：将您的 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。
  - 您的 HDFS 集群启用了 View File System (ViewFs)：将您的 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。

> **注意**
>
> 如果在发送查询时返回了未知主机的错误，则必须将您的 HDFS 集群节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径。

### Kerberos 身份验证

如果为您的 HDFS 集群或 Hive metastore 启用了 Kerberos 身份验证，请按以下步骤配置您的 StarRocks 集群：

- 在每个 FE 和每个 BE 上运行 `kinit -kt keytab路径 principal` 命令，从密钥分发中心 (KDC) 获取票据授权票据 (TGT)。要运行此命令，您必须具有访问您的 HDFS 集群和 Hive metastore 的权限。请注意，使用该命令访问 KDC 是有时间限制的。因此，您需要使用 cron 定期运行此命令。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改路径。

## 创建 Hive 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <目录名称>
[COMMENT <描述>]
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

#### 目录名称

Hive 目录的名称。命名约定如下：

- 名称可包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### 描述

Hive 目录的描述。此参数为可选参数。

#### 类型

数据源的类型。将值设置为 `hive`。

#### GeneralParams

一组通用参数。

以下表格描述了您可在 `GeneralParams` 中配置的参数。

| 参数                   | 是否必须 | 描述                                                         |
| ---------------------- | -------- | ------------------------------------------------------------ |
| enable_recursive_listing | 否       | 指定 StarRocks 是否从表及其分区以及表和其分区的物理位置中的子目录中读取数据。有效值：`true` 和 `false`。默认值：`false`。值 `true` 指定递归列表子目录，值 `false` 指定忽略子目录。 |

#### MetastoreParams

关于 StarRocks 与数据源元数据存储集成的一组参数。

##### Hive metastore

如果选择 Hive metastore 作为数据源的元数据存储，请按以下步骤配置 `MetastoreParams`：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 查询 Hive 数据之前，您必须将您的 Hive metastore 节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径。否则，当启动查询时，StarRocks 可能无法访问您的 Hive metastore。

以下表格描述了您需要在 `MetastoreParams` 中配置的参数。

| 参数                      | 是否必须 | 描述                                                         |
| ------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | 是 | 您用于 Hive 集群的元数据存储的类型。将值设置为 `hive`。 |
| hive.metastore.uris | 是 | 您的 Hive metastore 的 URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />如果为您的 Hive metastore 启用了高可用 (HA)，则可以指定多个元数据存储 URI，并使用逗号 (`,`) 分隔它们，例如，"`thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>`"。 |

##### AWS Glue

如果选择 AWS Glue 作为数据源的元数据存储，仅当您选择 AWS S3 作为存储时才受支持，请执行以下操作之一：

- 要选择基于实例配置文件的验证方法，请按以下步骤配置 `MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择基于角色的身份验证方法，请将`MetastoreParams`配置如下：

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "true",
"aws.glue.iam_role_arn" = "<iam_role_arn>",
"aws.glue.region" = "<aws_glue_region>"
```

- 要选择基于IAM用户的身份验证方法，请将`MetastoreParams`配置如下：

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "false",
"aws.glue.access_key" = "<iam_user_access_key>",
"aws.glue.secret_key" = "<iam_user_secret_key>",
"aws.glue.region" = "<aws_s3_region>"
```

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数                           | 必需     | 描述                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | 是       | 您用于Hive集群的metastore类型。将值设置为`glue`。               |
| aws.glue.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证。有效值：`true`和`false`。默认值：`false`。     |
| aws.glue.iam_role_arn         | 否       | 在AWS Glue数据目录上具有权限的IAM角色的ARN。如果您使用假定基于角色的身份验证方法访问AWS Glue，则必须指定此参数。      |
| aws.glue.region               | 是       | 您的AWS Glue数据目录所在的地区。示例：`us-west-1`。                   |
| aws.glue.access_key           | 否       | 您的AWS IAM用户的访问密钥。如果您使用IAM用户-based的身份验证方法访问AWS Glue，则必须指定此参数。        |
| aws.glue.secret_key           | 否       | 您的AWS IAM用户的秘密密钥。如果您使用IAM用户-based的身份验证方法访问AWS Glue，则必须指定此参数。        |

有关如何选择访问AWS Glue的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS Glue的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

关于StarRocks如何与您的存储系统集成的一组参数。此参数集是可选的。

如果您使用HDFS作为存储，您不需要配置`StorageCredentialParams`。

如果您使用AWS S3、其他兼容S3的存储系统、Microsoft Azure Storage或Google GCS作为存储，您必须配置`StorageCredentialParams`。

##### AWS S3

如果您选择AWS S3作为Hive集群的存储，请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请将`StorageCredentialParams`配置如下：

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 要选择假定基于角色的身份验证方法，请将`StorageCredentialParams`配置如下：

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<iam_role_arn>",
"aws.s3.region" = "<aws_s3_region>"
```

- 要选择基于IAM用户的身份验证方法，请将`StorageCredentialParams`配置如下：

```SQL
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                           | 必需     | 描述                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和假定基于角色的身份验证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.s3.iam_role_arn         | 否       | 具有对您的AWS S3存储桶特权的IAM角色的ARN。如果您使用假定基于角色的身份验证方法访问AWS S3，则必须指定此参数。   |
| aws.s3.region               | 是       | 您的AWS S3存储桶所在的地区。示例：`us-west-1`。                        |
| aws.s3.access_key           | 否       | 您的IAM用户的访问密钥。如果您使用IAM用户-based的身份验证方法访问AWS S3，则必须指定此参数。           |
| aws.s3.secret_key           | 否       | 您的IAM用户的秘密密钥。如果您使用IAM用户-based的身份验证方法访问AWS S3，则必须指定此参数。            |

有关如何选择访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS S3的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容S3的存储系统

Hive目录从v2.5开始支持兼容S3的存储系统。

如果您选择兼容S3的存储系统（例如MinIO）作为Hive集群的存储，请配置`StorageCredentialParams`如下以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                             | 必需     | 描述                                                         |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | 是       | 指定是否启用SSL连接。<br />有效值：`true`和`false`。默认值：`true`。  |
| aws.s3.enable_path_style_access  | 是       | 指定是否启用路径样式访问。<br />有效值：`true`和`false`。默认值：`false`。对于MinIO，您必须将值设置为`true`。<br />路径样式URL采用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈州）区域创建名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且要访问该存储桶中名为`alice.jpg`的对象，您可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | 是       | 用于连接到兼容S3存储系统而不是AWS S3的终点。                      |
| aws.s3.access_key                | 是       | 您的IAM用户的访问密钥。                                           |
| aws.s3.secret_key                | 是       | 您的IAM用户的秘密密钥。                                           |

##### Microsoft Azure Storage

Hive目录从v3.0开始支持Microsoft Azure Storage。

###### Azure Blob Storage

如果您选择Blob Storage作为Hive集群的存储，请执行以下操作之一：

- 要选择共享密钥认证方法，请将`StorageCredentialParams`配置如下：

```SQL
"azure.blob.storage_account" = "<blob_storage_account_name>",
"azure.blob.shared_key" = "<blob_storage_account_shared_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                          | 必需      | 描述                                                         |
| -------------------------  | -------- | ------------------------------------------------------------ |
| azure.blob.storage_account | 是       | 您的Blob Storage帐户的用户名。                                   |
| azure.blob.shared_key       | 是       | 您的Blob Storage帐户的共享密钥。                                 |

- 要选择SAS令牌认证方法，请将`StorageCredentialParams`配置如下：

```SQL
"azure.blob.account_name" = "<blob_storage_account_name>",
"azure.blob.container_name" = "<blob_container_name>",
"azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                      | 必需      | 描述                                                         |
| ----------------------- | -------- | ------------------------------------------------------------ |
| azure.blob.account_name | 是       | 您的Blob Storage帐户的用户名。                                   |
| azure.blob.container_name | 是      | 存储数据的blob容器的名称。                                      |
| azure.blob.sas_token    | 是       | 用于访问Blob Storage帐户的SAS令牌。                             |

###### Azure Data Lake Storage Gen1

如果您选择Data Lake Storage Gen1作为Hive集群的存储，请执行以下操作之一：

- 要选择托管服务标识身份验证方法，请将`StorageCredentialParams`配置如下：

```SQL
"azure.adls1.use_managed_service_identity" = "true"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                            | **必需** | **说明**                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是          | 指定是否启用托管服务标识的认证方法。将值设置为 `true`。 |

- 要选择服务主体认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                 | **必需** | **说明**                                                     |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是          | 服务主体的客户（应用程序）ID。        |
  | azure.adls1.oauth2_credential | 是         | 创建的新客户（应用程序）密钥的值。    |
  | azure.adls1.oauth2_endpoint   | 是          | 服务主体或应用程序的OAuth 2.0令牌终结点(v1)。 |

###### Azure Data Lake Storage Gen2

如果您选择Data Lake Storage Gen2作为Hive集群的存储方式，请执行以下操作之一:

- 要选择托管标识认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                           | **必需** | **说明**                                                     |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是          | 指定是否启用托管标识认证方法。将值设置为`true`。                   |
  | azure.adls2.oauth2_tenant_id            | 是          | 您要访问其数据的租户的ID。        |
  | azure.adls2.oauth2_client_id            | 是          | 托管标识的客户（应用程序）ID。        |

- 要选择共享密钥认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**               | **必需** | **说明**                                                     |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是         | 您的Data Lake Storage Gen2存储账户的用户名。       |
  | azure.adls2.shared_key      | 是         | 您的Data Lake Storage Gen2存储账户的共享密钥。       |

- 要选择服务主体认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下表格描述了您需要配置`StorageCredentialParams`中的参数。

  | **参数**                      | **必需** | **说明**                                                     |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是          | 服务主体的客户（应用程序）ID。        |
  | azure.adls2.oauth2_client_secret   | 是          | 创建的新客户（应用程序）密钥的值。    |
  | azure.adls2.oauth2_client_endpoint | 是          | 服务主体或应用程序的OAuth 2.0令牌终结点(v1)。 |

##### Google GCS

Hive目录从v3.0版本开始支持Google GCS。

如果您选择Google GCS作为Hive集群的存储方式，请执行以下操作之一:

- 要选择基于VM的认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                              | **默认值** | **值示例** | **说明**                                                     |
  | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false         | true          | 指定是否直接使用绑定到您的计算引擎的服务帐户。                     |

- 要选择基于服务帐户的认证方法，请将`StorageCredentialParams`配置如下:

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                          | **默认值** | **值示例**                                               | **说明**                                                     |
  | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)"  | JSON文件中在创建服务帐户时生成的电子邮件地址。       |
  | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                 | JSON文件中在创建服务帐户时生成的私钥ID。       |
  | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"   | JSON文件中在创建服务帐户时生成的私钥。       |

- 要选择基于模拟的认证方法，请将`StorageCredentialParams`配置如下:

  - 使VM实例模拟服务帐户:

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                              | **默认值** | **值示例** | **说明**                                                     |
    | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false         | true          | 指定是否直接使用绑定到您的计算引擎的服务帐户。                     |
    | gcp.gcs.impersonation_service_account      | ""            | "hello"       | 您要模拟的服务帐户。                                       |

  - 使一个服务帐户（临时命名为元服务帐户）模拟另一个服务帐户（临时命名为数据服务帐户）:

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                          | **默认值** | **值示例**                                               | **说明**                                                     |
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)"  | JSON文件中在创建元服务帐户时生成的电子邮件地址。       |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                 | JSON文件中在创建元服务帐户时生成的私钥ID。       |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"   | JSON文件中在创建元服务帐户时生成的私钥。       |
    | gcp.gcs.impersonation_service_account  | ""            | "hello"        | 您要模拟的数据服务帐户。                                     |

#### MetadataUpdateParams

有关StarRocks如何更新Hive缓存元数据的一组参数。此参数集是可选的。

StarRocks默认实现了[自动异步更新策略](#附录-了解元数据自动异步更新)。

在大多数情况下，您可以忽略`MetadataUpdateParams`，不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了开箱即用的性能。

然而，如果Hive中数据更新的频率很高，您可以调整这些参数以进一步优化自动异步更新的性能。

> **注意**
>
> 在大多数情况下，如果您的Hive数据以1小时或更短的时间间隔进行更新，则数据更新频率被认为是高的。

| 参数                                  | 必填     | 描述                                                         |
|---------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                | 否       | 指定StarRocks是否缓存Hive表的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| enable_remote_file_cache              | 否       | 指定StarRocks是否缓存Hive表或分区的底层数据文件的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| metastore_cache_refresh_interval_sec  | 否       | StarRocks异步更新自身中缓存的Hive表或分区的元数据的时间间隔。单位：秒。默认值：`7200`，即2小时。 |
| remote_file_cache_refresh_interval_sec| 否       | StarRocks异步更新自身中缓存的Hive表或分区的底层数据文件的元数据的时间间隔。单位：秒。默认值：`60`。 |
| metastore_cache_ttl_sec               | 否       | StarRocks自动丢弃自身缓存的Hive表或分区的元数据的时间间隔。单位：秒。默认值：`86400`，即24小时。 |
| remote_file_cache_ttl_sec             | 否       | StarRocks自动丢弃自身缓存的Hive表或分区的底层数据文件的元数据的时间间隔。单位：秒。默认值：`129600`，即36小时。 |
| enable_cache_list_names               | 否       | 指定StarRocks是否缓存Hive分区名称。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |

### 示例

以下示例根据您使用的Hive元数据存储类型创建名为`hive_catalog_hms`或`hive_catalog_glue`的Hive目录，以从Hive集群中查询数据。

#### HDFS

如果您将HDFS用作存储，运行如下命令：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### 基于实例配置文件的身份验证

- 如果您在Hive集群中使用Hive元数据存储，在运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，运行如下命令：

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

##### 基于假定角色的身份验证

- 如果您在Hive集群中使用Hive元数据存储，运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，运行如下命令：

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

##### 基于IAM用户的身份验证

- 如果您在Hive集群中使用Hive元数据存储，运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Hive集群中使用AWS Glue，运行如下命令：

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

#### 兼容S3存储系统

以MinIO为例，运行如下命令：

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### Microsoft Azure存储

##### Azure Blob 存储

- 如果选择共享密钥身份验证方法，运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果选择SAS令牌身份验证方法，运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure 数据湖存储 Gen1

- 如果选择托管服务标识的身份验证方法，运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务帐号的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果使 VM 实例模拟服务帐号，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果使服务帐号模拟另一个服务帐号，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 查看 Hive 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 来查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您也可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 来查询外部目录的创建语句。以下示例查询名为 `hive_catalog_glue` 的 Hive 目录的创建语句：

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## 切换到 Hive 目录及其中的数据库

您可以使用以下方法之一来切换到 Hive 目录及其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 来指定当前会话中的 Hive 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 来指定活动数据库：

  ```SQL
  -- 切换到当前会话中指定的目录：
  SET CATALOG <catalog_name>
  -- 在当前会话中指定活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 来切换到 Hive 目录及其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除 Hive 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 来删除外部目录。

以下示例删除名为 `hive_catalog_glue` 的 Hive 目录：

```SQL
DROP Catalog hive_catalog_glue;
```

## 查看 Hive 表的模式

您可以使用以下语法之一来查看 Hive 表的模式：

- 查看模式

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 查看创建语句中的模式和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询 Hive 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 来查看 Hive 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到 Hive 目录及其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)。

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从 Hive 加载数据

假设您有一个名为 `olap_tbl` 的 OLAP 表，您可以像下面这样转换和加载数据：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table
```

## 授予 Hive 表和视图的权限

您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 语句将特定角色对 Hive 目录中所有表或视图的权限授予给指定角色。

- 授予角色查询 Hive 目录中所有表的权限：

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- 授予角色查询 Hive 目录中所有视图的权限：

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```

例如，使用以下命令来创建名为 `hive_role_table` 的角色，切换到 Hive 目录 `hive_catalog`，然后授予角色 `hive_role_table` 查询 Hive 目录 `hive_catalog` 中所有表和视图的权限：

```SQL
-- 创建名为 hive_role_table 的角色。
CREATE ROLE hive_role_table;

-- 切换到 Hive 目录 hive_catalog。
SET CATALOG hive_catalog;

-- 授予角色 hive_role_table 查询 Hive 目录 hive_catalog 中所有表的权限。
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;
```
-- 授予角色hive_role_table在Hive目录hive_catalog中查询所有视图的权限。
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;


## 创建Hive数据库

类似于StarRocks的内部目录，如果您在Hive目录上具有[CREATE DATABASE](../../administration/privilege_item.md#catalog)权限，则可以使用[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)语句在该Hive目录中创建数据库。此功能从v3.2版本开始支持。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

[切换到Hive目录](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语句在该目录中创建Hive数据库：

```sql
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location`参数指定要创建数据库的文件路径，可以是HDFS或云存储。

- 当您使用Hive元存储作为Hive集群的元存储时，默认情况下`location`参数为`<warehouse_location>/<database_name.db>`，如果在数据库创建时未指定该参数，则Hive元存储支持此设置。
- 当您使用AWS Glue作为Hive集群的元存储时，`location`参数没有默认值，因此您必须在数据库创建时指定该参数。

`prefix`基于您使用的存储系统而异：

| 存储系统                                         | `prefix`值                                       |
| ------------------------------------------ | ------------------------------------------------ |
| HDFS                                       | `hdfs`                                           |
| Google GCS                                 | `gs`                                             |
| Azure Blob Storage                         | <ul><li>如果您的存储账户允许通过HTTP访问，则`prefix`为`wasb`。</li><li>如果您的存储账户允许通过HTTPS访问，则`prefix`为`wasbs`。</li></ul> |
| Azure Data Lake Storage Gen1               | `adl`                                            |
| Azure Data Lake Storage Gen2               | <ul><li>如果您的存储账户允许通过HTTP访问，则`prefix`为`abfs`。</li><li>如果您的存储账户允许通过HTTPS访问，则`prefix`为`abfss`。</li></ul> |
| AWS S3或其他S3兼容存储（例如MinIO） | `s3`                                             |

## 删除Hive数据库

类似于StarRocks的内部数据库，如果您对Hive数据库具有[DROP](../../administration/privilege_item.md#database)权限，则可以使用[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)语句删除该Hive数据库。此功能从v3.2版本开始支持。您只能删除空数据库。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

删除Hive数据库时，数据库在HDFS集群或云存储上的文件路径不会随数据库一起删除。

[切换到Hive目录](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语句在该目录中删除Hive数据库：

```sql
DROP DATABASE <database_name>
```

## 创建Hive表

类似于StarRocks的内部数据库，如果您对Hive数据库具有[CREATE TABLE](../../administration/privilege_item.md#database)权限，则可以使用[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)或[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)语句在该Hive数据库中创建托管表。此功能从v3.2版本开始支持。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

[切换到Hive目录和其中的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语法在该数据库中创建Hive托管表。

### 语法

```sql
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### 参数

#### column_definition

`column_definition`的语法如下：

```sql
col_name col_type [COMMENT 'comment']
```

下表描述了参数。

| 参数       | 描述                         |
| ---------- | ---------------------------- |
| col_name   | 列的名称。                   |
| col_type   | 列的数据类型。支持以下数据类型：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP和STRUCT。不支持LARGEINT、HLL和BITMAP数据类型。 |

> **注意**
>
> 所有非分区列的默认值必须使用`NULL`。这意味着您必须在表创建语句中为每个非分区列指定`DEFAULT "NULL"`。此外，分区列必须在非分区列后定义，并且不能使用`NULL`作为默认值。

#### partition_desc

`partition_desc`的语法如下：

```sql
PARTITION BY (par_col1[, par_col2...])
```

当前，StarRocks仅支持标识转换，这意味着StarRocks为每个唯一的分区值创建一个分区。

> **注意**
>
> 分区列必须在非分区列后定义。分区列支持除FLOAT、DOUBLE、DECIMAL和DATETIME之外的所有数据类型，并且不能使用`NULL`作为默认值。此外，`partition_desc`中声明的分区列的顺序必须与`column_definition`中定义的列的顺序一致。

#### PROPERTIES

您可以在`properties`中使用`"key" = "value"`格式指定表属性。

下表描述了一些关键属性。

| **属性**            | **描述**                                     |
| ------------------- | -------------------------------------------- |
| location            | 要创建托管表的文件路径。当使用HMS作为元存储时，您无需指定`location`参数，因为StarRocks将在当前Hive目录的默认文件路径创建表。当您使用AWS Glue作为元数据服务时：<ul><li>如果您为要创建表的数据库指定了`location`参数，则无需为表指定`location`参数。因此，表默认为属于的数据库的文件路径。</li><li>如果您未为要创建表的数据库指定`location`，则必须为表指定`location`参数。</li></ul> |
| file_format         | 托管表的文件格式。仅支持Parquet格式。默认值：`parquet`。 |
| compression_codec   | 用于托管表的压缩算法。支持的压缩算法是SNAPPY、GZIP、ZSTD和LZ4。默认值：`gzip`。 |

### 示例

1. 创建名为`unpartition_tbl`的非分区表。表包含`id`和`score`两列，如下所示：

   ```sql
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. 创建名为`partition_tbl_1`的分区表。表包含`action`、`id`和`dt`三列，其中`id`和`dt`被定义为分区列，如下所示：

   ```sql
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 查询名为`partition_tbl_1`的现有表，并基于`partition_tbl_1`的查询结果创建名为`partition_tbl_2`的分区表。对于`partition_tbl_2`，`id`和`dt`被定义为分区列，如下所示：

   ```sql
   CREATE TABLE partition_tbl_2
   PARTITION BY (k1, k2)
   AS SELECT * from partition_tbl_1;
   ```

## 将数据下沉到Hive表

与StarRocks的内部表类似，如果您在Hive表（可以是托管表或外部表）上具有[INSERT](../../administration/privilege_item.md#table)权限，则可以使用[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)语句将StarRocks表的数据导入到该Hive表中（目前仅支持Parquet格式的Hive表）。此功能从v3.2版本开始提供。默认情况下，禁用将数据导入到外部表。 若要将数据导入到外部表中，必须将[系统变量 `ENABLE_WRITE_HIVE_EXTERNAL_TABLE`](../../reference/System_variable.md)设置为`true`。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

[切换到Hive目录及其内部的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语法将StarRocks表的数据导入到该数据库中的Parquet格式Hive表中。

### 语法

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 如果要将数据导入指定的分区，请使用以下语法：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> 分区列不允许有`NULL`值。因此，您必须确保没有空值加载到Hive表的分区列中。

### 参数

| 参数          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
 | INTO         | 将StarRocks表的数据附加到Hive表中。                        |
 | OVERWRITE    | 用StarRocks表的数据覆盖Hive表中的现有数据。                |
 | column_name  | 您要加载数据的目标列的名称。您可以指定一个或多个列。如果指定多个列，请用逗号(,)分隔。您只能指定实际存在于Hive表中的列，且您指定的目标列必须包括Hive表的分区列。您指定的目标列将按顺序一对一地映射到StarRocks表的列，而不管目标列的名称是什么。如果未指定目标列，数据将加载到Hive表的所有列中。如果StarRocks表的非分区列无法映射到Hive表的任何列，StarRocks将向Hive表列中写入默认值`NULL`。如果INSERT语句包含一个查询语句，其返回的列类型与目标列的数据类型不匹配，则StarRocks会对不匹配的列进行隐式转换。如果转换失败，将返回语法解析错误。 |
 | expression   | 分配值给目标列的表达式。                                   |
 | DEFAULT      | 为目标列分配默认值。                                        |
 | query        | 查询语句的结果将加载到Hive表中。它可以是StarRocks支持的任何SQL语句。         |
 | PARTITION    | 您要加载数据的分区。您必须在此属性中指定Hive表的所有分区列。您在此属性中指定的分区列可以与表创建语句中定义的分区列的顺序不同。如果指定了此属性，则不能指定`column_name`属性。               |

### 示例

1. 将三行数据插入`partition_tbl_1`表中：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("购买", 1, "2023-09-01"),
       ("出售", 2, "2023-09-02"),
       ("购买", 3, "2023-09-03");
   ```

2. 将SELECT查询的结果（其中包含简单计算）插入`partition_tbl_1`表中：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, '购买', '2023-09-03';
   ```

3. 将从`partition_tbl_1`表中读取数据的SELECT查询结果插入到同一表中：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT '购买', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 将SELECT查询的结果插入到满足`dt='2023-09-01'`和`id=1`两个条件的分区中的`partition_tbl_2`表中：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT '订单', 1, '2023-09-01';
   ```

   或

   ```SQL
   INSERT INTO partition_tbl_2 PARTITION(dt='2023-09-01',id=1) SELECT '订单';
   ```

5. 用`close`覆盖`partition_tbl_1`表中满足`dt='2023-09-01'`和`id=1`两个条件的分区中的所有`action`列值：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   或

   ```SQL
   INSERT OVERWRITE partition_tbl_1 PARTITION(dt='2023-09-01',id=1) SELECT 'close';
   ```

## 删除Hive表

类似于StarRocks的内部表，如果您在Hive表上具有[DROP](../../administration/privilege_item.md#table)权限，则可以使用[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)语句删除该Hive表。此功能从v3.1版本开始提供。请注意，目前StarRocks仅支持删除Hive的托管表。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

删除Hive表时，必须在DROP TABLE语句中指定`FORCE`关键字。操作完成后，表的文件路径保留不变，但是删除表的同时，存储在HDFS集群或云存储中的表数据也全部被删除。在执行此操作删除Hive表时需要谨慎。

[切换到Hive目录及其内部的数据库](#switch-to-a-hive-catalog-and-a-database-in-it)，然后使用以下语句在该数据库中删除Hive表。

```SQL
DROP TABLE <table_name> FORCE
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks缓存Hive的元数据并自动以异步模式更新元数据以提供更好的性能。此外，在对Hive表进行某些模式更改或表更新后，您还可以使用[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)来手动更新其元数据，从而确保StarRocks能够在最早的机会获取最新的元数据并生成适当的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

在以下情况下需要手动更新元数据：

- 对现有分区中的数据文件进行更改，例如，运行`INSERT OVERWRITE ... PARTITION ...`命令。
- 对Hive表进行模式更改。
- 使用DROP语句删除现有的Hive表，并创建与已删除的Hive表同名的新Hive表。

  如果您在Hive目录的创建中指定了`"enable_cache_list_names" = "true"`，并且要查询您在Hive集群上刚创建的新分区。

  > **注意**
  >
  > 从v2.5.5版本开始，StarRocks提供了周期性的Hive元数据缓存刷新功能。有关更多信息，请参见以下主题中的“[定期刷新元数据缓存](#定期刷新元数据缓存)”部分。启用此功能后，默认情况下，StarRocks每10分钟刷新Hive元数据缓存。因此，在大多数情况下不需要手动更新。仅当希望在Hive集群上创建新分区后立即查询新分区时才需要手动更新。

请注意，REFRESH EXTERNAL TABLE仅刷新FE中缓存的表和分区。

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略使StarRocks集群中的FE能够读取从您的Hive元存储中提取的事件，例如添加列、删除分区和更新数据。StarRocks可以根据这些事件自动更新FE中缓存的元数据。这意味着您无需手动更新Hive表的元数据。

此功能可能会给HMS带来重大压力，因此在使用此功能时要小心。我们建议您使用[定期刷新元数据缓存](#定期刷新元数据缓存)。

要启用自动增量更新，请按照以下步骤操作：
#### 步骤 1：配置 Hive metastore 的事件监听器

Hive metastore v2.x 和 v3.x 都支持配置事件监听器。本步骤以配置 Hive metastore v3.1.2 中使用的事件监听器配置作为示例。将以下配置项目添加到 **$HiveMetastore/conf/hive-site.xml** 文件中，然后重新启动 Hive metastore：

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

您可以在 FE 日志文件中搜索 `event id`，以检查事件监听器是否成功配置。如果配置失败，`event id` 的值将为 `0`。

#### 步骤 2：启用 StarRocks 的自动增量更新

您可以为单个 Hive 目录或您 StarRocks 集群中的所有 Hive 目录启用自动增量更新。

- 若要为单个 Hive 目录启用自动增量更新，在创建 Hive 目录时，将 `enable_hms_events_incremental_sync` 参数设置为 `true`，如下所示：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- 若要为所有 Hive 目录启用自动增量更新，将 `"enable_hms_events_incremental_sync" = "true"` 添加到每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中，然后重新启动每个 FE 以使参数设置生效。

根据业务需求，您还可以调整每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中的以下参数，然后重新启动每个 FE 以使参数设置生效。

| 参数                              | 描述                                                    |
| --------------------------------- | ------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks 从 Hive metastore 读取事件的时间间隔。默认值：`5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks 可以一次读取的最大事件数。默认值：`500`。 |
| enable_hms_parallel_process_evens | 指定 StarRocks 是否在读取事件时并行处理事件。有效值：`true` 和 `false`。默认值：`true`。值 `true` 表示启用并行处理，值 `false` 表示禁用并行处理。 |
| hms_process_events_parallel_num   | StarRocks 可以并行处理的最大事件数。默认值：`4`。 |

## 定期刷新元数据缓存

从 v2.5.5 版本开始，StarRocks 可以定期刷新频繁访问的 Hive 目录的缓存元数据以感知数据变化。您可以通过以下 [前端参数](../../administration/Configuration.md#fe-configuration-items) 配置 Hive 元数据缓存刷新：

| 配置项目                                           | 默认                            | 描述                       |
| ------------------------------------------------------------ | ------------------------------------ | ------------------------------------ |
| enable_background_refresh_connector_metadata                 | 在 v3.0 中为 `true`<br />在 v2.5 中为 `false`  | 是否启用定期 Hive 元数据缓存刷新。启用后，StarRocks 会轮询您的 Hive 集群的元数据存储（Hive metastore 或 AWS Glue），并刷新频繁访问的 Hive 目录的缓存元数据以感知数据变化。`true` 表示启用 Hive 元数据缓存刷新，`false` 表示禁用。此项目是一个 [前端动态参数](../../administration/Configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令来修改它。 |
| background_refresh_metadata_interval_millis                  | `600000`（10 分钟）                | 两次连续的 Hive 元数据缓存刷新之间的时间间隔。单位：毫秒。此项目是一个 [前端动态参数](../../administration/Configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令来修改它。 |
| background_refresh_metadata_time_secs_since_last_access_secs | `86400`（24 小时）                   | Hive 元数据缓存刷新任务的过期时间。对于已经访问过的 Hive 目录，如果在指定时间内未再次访问，则 StarRocks 停止刷新其缓存元数据。对于未访问的 Hive 目录，StarRocks 不会刷新其缓存元数据。单位：秒。此项目是一个 [前端动态参数](../../administration/Configuration.md#configure-fe-dynamic-parameters)。您可以使用 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令来修改它。 |

将定期 Hive 元数据缓存刷新功能与元数据自动异步更新策略一起使用，可以显著加速数据访问，减少来自外部数据源的读取负载，并提高查询性能。

## 附录：了解元数据自动异步更新

自动异步更新是 StarRocks 用来更新 Hive 目录中的元数据的默认策略。

默认情况下（即当 `enable_metastore_cache` 和 `enable_remote_file_cache` 参数都设置为 `true` 时），如果查询命中了 Hive 表的某个分区，StarRocks 会自动缓存该分区的元数据和该分区的底层数据文件的元数据。缓存的元数据使用延迟更新策略进行更新。

例如，有一个名为 `table2` 的 Hive 表，它有四个分区：`p1`、`p2`、`p3` 和 `p4`。查询命中了 `p1`，StarRocks 缓存了 `p1` 的元数据以及 `p1` 的底层数据文件的元数据。假设更新和丢弃缓存的元数据的默认时间间隔如下：

- 异步更新 `p1` 的缓存元数据的时间间隔（由 `metastore_cache_refresh_interval_sec` 参数指定）为 2 小时。
- 异步更新 `p1` 底层数据文件的缓存元数据的时间间隔（由 `remote_file_cache_refresh_interval_sec` 参数指定）为 60 秒。
- 自动丢弃 `p1` 的缓存元数据的时间间隔（由 `metastore_cache_ttl_sec` 参数指定）为 24 小时。
- 自动丢弃 `p1` 底层数据文件的缓存元数据的时间间隔（由 `remote_file_cache_ttl_sec` 参数指定）为 36 小时。

下图在时间线上显示了这些时间间隔，以便更好地理解。

![更新和丢弃缓存元数据的时间轴](../../assets/catalog_timeline.png)

然后，StarRocks 根据以下规则来更新或丢弃元数据：

- 如果另一个查询再次命中了 `p1`，且从上次更新至当前的时间小于 60 秒，则 StarRocks 不会更新 `p1` 或 `p1` 的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了 `p1`，且从上次更新至当前的时间大于 60 秒，则 StarRocks 更新 `p1` 的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了 `p1`，且从上次更新至当前的时间大于 2 小时，则 StarRocks 更新 `p1` 的缓存元数据。
- 如果从上次更新起，`p1` 在 24 小时内未被访问，则 StarRocks 丢弃 `p1` 的缓存元数据。在下一次查询时，元数据将被缓存。
- 如果从上次更新起，`p1` 在 36 小时内未被访问，则 StarRocks 丢弃 `p1` 的底层数据文件的缓存元数据。在下一次查询时，元数据将被缓存。