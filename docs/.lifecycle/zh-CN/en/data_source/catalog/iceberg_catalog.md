---
displayed_sidebar: "Chinese"
---

# 冰山目录

从v2.4版本开始，冰山目录是StarRocks支持的一种外部目录类型。使用冰山目录，您可以：

- 直接查询存储在Iceberg中的数据，无需手动创建表。
- 使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)或者异步材料化视图（从v2.5版本开始支持）处理存储在Iceberg中的数据，并将数据加载到StarRocks中。
- 在StarRocks上执行操作以创建或删除Iceberg数据库和表，或者通过使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)（此功能从v3.1版本开始支持）将数据从StarRocks表下沉到Parquet格式的Iceberg表中。

为了确保您的Iceberg集群上的SQL工作负载成功运行，您的StarRocks集群需要集成两个重要组件：

- 分布式文件系统（HDFS）或对象存储（如AWS S3、Microsoft Azure存储、Google GCS或其他兼容S3的存储系统（例如MinIO））

- 元数据存储（Hive metastore、AWS Glue或Tabular）

  > **注意**
  >
  > - 如果您选择AWS S3作为存储，可以使用HMS或AWS Glue作为元数据存储。如果选择任何其他存储系统，只能使用HMS作为元数据存储。
  > - 如果您选择Tabular作为元数据存储，您需要使用Iceberg REST目录。

## 使用注意事项

StarRocks支持的Iceberg文件格式是Parquet和ORC：

- Parquet文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP和NO_COMPRESSION。
- ORC文件支持以下压缩格式：ZLIB、SNAPPY、LZO、LZ4、ZSTD和NO_COMPRESSION。

Iceberg目录支持v1表，并从StarRocks v3.0版本开始支持ORC格式的v2表。
Iceberg目录支持v1表。此外，Iceberg目录还从StarRocks v3.0版本开始支持ORC格式的v2表，并从StarRocks v3.1版本开始支持Parquet格式的v2表。

## 集成准备

在创建Iceberg目录之前，请确保您的StarRocks集群可以与Iceberg集群的存储系统和元数据存储集成。

### AWS IAM

如果您的Iceberg集群使用AWS S3作为存储或AWS Glue作为元数据存储，请选择适合的身份验证方法，并做好必要的准备工作，以确保您的StarRocks集群可以访问相关的AWS云资源。

建议使用以下身份验证方法：

- 实例配置文件
- 假定的角色
- IAM用户

在上述三种身份验证方法中，实例配置文件是最常用的。

有关更多信息，请参见[AWS IAM身份验证准备](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果您选择HDFS作为存储，配置您的StarRocks集群如下：

- （可选）设置用于访问您的HDFS集群和Hive元数据存储的用户名。默认情况下，StarRocks使用FE和BE进程的用户名来访问您的HDFS集群和Hive元数据存储。您还可以通过在每个FE的**fe/conf/hadoop_env.sh**文件和每个BE的**be/conf/hadoop_env.sh**文件的开头添加`export HADOOP_USER_NAME="<user_name>"`来设置用户名。在设置了这些文件中的用户名后，请重新启动每个FE和每个BE，以使参数设置生效。您只能为每个StarRocks集群设置一个用户名。
- 在查询Iceberg数据时，您的StarRocks集群的FE和BE使用HDFS客户端访问您的HDFS集群。在大多数情况下，您无需配置StarRocks集群即可实现该目的，StarRocks会使用默认配置启动HDFS客户端。您只需要在以下情况下配置StarRocks集群：

  - 您的HDFS集群启用了高可用性（HA）：将您的HDFS集群的**hdfs-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。
  - 您的HDFS集群启用了视图文件系统（ViewFs）：将您的HDFS集群的**core-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。

  > **注意**
  >
  > 如果发送查询时返回了指示未知主机的错误，请将您的HDFS集群节点的主机名和IP地址的映射添加到**/etc/hosts**路径。

### Kerberos身份验证

如果您的HDFS集群或Hive元数据存储启用了Kerberos身份验证，请配置您的StarRocks集群如下：

- 在每个FE和每个BE上运行`kinit -kt keytab_path principal`命令，从密钥分发中心（KDC）获取票据授予票证（TGT）。要运行此命令，您必须具有访问您的HDFS集群和Hive元数据存储的权限。请注意，使用此命令访问KDC是时效性的。因此，您需要使用cron定期运行此命令。
- 在每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf`是**krb5.conf**文件的保存路径。您可以根据需要修改路径。

## 创建Iceberg目录

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

### 参数

#### catalog_name

Iceberg目录的名称。命名规则如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，长度不能超过1023个字符。

#### comment

Iceberg目录的描述。此参数是可选的。

#### type

您的数据源类型。将值设置为`iceberg`。

#### MetastoreParams

关于StarRocks与数据源的元数据存储集成的一组参数。

##### Hive元数据存储

如果您选择Hive元数据存储作为数据源的元数据存储，请按以下方式配置`MetastoreParams`：

```SQL
"iceberg.catalog.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询Iceberg数据之前，您必须将Hive元数据存储节点的主机名和IP地址的映射添加到**/etc/hosts**路径。否则，在启动查询时，StarRocks可能无法访问您的Hive元数据存储。

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数                         | 是否必需 | 描述                                                       |
| --------------------------- | -------- | --------------------------------------------------------- |
| iceberg.catalog.type        | 是       | 您用于Iceberg集群的元数据存储类型。将值设置为`hive`。    |
| hive.metastore.uris         | 是       | 您的Hive元数据存储的URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />如果启用了Hive元数据存储的高可用性（HA），您可以指定多个元数据存储URI，并用逗号（,）分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

如果您选择AWS Glue作为数据源的元数据存储，并且选择了AWS S3作为存储，则执行以下操作之一：

- 选择基于实例配置文件的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 选择假设的角色为基础的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 选择IAM用户为基础的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "iceberg.catalog.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
```
"aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数                         | 是否必须 | 描述                                                         |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| iceberg.catalog.type          | 是       | 您用于 Iceberg 集群的存储库类型。将该值设置为`glue`。 |
| aws.glue.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值为：`true` 和 `false`。默认值为：`false`。 |
| aws.glue.iam_role_arn         | 否       | 在您的 AWS Glue 数据目录上具有权限的 IAM 角色的 Amazon 资源名称 (ARN)。如果您使用基于假定角色的身份验证方法来访问 AWS Glue，则必须指定此参数。 |
| aws.glue.region               | 是       | 您的 AWS Glue 数据目录所在的区域。示例：`us-west-1`。 |
| aws.glue.access_key           | 否       | 您的 AWS IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key           | 否       | 您的 AWS IAM 用户的机密密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS Glue，则必须指定此参数。 |

有关如何选择用于访问 AWS Glue 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参见[访问 AWS Glue 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

##### 表格

如果您将 Tabular 用作存储库，则必须将存储库类型设置为 REST (`"iceberg.catalog.type" = "rest"`)。配置`MetastoreParams`如下所示：

```SQL
"iceberg.catalog.type" = "rest",
"iceberg.catalog.uri" = "<rest_server_api_endpoint>",
"iceberg.catalog.credential" = "<credential>",
"iceberg.catalog.warehouse" = "<identifier_or_path_to_warehouse>"
```

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数                        | 是否必须 | 描述                                                     |
| -------------------------- | -------- | -------------------------------------------------------- |
| iceberg.catalog.type       | 是       | 您用于 Iceberg 集群的存储库类型。将该值设置为`rest`。          |
| iceberg.catalog.uri        | 是       | Tabular 服务端点的 URI。示例：`https://api.tabular.io/ws`。      |
| iceberg.catalog.credential | 是       | Tabular 服务的身份验证信息。                                     |
| iceberg.catalog.warehouse  | 否       | Iceberg 存储库的位置或标识符。示例：`s3://my_bucket/warehouse_location` 或 `sandbox`。 |

以下示例创建了一个名为`tabular`的 Iceberg 存储库，该存储库使用 Tabular 作为存储库：

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

#### `StorageCredentialParams`

关于 StarRocks 如何与您的存储系统集成的一组参数。此参数集是可选的。

请注意以下几点：

- 如果您使用 HDFS 作为存储系统，无需配置`StorageCredentialParams`，可以跳过此部分。如果您使用 AWS S3、其他兼容 S3 的存储系统、Microsoft Azure 存储或 Google GCS 作为存储系统，必须配置`StorageCredentialParams`。

- 如果您将 Tabular 用作存储库，则无需配置`StorageCredentialParams`，可以跳过此部分。如果您将 HMS 或 AWS Glue 用作存储库，则必须配置`StorageCredentialParams`。

##### AWS S3

如果您选择 AWS S3 作为 Iceberg 集群的存储系统，请执行以下操作之一：

- 若要选择基于实例配置文件的身份验证方法，请将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 若要选择基于假定角色的身份验证方法，请将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 若要选择基于 IAM 用户的身份验证方法，请将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                        | 是否必须 | 描述                                                         |
| --------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值为：`true` 和 `false`。默认值为：`false`。 |
| aws.s3.iam_role_arn         | 否       | 具有对您的 AWS S3 存储桶具有权限的 IAM 角色的 Amazon 资源名称 (ARN)。如果您使用基于假定角色的身份验证方法来访问 AWS S3，则必须指定此参数。 |
| aws.s3.region               | 是       | 您的 AWS S3 存储桶所在的区域。示例：`us-west-1`。 |
| aws.s3.access_key           | 否       | 您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS S3，则必须指定此参数。 |
| aws.s3.secret_key           | 否       | 您的 IAM 用户的机密密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS S3，则必须指定此参数。 |

有关如何选择用于访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参见[访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容 S3 的存储系统

Iceberg 存储库从 v2.5 开始支持兼容 S3 的存储系统。

如果您选择兼容 S3 的存储系统（如 MinIO）作为 Iceberg 集群的存储系统，为确保成功集成，请将`StorageCredentialParams`配置如下：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                               | 是否必须 | 描述                                                         |
| ---------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                   | 是       | 指定是否启用 SSL 连接。有效值：`true` 和 `false`。默认值为：`true`。 |
| aws.s3.enable_path_style_access     | 是       | 指定是否启用路径样式访问。有效值：`true` 和 `false`。默认值为：`false`。对于 MinIO，您必须将该值设置为`true`。<br />路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈州）区域创建了一个名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且要访问该存储桶中的`alice.jpg`对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                    | 是       | 用于连接到兼容 S3 存储系统的端点。                                      |
| aws.s3.access_key                  | 是       | 您的 IAM 用户的访问密钥。                                            |
| aws.s3.secret_key                  | 是       | 您的 IAM 用户的机密密钥。                                            |

##### Microsoft Azure 存储

Iceberg 存储库从 v3.0 开始支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果您选择 Blob 存储作为 Iceberg 集群的存储系统，请执行以下操作之一：

- 若要选择共享密钥身份验证方法，请将`StorageCredentialParams`配置如下：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                    | **是否必须** | **描述**                         |
  | -------------------------- | ------------ | -------------------------------- |
  | azure.blob.storage_account | 是           | 您的 Blob 存储帐户的用户名。      |
  | azure.blob.shared_key      | 是           | 您的 Blob 存储帐户的共享密钥。    |
  ```
```
      + {R}
      + {R}
    + {R}
  + {R}
```
| gcp.gcs.service_account_email          | ""                | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | ""                | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 在创建元服务帐户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key    | ""                | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 在创建元服务帐户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""                | "hello"                                                      | 要冒充的数据服务帐户。       |

### 例子

以下示例根据您使用的元数据存储库类型创建名为 `iceberg_catalog_hms` 或 `iceberg_catalog_glue` 的 Iceberg 目录，以从 Iceberg 集群查询数据。

#### HDFS

如果您使用 HDFS 作为存储，运行以下命令：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### 如果您选择基于实例配置文件的凭据

- 如果您在 Iceberg 集群中使用 Hive metastore，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，运行以下命令：

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

##### 如果您选择基于角色的凭据

- 如果您在 HIceberg 集群中使用 Hive metastore，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，运行以下命令：

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

##### 如果您选择基于 IAM 用户的凭据

- 如果您在 Iceberg 集群中使用 Hive metastore，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Iceberg 集群中使用 AWS Glue，运行以下命令：

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

#### 兼容 S3 存储系统

以 MinIO 为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG iceberg_catalog_hms
PROPERTIES
(
    "type" = "iceberg",
    "iceberg.catalog.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### 微软 Azure 存储

##### Azure Blob 存储

- 如果您选择共享密钥验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果您选择 SAS 令牌验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure 数据湖存储 Gen1

- 如果您选择托管服务身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果您选择服务主体验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure 数据湖存储 Gen2

- 如果您选择托管标识验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果您选择Shared Key身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果您选择Service Principal身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果您选择基于VM的身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果您选择基于服务账户的身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG iceberg_catalog_hms
  PROPERTIES
  (
      "type" = "iceberg",
      "iceberg.catalog.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果您选择基于冒充的身份验证方法：

  - 如果您让VM实例冒充服务账户，请运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果您让服务账户冒充另一个服务账户，请运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG iceberg_catalog_hms
    PROPERTIES
    (
        "type" = "iceberg",
        "iceberg.catalog.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    );
    ```

## 查看Iceberg目录

您可以使用[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)来查询当前StarRocks集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您也可以使用[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)来查询外部目录的创建语句。以下示例查询名为`iceberg_catalog_glue`的Iceberg目录的创建语句：

```SQL
SHOW CREATE CATALOG iceberg_catalog_glue;
```

## 切换到Iceberg目录及其中的数据库

您可以使用以下方法之一来切换到Iceberg目录及其中的数据库：

- 使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)来指定当前会话中的Iceberg目录，然后使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来指定活动数据库：

  ```SQL
  -- 切换到当前会话中指定的目录：
  SET CATALOG <catalog_name>
  -- 在当前会话中指定活动数据库：
  USE <db_name>
  ```

- 直接使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来切换到Iceberg目录及其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除Iceberg目录

您可以使用[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)来删除外部目录。

以下示例删除名为`iceberg_catalog_glue`的Iceberg目录：

```SQL
DROP CATALOG iceberg_catalog_glue;
```

## 查看Iceberg表的模式

您可以使用以下语法之一来查看Iceberg表的模式：

- 查看模式

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 查看来自CREATE语句的模式和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询Iceberg表

1. 使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)来查看Iceberg集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到Iceberg目录及其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)。

3. 使用[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)来查询指定数据库中目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 创建Iceberg数据库

与StarRocks的内部目录类似，如果您在Iceberg目录上有[CREATE DATABASE](../../administration/privilege_item.md#catalog)权限，您可以使用[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)语句在该Iceberg目录中创建数据库。从v3.1版本开始支持此功能。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

[切换到Iceberg目录](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句在该目录中创建Iceberg数据库：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

您可以使用`location`参数来指定要创建数据库的文件路径。支持HDFS和云存储。如果未指定`location`参数，则StarRocks将在Iceberg目录的默认文件路径中创建数据库。

`prefix`根据您使用的存储系统而变化：

| **存储系统**                                         | **`前缀`值**                                       |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| HDFS                                                       | `hdfs`                                                       |
| Google GCS                                                 | `gs`                                                         |
| Azure Blob Storage                                         | <ul><li>如果您的存储账户允许通过HTTP访问，则`前缀`为`wasb`。</li><li>如果您的存储账户允许通过HTTPS访问，则`前缀`为`wasbs`。</li></ul> |
| Azure Data Lake Storage Gen1                               | `adl`                                                        |
| Azure Data Lake Storage Gen2                               | <ul><li>如果您的存储账户允许通过HTTP访问，则`前缀`为`abfs`。</li><li>如果您的存储账户允许通过HTTPS访问，则`前缀`为`abfss`。</li></ul> |
| AWS S3或其他兼容S3的存储（例如MinIO） | `s3`                                                         |

## 删除Iceberg数据库
```
与StarRocks的内部数据库类似，如果您在Iceberg数据库上拥有[DROP](../../administration/privilege_item.md#database) 权限，您可以使用[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) 语句删除该Iceberg数据库。此功能从v3.1版本开始支持。您只能删除空数据库。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

当您删除一个Iceberg数据库时，数据库在HDFS集群或云存储上的文件路径不会随着数据库一起删除。

[切换到Iceberg catalog](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句在该catalog中删除Iceberg数据库：

```SQL
DROP DATABASE <database_name>;
```

## 创建Iceberg表

与StarRocks的内部数据库类似，如果您在Iceberg数据库上拥有[CREATE TABLE](../../administration/privilege_item.md#database) 权限，您可以使用[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) 或 [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) 语句在Iceberg数据库中创建表。此功能从v3.1版本开始支持。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

[切换到Iceberg catalog和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语法在该数据库中创建一个Iceberg表。

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

`column_definition`的语法如下：

```SQL
col_name col_type [COMMENT 'comment']
```

以下是参数描述。

| **参数**            | **描述**                      |
| --------------------- |------------------------------------------------------ |
| col_name              | 列的名称。                                                |
| col_type                | 列的数据类型。支持以下数据类型：TINYINT、SMALLINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL、DATE、DATETIME、CHAR、VARCHAR[(length)]、ARRAY、MAP、和STRUCT。不支持LARGEINT、HLL和BITMAP数据类型。 |

> **注意**
>
> 所有非分区列必须使用`NULL`作为默认值。这意味着您必须在表创建语句中为每个非分区列指定`DEFAULT "NULL"`。此外，分区列必须在非分区列后定义，并且不能使用`NULL`作为默认值。

#### partition_desc

`partition_desc`的语法如下：

```SQL
PARTITION BY (par_col1[, par_col2...])
```

目前StarRocks仅支持[identity transforms](https://iceberg.apache.org/spec/#partitioning)，这意味着StarRocks为每个唯一的分区值创建一个分区。

> **注意**
>
> 分区列必须在非分区列后定义。分区列支持除FLOAT、DOUBLE、DECIMAL和DATETIME之外的所有数据类型，并且不能使用`NULL`作为默认值。

#### PROPERTIES

您可以在`PROPERTIES`中以`"key" = "value"`格式指定表属性。请参阅[Iceberg表属性](https://iceberg.apache.org/docs/latest/configuration/)。

以下表格描述了一些关键属性。

| **属性**        | **描述**                   |
| ----------------- | ---------------------------------------------------- |
| location            | 您希望创建Iceberg表的文件路径。当您使用HMS作为元存储时，您不需要指定`location`参数，因为StarRocks会将表创建在当前Iceberg catalog的默认文件路径中。当您使用AWS Glue作为元存储:<ul><li>如果您已经为要创建表的数据库指定了`location`参数，则不需要为表指定`location`参数。因此，表默认为属于的数据库的文件路径。</li><li>如果您没有为要创建表的数据库指定`location`参数，则必须为表指定`location`参数。</li></ul> |
| file_format     | Iceberg表的文件格式。仅支持Parquet格式。默认值：`parquet`。 |
| compression_codec | 用于Iceberg表的压缩算法。支持的压缩算法有SNAPPY、GZIP、ZSTD和LZ4。默认值：`gzip`。 |

### 示例

1. 创建一个名为`unpartition_tbl`的非分区表。该表包括两列`id`和`score`，如下所示：

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. 创建一个名为`partition_tbl_1`的分区表。该表包括三列`action`、`id`和`dt`，其中`id`和`dt`被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. 查询名为`partition_tbl_1`的现有表，并基于`partition_tbl_1`的查询结果创建名为`partition_tbl_2`的分区表。对于`partition_tbl_2`，`id`和`dt`被定义为分区列，如下所示：

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from employee;
   ```

## 向Iceberg表中的数据接入数据

类似于StarRocks的内部表，如果您在Iceberg表上具有[INSERT](../../administration/privilege_item.md#table) 权限，您可以使用[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) 语句将StarRocks表的数据接入至该Iceberg表（当前仅支持格式为Parquet的Iceberg表）。此功能从v3.1版本开始支持。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 来授予和撤销权限。

[切换到Iceberg catalog和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语法将StarRocks表的数据接入到该数据库中的格式为Parquet的Iceberg表中。

### 语法

```SQL
INSERT {INTO | OVERWRITE} <table_name>
(列名 [, ...])
{ VALUES ( { 表达式 | DEFAULT } [, ...] ) [, ...] | query }

-- 如果您想要将数据接入到指定分区，请使用以下语法：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { 表达式 | DEFAULT } [, ...] ) [, ...] | query }
```

> **注意**
>
> 分区列不允许`NULL`值。因此，您必须确保不将空值加载到Iceberg表的分区列中。

### 参数

| 参数          | 描述                                                      |
| ------------- | ------------------------------------------------------------ |
| INTO          | 将StarRocks表的数据追加到Iceberg表中。                                    |
| OVERWRITE     | 用StarRocks表的数据覆盖Iceberg表的现有数据。                                    |
| column_name | 您要将数据加载到的目标列的名称。您可以指定一个或多个列。如果指定多个列，请用逗号(`,`)将它们分隔开。您只能指定实际存在于Iceberg表中的列，您指定的目标列必须包括Iceberg表的分区列。您指定的目标列按顺序一一对应地映射到StarRocks表的列，不管目标列名称是什么。如果未指定目标列，数据将加载到Iceberg表的所有列中。如果StarRocks表的非分区列无法映射到Iceberg表的任何列，StarRocks将向Iceberg表列写入默认值`NULL`。如果INSERT语句包含一个查询语句，其返回的列类型与目标列的数据类型不匹配，则StarRocks会对不匹配的列执行隐式转换。如果转换失败，将返回语法解析错误。 |
| 表达式      | 分配值给目标列的表达式。                                                     |
| DEFAULT   | 为目标列分配默认值。                                                       |
| 查询       | 查询语句的结果将加载到 Iceberg 表中。它可以是 StarRocks 支持的任何 SQL 语句。 |
| 分区       | 您希望加载数据的分区。您必须在此属性中指定 Iceberg 表的所有分区列。在此属性中指定的分区列的顺序可以与您在表创建语句中定义的分区列的顺序不同。如果指定此属性，则不能指定 `列名` 属性。 |

### 示例

1. 将三行数据插入到 `partition_tbl_1` 表中：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 将包含简单计算的 SELECT 查询结果插入到 `partition_tbl_1` 表中：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. 将从 `partition_tbl_1` 表中读取数据的 SELECT 查询结果插入到相同的表中：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 将 SELECT 查询结果插入到满足 `dt='2023-09-01'` 和 `id=1` 两个条件的分区中的 `partition_tbl_2` 表中：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   或者

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 用 `close` 覆盖满足 `dt='2023-09-01'` 和 `id=1` 两个条件的分区中的 `partition_tbl_1` 表中的所有 `action` 列值：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   或者

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## 删除 Iceberg 表

与 StarRocks 的内部表类似，如果您在 Iceberg 表上具有 [DROP](../../administration/privilege_item.md#table) 权限，则可以使用 [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 语句删除该 Iceberg 表。支持该功能的起始版本为 v3.1。

> **注意**
>
> 您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 授予和撤销权限。

删除 Iceberg 表时，表的文件路径和 HDFS 集群或云存储中的数据将不会随表一起删除。

当您强制删除 Iceberg 表（即在 DROP TABLE 语句中指定了 `FORCE` 关键字）时，表在 HDFS 集群或云存储中的数据将随表一起删除，但表的文件路径将被保留。

[切换到 Iceberg 目录及其中的数据库](#切换到 Iceberg 目录及其中的数据库)，然后使用以下语句删除该数据库中的 Iceberg 表。

```SQL
DROP TABLE <table_name> [FORCE];
```

## 配置元数据缓存

您的 Iceberg 集群的元数据文件可以存储在 AWS S3 或 HDFS 等远程存储中。默认情况下，StarRocks 在内存中缓存 Iceberg 元数据。为了加速查询，StarRocks 采用了两级元数据缓存机制，可以同时在内存和磁盘上缓存元数据。对于每个初始查询，StarRocks 都会缓存它们的计算结果。如果发出了与之前查询语义等价的后续查询，StarRocks 首先尝试从其缓存中检索请求的元数据，当无法在缓存中命中元数据时，它才从远程存储中检索元数据。

StarRocks 使用最近最少使用（LRU）算法来缓存和清除数据。基本规则如下：

- StarRocks 首先尝试从内存中检索请求的元数据。如果在内存中无法命中元数据，则尝试从磁盘中检索元数据。从磁盘中检索到的元数据将被加载到内存中。如果在磁盘中也无法命中元数据，则从远程存储中检索元数据，并将检索到的元数据缓存到内存中。
- StarRocks 将从内存中清除的元数据写入磁盘，但直接丢弃从磁盘中清除的元数据。

以下表描述了 FE 配置项，您可以使用这些配置项来配置 Iceberg 的元数据缓存机制。

| **配置项**                           | **单位** | **默认值**                                    | **描述**                                              |
| ------------------------------------ | -------- | ---------------------------------------------- | ---------------------------------------------------- |
| enable_iceberg_metadata_disk_cache   | N/A      | `false`                                        | 指定是否启用磁盘缓存。                                 |
| iceberg_metadata_cache_disk_path     | N/A      | `StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"` | 磁盘上缓存的元数据文件的保存路径。                    |
| iceberg_metadata_disk_cache_capacity | 字节     | `2147483648`，相当于 2 GB                     | 允许在磁盘上缓存的元数据的最大大小。                  |
| iceberg_metadata_memory_cache_capacity | 字节   | `536870912`，相当于 512 MB                    | 内存中允许缓存的元数据的最大大小。                    |
| iceberg_metadata_memory_cache_expiration_seconds | 秒  | `86500`                                    | 从最后访问开始计时，缓存中的条目在这段时间后到期。 |
| iceberg_metadata_disk_cache_expiration_seconds | 秒    | `604800`，相当于一周                           | 从最后访问开始计时，磁盘上的缓存中的条目在这段时间后到期。|
| iceberg_metadata_cache_max_entry_size | 字节      | `8388608`，相当于 8 MB                         | 可以缓存的文件的最大大小。超过此参数值的文件将无法缓存。如果查询请求这些文件，StarRocks 将从远程存储中检索它们。 |