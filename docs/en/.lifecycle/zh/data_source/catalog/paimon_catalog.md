---
displayed_sidebar: English
---

# 派蒙目录

StarRocks 从 v3.1 开始支持派蒙目录。

派蒙目录是一种外部目录，可以让您在不进行数据摄取的情况下查询 Apache Paimon 中的数据。

此外，您还可以根据派蒙目录，使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)直接转换和加载派蒙的数据。

为了确保 Paimon 集群上的 SQL 工作负载成功，您的 StarRocks 集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或对象存储，例如 AWS S3、Microsoft Azure 存储、Google GCS 或其他兼容 S3 的存储系统（例如 MinIO）
- 元存储，例如您的文件系统或 Hive 元存储

## 使用说明

您只能使用派蒙目录来查询数据。无法使用派蒙目录在 Paimon 集群中删除、丢弃或插入数据。

## 集成准备

在创建派蒙目录之前，请确保您的 StarRocks 集群可以与 Paimon 集群的存储系统和元存储集成。

### AWS IAM

如果您的 Paimon 集群使用 AWS S3 作为存储，选择合适的身份验证方法并做好必要的准备工作，以确保您的 StarRocks 集群可以访问相关的 AWS 云资源。

建议使用以下身份验证方法：

- 实例配置文件（推荐）
- 承担的角色
- IAM 用户

在上述三种身份验证方法中，实例配置文件是应用最广泛的。

有关更多信息，请参阅 [在 AWS IAM 中准备身份验证](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)。

### HDFS

如果您选择 HDFS 作为存储，请按照以下方式配置您的 StarRocks 集群：

- （可选）设置用于访问您的 HDFS 集群和 Hive 元存储的用户名。默认情况下，StarRocks 使用 FE 和 BE 进程的用户名访问您的 HDFS 集群和 Hive 元存储。您还可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。设置用户名后，请重新启动每个 FE 和每个 BE 以使参数设置生效。每个 StarRocks 集群只能设置一个用户名。
- 查询 Paimon 数据时，您的 StarRocks 集群的 FE 和 BE 使用 HDFS 客户端访问您的 HDFS 集群。在大多数情况下，您无需配置 StarRocks 集群即可实现此目的，StarRocks 会使用默认配置启动 HDFS 客户端。只有在以下情况下，您才需要配置 StarRocks 集群：
  - 为您的 HDFS 集群启用了高可用性（HA）：将您的 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。
  - 为您的 HDFS 集群启用了 View 文件系统（ViewFs）：将您的 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。

> **注意**
>
> 如果在发送查询时返回未知主机错误，请将您的 HDFS 集群节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径中。

### Kerberos 身份验证

如果您的 HDFS 集群或 Hive 元存储启用了 Kerberos 身份验证，请按照以下方式配置您的 StarRocks 集群：

- 在每个 FE 和每个 BE 上运行 `kinit -kt keytab_path principal` 命令，从密钥分发中心（KDC）获取票证授权票（TGT）。运行此命令时，您必须具有访问您的 HDFS 集群和 Hive 元存储的权限。请注意，使用 cron 定期运行此命令，因为使用此命令访问 KDC 是时间敏感的。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改路径。

## 创建派蒙目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "paimon",
    CatalogParams,
    StorageCredentialParams,
)
```

### 参数

#### catalog_name

派蒙目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### comment

派蒙目录的描述。此参数是可选的。

#### type

数据源的类型。将值设置为 `paimon`。

#### CatalogParams

关于 StarRocks 如何访问您的 Paimon 集群元数据的一组参数。

下表描述了您需要在 `CatalogParams` 中配置的参数。

| 参数                | 必填 | 描述                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| paimon.catalog.type      | 是      | 用于您的 Paimon 集群的元存储类型。将此参数设置为 `filesystem` 或 `hive`。 |
| paimon.catalog.warehouse | 是      | 您的 Paimon 数据的仓库存储路径。 |
| hive.metastore.uris      | 否       | 您的 Hive 元存储的 URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。如果为您的 Hive 元存储启用了高可用性（HA），您可以指定多个元存储 URI，并使用逗号（`,`）分隔它们，例如 `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

> **注意**
>
> 如果您使用 Hive 元存储，在查询 Paimon 数据之前，您需要将您的 Hive 元存储节点的主机名和 IP 地址的映射添加到 `/etc/hosts` 路径中。否则，当您发起查询时，StarRocks 可能无法访问您的 Hive 元存储。

#### StorageCredentialParams

关于 StarRocks 如何与您的存储系统集成的一组参数。此参数集是可选的。

如果您使用 HDFS 作为存储，无需配置 `StorageCredentialParams`。

如果您使用 AWS S3、其他兼容 S3 的存储系统、Microsoft Azure 存储或 Google GCS 作为存储，您必须配置 `StorageCredentialParams`。

##### AWS S3

如果您选择 AWS S3 作为您的 Paimon 集群的存储，执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择假定的基于角色的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择 IAM 用户的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                   | 必填 | 描述                                                  |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和假定的基于角色的身份验证方法。有效值：`true` 和 `false`。默认值：`false`。 |
| aws.s3.iam_role_arn         | 否       | 具有权限访问您的 AWS S3 存储桶的 IAM 角色的 ARN。如果您使用假定的基于角色的身份验证方法访问 AWS S3，则必须指定此参数。 |
| aws.s3.region               | 是      | 您的 AWS S3 存储桶所在的区域。示例：`us-west-1`。 |
| aws.s3.access_key           | 否       | 您的 IAM 用户的访问密钥。如果您使用 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | 您的 IAM 用户的私有密钥。如果您使用 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。 |

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储系统

如果您选择 MinIO 等兼容 S3 的存储系统作为您的 Paimon 集群的存储，请按照以下方式配置 `StorageCredentialParams` 以确保成功集成。

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数                       | 必填 | 描述                                                  |
| ------------------------------- | -------- | ------------------------------------------------------------ |

| aws.s3.enable_ssl               | 是       | 指定是否启用 SSL 连接。 <br />有效值： `true` 和 `false`。默认值： `true`。 |
| aws.s3.enable_path_style_access | 是       | 指定是否启用路径样式访问。<br />有效值： `true` 和 `false`。默认值：`false`。对于 MinIO，必须将值设置为 `true`。<br /> 路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`.例如，如果您在美国西部（俄勒冈）区域创建了名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且想要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                 | 是       | 用于连接到与 S3 兼容的存储系统（而不是 AWS S3）的终端节点。 |
| aws.s3.access_key               | 是       | IAM 用户的访问密钥。                             |
| aws.s3.secret_key               | 是       | IAM 用户的私有密钥。                             |

##### Microsoft Azure 存储

###### Azure Blob 存储

如果您选择 Blob 存储作为 Paimon 集群的存储，执行以下操作之一：

- 要选择共享密钥身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                  | 必填 | 描述                                  |
  | -------------------------- | -------- | -------------------------------------------- |
  | azure.blob.storage_account | 是       | Blob 存储帐户的用户名。   |
  | azure.blob.shared_key      | 是       | Blob 存储帐户的共享密钥。 |

- 要选择 SAS 令牌身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                 | 必填 | 描述                                                  |
  | ------------------------- | -------- | ------------------------------------------------------------ |
  | azure.blob.storage_account| 是       | Blob 存储帐户的用户名。                   |
  | azure.blob.container      | 是       | 存储数据的 Blob 容器的名称。        |
  | azure.blob.sas_token      | 是       | 用于访问 Blob 存储帐户的 SAS 令牌。 |

###### Azure Data Lake Storage Gen1

如果您选择 Data Lake Storage Gen1 作为 Paimon 群集的存储，执行以下操作之一：

- 要选择托管服务标识身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                                | 必填 | 描述                                                  |
  | ---------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是       | 指定是否启用托管服务标识身份验证方法。将值设置为 `true`。 |

- 要选择服务主体身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                     | 必填 | 描述                                                  |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是       | 服务主体的客户端（应用程序）ID。        |
  | azure.adls1.oauth2_credential | 是       | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls1.oauth2_endpoint   | 是       | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

###### Azure Data Lake Storage Gen2

如果您选择 Data Lake Storage Gen2 作为 Paimon 群集的存储，执行以下操作之一：

- 要选择托管标识身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                               | 必填 | 描述                                                  |
  | --------------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_use_managed_identity | 是       | 指定是否启用托管标识身份验证方法。将值设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是       | 要访问其数据的租户的 ID。          |
  | azure.adls2.oauth2_client_id            | 是       | 托管标识的客户端（应用程序）ID。         |

- 要选择共享密钥身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                   | 必填 | 描述                                                  |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.storage_account | 是       | Data Lake Storage Gen2 存储帐户的用户名。 |
  | azure.adls2.shared_key      | 是       | Data Lake Storage Gen2 存储帐户的共享密钥。 |

- 要选择服务主体身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                          | 必填 | 描述                                                  |
  | ---------------------------------- | -------- | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是       | 服务主体的客户端（应用程序）ID。        |
  | azure.adls2.oauth2_client_secret   | 是       | 创建的新客户端（应用程序）机密的值。    |
  | azure.adls2.oauth2_client_endpoint | 是       | 服务主体或应用程序的 OAuth 2.0 令牌终结点 （v1）。 |

##### 谷歌GCS

如果您选择 Google GCS 作为 Paimon 集群的存储，执行以下操作之一：

- 要选择基于 VM 的身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                                  | 默认值 | 值示例 | 描述                                                  |
  | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
  | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |

- 要选择基于服务帐户的身份验证方法，请配置 `StorageCredentialParams` 如下：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数                              | 默认值 | 值示例                                                | 描述                                                  |
  | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 创建服务帐户时生成的 JSON 文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 创建服务帐户时生成的 JSON 文件中的私钥 ID。 |
  | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 创建服务帐户时生成的 JSON 文件中的私钥。 |

- 要选择基于模拟的身份验证方法，请配置 `StorageCredentialParams` 如下：

  - 使 VM 实例模拟服务帐户：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | 参数                                  | 默认值 | 值示例 | 描述                                                  |
    | ------------------------------------------ | ------------- | ------------- | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | FALSE         | TRUE          | 指定是否直接使用与 Compute Engine 绑定的服务帐号。 |
    | gcp.gcs.impersonation_service_account      | ""            | "hello"       | 要模拟的服务帐户。            |

  - 使服务帐户（暂时命名为元服务帐户）模拟另一个服务帐户（暂时称为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | 参数                              | 默认值 | 值示例                                                | 描述                                                  |
    | -------------------------------------- | ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""            | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""            | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 在创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key    | ""            | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 在创建元服务帐户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""            | "hello"                                                      | 您想要模拟的数据服务帐户。       |

### 示例

以下示例将创建一个名为 `paimon_catalog_fs` 的 Paimon 目录，其元存储类型 `paimon.catalog.type` 设置为 `filesystem`，以从您的 Paimon 集群中查询数据。

#### AWS S3

- 如果您选择基于实例配置文件的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您选择假定角色为基础的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您选择基于 IAM 用户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<s3_paimon_warehouse_path>",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

#### 兼容 S3 的存储系统

以 MinIO 为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<paimon_warehouse_path>",
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
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果选择 SAS 令牌身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<blob_paimon_warehouse_path>",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- 如果选择托管服务标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.use_managed_service_identity" = "true"
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls1_paimon_warehouse_path>",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- 如果选择托管标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<adls2_paimon_warehouse_path>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果选择基于 VM 的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.use_compute_engine_service_account" = "true"
  );
  ```

- 如果选择基于服务帐户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG paimon_catalog_fs
  PROPERTIES
  (
      "type" = "paimon",
      "paimon.catalog.type" = "filesystem",
      "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果使 VM 实例模拟服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    );
    ```

  - 如果使一个服务帐户模拟另一个服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG paimon_catalog_fs
    PROPERTIES
    (
        "type" = "paimon",
        "paimon.catalog.type" = "filesystem",
        "paimon.catalog.warehouse" = "<gcs_paimon_warehouse_path>",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```

## 查看 Paimon 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `paimon_catalog_fs` 的 Paimon 目录的创建语句：

```SQL
SHOW CREATE CATALOG paimon_catalog_fs;
```

## 删除 Paimon 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `paimon_catalog_fs` 的 Paimon 目录：

```SQL
DROP Catalog paimon_catalog_fs;
```

## 查看 Paimon 表的模式

您可以使用以下语法之一查看 Paimon 表的模式：

- 查看模式

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>;
  ```

- 从 CREATE 语句查看模式和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>;
  ```

## 查询 Paimon 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Paimon 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>;
   ```

2. 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中切换到目标目录：

   ```SQL
   SET CATALOG <catalog_name>;
   ```

   然后，使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 来指定当前会话中的活动数据库：

   ```SQL
   USE <db_name>;
   ```

   或者，您可以使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 直接指定目标目录中的活动数据库：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 来查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## 从派蒙加载数据

假设您有一个名为 `olap_tbl` 的 OLAP 表，您可以像下面这样转换和加载数据：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;