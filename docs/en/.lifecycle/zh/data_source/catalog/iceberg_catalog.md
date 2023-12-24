---
displayed_sidebar: English
toc_max_heading_level: 3
---
从“@theme/Tabs”导入选项卡;
从 '@theme/TabItem' 导入 TabItem;

# 冰山目录

Iceberg 目录是 StarRocks 从 v2.4 开始支持的一种外部目录。使用 Iceberg 目录，您可以：

- 直接查询存储在 Iceberg 中的数据，无需手动创建表。
- 使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 或异步物化视图（从 v2.5 开始支持）处理存储在 Iceberg 中的数据，并将数据加载到 StarRocks 中。
- 在 StarRocks 上执行创建或删除 Iceberg 数据库和表的操作，或者使用 [INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 将 StarRocks 表中的数据下沉到 Parquet 格式的 Iceberg 表中（从 v3.1 开始支持此功能）。

为了确保 Iceberg 集群上的 SQL 工作负载成功，StarRocks 集群需要与两个重要组件集成：

- 分布式文件系统 （HDFS） 或对象存储，如 AWS S3、Microsoft Azure 存储、Google GCS 或其他与 S3 兼容的存储系统（例如 MinIO）

- 元存储，如 Hive 元存储、AWS Glue 或 Tabular

:::注意

- 如果您选择 AWS S3 作为存储，则可以使用 HMS 或 AWS Glue 作为元存储。如果选择其他存储系统，则只能使用 HMS 作为元存储。
- 如果选择 Tabular 作为元存储，则需要使用 Iceberg REST 目录。

:::

## 使用说明

- StarRocks 支持的 Iceberg 文件格式为 Parquet 和 ORC：

  - Parquet 文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP 和 NO_COMPRESSION。
  - ORC 文件支持以下压缩格式：ZLIB、SNAPPY、LZO、LZ4、ZSTD 和 NO_COMPRESSION。

- Iceberg 目录支持 v1 表，从 StarRocks v3.0 开始支持 ORC 格式的 v2 表。
- Iceberg 目录支持 v1 表。此外，Iceberg 目录支持 StarRocks v3.0 及以上的 ORC 格式的 v2 表，以及 StarRocks v3.1 及以上的 Parquet 格式的 v2 表。

## 集成准备

在创建 Iceberg 目录之前，请确保您的 StarRocks 集群可以与 Iceberg 集群的存储系统和元存储集成。

---

### 存储

选择与您的存储类型匹配的选项卡：

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

如果您的 Iceberg 集群使用 AWS S3 作为存储，或者使用 AWS Glue 作为元存储，请选择合适的身份验证方式并做好必要的准备工作，以确保您的 StarRocks 集群能够访问相关的 AWS 云资源。

建议使用以下身份验证方法：

- 实例配置文件
- 承担的角色
- IAM 用户

在上述三种身份验证方法中，实例配置文件是应用最广泛的。

有关更多信息，请参阅 [在 AWS IAM 中准备身份验证](../../integrations/authenticate_to_aws_resources.md#preparations)。

</TabItem>

<TabItem value="HDFS" label="HDFS" >

如果您选择 HDFS 作为存储，请按如下方式配置 StarRocks 集群：

- （可选）设置用于访问 HDFS 集群和 Hive 元存储的用户名。StarRocks 默认使用 FE 和 BE 进程的用户名访问 HDFS 集群和 Hive 元存储。也可以通过在每个 FE 的 **fe/conf/hadoop_env.sh** 文件的开头和每个 BE 的 **be/conf/hadoop_env.sh** 文件的开头添加 `export HADOOP_USER_NAME="<user_name>"` 来设置用户名。设置用户名后，重启每个 FE 和每个 BE，使参数设置生效。每个 StarRocks 集群只能设置一个用户名。
- 当您查询 Iceberg 数据时，StarRocks 集群的 FE 和 BE 会使用 HDFS 客户端访问 HDFS 集群。大多数情况下，您不需要配置 StarRocks 集群来达到这个目的，StarRocks 会使用默认配置来启动 HDFS 客户端。只有在以下情况下，您才需要配置 StarRocks 集群：

  - 开启 HDFS 集群高可用 （HA）：将 HDFS 集群的 **hdfs-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径中。
  - 开启 HDFS 集群的 View 文件系统 （ViewFs）：将 HDFS 集群的 **core-site.xml** 文件添加到每个 FE 的 **$FE_HOME/conf** 路径和每个 BE 的 **$BE_HOME/conf** 路径中。

:::提示

如果发送查询时返回未知主机的错误，则需要将 HDFS 集群节点的主机名和 IP 地址之间的映射添加到 **/etc/hosts** 路径中。

:::

---

#### Kerberos 身份验证

如果您的 HDFS 集群或 Hive 元存储开启了 Kerberos 认证，请按如下方式配置 StarRocks 集群：

- 在每个 FE 和每个 BE 上运行命令 `kinit -kt keytab_path principal`，从密钥分发中心（KDC）获取 Ticket Granting Ticket（TGT）。若要运行此命令，您必须具有访问 HDFS 群集和 Hive 元存储的权限。请注意，使用此命令访问 KDC 是时间敏感的。因此，您需要使用 cron 定期运行此命令。
- 在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件和每个 BE 的 **$BE_HOME/conf/be.conf** 文件中添加 `JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf` 是 **krb5.conf** 文件的保存路径。您可以根据需要修改路径。

</TabItem>

</Tabs>

---

## 创建 Iceberg 目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "iceberg",
    MetastoreParams,
    StorageCredentialParams
)
```

---

### 参数

#### catalog_name

Iceberg 目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过 1023 个字符。

#### 评论

Iceberg 目录的描述。此参数是可选的。

#### 类型

数据源的类型。将值设置为 `iceberg`。

#### MetastoreParams

一组关于 StarRocks 如何与数据源的元存储集成的参数。选择与元存储类型匹配的选项卡：

<Tabs groupId="metastore">
<TabItem value="HIVE" label="Hive metastore" default>

##### Hive 元存储

如果选择 Hive 元存储作为数据源的元存储，请按如下方式进行配置 `MetastoreParams` ：

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

:::注意

在查询 Iceberg 数据之前，您需要将 Hive 元存储节点的主机名和 IP 地址之间的映射添加到路径中 `/etc/hosts`。否则，StarRocks 可能会在您发起查询时无法访问您的 Hive 元存储。

:::

下表描述了您需要在 `MetastoreParams` 中配置的参数。

##### iceberg.catalog.type

必需：是
描述：用于 Iceberg 群集的元存储类型。将值设置为 `hive`。

##### hive.metastore.uris

必需：是
说明：Hive 元存储的 URI。格式： `thrift://<metastore_IP_address>:<metastore_port>`。如果为 Hive 元存储启用了高可用性（HA），则可以指定多个元存储 URI，并用逗号（,）分隔它们，例如 `"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。

</TabItem>
<TabItem value="GLUE" label="AWS Glue">

##### AWS Glue

如果您选择 AWS Glue 作为数据源的元存储（仅当您选择 AWS S3 作为存储时才支持），请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择假定的基于角色的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择 IAM 基于用户的身份验证方法，`MetastoreParams` 请按如下方式进行配置：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

`MetastoreParams` 对于 AWS Glue：

###### iceberg.catalog.type

必需：是
描述：用于 Iceberg 群集的元存储类型。将值设置为 `glue`。

###### aws.glue.use_instance_profile

必需：是
描述：指定是否启用基于实例配置文件的身份验证方法和假定的基于角色的身份验证方法。有效值：`true` 和 `false`。默认值：`false`。

###### aws.glue.iam_role_arn

必需：否
描述：具有权限访问您的 AWS Glue 数据目录的 IAM 角色的 ARN。如果您使用基于假定角色的身份验证方法访问 AWS Glue，则必须指定此参数。 

###### aws.glue.region

必填：是
描述：您的 AWS Glue 数据目录所在的区域。示例： `us-west-1`. 

###### aws.glue.access_key

必填：否
描述：您的 AWS IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS Glue，则必须指定此参数。

###### aws.glue.secret_key

必填：否
描述：您的 AWS IAM 用户的私有密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS Glue，则必须指定此参数。

有关如何选择用于访问 AWS Glue 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS Glue 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

</TabItem>
<TabItem value="TABULAR" label="表格式的">

##### 表格式的

如果您将 Tabular 用作元存储，则必须将元存储类型指定为 REST（`"iceberg.catalog.type" = "rest"`）。配置 `MetastoreParams` 如下：

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

Tabular 的 `MetastoreParams`：

###### iceberg.catalog.type

必填：是
描述：您在 Iceberg 群集中使用的元存储类型。将值设置为 `rest`。

###### iceberg.catalog.uri

必填：是
描述：Tabular 服务终结点的 URI。示例： `https://api.tabular.io/ws`.      

###### iceberg.catalog.credential

必填：是
描述：Tabular 服务的身份验证信息。

###### iceberg.catalog.warehouse

必填：否
描述：Iceberg 目录的仓库位置或标识符。示例： `s3://my_bucket/warehouse_location` 或 `sandbox`。

以下示例创建了一个名为 `tabular` 的 Iceberg 目录，该目录使用 Tabular 作为元存储：

```SQL
CREATE EXTERNAL CATALOG tabular
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "rest",
    "iceberg.catalog.uri" = "https://api.tabular.io/ws",
    "iceberg.catalog.credential" = "t-5Ii8e3FIbT9m0:aaaa-3bbbbbbbbbbbbbbbbbbb",
    "iceberg.catalog.warehouse" = "sandbox"
);
```
</TabItem>

</Tabs>

---

#### `StorageCredentialParams`

一组关于 StarRocks 如何与您的存储系统集成的参数。此参数集是可选的。

请注意以下几点：

- 如果您使用 HDFS 作为存储，则无需配置 `StorageCredentialParams`，可以跳过此部分。如果您使用 AWS S3、其他兼容 S3 的存储系统、Microsoft Azure 存储或 Google GCS 作为存储，则必须配置 `StorageCredentialParams`。

- 如果您使用 Tabular 作为元存储，则无需配置 `StorageCredentialParams`，可以跳过此部分。如果您使用 HMS 或 AWS Glue 作为元存储，则必须配置 `StorageCredentialParams`。

选择与您的存储类型匹配的选项卡：

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

##### AWS S3

如果您选择 AWS S3 作为 Iceberg 集群的存储，执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择假定角色身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于 IAM 用户的身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

AWS S3 的 `StorageCredentialParams`：

###### aws.s3.use_instance_profile

必填：是
描述：指定是否启用基于实例配置文件的身份验证方法和假定角色身份验证方法。有效值： `true` 和 `false`。默认值： `false`。 

###### aws.s3.iam_role_arn

必填：否
描述：具有权限访问您的 AWS S3 存储桶的 IAM 角色的 ARN。如果您使用假定角色身份验证方法访问 AWS S3，则必须指定此参数。 

###### aws.s3.region

必填：是
描述：您的 AWS S3 存储桶所在的区域。示例： `us-west-1`. 

###### aws.s3.access_key

必填：否
描述：IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。

###### aws.s3.secret_key

必填：否
描述：IAM 用户的私有密钥。如果您使用基于 IAM 用户的身份验证方法访问 AWS S3，则必须指定此参数。

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

</TabItem>

<TabItem value="HDFS" label="HDFS" >

使用 HDFS 存储时，请跳过存储凭据。

</TabItem>

<TabItem value="MINIO" label="MinIO" >

##### 兼容 S3 的存储系统

Iceberg 目录从 v2.5 开始支持兼容 S3 的存储系统。

如果您选择兼容 S3 的存储系统（如 MinIO）作为 Iceberg 集群的存储，按如下方式配置 `StorageCredentialParams` 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

MinIO 和其他 S3 兼容系统的 `StorageCredentialParams`：

###### aws.s3.enable_ssl

必填：是
描述：指定是否启用 SSL 连接。<br />有效值： `true` 和 `false`。默认值： `true`。

###### aws.s3.enable_path_style_access

必填：是
描述：指定是否启用路径样式访问。<br />有效值： `true` 和 `false`。默认值：`false`。对于 MinIO，必须将值设置为 `true`。<br />路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建了名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且想要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。

###### aws.s3.endpoint

必填：是
描述：用于连接到与 S3 兼容的存储系统而不是 AWS S3 的终端节点。

###### aws.s3.access_key

必填：是
描述：IAM 用户的访问密钥。

###### aws.s3.secret_key

必填：是
描述：IAM 用户的私有密钥。

</TabItem>

<TabItem value="AZURE" label="Microsoft Azure Blob Storage" >

##### Microsoft Azure 存储

Iceberg 目录从 v3.0 开始支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果选择“Blob 存储”作为 Iceberg 集群的存储，执行以下操作之一：

- 要选择共享密钥身份验证方法，请按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

- 若要选择 SAS 令牌身份验证方法，按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

Microsoft Azure 的 `StorageCredentialParams`：

###### azure.blob.storage_account

必填：是
描述：Blob 存储帐户的用户名。

###### azure.blob.shared_key

必填：是
描述：Blob 存储帐户的共享密钥。

###### azure.blob.account_name

必填：是
描述：Blob 存储帐户的用户名。

###### azure.blob.container

必填：是
描述：存储数据的 Blob 容器的名称。

###### azure.blob.sas_token

必填：是
描述：用于访问 Blob 存储帐户的 SAS 令牌。

###### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为 Iceberg 集群的存储，执行以下操作之一：

- 若要选择托管服务标识身份验证方法，按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

或：

- 若要选择服务主体身份验证方法，按如下方式配置 `StorageCredentialParams` ：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

###### Azure Data Lake Storage Gen2


如果选择 Data Lake Storage Gen2 作为 Iceberg 群集的存储，请执行以下操作之一：

- 若要选择托管标识身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  或：

- 若要选择共享密钥身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  或：

- 若要选择服务主体身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

</TabItem>

<TabItem value="GCS" label="Google GCS" >

##### Google GCS

Iceberg 目录从 v3.0 开始支持 Google GCS。

如果选择 Google GCS 作为 Iceberg 群集的存储，请执行以下操作之一：

- 若要选择基于 VM 的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

- 若要选择基于服务帐户的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

- 若要选择基于模拟的身份验证方法，请按以下方式配置 `StorageCredentialParams`：

  - 使 VM 实例模拟服务帐户：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

  - 使服务帐户（暂时命名为元服务帐户）模拟另一个服务帐户（暂时命名为数据服务帐户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

`StorageCredentialParams` 对于 Google GCS：

###### gcp.gcs.service_account_email

默认值: ""
示例: "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)"
说明: 在创建服务帐户时生成的 JSON 文件中的电子邮件地址。

###### gcp.gcs.service_account_private_key_id

默认值: ""
示例: "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"
说明: 在创建服务帐户时生成的 JSON 文件中的私钥 ID。

###### gcp.gcs.service_account_private_key

默认值: ""
示例: "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  
说明: 在创建服务帐户时生成的 JSON 文件中的私钥。

###### gcp.gcs.impersonation_service_account

默认值: ""  
示例: "hello"  
说明: 要模拟的服务帐户。

</TabItem>

</Tabs>

---

### 例子

以下示例创建一个名为 `iceberg_catalog_hms` 或 `iceberg_catalog_glue` 的 Iceberg 目录，具体取决于您使用的元存储类型，以查询 Iceberg 群集中的数据。选择与您的存储类型匹配的选项卡：

<Tabs groupId="storage">
<TabItem value="AWS" label="AWS S3" default>

#### AWS S3

##### 如果您选择基于实例配置文件的凭证

- 如果在 Iceberg 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果选择假定的基于角色的凭据

- 如果在 Iceberg 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 如果选择 IAM 基于用户的凭证

- 如果在 Iceberg 群集中使用 Hive 元存储，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_glue
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "glue",
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
</TabItem>

<TabItem value="HDFS" label="HDFS" >

#### HDFS

如果您使用 HDFS 作为存储，请运行以下命令：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

</TabItem>

<TabItem value="MINIO" label="MinIO" >

#### 兼容 S3 的存储系统

以 MinIO 为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```
</TabItem>

<TabItem value="AZURE" label="Microsoft Azure Blob Storage" >

#### Microsoft Azure 存储

##### Azure Blob 存储

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果选择 SAS 令牌身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.container" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- 如果选择托管服务标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",

      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- 如果选择托管标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

</TabItem>

<TabItem value="GCS" label="Google GCS" >

#### Google GCS

- 如果选择基于 VM 的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务帐户的身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果使 VM 实例模拟服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果使一个服务帐户模拟另一个服务帐户，请运行以下命令：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```
</TabItem>

</Tabs>
 
 ---

## 使用您的目录

### 查看 Iceberg 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `iceberg_catalog_glue` 的 Iceberg 目录的创建语句：

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

---

### 切换到 Iceberg 目录和其中的数据库

您可以使用以下方法之一切换到 Iceberg 目录和其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 指定当前会话中的 Iceberg 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定活动数据库：

  ```SQL
  -- 切换到当前会话中的指定目录：
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到 Iceberg 目录和其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```
 
---

### 删除 Iceberg 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `iceberg_catalog_glue` 的 Iceberg 目录：

```SQL
DROP Catalog iceberg_catalog_glue;
```

---

### 查看 Iceberg 表的结构

您可以使用以下语法之一来查看 Iceberg 表的结构：

- 查看架构

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 从 CREATE 语句查看架构和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

---

### 查询 Iceberg 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Iceberg 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到 Iceberg 目录和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)。

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

---

### 创建 Iceberg 数据库

与 StarRocks 的内部目录类似，如果您拥有对 Iceberg 目录的 [CREATE DATABASE](../../administration/privilege_item.md#catalog) 权限，则可以使用 [CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 语句在该 Iceberg 目录中创建数据库。此功能从 v3.1 版本开始支持。

:::tip

您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 授予和撤消权限。

:::

[切换到 Iceberg 目录](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句在该目录中创建 Iceberg 数据库：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

您可以使用 `location` 参数指定要在其中创建数据库的文件路径。支持 HDFS 和云存储。如果不指定 `location` 参数，StarRocks 会在 Iceberg 目录的默认文件路径下创建数据库。

`prefix` 根据您使用的存储系统而变化：

#### HDFS

`prefix` 值： `hdfs`

#### 谷歌GCS

`prefix` 值： `gs`

#### Azure Blob 存储

`prefix` 值：

- 如果您的存储帐户允许通过 HTTP 进行访问，则 `prefix` 为 `wasb`。
- 如果您的存储帐户允许通过 HTTPS 进行访问，则 `prefix` 为 `wasbs`。

#### Azure Data Lake Storage Gen1

`prefix` 值： `adl`

#### Azure Data Lake Storage Gen2

`prefix` 值：

- 如果您的存储帐户允许通过 HTTP 进行访问，则 `prefix` 为 `abfs`。
- 如果您的存储帐户允许通过 HTTPS 进行访问，则 `prefix` 为 `abfss`。

#### AWS S3 或其他兼容 S3 存储（例如，MinIO）

`prefix` 值： `s3`

---

### 删除 Iceberg 数据库

与 StarRocks 的内部数据库类似，如果您对 Iceberg 数据库有 [DROP](../../administration/privilege_item.md#database) 权限，则可以使用 [DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) 语句删除该 Iceberg 数据库。此功能从 v3.1 版本开始支持。只能删除空数据库。

删除 Iceberg 数据库时，数据库在 HDFS 集群或云存储上的文件路径不会随数据库一起删除。

[切换到 Iceberg 目录](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句删除该目录中的 Iceberg 数据库：

```SQL
DROP DATABASE <database_name>;
```

---

### 创建 Iceberg 表

与 StarRocks 的内部数据库类似，如果您对 Iceberg 数据库有 [CREATE TABLE](../../administration/privilege_item.md#database) 权限，则可以使用 [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 或 [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) 语句在该 Iceberg 数据库中创建表。此功能从 v3.1 版本开始支持。

[切换到 Iceberg 目录和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语法在该数据库中创建一个 Iceberg 表。

#### 语法

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

#### 参数

##### column_definition


`column_definition` 的语法如下：

```SQL
col_name col_type [COMMENT 'comment']
```

:::note

所有非分区列必须使用 `NULL` 作为默认值。这意味着在创建表的语句中，您必须为每个非分区列指定 `DEFAULT "NULL"`。此外，分区列必须在非分区列之后定义，并且不能使用 `NULL` 作为默认值。

:::

##### partition_desc

`partition_desc` 的语法如下：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

目前 StarRocks 仅支持 [身份转换](https://iceberg.apache.org/spec/#partitioning)，这意味着 StarRocks 为每个唯一的分区值创建一个分区。

:::note

分区列必须在非分区列之后定义。分区列支持除了 FLOAT、DOUBLE、DECIMAL 和 DATETIME 之外的所有数据类型，并且不能使用 `NULL` 作为默认值。

:::

##### PROPERTIES

您可以按照 `"key" = "value"` 的格式在 `PROPERTIES` 中指定表属性。请参阅 [Iceberg 表属性](https://iceberg.apache.org/docs/latest/configuration/)。

下表描述了一些关键属性。

###### location

描述：要创建 Iceberg 表的文件路径。当您使用 HMS 作为元存储时，无需指定 `location` 参数，因为 StarRocks 会在当前 Iceberg 目录的默认文件路径下创建表。当您使用 AWS Glue 作为元存储时：

- 如果已为要创建表的数据库指定了 `location` 参数，则无需为表指定 `location` 参数。因此，该表默认为其所属数据库的文件路径。
- 如果尚未为要创建表的数据库指定 `location`，则必须为表指定 `location` 参数。

###### file_format

描述：Iceberg 表的文件格式。仅支持 Parquet 格式。默认值： `parquet`。

###### compression_codec

描述：Iceberg 表使用的压缩算法。支持的压缩算法包括 SNAPPY、GZIP、ZSTD 和 LZ4。默认值： `gzip`。

---

### 例子

1. 创建一个名为 `unpartition_tbl` 的非分区表。该表由两列 `id` 和 `score` 组成，如下所示：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. 创建一个名为 `partition_tbl_1` 的分区表。该表由三列 `action`、`id` 和 `dt` 组成，其中 `id` 和 `dt` 被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 查询现有表 `partition_tbl_1`，并根据查询结果创建一个名为 `partition_tbl_2` 的分区表。对于 `partition_tbl_2`，`id` 和 `dt` 被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from employee;
   ```

---

### 将数据下沉到 Iceberg 表

与 StarRocks 的内部表类似，如果您对 Iceberg 表有 [INSERT](../../administration/privilege_item.md#table) 权限，您可以使用 [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 语句将 StarRocks 表的数据下沉到该 Iceberg 表中（目前仅支持 Parquet 格式的 Iceberg 表）。此功能从 v3.1 版本开始支持。

[切换到 Iceberg 目录和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语法将 StarRocks 表的数据下沉到该数据库中的 Parquet 格式的 Iceberg 表中。

#### 语法

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 如果要将数据下沉到指定的分区，请使用以下语法：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

:::note

分区列不允许使用 `NULL` 值。因此，您必须确保不会将空值加载到 Iceberg 表的分区列中。

:::

#### 参数

##### INTO

将 StarRocks 表的数据附加到 Iceberg 表中。

##### OVERWRITE

使用 StarRocks 表的数据覆盖 Iceberg 表的现有数据。

##### column_name

要将数据加载到的目标列的名称。您可以指定一个或多个列。如果指定多个列，请用逗号（`,`）分隔。您只能指定 Iceberg 表中实际存在的列，并且您指定的目标列必须包括 Iceberg 表的分区列。无论目标列的实际名称是什么，您指定的目标列都会依次映射到 StarRocks 表的列中。如果未指定目标列，则数据将加载到 Iceberg 表的所有列中。如果无法将 StarRocks 表的非分区列映射到 Iceberg 表的任何列，StarRocks 将默认值 `NULL` 写入 Iceberg 表的列。如果 INSERT 语句包含一个查询语句，其返回的列类型与目标列的数据类型不同，StarRocks 将对不匹配的列进行隐式转换。如果转换失败，将返回语法解析错误。

##### expression

将值分配给目标列的表达式。

##### DEFAULT

将默认值分配给目标列。

##### query

查询语句的结果将加载到 Iceberg 表中。它可以是 StarRocks 支持的任何 SQL 语句。

##### PARTITION

要加载数据的分区。您必须在此属性中指定 Iceberg 表的所有分区列。在此属性中指定的分区列的顺序可以与在表创建语句中定义的分区列的顺序不同。如果指定此属性，则不能指定 `column_name` 属性。

#### 例子

1. 将三行数据插入到 `partition_tbl_1` 表中：

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
   INSERT INTO partition_tbl_2 PARTITION (dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 用以下命令将满足两个条件 `dt='2023-09-01'` 和 `id=1` 的分区中的所有 `action` 列值覆盖为 `close`：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   或

   ```SQL
   INSERT OVERWRITE partition_tbl_1 PARTITION (dt='2023-09-01',id=1) SELECT 'close';
   ```

---

### 删除 Iceberg 表

与 StarRocks 的内部表类似，如果您对 Iceberg 表有 [DROP](../../administration/privilege_item.md#table) 权限，您可以使用 [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 语句删除该 Iceberg 表。此功能从 v3.1 版本开始支持。

删除 Iceberg 表时，该表的文件路径和 HDFS 集群或云存储上的数据不会随表一起删除。

当您强制删除 Iceberg 表时（即在 DROP TABLE 语句中指定 `FORCE` 关键字），该表在 HDFS 集群或云存储上的数据将与该表一起删除，但表的文件路径将被保留。

[切换到 Iceberg 目录和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句删除该数据库中的 Iceberg 表。

```SQL
DROP TABLE <table_name> [FORCE];
```

---

### 配置元数据缓存

您的 Iceberg 集群的元数据文件可能存储在 AWS S3 或 HDFS 等远程存储中。StarRocks 默认将 Iceberg 元数据缓存在内存中。为了加速查询，StarRocks 采用了两级元数据缓存机制，可以将元数据缓存在内存和磁盘上。对于每个初始查询，StarRocks 都会缓存其计算结果。如果后续发出的查询在语义上等同于前一个查询，StarRocks 会首先尝试从缓存中检索请求的元数据，并且只有在缓存中无法命中元数据时，才会从远端存储中获取元数据。

StarRocks 使用最近最少使用（LRU）算法对数据进行缓存和逐出。基本规则如下：

- StarRocks 首先尝试从内存中检索请求的元数据。如果内存中无法命中元数据，StarRocks 尝试从磁盘中获取元数据。StarRocks 从磁盘中获取到的元数据将被加载到内存中。如果磁盘中也无法命中元数据，StarRocks 会从远程存储中获取元数据，并将检索到的元数据缓存到内存中。
- StarRocks 将内存中驱逐出的元数据写入磁盘，但直接丢弃磁盘中驱逐出的元数据。

#### Iceberg 元数据缓存参数

##### enable_iceberg_metadata_disk_cache

单位：N/A
默认值： `false`
描述：指定是否启用磁盘缓存。

##### iceberg_metadata_cache_disk_path

单位：N/A
默认值： `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"`
描述：磁盘上缓存元数据文件的保存路径。

##### iceberg_metadata_disk_cache_capacity

单位：字节
默认值： `2147483648`，相当于 2 GB
描述：磁盘上允许的缓存元数据的最大大小。

##### iceberg_metadata_memory_cache_capacity

单位：字节
默认值： `536870912`，相当于 512 MB
描述：内存中允许的缓存元数据的最大大小。

##### iceberg_metadata_memory_cache_expiration_seconds

单位：秒
默认值： `86500`
描述：缓存条目在内存中的最后访问后经过的时间后过期。

##### iceberg_metadata_disk_cache_expiration_seconds

单位：秒
默认值： `604800`，相当于一周
描述：缓存条目在磁盘上的最后访问后经过的时间后过期。

##### iceberg_metadata_cache_max_entry_size

单位：字节
默认值： `8388608`，相当于 8 MB
描述：可以缓存的文件的最大大小。超过此参数值的文件将无法缓存，StarRocks 将从远程存储中检索这些文件。