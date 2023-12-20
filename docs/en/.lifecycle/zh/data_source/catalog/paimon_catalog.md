---
displayed_sidebar: English
---

# Paimon 目录

StarRocks 从 v3.1 版本开始支持 Paimon 目录。

Paimon 目录是一种外部目录，允许您在无需数据摄入的情况下，从 Apache Paimon 查询数据。

此外，您还可以直接使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 基于 Paimon 目录来转换和加载来自 Paimon 的数据。

为了确保您的 Paimon 集群上的 SQL 工作负载成功，您的 StarRocks 集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或对象存储，如 AWS S3、Microsoft Azure Storage、Google GCS 或其他兼容 S3 的存储系统（例如 MinIO）
- 元数据存储，如文件系统或 Hive metastore

## 使用说明

您只能使用 Paimon 目录来查询数据。您不能使用 Paimon 目录来删除、删除或插入数据到您的 Paimon 集群中。

## 集成准备

在创建 Paimon 目录之前，请确保您的 StarRocks 集群能够与您的 Paimon 集群的存储系统和元数据存储集成。

### AWS IAM

如果您的 Paimon 集群使用 AWS S3 作为存储，请选择合适的认证方法并做好必要的准备，以确保您的 StarRocks 集群可以访问相关的 AWS 云资源。

推荐以下认证方法：

- 实例配置文件（推荐）
- 假定角色
- IAM 用户

在上述三种认证方法中，实例配置文件是最广泛使用的。

有关更多信息，请参见 [AWS IAM 中的认证准备](../../integrations/authenticate_to_aws_resources.md#preparation-for-authentication-in-aws-iam)。

### HDFS

如果您选择 HDFS 作为存储，请按以下方式配置您的 StarRocks 集群：

- （可选）设置用于访问您的 HDFS 集群和 Hive metastore 的用户名。默认情况下，StarRocks 使用 FE 和 BE 进程的用户名来访问您的 HDFS 集群和 Hive metastore。您也可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件开头和每个 BE 的 **be/conf/hadoop_env.sh** 文件开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。设置用户名后，需要重启每个 FE 和 BE 以使参数设置生效。您只能为每个 StarRocks 集群设置一个用户名。
- 当您查询 Paimon 数据时，您的 StarRocks 集群的 FE 和 BE 使用 HDFS 客户端访问您的 HDFS 集群。在大多数情况下，您不需要配置 StarRocks 集群即可实现此目的，StarRocks 会使用默认配置启动 HDFS 客户端。只有在以下情况下，您才需要配置 StarRocks 集群：
  - 如果为您的 HDFS 集群启用了高可用性（HA）：将您的 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。
  - 如果为您的 HDFS 集群启用了 View File System（ViewFs）：将您的 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径。

> **注意**
> 如果在发送查询时返回未知主机错误，您必须将您的 HDFS 集群节点的主机名和 IP 地址的映射添加到 **/etc/hosts** 路径中。

### Kerberos 认证

如果为您的 HDFS 集群或 Hive metastore 启用了 Kerberos 认证，请按以下方式配置您的 StarRocks 集群：

- 在每个 FE 和 BE 上运行 `kinit -kt keytab_path principal` 命令，从 Key Distribution Center (KDC) 获取 Ticket Granting Ticket (TGT)。要运行此命令，您必须有权限访问您的 HDFS 集群和 Hive metastore。请注意，使用此命令访问 KDC 是有时间限制的。因此，您需要使用 cron 定期运行此命令。
- 将 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"` 添加到每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改该路径。

## 创建 Paimon 目录

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

Paimon 目录的名称。命名规则如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### comment

Paimon 目录的描述。此参数是可选的。

#### type

您的数据源类型。将值设置为 `paimon`。

#### CatalogParams

一组参数，关于 StarRocks 如何访问您的 Paimon 集群的元数据。

下表描述了您需要在 `CatalogParams` 中配置的参数。

|参数|必需|描述|
|---|---|---|
|paimon.catalog.type|是|您用于 Paimon 集群的元存储类型。将此参数设置为 `filesystem` 或 `hive`。|
|paimon.catalog.warehouse|是|您的 Paimon 数据的仓库存储路径。|
|hive.metastore.uris|否|您的 Hive metastore 的 URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。如果为您的 Hive metastore 启用了高可用性（HA），您可以指定多个 metastore URI 并用逗号（,）分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。|

> **注意**
> 如果您使用 Hive metastore，必须在查询 Paimon 数据之前将 Hive metastore 节点的主机名和 IP 地址的映射添加到 `/etc/hosts` 路径中。否则，当您开始查询时，StarRocks 可能无法访问您的 Hive metastore。

#### StorageCredentialParams

一组参数，关于 StarRocks 如何与您的存储系统集成。此参数集是可选的。

如果您使用 HDFS 作为存储，则不需要配置 `StorageCredentialParams`。

如果您使用 AWS S3、其他 S3 兼容存储系统、Microsoft Azure Storage 或 Google GCS 作为存储，则必须配置 `StorageCredentialParams`。

##### AWS S3

如果您选择 AWS S3 作为您的 Paimon 集群的存储，请采取以下操作之一：

- 要选择基于实例配置文件的认证方法，请如下配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于假定角色的认证方法，请如下配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于 IAM 用户的认证方法，请如下配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|参数|必需|描述|
|---|---|---|
|aws.s3.use_instance_profile|是|指定是否启用基于实例配置文件的认证方法和基于假定角色的认证方法。有效值：`true` 和 `false`。默认值：`false`。|
|aws.s3.iam_role_arn|否|拥有您的 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您使用基于假定角色的认证方法来访问 AWS S3，则必须指定此参数。|
|aws.s3.region|是|您的 AWS S3 存储桶所在的区域。例如：`us-west-1`。|
|aws.s3.access_key|否|您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS S3，则必须指定此参数。|
|aws.s3.secret_key|否|您的 IAM 用户的密钥。如果您使用基于 IAM 用户的认证方法来访问 AWS S3，则必须指定此参数。|

有关如何选择访问 AWS S3 的认证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参见 [访问 AWS S3 的认证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储系统

如果您选择了兼容 S3 的存储系统（例如 MinIO）作为您的 Paimon 集群的存储，请按以下方式配置 `StorageCredentialParams` 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```
```
下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

|参数|必填|说明|
|---|---|---|
|aws.s3.enable_ssl|是|指定是否启用 SSL 连接。有效值：`true` 和 `false`。默认值：`true`。|
|aws.s3.enable_path_style_access|是|指定是否启用路径样式访问。有效值：`true` 和 `false`。默认值：`false`。对于 MinIO，您必须将该值设置为 `true`。路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且想要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
|aws.s3.endpoint|是|用于连接到 S3 兼容存储系统而不是 AWS S3 的终端节点。|
|aws.s3.access_key|是|您的 IAM 用户的访问密钥。|
|aws.s3.secret_key|是|您的 IAM 用户的密钥。|

##### Microsoft Azure 存储

###### Azure Blob 存储

如果您选择 Blob 存储作为 Paimon 集群的存储，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.blob.storage_account|是|Blob 存储账户的用户名。|
  |azure.blob.shared_key|是|Blob 存储账户的共享密钥。|

- 要选择 SAS 令牌身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.blob.storage_account|是|Blob 存储账户的用户名。|
  |azure.blob.container|是|存储数据的 Blob 容器的名称。|
  |azure.blob.sas_token|是|用于访问 Blob 存储账户的 SAS 令牌。|

###### Azure Data Lake Storage Gen1

如果您选择 Data Lake Storage Gen1 作为 Paimon 集群的存储，请执行以下操作之一：

- 要选择托管服务身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.adls1.use_managed_service_identity|是|指定是否启用托管服务身份验证方法。将值设置为 `true`。|

- 要选择服务主体身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.adls1.oauth2_client_id|是|服务主体的客户端（应用程序）ID。|
  |azure.adls1.oauth2_credential|是|创建的新客户端（应用程序）密钥的值。|
  |azure.adls1.oauth2_endpoint|是|服务主体或应用程序的 OAuth 2.0 令牌端点（v2）。|

###### Azure Data Lake Storage Gen2

如果您选择 Data Lake Storage Gen2 作为 Paimon 集群的存储，请执行以下操作之一：

- 要选择托管身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.adls2.oauth2_use_managed_identity|是|指定是否启用托管身份验证方法。将值设置为 `true`。|
  |azure.adls2.oauth2_tenant_id|是|要访问其数据的租户的 ID。|
  |azure.adls2.oauth2_client_id|是|托管身份的客户端（应用程序）ID。|

- 要选择共享密钥身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.adls2.storage_account|是|Data Lake Storage Gen2 存储账户的用户名。|
  |azure.adls2.shared_key|是|Data Lake Storage Gen2 存储账户的共享密钥。|

- 要选择服务主体身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|必填|说明|
|---|---|---|
  |azure.adls2.oauth2_client_id|是|服务主体的客户端（应用程序）ID。|
  |azure.adls2.oauth2_client_secret|是|创建的新客户端（应用程序）密钥的值。|
  |azure.adls2.oauth2_client_endpoint|是|服务主体或应用程序的 OAuth 2.0 令牌端点（v1）。|

##### Google GCS

如果您选择 Google GCS 作为 Paimon 集群的存储，请执行以下操作之一：

- 要选择基于虚拟机的身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|默认值|值示例|说明|
|---|---|---|---|
  |gcp.gcs.use_compute_engine_service_account|`FALSE`|`TRUE`|指定是否直接使用绑定到您的 Compute Engine 的服务账户。|

- 要选择基于服务账户的身份验证方法，请配置 `StorageCredentialParams`，如下所示：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  |参数|默认值|值示例|说明|
|---|---|---|---|
  |gcp.gcs.service_account_email|""|`user@hello.iam.gserviceaccount.com`|创建服务账户时生成的 JSON 文件中的电子邮件地址。|
  |gcp.gcs.service_account_private_key_id|""|`61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea`|创建服务账户时生成的 JSON 文件中的私钥 ID。|
  |gcp.gcs.service_account_private_key|""|`-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n`|创建服务账户时生成的 JSON 文件中的私钥。|

- 要选择基于模拟的身份验证方法，请配置 `StorageCredentialParams`，如下所示：

-   让 VM 实例模拟服务账户：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    |参数|默认值|值示例|说明|
|---|---|---|---|
    |gcp.gcs.use_compute_engine_service_account|`FALSE`|`TRUE`|指定是否直接使用绑定到您的 Compute Engine 的服务账户。|
    |gcp.gcs.impersonation_service_account|""|`hello`|您要模拟的服务账户。|

-   让一个服务账户（暂时命名为元服务账户）模拟另一个服务账户（暂时命名为数据服务账户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    |参数|默认值|值示例|说明|
|---|---|---|---|
    |gcp.gcs.service_account_email|""|`user@hello.iam.gserviceaccount.com`|创建元服务账户时生成的 JSON 文件中的电子邮件地址。|
    |gcp.gcs.service_account_private_key_id|""|`61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea`|创建元服务账户时生成的 JSON 文件中的私钥 ID。|
    |gcp.gcs.service_account_private_key|""|`-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n`|创建元服务账户时生成的 JSON 文件中的私钥。|
    |gcp.gcs.impersonation_service_account|""|`hello`|您要模拟的数据服务账户。|

### 示例
以下示例创建一个名为 `paimon_catalog_fs` 的 Paimon 目录，其元存储类型 `paimon.catalog.type` 设置为 `filesystem`，以从 Paimon 集群查询数据。

#### AWS S3

- 如果您选择基于实例配置文件的身份验证方法，请运行如下命令：

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

- 如果您选择基于角色的身份验证方法，请运行如下命令：

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

- 如果您选择基于 IAM 用户的身份验证方法，请运行如下命令：

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

#### S3 兼容存储系统

以 MinIO 为例。运行如下命令：

```SQL
CREATE EXTERNAL CATALOG paimon_catalog_fs
PROPERTIES
(
    "type" = "paimon",
    "paimon.catalog.type" = "filesystem",
    "paimon.catalog.warehouse" = "<paimon_warehouse_path>",
    "aws.s3.enable_ssl" = "false",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### 微软 Azure 存储

##### Azure Blob 存储

- 如果您选择共享密钥身份验证方法，请运行如下命令：

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

- 如果您选择 SAS 令牌身份验证方法，请运行如下命令：

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

- 如果您选择托管服务身份验证方法，请运行如下命令：

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

- 如果您选择服务主体身份验证方法，请运行如下命令：

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

- 如果您选择托管身份验证方法，请运行如下命令：

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

- 如果您选择共享密钥身份验证方法，请运行如下命令：

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

- 如果您选择服务主体身份验证方法，请运行如下命令：

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

#### 谷歌 GCS

- 如果您选择基于虚拟机的身份验证方法，请运行如下命令：

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

- 如果您选择基于服务账户的身份验证方法，请运行如下命令：

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

- 如果您选择基于模拟的身份验证方法：

    - 如果您让 VM 实例模拟服务账户，请运行如下命令：

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

    - 如果您让一个服务账户模拟另一个服务账户，请运行如下命令：

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
DROP CATALOG paimon_catalog_fs;
```

## 查看 Paimon 表的架构

您可以使用以下语法之一来查看 Paimon 表的架构：

- 查看架构

  ```SQL
  DESCRIBE <catalog_name>.<database_name>.<table_name>;
  ```

- 从 CREATE 语句查看架构和位置

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

   然后，使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定当前会话中的活动数据库：

   ```SQL
   USE <db_name>;
   ```
或者，您可以使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 直接指定目标目录中的活跃数据库：

   ```SQL
   USE <catalog_name>.<db_name>;
   ```

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 在指定数据库中查询目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10;
   ```

## 从 Paimon 加载数据

假设您有一个名为 `olap_tbl` 的 OLAP 表，您可以像下面这样转换和加载数据：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM paimon_table;
```