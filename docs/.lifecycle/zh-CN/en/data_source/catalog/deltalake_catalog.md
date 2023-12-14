---
displayed_sidebar: "英语"
---

# Delta Lake 目录

Delta Lake 目录是一种外部目录，它使您能够在不经过摄入的情况下查询 Delta Lake 中的数据。

此外，您还可以直接基于 Delta Lake 目录使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 来转换和加载 Delta Lake 中的数据。StarRocks 从 v2.5 开始支持 Delta Lake 目录。

为确保您的 Delta Lake 集群能够成功完成 SQL 工作负载，您的 StarRocks 集群需要集成两个重要组件：

- 分布式文件系统 (HDFS) 或类似 AWS S3、Microsoft Azure Storage、Google GCS 或其他兼容 S3 的对象存储系统（例如 MinIO）这样的对象存储系统
- 像 Hive metastore 或 AWS Glue 这样的元存储

  > **注意**
  >
  > 如果您选择 AWS S3 作为存储系统，则可以使用 HMS 或 AWS Glue 作为元存储。如果您选择其他任何存储系统，则只能使用 HMS 作为元存储。

## 使用说明

StarRocks 支持的 Delta Lake 文件格式是 Parquet。Parquet 文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP 和 NO_COMPRESSION。

StarRocks 不支持的 Delta Lake 数据类型包括 MAP 和 STRUCT。

## 集成准备工作

在创建 Delta Lake 目录之前，请确保您的 StarRocks 集群可以与 Delta Lake 集群的存储系统和元存储集成。

### AWS IAM

如果您的 Delta Lake 集群使用 AWS S3 作为存储系统或 AWS Glue 作为元存储，请选择合适的身份验证方法并进行必要的准备，以确保您的 StarRocks 集群能够访问相关的 AWS 云资源。

建议的身份验证方法如下：

- 实例配置文件
- 假定角色
- IAM 用户

在上述三种身份验证方法中，实例配置文件是最常用的。

有关更多信息，请参见 [AWS IAM 身份验证准备](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)。

### HDFS

如果您选择 HDFS 作为存储系统，请按照以下步骤配置您的 StarRocks 集群：

- (可选) 设置用于访问您的 HDFS 集群和 Hive metastore 的用户名。默认情况下，StarRocks 使用 FE 和 BE 进程的用户名来访问您的 HDFS 集群和 Hive metastore。您还可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。在这些文件中设置用户名后，重新启动每个 FE 和每个 BE 以使参数设置生效。每个 StarRocks 集群只能设置一个用户名。
- 当您查询 Delta Lake 数据时，StarRocks 集群的 FE 和 BE 使用 HDFS 客户端访问您的 HDFS 集群。在大多数情况下，您不需要配置您的 StarRocks 集群来实现此目的，StarRocks 使用默认配置启动 HDFS 客户端。以下情况下，您需要配置您的 StarRocks 集群：

  - 为您的 HDFS 集群启用了高可用性 (HA)：将 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。
  - 为您的 HDFS 集群启用了 View File System (ViewFs)：将 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。

> **注意**
>
> 如果发送查询时返回了指示未知主机的错误，请将您的 HDFS 集群节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径。

### Kerberos 身份验证

如果为您的 HDFS 集群或 Hive metastore 启用了 Kerberos 身份验证，请按以下步骤配置您的 StarRocks 集群：

- 在每个 FE 和每个 BE 上运行 `kinit -kt keytab_path principal` 命令，以从密钥分发中心 (KDC) 获取票证授权票（TGT）。运行此命令时，您必须具有访问您的 HDFS 集群和 Hive metastore 的权限。请注意，在规定时间内通过该命令访问 KDC 是受时间限制的。因此，您需要使用 cron 定期运行此命令。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据自己的需求修改该路径。

## 创建 Delta Lake 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "deltalake",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### 参数

#### catalog_name

Delta Lake 目录的名称。命名约定如下：

- 名称可以包含字母、数字 (0-9) 和下划线 (_)。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### comment

Delta Lake 目录的描述。此参数是可选的。

#### type

数据源的类型。将值设置为 `deltalake`。

#### MetastoreParams

关于 StarRocks 如何与数据源的元存储集成的一组参数。

##### Hive metastore

如果您选择 Hive metastore 作为您的数据源的元存储，请按以下方式配置 `MetastoreParams`：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询 Delta Lake 数据之前，您必须将您的 Hive metastore 节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径。否则，当您开始查询时，StarRocks 可能无法访问您的 Hive metastore。

以下表格描述了您需要配置在 `MetastoreParams` 中的参数。

| 参数                | 是否必需 | 描述                                                        |
| ------------------- | -------- | ----------------------------------------------------------- |
| hive.metastore.type | 是       | 您用于 Delta Lake 集群的元存储类型。将值设置为 `hive`。    |
| hive.metastore.uris | 是       | 您的 Hive metastore 的 URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />如果为您的 Hive metastore 启用了高可用性 (HA)，您可以指定多个元存储 URI，并使用逗号 (`,`) 进行分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

如果您选择 AWS Glue 作为您的数据源的元存储（仅在选择 AWS S3 作为存储系统时受支持），请执行以下操作中的一个：

- 选择基于实例配置文件的身份验证方法，请按以下方式配置 `MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 选择基于假定角色的身份验证方法，请按以下方式配置 `MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 选择基于 IAM 用户的身份验证方法，请按以下方式配置 `MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要配置在 `MetastoreParams` 中的参数。

| 参数                         | 是否必需 | 描述                                                                                        |
| ---------------------------- | -------- | ------------------------------------------------------------------------------------------- |
| hive.metastore.type          | 是       | 您用于 Delta Lake 集群的元存储类型。将值设置为 `glue`。                                     |
| aws.glue.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：`true` 和 `false`。默认值：`false`。 |
| aws.glue.iam_role_arn         | No       | AWS Glue数据目录访问权限的IAM角色的ARN。如果您使用基于假定角色的身份验证方法访问AWS Glue，则必须指定此参数。 |
| aws.glue.region               | 是       | AWS Glue数据目录所在的区域。示例：“us-west-1”。 |
| aws.glue.access_key           | No       | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法访问AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | No       | 您的AWS IAM用户的秘密密钥。如果您使用基于IAM用户的身份验证方法访问AWS Glue，则必须指定此参数。 |

有关如何选择访问AWS Glue的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS Glue的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

关于StarRocks如何与您的存储系统集成的一组参数。此参数集是可选的。

如果您使用HDFS作为存储，您无需配置`StorageCredentialParams`。

如果您使用AWS S3、其他与S3兼容的存储系统、Microsoft Azure Storage或Google GCS作为存储，您必须配置`StorageCredentialParams`。

##### AWS S3

如果您选择AWS S3作为Delta Lake cluster的存储，执行以下操作之一：

- 选择基于实例配置文件的身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 选择基于假定角色的身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 选择基于IAM用户的身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                        | 必需     | 描述                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：“true”和“false”。默认值：“false”。 |
| aws.s3.iam_role_arn         | No       | 拥有特权访问AWS S3存储桶的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS S3，则必须指定此参数。 |
| aws.s3.region               | 是       | 您的AWS S3存储桶所在的区域。示例：“us-west-1”。 |
| aws.s3.access_key           | No       | 您的IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | No       | 您的IAM用户的秘密密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。 |

有关如何选择访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS S3的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 与S3兼容的存储系统

Delta Lake目录支持从v2.5开始的与S3兼容的存储系统。

如果您选择与S3兼容的存储系统（例如MinIO）作为Delta Lake cluster的存储，配置`StorageCredentialParams`如下，以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                        | 必需     | 描述                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl           | 是       | 指定是否启用SSL连接。<br />有效值：“true”和“false”。默认值：“true”。 |
| aws.s3.enable_path_style_access | 是    | 指定是否启用路径样式访问。<br />有效值：“true”和“false”。默认值：“false”。对于MinIO，您必须将值设置为“true”。<br />路径样式URL使用以下格式：“https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>”。例如，如果您在美国西部（俄勒冈州）区域创建一个名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且您想要访问该存储桶中的`alice.jpg`对象，则可以使用以下路径样式URL：“https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg”。 |
| aws.s3.endpoint             | 是       | 用于连接到与AWS S3不同的S3兼容存储系统的终端点。              |
| aws.s3.access_key           | 是       | 您的IAM用户的访问密钥。                                        |
| aws.s3.secret_key           | 是       | 您的IAM用户的秘密密钥。                                        |

##### Microsoft Azure Storage

Delta Lake目录支持从v3.0开始的Microsoft Azure Storage。

###### Azure Blob Storage

如果您选择Blob Storage作为Delta Lake cluster的存储，执行以下操作之一：

- 选择Shared Key身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                 | **必需**  | **描述**                   |
  | ------------------------- | ---------- | -------------------------- |
  | azure.blob.storage_account | 是       | 您Blob Storage账户的用户名。     |
  | azure.blob.shared_key      | 是       | 您Blob Storage账户的共享密钥。 |

- 选择SAS Token身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**              | **必需**  | **描述**                         |
  | ---------------------- | ---------- | -------------------------------- |
  | azure.blob.account_name | 是       | 您Blob Storage账户的用户名。      |
  | azure.blob.container_name | 是      | 存储数据的blob容器的名称。       |
  | azure.blob.sas_token   | 是       | 用于访问您的Blob Storage账户的SAS令牌。 |

###### Azure Data Lake Storage Gen1

如果您选择Data Lake Storage Gen1作为Delta Lake cluster的存储，执行以下操作之一：

- 选择托管服务标识身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                      | **必需**  | **描述**                                 |
  | ------------------------------ | ---------- | ---------------------------------------- |
  | azure.adls1.use_managed_service_identity | 是   | 指定是否启用托管服务标识身份验证方法。将值设置为“true”。

- 选择服务主体身份验证方法，配置`StorageCredentialParams`如下：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                  | **必需**  | **描述**                         |
  | -------------------------- | ---------- | -------------------------------- |
  | azure.adls1.oauth2_client_id | 是       | 服务主体的客户端（应用程序）ID。  |
  | azure.adls1.oauth2_credential | 是      | 新创建的客户端（应用程序）秘密的值。 |
  | azure.adls1.oauth2_endpoint | 是       | 服务主体或应用程序的OAuth 2.0令牌端点（v1）。|

###### Azure Data Lake Storage Gen2

如果您选择Data Lake Storage Gen2作为Delta Lake集群的存储，可以执行以下操作之一：

- 要选择托管标识身份验证方法，请配置`StorageCredentialParams`如下：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                            | **必需** | **描述**                                    |
  | --------------------------------- | ------ | ---------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | 是      | 指定是否启用托管标识身份验证方法。将值设置为`true`。     |
  | azure.adls2.oauth2_tenant_id            | 是      | 您要访问其数据的租户的ID。                     |
  | azure.adls2.oauth2_client_id            | 是      | 托管标识的客户（应用程序）ID。                    |

- 要选择共享密钥身份验证方法，请配置`StorageCredentialParams`如下：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**               | **必需** | **描述**                                    |
  | ---------------------- | ------ | ---------------------------------------- |
  | azure.adls2.storage_account | 是      | 您的Data Lake Storage Gen2存储帐户的用户名。            |
  | azure.adls2.shared_key      | 是      | 您的Data Lake Storage Gen2存储帐户的共享密钥。          |

- 要选择服务主体身份验证方法，请配置`StorageCredentialParams`如下：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                   | **必需** | **描述**                                    |
  | --------------------------- | ------ | ---------------------------------------- |
  | azure.adls2.oauth2_client_id       | 是      | 服务主体的客户（应用程序）ID。                    |
  | azure.adls2.oauth2_client_secret   | 是      | 创建的新客户（应用程序）密钥的值。                 |
  | azure.adls2.oauth2_client_endpoint | 是      | 服务主体或应用程序的OAuth 2.0令牌端点（v1）。           |

##### Google GCS

Delta Lake目录从v3.0开始支持Google GCS。

如果您选择Google GCS作为Delta Lake集群的存储，可以执行以下操作之一：

- 要选择基于VM的身份验证方法，请配置`StorageCredentialParams`如下：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                                  | **默认值** | **值示例**  | **描述**                                    |
  | --------------------------------------- | -------- | ----------- | ---------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false    | true        | 指定是否直接使用绑定到您的Compute Engine的服务帐户。 |

- 要选择基于服务帐户的身份验证方法，请配置`StorageCredentialParams`如下：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
  ```

  下表描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                         | **默认值** | **值示例**                                     | **描述**                                    |
  | --------------------------------- | -------- | ---------------------------------------- | ---------------------------------------- |
  | gcp.gcs.service_account_email     | ""       | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建服务帐户时生成的JSON文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | ""       | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"        | 在创建服务帐户时生成的JSON文件中的私钥ID。       |
  | gcp.gcs.service_account_private_key    | ""       | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建服务帐户时生成的JSON文件中的私钥。       |

- 要选择基于拟扮身份的身份验证方法，请配置`StorageCredentialParams`如下：

  - 使VM实例拟扮为服务帐户：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                                  | **默认值** | **值示例**  | **描述**                                    |
    | --------------------------------------- | -------- | ----------- | ---------------------------------------- |
    | gcp.gcs.use_compute_engine_service_account | false    | true        | 指定是否直接使用绑定到您的Compute Engine的服务帐户。 |
    | gcp.gcs.impersonation_service_account     | ""       | "hello"     | 您想要拟扮的服务帐户。                           |

  - 使服务帐户（临时命名为元服务帐户）拟扮另一个服务帐户（临时命名为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                         | **默认值** | **值示例**                                     | **描述**                                    |
    | --------------------------------- | -------- | ---------------------------------------- | ---------------------------------------- |
    | gcp.gcs.service_account_email     | ""       | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐户时生成的JSON文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""       | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"        | 在创建元服务帐户时生成的JSON文件中的私钥ID。       |
    | gcp.gcs.service_account_private_key    | ""       | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建元服务帐户时生成的JSON文件中的私钥。       |
    | gcp.gcs.impersonation_service_account  | ""       | "hello"                                     | 您想要拟扮的数据服务帐户。                         |

#### MetadataUpdateParams

关于StarRocks如何更新Delta Lake缓存的元数据的一组参数。此参数集是可选的。

StarRocks默认实现[自动异步更新策略](#附录-了解元数据自动异步更新)。

在大多数情况下，您可以忽略`MetadataUpdateParams`，不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了即插即用的性能。

但是，如果Delta Lake中数据的更新频率很高，则可以调整这些参数以进一步优化自动异步更新的性能。

> **注意**
>
> 在大多数情况下，如果您的Delta Lake数据以1小时或更短的粒度更新，那么数据更新频率被视为高。

| 参数                                  | 必需   | 描述                                       |
|----------------------------------------| ------ | ----------------------------------------- |
| enable_metastore_cache                 | 否      | 指定StarRocks是否缓存Delta Lake表的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| enable_remote_file_cache               | 否      | 指定StarRocks是否缓存Delta Lake表或分区的底层数据文件的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| metastore_cache_refresh_interval_sec   | 否      | StarRocks异步更新自身缓存的Delta Lake表或分区的元数据的时间间隔。单位：秒。默认值：`7200`，即2小时。 |
| remote_file_cache_refresh_interval_sec | No | StarRocks异步更新其自身缓存的Delta Lake表或分区的基础数据文件的元数据的时间间隔。单位：秒。默认值：`60`。|
| metastore_cache_ttl_sec | No | StarRocks自动丢弃其自身缓存的Delta Lake表或分区的元数据的时间间隔。单位：秒。默认值：`86400`，即24小时。|
| remote_file_cache_ttl_sec | No | StarRocks自动丢弃其自身缓存的Delta Lake表或分区的基础数据文件的元数据的时间间隔。单位：秒。默认值：`129600`，即36小时。|

### 示例

以下示例将创建名为`deltalake_catalog_hms`或`deltalake_catalog_glue`的Delta Lake目录，具体取决于您使用的Hive元数据存储。用于从Delta Lake集群查询数据。

#### HDFS

如果您将HDFS作为存储，运行以下命令：

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### 如果您选择基于实例配置文件的凭据

- 如果您在Delta Lake集群中使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Delta Lake集群中使用AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果您选择基于假定角色的凭据

- 如果您在Delta Lake集群中使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Delta Lake集群中使用AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果您选择基于IAM用户的凭据

- 如果您在Delta Lake集群中使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Delta Lake集群中使用AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_glue
  PROPERTIES
  (
      "type" = "deltalake",
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

以MinIO为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG deltalake_catalog_hms
PROPERTIES
(
    "type" = "deltalake",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### 微软Azure存储

##### Azure Blob存储

- 如果您选择共享密钥认证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果您选择SAS令牌认证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure数据湖存储Gen1

- 如果您选择托管服务身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果您选择服务主体认证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure数据湖存储Gen2

- 如果您选择托管身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );


- 如果选择使用共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择使用服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果选择基于VM的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务账户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG deltalake_catalog_hms
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于冒充的身份验证方法：

  - 如果使VM实例冒充服务账户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果使服务账户冒充另一个服务账户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG deltalake_catalog_hms
    PROPERTIES
    (
        "type" = "deltalake",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 查看Delta Lake目录

可以使用[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)查询当前StarRocks集群中的所有目录：

```SQL
SHOW CATALOGS;
```

还可以使用[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)查询外部目录的创建语句。以下示例查询名为`deltalake_catalog_glue`的Delta Lake目录的创建语句：

```SQL
SHOW CREATE CATALOG deltalake_catalog_glue;
```

## 切换到Delta Lake目录及其中的数据库

可以使用以下方法之一切换到Delta Lake目录及其中的数据库：

- 使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)指定当前会话中的Delta Lake目录，然后使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)指定活动数据库：

  ```SQL
  -- 切换到当前会话中指定的目录：
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库：
  USE <db_name>
  ```

- 直接使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)切换到Delta Lake目录及其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除Delta Lake目录

可以使用[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)删除外部目录。

以下示例删除名为`deltalake_catalog_glue`的Delta Lake目录：

```SQL
DROP Catalog deltalake_catalog_glue;
```

## 查看Delta Lake表的架构

可以使用以下语法之一查看Delta Lake表的架构：

- 查看架构

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 查看来自CREATE语句的架构和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询Delta Lake表

1. 使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)查看Delta Lake集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到Delta Lake目录及其中的数据库](#切换到delta-lake目录及其中的数据库).

3. 使用[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从Delta Lake加载数据

假设有名为`olap_tbl`的OLAP表，您可以执行以下操作对数据进行转换和加载：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM deltalake_table
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks缓存Delta Lake的元数据，并以异步模式自动更新元数据，以提供更好的性能。此外，在对Delta Lake表进行某些架构更改或表更新后，您还可以使用[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)手动更新其元数据，以确保StarRocks可以尽快获取最新的元数据并生成适当的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略使您StarRocks集群中的FE可以从Hive元数据存储中读取事件（例如添加列、删除分区和更新数据）。StarRocks可以根据这些事件自动更新FE中缓存的元数据。这意味着您无需手动更新Delta Lake表的元数据。

要启用自动增量更新，请执行以下步骤：

#### 步骤1：为Hive元数据存储配置事件监听器

Hive元数据存储v2.x和v3.x均支持配置事件监听器。下面的步骤以Hive元数据存储v3.1.2的事件监听器配置为例。将以下配置项添加到**$HiveMetastore/conf/hive-site.xml**文件中，然后重新启动Hive元数据存储：

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
```xml
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

您可以在 FE 日志文件中搜索`event id`，以检查事件监听器是否成功配置。如果配置失败，则`event id`值为`0`。

#### 步骤2: 在 StarRocks 上启用自动增量更新

您可以为单个 Delta Lake 目录或您的 StarRocks 集群中的所有 Delta Lake 目录启用自动增量更新。

- 要为单个 Delta Lake 目录启用自动增量更新，请在创建 Delta Lake 目录时，像下面这样在 `PROPERTIES` 中将`enable_hms_events_incremental_sync`参数设置为`true`：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "deltalake",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- 要为所有 Delta Lake 目录启用自动增量更新，请将 `"enable_hms_events_incremental_sync" = "true"` 添加到每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中，然后重新启动每个 FE 以使参数设置生效。

您还可以根据业务需求，在每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中调整以下参数，然后重新启动每个 FE 以使参数设置生效。

| 参数                              | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks 从 Hive 元数据存储中读取事件的时间间隔。默认值： `5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks 一次可以读取的事件的最大数量。默认值：`500`。      |
| enable_hms_parallel_process_evens | 指定 StarRocks 是否并行处理事件以及读取事件。有效值: `true` 和 `false`。默认值：`true`。`true`表示启用并行处理，`false`表示禁用并行处理。 |
| hms_process_events_parallel_num   | StarRocks 可以并行处理的事件的最大数量。默认值：`4`。        |

## 附录：理解元数据自动异步更新

自动异步更新是 StarRocks 用来更新 Delta Lake 目录中的元数据的默认策略。

默认情况下（即当`enable_metastore_cache`和`enable_remote_file_cache`参数均设置为`true`时），如果查询命中 Delta Lake 表的分区，则 StarRocks 会自动缓存分区的元数据和分区底层数据文件的元数据。缓存的元数据是使用延迟更新策略更新的。

例如，有一个名为 `table2` 的 Delta Lake 表，它有四个分区：`p1`、`p2`、`p3`和`p4`。一个查询命中了`p1`，StarRocks缓存了`p1`的元数据和`p1`的底层数据文件的元数据。假设默认的更新和丢弃缓存的元数据的时间间隔为：

- 异步更新`p1`的缓存元数据的时间间隔（由`metastore_cache_refresh_interval_sec`参数指定）为2小时。
- 异步更新`p1`的底层数据文件的缓存元数据的时间间隔（由`remote_file_cache_refresh_interval_sec`参数指定）为60秒。
- 自动丢弃`p1`的缓存元数据的时间间隔（由`metastore_cache_ttl_sec`参数指定）为24小时。
- 自动丢弃`p1`的底层数据文件的缓存元数据的时间间隔（由`remote_file_cache_ttl_sec`参数指定）为36小时。

下面的图示显示了时间轴上的时间间隔，以便更容易理解。

![更新和丢弃缓存的元数据的时间轴](../../assets/catalog_timeline.png)

然后，StarRocks根据以下规则更新或丢弃元数据：

- 如果另一个查询再次命中了`p1`，并且当前时间距上次更新时间少于60秒，则StarRocks不会更新`p1`的缓存元数据或`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了`p1`，并且当前时间距上次更新时间超过60秒，则StarRocks会更新`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了`p1`，并且当前时间距上次更新时间超过2小时，则StarRocks会更新`p1`的缓存元数据。
- 如果从上次更新已经过去了24小时，而未访问`p1`，则StarRocks会丢弃`p1`的缓存元数据。元数据将在下次查询时被缓存。
- 如果从上次更新已经过去了36小时，而未访问`p1`，则StarRocks会丢弃`p1`的底层数据文件的缓存元数据。元数据将在下次查询时被缓存。