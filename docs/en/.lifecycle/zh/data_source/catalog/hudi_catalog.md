---
displayed_sidebar: English
---

# Hudi目录

Hudi目录是一种外部目录，可让您在不进行摄取的情况下查询 Apache Hudi 中的数据。

此外，您还可以使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 来直接基于 Hudi目录转换和加载数据。StarRocks 从 v2.4 开始支持 Hudi目录。

为了确保 Hudi集群上的 SQL 工作负载成功，您的 StarRocks集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或像 AWS S3、Microsoft Azure Storage、Google GCS 或其他兼容 S3 的存储系统（例如 MinIO）这样的对象存储

- Hive元存储或 AWS Glue等元存储

  > **注意**
  >
  > 如果您选择 AWS S3 作为存储，可以使用 HMS 或 AWS Glue 作为元存储。如果选择其他存储系统，则只能使用 HMS 作为元存储。

## 使用说明

- StarRocks支持的 Hudi文件格式为 Parquet。Parquet文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP 和 NO_COMPRESSION。
- StarRocks为 Hudi的Copy On Write（COW）表和Merge On Read（MOR）表提供完整支持。

## 集成准备

在创建 Hudi目录之前，请确保您的 StarRocks集群可以与 Hudi集群的存储系统和元存储集成。

### AWS IAM

如果您的 Hudi集群使用 AWS S3 作为存储或 AWS Glue 作为元存储，请选择合适的身份验证方法并做好必要的准备工作，以确保您的 StarRocks集群能够访问相关的 AWS 云资源。

建议使用以下身份验证方法：

- 实例配置文件
- 承担的角色
- IAM用户

在上述三种身份验证方法中，实例配置文件是最广泛使用的。

有关更多信息，请参阅 [在 AWS IAM 中准备身份验证](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果您选择 HDFS 作为存储，请按如下方式配置您的 StarRocks集群：

- （可选）设置用于访问您的 HDFS集群和 Hive元存储的用户名。StarRocks默认使用 FE 和 BE 进程的用户名来访问您的 HDFS集群和 Hive元存储。您也可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。设置了用户名后，请重新启动每个 FE 和 BE 以使参数设置生效。每个 StarRocks集群只能设置一个用户名。
- 查询 Hudi数据时，您的 StarRocks集群的 FE 和 BE 使用 HDFS客户端来访问您的 HDFS集群。在大多数情况下，您不需要配置您的 StarRocks集群来实现这一目的，StarRocks会使用默认配置启动 HDFS客户端。只有在以下情况下，您才需要配置您的 StarRocks集群：

  - 为您的 HDFS集群启用了高可用性（HA）：将您的 HDFS集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。
  - 为您的 HDFS集群启用了 View 文件系统（ViewFs）：将您的 HDFS集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。

> **注意**
>
> 如果在发送查询时返回了未知主机的错误，则必须将您的 HDFS集群节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径中。

### Kerberos身份验证

如果为您的 HDFS集群或 Hive元存储启用了 Kerberos身份验证，请按如下方式配置您的 StarRocks集群：

- 在每个 FE 和 BE 上运行 `kinit -kt keytab_path principal` 命令，以从密钥分发中心（KDC）获取票证授予票证（TGT）。要运行此命令，您必须具有访问您的 HDFS集群和 Hive元存储的权限。请注意，使用此命令访问 KDC 是时间敏感的。因此，您需要使用 cron 定期运行此命令。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改路径。

## 创建 Hudi目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "hudi",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### 参数

#### catalog_name

Hudi目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### comment

Hudi目录的描述。此参数是可选的。

#### type

数据源的类型。将值设置为 `hudi`。

#### MetastoreParams

一组关于 StarRocks如何与数据源的元存储集成的参数。

##### Hive元存储

如果选择 Hive元存储作为数据源的元存储，请按如下方式配置 `MetastoreParams` ：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询 Hudi数据之前，您必须将 Hive元存储节点的主机名和 IP 地址的映射添加到 `/etc/hosts` 路径中。否则，当您发起查询时，StarRocks可能无法访问您的 Hive元存储。

下表描述了您需要在 `MetastoreParams` 中配置的参数。

| 参数           | 必填 | 描述                                                  |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | 是      | 用于 Hudi集群的元存储类型。将值设置为 `hive`。 |
| hive.metastore.uris | 是      | Hive元存储的 URI。格式： `thrift://<metastore_IP_address>:<metastore_port>`。<br />如果为 Hive元存储启用了高可用性（HA），则可以指定多个元存储 URI，并用逗号（,）分隔它们，例如 `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

如果选择 AWS Glue 作为数据源的元存储（仅当您选择 AWS S3 作为存储时才支持），请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择假定的基于角色的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择 IAM基于用户的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `MetastoreParams` 中配置的参数。

| 参数                     | 必填 | 描述                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | 是      | 用于 Hudi集群的元存储类型。将值设置为 `glue`。 |
| aws.glue.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和假定的基于角色的身份验证方法。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.glue.iam_role_arn         | 否       | 在 AWS Glue 数据目录上具有权限的 IAM 角色的 ARN。如果您使用假定的基于角色的身份验证方法访问 AWS Glue，则必须指定此参数。 |
| aws.glue.区域               | 是      | 您的 AWS Glue 数据目录所在的区域。示例： `us-west-1`。 |
| aws.glue.access_key           | 否       | 您的 AWS IAM 用户的访问密钥。如果您使用 IAM基于用户的身份验证方法访问 AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | 否       | 您的 AWS IAM 用户的私有密钥。如果您使用 IAM基于用户的身份验证方法访问 AWS Glue，则必须指定此参数。 |

有关如何选择用于访问 AWS Glue 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅 [用于访问 AWS Glue 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

一组关于 StarRocks如何与您的存储系统集成的参数。此参数集是可选的。

如果使用 HDFS 作为存储，则无需配置 `StorageCredentialParams`。

如果您使用 AWS S3、其他兼容 S3 的存储系统、Microsoft Azure Storage 或 Google GCS 作为存储，则必须配置 `StorageCredentialParams`。

##### AWS S3的

如果您选择 AWS S3 作为 Hudi集群的存储，请执行下列操作之一：

- 要选择基于实例配置文件的身份验证方法，`StorageCredentialParams` 请按如下方式进行配置：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择假定的基于角色的身份验证方法，`StorageCredentialParams` 请按如下方式进行配置：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择 IAM基于用户的身份验证方法，`StorageCredentialParams` 请按如下方式进行配置：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |

| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值： `true` 和 `false`。默认值： `false`。 |
| aws.s3.iam_role_arn         | 否       | 对您的 AWS S3 存储桶具有权限的 IAM 角色的 ARN。如果您使用假定的基于角色的身份验证方法访问 AWS S3，则必须指定此参数。 |
| aws.s3.region               | 是       | 您的 AWS S3 存储桶所在的区域。示例： `us-west-1`. |
| aws.s3.access_key           | 否       | IAM 用户的访问密钥。如果您使用 IAM 用户基于的身份验证方法访问 AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | IAM 用户的私有密钥。如果您使用 IAM 用户基于的身份验证方法访问 AWS S3，则必须指定此参数。 |

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅 [访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储系统

Hudi 目录从 v2.5 开始支持 S3 兼容的存储系统。

如果选择 S3 兼容的存储系统（如 MinIO）作为 Hudi 集群的存储，请按以下方式配置 `StorageCredentialParams` 以确保成功集成：

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
| aws.s3.enable_ssl                | 是       | 指定是否启用 SSL 连接。<br />有效值： `true` 和 `false`。默认值： `true`。 |
| aws.s3.enable_path_style_access  | 是       | 指定是否启用路径样式访问。<br />有效值： `true` 和 `false`。默认值：`false`。对于 MinIO，必须将值设置为 `true`。<br />路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建了名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | 是       | 用于连接到与 S3 兼容的存储系统（而不是 AWS S3）的终端节点。 |
| aws.s3.access_key                | 是       | IAM 用户的访问密钥。 |
| aws.s3.secret_key                | 是       | IAM 用户的私有密钥。 |

##### Microsoft Azure 存储

Hudi 目录从 v3.0 开始支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果选择 Blob 存储作为 Hudi 群集的存储，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**              | **必填** | **描述**                              |
  | -------------------------- | ------------ | -------------------------------------------- |
  | azure.blob.storage_account | 是       | Blob 存储帐户的用户名。   |
  | azure.blob.shared_key      | 是       | Blob 存储帐户的共享密钥。 |

- 若要选择 SAS 令牌身份验证方法， `StorageCredentialParams` 请按以下方式配置：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**             | **必填** | **描述**                                              |
  | ------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.blob.storage_account| 是       | Blob 存储帐户的用户名。                   |
  | azure.blob.container      | 是       | 存储数据的 Blob 容器的名称。        |
  | azure.blob.sas_token      | 是       | 用于访问 Blob 存储帐户的 SAS 令牌。 |

###### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为 Hudi 群集的存储，请执行以下操作之一：

- 若要选择托管服务标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                            | **必填** | **描述**                                              |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是       | 指定是否启用托管服务标识身份验证方法。将值设置为 `true`。 |

- 若要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                 | **必填** | **描述**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是       | 服务主体的客户端（应用程序）ID。        |
  | azure.adls1.oauth2_credential | 是       | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls1.oauth2_endpoint   | 是       | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

###### Azure Data Lake Storage Gen2

如果选择 Data Lake Storage Gen2 作为 Hudi 群集的存储，请执行以下操作之一：

- 若要选择托管标识身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                           | **必填** | **描述**                                              |
  | --------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是       | 指定是否启用托管标识身份验证方法。将值设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是       | 要访问其数据的租户的 ID。          |
  | azure.adls2.oauth2_client_id            | 是       | 托管标识的客户端（应用程序）ID。         |

- 要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**               | **必填** | **描述**                                              |
  | --------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是       | Data Lake Storage Gen2 存储帐户的用户名。 |
  | azure.adls2.shared_key      | 是       | Data Lake Storage Gen2 存储帐户的共享密钥。 |

- 若要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                      | **必填** | **描述**                                              |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是       | 服务主体的客户端（应用程序）ID。        |
  | azure.adls2.oauth2_client_secret   | 是       | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls2.oauth2_client_endpoint | 是       | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

##### Google GCS

Hudi 目录从 v3.0 开始支持 Google GCS。

如果选择 Google GCS 作为 Hudi 群集的存储，请执行以下操作之一：

- 若要选择基于 VM 的身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                              | **默认值** | **值****示例** | **描述**                                              |
  | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |

- 若要选择基于服务帐户的身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | **参数**                          | **默认值** | **值****示例**                                        | **描述**                                              |
  | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""                | “[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)” | 创建服务帐户时生成的 JSON 文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | ""                | “61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea”                   | 创建服务帐户时生成的 JSON 文件中的私钥 ID。 |
  | gcp.gcs.service_account_private_key    | ""                | “-----开始私钥----xxxx-----结束私钥-----\n”  | 创建服务帐户时生成的 JSON 文件中的私钥。 |

- 若要选择基于模拟的身份验证方法，请按以下方式配置 `StorageCredentialParams` ：

  - 使 VM 实例模拟服务帐户：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                              | **默认值** | **值****示例** | **描述**                                              |
    | ------------------------------------------ | ----------------- | --------------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false             | true                  | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |

    | gcp.gcs.impersonation_service_account      | ""                | "hello"               | 您要模拟的服务帐户。            |

  - 使一个服务帐户（临时命名为元服务帐户）模拟另一个服务帐户（临时命名为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | **参数**                          | **默认值** | **示例值**                                        | **描述**                                              |
    | -------------------------------------- | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 在创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 在创建元服务帐户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 您要模拟的数据服务帐户。       |

#### MetadataUpdateParams

关于 StarRocks 如何更新 Hudi 缓存元数据的一组参数。此参数集是可选的。

StarRocks 默认实现 [自动异步更新策略](#appendix-understand-metadata-automatic-asynchronous-update) 。

在大多数情况下，您可以忽略 `MetadataUpdateParams` 并且不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了开箱即用的性能。

但是，如果 Hudi 中数据更新的频率较高，则可以调整这些参数，以进一步优化自动异步更新的性能。

> **注意**
>
> 在大多数情况下，如果 Hudi 数据以 1 小时或更短的粒度更新，则认为数据更新频率较高。

| 参数                              | 必填 | 描述                                                  |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | 否       | 指定 StarRocks 是否缓存 Hudi 表的元数据。有效值： `true` 和 `false`。默认值：`true`。该`true`值启用缓存，该值 `false` 禁用缓存。 |
| enable_remote_file_cache               | 否       | 指定 StarRocks 是否缓存 Hudi 表或分区的底层数据文件的元数据。有效值： `true` 和 `false`。默认值：`true`。该`true`值启用缓存，该值 `false` 禁用缓存。 |
| metastore_cache_refresh_interval_sec   | 否       | StarRocks 异步更新自身缓存的 Hudi 表或分区元数据的时间间隔。单位：秒。默认值： `7200`，即 2 小时。 |
| remote_file_cache_refresh_interval_sec | 否       | StarRocks 异步更新自身缓存的 Hudi 表或分区底层数据文件的元数据的时间间隔。单位：秒。默认值： `60`。 |
| metastore_cache_ttl_sec                | 否       | StarRocks 自动丢弃自身缓存的 Hudi 表或分区元数据的时间间隔。单位：秒。默认值： `86400`，即 24 小时。 |
| remote_file_cache_ttl_sec              | 否       | StarRocks 自动丢弃自身缓存的 Hudi 表或分区底层数据文件的元数据的时间间隔。单位：秒。默认值： `129600`，即 36 小时。 |

### 例子

以下示例创建一个名为 `hudi_catalog_hms` 或 `hudi_catalog_glue` 的 Hudi 目录，具体取决于您使用的元存储类型，用于查询 Hudi 群集中的数据。

#### HDFS

如果您使用 HDFS 作为存储，运行以下命令：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

#### AWS S3

##### 如果您选择基于实例配置文件的凭证

- 如果您在 Hudi 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果选择假定的基于角色的凭据

- 如果您在 Hudi 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果选择 IAM 用户凭据

- 如果您在 Hudi 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_glue
  PROPERTIES
  (
      "type" = "hudi",
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
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
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
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果选择 SAS 令牌身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
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
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
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
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",

      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### 谷歌GCS

- 如果选择基于 VM 的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务帐户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果使 VM 实例模拟服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果使一个服务帐户模拟另一个服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 查看Hudi产品目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群下的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `hudi_catalog_glue` 的 Hudi 目录的创建语句：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## 切换到 Hudi 目录和其中的数据库

您可以使用以下方法之一切换到 Hudi 目录和其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 指定当前会话中的 Hudi 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定活动数据库：

  ```SQL
  -- 在当前会话中切换到指定的目录：
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到 Hudi 目录和其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除 Hudi 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `hudi_catalog_glue` 的 Hudi 目录：

```SQL
DROP Catalog hudi_catalog_glue;
```

## 查看 Hudi 表的架构

您可以使用以下语法之一来查看 Hudi 表的架构：

- 查看架构

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 从 CREATE 语句查看架构和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询 Hudi 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Hudi 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到 Hudi 目录和其中的数据库](#switch-to-a-hudi-catalog-and-a-database-in-it)。

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从 Hudi 加载数据

假设您有一个名为 `olap_tbl` 的 OLAP 表，您可以像下面这样转换和加载数据：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks 会缓存 Hudi 的元数据，并以异步模式自动更新元数据，以提供更好的性能。此外，在对 Hudi 表进行一些 Schema 更改或表更新后，您还可以使用 [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) 手动更新其元数据，从而确保 StarRocks 能够第一时间获取到最新的元数据，并生成合适的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略允许 StarRocks 集群中的 FE 从 Hive 元存储中读取事件，例如添加列、删除分区和更新数据。StarRocks 可以根据这些事件自动更新 FE 中缓存的元数据。这意味着您无需手动更新 Hudi 表的元数据。

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

您可以在 FE 日志文件中搜索 `event id`，以检查事件侦听器是否成功配置。如果配置失败，`event id` 的值将为 `0`。

#### 第 2 步：开启 StarRocks 自动增量更新

您可以为 StarRocks 集群中的单个 Hudi 目录或所有 Hudi 目录启用自动增量更新。

- 要为单个 Hudi 目录启用自动增量更新，请在创建 Hudi 目录时将 `enable_hms_events_incremental_sync` 参数设置为 `true`，如下所示：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- 要启用所有 Hudi 目录的自动增量更新，请将 `"enable_hms_events_incremental_sync" = "true"` 添加到每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中，然后重新启动每个 FE 以使参数设置生效。

您还可以根据业务需求在每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中调整以下参数，然后重新启动每个 FE 以使参数设置生效。

| 参数                         | 描述                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks 从 Hive 元存储读取事件的时间间隔。默认值： `5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks 一次可读取的最大事件数。默认值： `500`。 |
| enable_hms_parallel_process_evens | 指定 StarRocks 在读取事件时是否并行处理事件。有效值： `true` 和 `false`。默认值：`true`。该值`true`启用并行性，该值 `false` 禁用并行性。 |
| hms_process_events_parallel_num   | StarRocks 可并行处理的最大事件数。默认值： `4`。 |

## 附录：了解元数据自动异步更新

自动异步更新是 StarRocks 用于更新 Hudi 目录下元数据的默认策略。

默认情况下（即当 `enable_metastore_cache` 和 `enable_remote_file_cache` 参数都设置为 `true`时），如果查询命中 Hudi 表的分区，StarRocks 会自动缓存该分区的元数据和该分区底层数据文件的元数据。缓存的元数据使用延迟更新策略进行更新。

例如，有一个名为 `table2` 的 Hudi 表，它有四个分区：`p1`、 `p2`、 `p3`和 `p4`。查询命中 `p1`，StarRocks 会缓存 `p1` 的元数据和底层数据文件的元数据。假设更新和丢弃缓存元数据的默认时间间隔如下：

- 异步更新缓存元数据 `metastore_cache_refresh_interval_sec` 的时间间隔（由参数指定）为 2 小时。
- 异步更新基础数据文件的缓存元数据 `remote_file_cache_refresh_interval_sec` 的时间间隔（由参数指定）为 60 秒。
- 自动丢弃缓存元数据 `metastore_cache_ttl_sec` 的时间间隔（由参数指定）为 24 小时。
- 自动丢弃基础数据文件的缓存元数据 `remote_file_cache_ttl_sec` 的时间间隔（由参数指定）为 36 小时。

下图显示了时间轴上的时间间隔，以便于理解。

![更新和丢弃缓存元数据的时间线](../../assets/catalog_timeline.png)

然后，StarRocks 会按照以下规则更新或丢弃元数据：


- 如果再次查询命中`p1`，并且距离上次更新的当前时间小于 60 秒，则 StarRocks 不会更新`p1`的缓存元数据，也不会更新`p1`的底层数据文件的缓存元数据。
- 如果再次查询命中`p1`，并且距离上次更新的当前时间超过 60 秒，StarRocks会更新`p1`的底层数据文件的缓存元数据。
- 如果再次查询命中`p1`，并且距离上次更新的当前时间超过 2 小时，StarRocks会更新`p1`的缓存元数据。
- 如果`p1`在上次更新后 24 小时内未被访问，StarRocks会丢弃`p1`的缓存元数据。元数据将在下一次查询时被缓存。
- 如果`p1`在上次更新后 36 小时内未被访问，StarRocks会丢弃`p1`的底层数据文件的缓存元数据。元数据将在下一次查询时被缓存。