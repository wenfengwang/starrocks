---
displayed_sidebar: English
---

# Hudi目录

Hudi目录是一种外部目录，使您能够在不摄取的情况下从Apache Hudi查询数据。

此外，您还可以直接使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)基于Hudi目录来转换和加载Hudi中的数据。StarRocks从v2.4版本开始支持Hudi目录。

为了确保您的Hudi集群上的SQL工作负载成功，您的StarRocks集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或对象存储，例如AWS S3、Microsoft Azure Storage、Google GCS或其他S3兼容存储系统（例如MinIO）

- 元存储，例如Hive元存储或AWS Glue

    > **注意**
    > 如果您选择AWS S3作为存储，您可以使用HMS或AWS Glue作为元存储。如果您选择任何其他存储系统，您只能使用HMS作为元存储。

## 使用说明

- StarRocks支持的Hudi文件格式是Parquet。Parquet文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP和NO_COMPRESSION。
- StarRocks为Hudi的写入时复制（COW）表和读取时合并（MOR）表提供完整支持。

## 集成准备工作

在创建Hudi目录之前，请确保您的StarRocks集群可以与Hudi集群的存储系统和元存储集成。

### AWS IAM

如果您的Hudi集群使用AWS S3作为存储或AWS Glue作为元存储，请选择合适的身份验证方法并做好必要的准备，以确保您的StarRocks集群可以访问相关的AWS云资源。

推荐使用以下认证方法：

- 实例配置文件
- 假定角色
- IAM用户

在上述三种身份验证方法中，实例配置文件是使用最广泛的。

有关更多信息，请参阅[在AWS IAM中进行身份验证准备](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果您选择HDFS作为存储，请按以下方式配置StarRocks集群：

- （可选）设置用于访问HDFS集群和Hive元存储的用户名。默认情况下，StarRocks使用FE和BE进程的用户名来访问您的HDFS集群和Hive元存储。您还可以通过在每个FE的**fe/conf/hadoop_env.sh**文件开头和每个BE的**be/conf/hadoop_env.sh**文件开头添加`export HADOOP_USER_NAME="<user_name>"`来设置用户名。在这些文件中设置用户名后，需要重新启动各个FE和BE以使参数设置生效。您只能为每个StarRocks集群设置一个用户名。
- 当您查询Hudi数据时，StarRocks集群的FE和BE使用HDFS客户端访问您的HDFS集群。在大多数情况下，您不需要配置StarRocks集群来实现该目的，StarRocks使用默认配置启动HDFS客户端。仅在以下情况下才需要配置StarRocks集群：

  - 为HDFS集群启用高可用性（HA）：将HDFS集群的**hdfs-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。
  - 为HDFS集群启用View File System（ViewFs）：将HDFS集群的**core-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。

> **注意**
> 如果发送查询时返回未知主机错误，则必须将HDFS集群节点的主机名和IP地址之间的映射添加到**/etc/hosts**路径中。

### Kerberos身份验证

如果您的HDFS集群或Hive元存储启用了Kerberos身份验证，请按以下方式配置您的StarRocks集群：

- 在每个FE和BE上运行`kinit -kt keytab_path principal`命令，从Key Distribution Center（KDC）获取Ticket Granting Ticket（TGT）。要运行此命令，您必须具有访问HDFS集群和Hive元存储的权限。请注意，使用此命令访问KDC是有时间限制的。因此，需要使用cron定期运行该命令。
- 将`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`添加到每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中。在此示例中，`/etc/krb5.conf`是**krb5.conf**文件的保存路径。您可以根据需要修改路径。

## 创建Hudi目录

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

- 名称可以包含字母、数字（0-9）和下划线（_）。它必须以字母开头。
- 名称区分大小写，长度不能超过1023个字符。

#### comment

Hudi目录的描述。此参数是可选的。

#### type

您的数据源的类型。将值设置为`hudi`。

#### MetastoreParams

一组参数，说明StarRocks如何与数据源的元存储集成。

##### Hive元存储

如果您选择Hive元存储作为数据源的元存储，请按以下方式配置`MetastoreParams`：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
> 在查询Hudi数据之前，您必须将Hive元存储节点的主机名和IP地址之间的映射添加到`/etc/hosts`路径中。否则，StarRocks可能无法访问您的Hive元存储当您启动查询时。

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| hive.metastore.type | 是 | 您用于Hudi集群的元存储的类型。将值设置为`hive`。 |
| hive.metastore.uris | 是 | 您的Hive元存储的URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />如果您的Hive元存储启用了高可用性（HA），您可以指定多个元存储URI，并用逗号（,）分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

如果您选择AWS Glue作为数据源的元存储，仅当您选择AWS S3作为存储时才支持，请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择基于假定角色的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择基于IAM用户的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数 | 必填 | 说明 |
| --- | --- | --- |
| hive.metastore.type | 是 | 您用于Hudi集群的元存储的类型。将值设置为`glue`。 |
| aws.glue.use_instance_profile | 是 | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.glue.iam_role_arn | 否 | 对您的AWS Glue数据目录拥有权限的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS Glue，则必须指定此参数。 |
| aws.glue.region | 是 | 您的AWS Glue数据目录所在的区域。示例：`us-west-1`。 |
| aws.glue.access_key | 否 | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，则必须指定此参数。 |
| aws.glue.secret_key | 否 | 您的AWS IAM用户的密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，则必须指定此参数。 |

有关如何选择用于访问AWS Glue的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS Glue的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

一组参数，说明StarRocks如何与您的存储系统集成。此参数集是可选的。

如果使用HDFS作为存储，则不需要配置`StorageCredentialParams`。

如果您使用AWS S3、其他S3兼容存储系统、Microsoft Azure存储或Google GCS作为存储，则必须配置`StorageCredentialParams`。

##### AWS S3

如果您选择AWS S3作为Hudi集群的存储，请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按以下方式配置`StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于假定角色的身份验证方法，请按以下方式配置`StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 要选择基于IAM用户的身份验证方法，请按以下方式配置`StorageCredentialParams`：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  ```
```markdown
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数 | 必填 | 说明 |
|---|---|---|
| aws.s3.use_instance_profile | 是 | 指定是否启用基于实例配置文件的身份验证方法和基于角色的身份验证方法。有效值：`true` 和 `false`。默认值：`false`。 |
| aws.s3.iam_role_arn | 否 | 拥有对您的 AWS S3 存储桶权限的 IAM 角色的 ARN。如果您使用基于角色的身份验证方法来访问 AWS S3，则必须指定此参数。 |
| aws.s3.region | 是 | 您的 AWS S3 存储桶所在的区域。示例：`us-west-1`。 |
| aws.s3.access_key | 否 | 您的 IAM 用户的访问密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS S3，则必须指定此参数。 |
| aws.s3.secret_key | 否 | 您的 IAM 用户的密钥。如果您使用基于 IAM 用户的身份验证方法来访问 AWS S3，则必须指定此参数。 |

有关如何选择访问 AWS S3 的身份验证方法以及如何在 AWS IAM 控制台中配置访问控制策略的信息，请参阅[访问 AWS S3 的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### S3 兼容存储系统

Hudi 目录从 v2.5 版本开始支持 S3 兼容的存储系统。

如果您选择 S3 兼容的存储系统（例如 MinIO）作为 Hudi 集群的存储，请按如下方式配置 `StorageCredentialParams` 以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

| 参数 | 必填 | 说明 |
|---|---|---|
| aws.s3.enable_ssl | 是 | 指定是否启用 SSL 连接。<br>有效值：`true` 和 `false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是 | 指定是否启用路径样式访问。<br>有效值：`true` 和 `false`。默认值：`false`。对于 MinIO，您必须将该值设置为 `true`。<br>路径样式 URL 使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并且想要访问该存储桶中的 `alice.jpg` 对象，则可以使用以下路径样式 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint | 是 | 用于连接到您的 S3 兼容存储系统的终端节点，而不是 AWS S3。 |
| aws.s3.access_key | 是 | 您的 IAM 用户的访问密钥。 |
| aws.s3.secret_key | 是 | 您的 IAM 用户的密钥。 |

##### 微软 Azure 存储

Hudi 目录从 v3.0 版本开始支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果您选择 Blob 存储作为 Hudi 集群的存储，请执行以下操作之一：

- 要选择共享密钥身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.blob.storage_account | 是 | 您的 Blob 存储账户的用户名。 |
  | azure.blob.shared_key | 是 | 您的 Blob 存储账户的共享密钥。 |

- 要选择 SAS 令牌身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.container" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.blob.storage_account | 是 | 您的 Blob 存储账户的用户名。 |
  | azure.blob.container | 是 | 存储数据的 Blob 容器的名称。 |
  | azure.blob.sas_token | 是 | 用于访问您的 Blob 存储账户的 SAS 令牌。 |

###### Azure 数据湖存储 Gen1

如果您选择 Data Lake Storage Gen1 作为 Hudi 集群的存储，请执行以下操作之一：

- 要选择托管服务身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.adls1.use_managed_service_identity | 是 | 指定是否启用托管服务身份验证方法。将值设置为 `true`。 |

- 要选择服务主体身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.adls1.oauth2_client_id | 是 | 服务主体的客户端（应用程序）ID。 |
  | azure.adls1.oauth2_credential | 是 | 创建的新客户端（应用程序）密钥的值。 |
  | azure.adls1.oauth2_endpoint | 是 | 服务主体或应用程序的 OAuth 2.0 令牌端点 (v1)。 |

###### Azure 数据湖存储 Gen2

如果您选择 Data Lake Storage Gen2 作为 Hudi 集群的存储，请执行以下操作之一：

- 要选择托管身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.adls2.oauth2_use_managed_identity | 是 | 指定是否启用托管身份验证方法。将值设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id | 是 | 您想要访问其数据的租户的 ID。 |
  | azure.adls2.oauth2_client_id | 是 | 托管身份的客户端（应用程序）ID。 |

- 要选择共享密钥身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.adls2.storage_account | 是 | 您的 Data Lake Storage Gen2 存储账户的用户名。 |
  | azure.adls2.shared_key | 是 | 您的 Data Lake Storage Gen2 存储账户的共享密钥。 |

- 要选择服务主体身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 必填 | 说明 |
|---|---|---|
  | azure.adls2.oauth2_client_id | 是 | 服务主体的客户端（应用程序）ID。 |
  | azure.adls2.oauth2_client_secret | 是 | 创建的新客户端（应用程序）密钥的值。 |
  | azure.adls2.oauth2_client_endpoint | 是 | 服务主体或应用程序的 OAuth 2.0 令牌端点 (v1)。 |

##### Google GCS

Hudi 目录从 v3.0 版本开始支持 Google GCS。

如果您选择 Google GCS 作为 Hudi 集群的存储，请执行以下操作之一：

- 要选择基于虚拟机的身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 默认值 | 值示例 | 说明 |
|---|---|---|---|
  | gcp.gcs.use_compute_engine_service_account | `false` | `true` | 指定是否直接使用绑定到您的 Compute Engine 的服务账户。 |

- 要选择基于服务账户的身份验证方法，请按如下方式配置 `StorageCredentialParams`：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

  | 参数 | 默认值 | 值示例 | 说明 |
|---|---|---|---|
  | gcp.gcs.service_account_email | "" | "user@hello.iam.gserviceaccount.com" | 创建服务账户时生成的 JSON 文件中的电子邮件地址。 |
  | gcp.gcs.service_account_private_key_id | "" | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea" | 创建服务账户时生成的 JSON 文件中的私钥 ID。 |
  | gcp.gcs.service_account_private_key | "" | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建服务账户时生成的 JSON 文件中的私钥。 |

- 要选择基于模拟的身份验证方法，请按如下方式配置 `StorageCredentialParams`：

-   让 VM 实例模拟服务账户：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | 参数 | 默认值 | 值示例 | 说明 |
|---|---|---|---|
    | gcp.gcs.use_compute_engine_service_account | `false` | `true` | 指定是否直接使用绑定到您的 Compute Engine 的服务账户。 |
    | gcp.gcs.impersonation_service_account | "" | "hello" | 您要模拟的服务账户。 |

-   让一个服务账户（暂时命名为元服务账户）模拟另一个服务账户（暂时命名为数据服务账户）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | 参数 | 默认值 | 值示例 | 说明 |
|---|---|---|---|
    | gcp.gcs.service_account_email | "" | "user@hello.iam.gserviceaccount.com" | 创建元服务账户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | "" | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea" | 创建元服务账户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key | "" | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建元服务账户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account | "" | "hello" | 您要模拟的数据服务账户。 |
```
```markdown
    "gcp.gcs.service_account_private_key" = "<google_service_private_key>",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    下表描述了您需要在 `StorageCredentialParams` 中配置的参数。

    | 参数 | 默认值 | 值示例 | 说明 |
    |---|---|---|---|
    | gcp.gcs.service_account_email | "" | "user@hello.iam.gserviceaccount.com" | 创建元服务账户时生成的 JSON 文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id | "" | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea" | 创建元服务账户时生成的 JSON 文件中的私钥 ID。 |
    | gcp.gcs.service_account_private_key | "" | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建元服务账户时生成的 JSON 文件中的私钥。 |
    | gcp.gcs.impersonation_service_account | "" | "hello" | 您要模拟的数据服务账户。 |

#### 元数据更新参数

一组关于 StarRocks 如何更新 Hudi 缓存元数据的参数。该参数集是可选的。

StarRocks 默认执行[自动异步更新策略](#appendix-understand-metadata-automatic-asynchronous-update)。

大多数情况下，您可以忽略 `MetadataUpdateParams`，不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了开箱即用的性能。

但如果 Hudi 中的数据更新频率较高，可以通过调整这些参数来进一步优化自动异步更新的性能。

> **注意**
> 在大多数情况下，如果您的 Hudi 数据更新的粒度为 1 小时或更小，则数据更新频率被认为很高。

| 参数 | 必填 | 说明 |
|---|---|---|
| enable_metastore_cache | 否 | 指定 StarRocks 是否缓存 Hudi 表的元数据。有效值：`true` 和 `false`。默认值：`true`。值 `true` 启用缓存，值 `false` 禁用缓存。 |
| enable_remote_file_cache | 否 | 指定 StarRocks 是否缓存 Hudi 表或分区的底层数据文件的元数据。有效值：`true` 和 `false`。默认值：`true`。值 `true` 启用缓存，值 `false` 禁用缓存。 |
| metastore_cache_refresh_interval_sec | 否 | StarRocks 异步更新自身缓存的 Hudi 表或分区元数据的时间间隔。单位：秒。默认值：`7200`，即 2 小时。 |
| remote_file_cache_refresh_interval_sec | 否 | StarRocks 异步更新自身缓存的 Hudi 表或分区的底层数据文件元数据的时间间隔。单位：秒。默认值：`60`。 |
| metastore_cache_ttl_sec | 否 | StarRocks 自动丢弃自身缓存的 Hudi 表或分区元数据的时间间隔。单位：秒。默认值：`86400`，即 24 小时。 |
| remote_file_cache_ttl_sec | 否 | StarRocks 自动丢弃自身缓存的 Hudi 表或分区的底层数据文件元数据的时间间隔。单位：秒。默认值：`129600`，即 36 小时。 |

### 示例

以下示例根据您使用的元存储类型创建一个名为 `hudi_catalog_hms` 或 `hudi_catalog_glue` 的 Hudi 目录，以从 Hudi 集群查询数据。

#### 分布式文件系统 (HDFS)

如果您使用 HDFS 作为存储，请运行如下命令：

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

- 如果您在 Hudi 集群中使用 Hive 元存储，请运行如下命令：

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

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行如下命令：

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

##### 如果您选择假定的基于角色的凭证

- 如果您在 Hudi 集群中使用 Hive 元存储，请运行如下命令：

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

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行如下命令：

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

##### 如果您选择 IAM 基于用户的凭证

- 如果您在 Hudi 集群中使用 Hive 元存储，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_secret_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在 Amazon EMR Hudi 集群中使用 AWS Glue，请运行如下命令：

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

#### S3 兼容存储系统

以 MinIO 为例。运行如下命令：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083",
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

- 如果您选择 SAS 令牌身份验证方法，请运行如下命令：

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

##### Azure 数据湖存储 Gen1

- 如果您选择托管服务身份验证方法，请运行如下命令：

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

- 如果您选择服务主体身份验证方法，请运行如下命令：

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

##### Azure 数据湖存储 Gen2

- 如果您选择托管身份验证方法，请运行如下命令：

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

- 如果您选择共享密钥身份验证方法，请运行如下命令：

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

- 如果您选择服务主体身份验证方法，请运行如下命令：

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
```
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

#### Google GCS

- 如果您选择基于虚拟机的认证方式，请运行以下命令：

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

- 如果您选择基于服务账户的认证方式，请运行以下命令：

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

- 如果您选择基于模拟的认证方式：

  - 如果您让 VM 实例模拟服务账户，请运行以下命令：

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

  - 如果您让一个服务账户模拟另一个服务账户，请运行以下命令：

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

## 查看 Hudi 目录

您可以使用 [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) 查询当前 StarRocks 集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您还可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录的创建语句。以下示例查询名为 `hudi_catalog_glue` 的 Hudi 目录的创建语句：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## 切换到 Hudi 目录和其中的数据库

您可以使用以下方法之一切换到 Hudi 目录及其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中指定一个 Hudi 目录，并使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 来指定一个活动数据库：

  ```SQL
  -- 在当前会话中切换到指定的目录：
  SET CATALOG <catalog_name>
  -- 在当前会话中指定活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到一个 Hudi 目录和其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除 Hudi 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `hudi_catalog_glue` 的 Hudi 目录：

```SQL
DROP CATALOG hudi_catalog_glue;
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

2. [切换到 Hudi 目录和其中的数据库](#切换到-hudi-目录和其中的数据库)。

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

默认情况下，StarRocks 缓存 Hudi 的元数据，并以异步方式自动更新元数据，以提供更好的性能。此外，在对 Hudi 表进行一些架构更改或表更新后，您还可以使用 [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) 手动更新其元数据，从而确保 StarRocks 能够尽早获取最新的元数据并生成合适的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略使 StarRocks 集群中的 FE 能够从 Hive 元存储中读取事件，例如添加列、删除分区和更新数据。StarRocks 可以根据这些事件自动更新 FE 中缓存的元数据。这意味着您不需要手动更新 Hudi 表的元数据。

要启用自动增量更新，请按照以下步骤操作：

#### 步骤 1：为 Hive 元存储配置事件监听器

Hive Metastore v2.x 和 v3.x 都支持配置事件监听器。本步骤以 Hive Metastore v3.1.2 使用的事件监听器配置为例。将以下配置项添加到 **$HiveMetastore/conf/hive-site.xml** 文件，然后重新启动 Hive 元存储：

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

您可以在 FE 日志文件中搜索 `event id` 来检查事件监听器是否配置成功。如果配置失败，`event id` 值为 `0`。

#### 步骤 2：在 StarRocks 上启用自动增量更新

您可以为 StarRocks 集群中的单个 Hudi 目录或所有 Hudi 目录启用自动增量更新。

- 要为单个 Hudi 目录启用自动增量更新，请在创建 Hudi 目录时将 `PROPERTIES` 中的 `enable_hms_events_incremental_sync` 参数设置为 `true`，如下所示：

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

- 要为所有 Hudi 目录启用自动增量更新，请在每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中添加 `"enable_hms_events_incremental_sync" = "true"`，然后重新启动每个 FE 以使参数设置生效。

您也可以根据业务需求调整每个 FE 的 `$FE_HOME/conf/fe.conf` 文件中的以下参数，然后重启每个 FE 以使参数设置生效。

| 参数 | 说明 |
| --- | --- |
| hms_events_polling_interval_ms | StarRocks 从 Hive 元存储读取事件的时间间隔。默认值：`5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc | StarRocks 一次可以读取的最大事件数。默认值：`500`。 |
| enable_hms_parallel_process_events | 指定 StarRocks 在读取事件时是否并行处理事件。有效值：`true` 和 `false`。默认值：`true`。值 `true` 启用并行性，值 `false` 禁用并行性。 |
| hms_process_events_parallel_num | StarRocks 可以并行处理的最大事件数。默认值：`4`。 |

## 附录：了解元数据自动异步更新

自动异步更新是 StarRocks 用于更新 Hudi 目录中元数据的默认策略。

默认情况下（即当 `enable_metastore_cache` 和 `enable_remote_file_cache` 参数都设置为 `true` 时），如果查询命中 Hudi 表的某个分区，StarRocks 会自动缓存该分区的元数据以及该分区底层数据文件的元数据。缓存的元数据使用惰性更新策略进行更新。

例如，有一个名为 `table2` 的 Hudi 表，它有四个分区：`p1`、`p2`、`p3` 和 `p4`。查询命中 `p1`，StarRocks 会缓存 `p1` 的元数据以及 `p1` 底层数据文件的元数据。假设更新和丢弃缓存元数据的默认时间间隔如下：

- 异步更新 `p1` 缓存元数据的时间间隔（由 `metastore_cache_refresh_interval_sec` 参数指定）为 2 小时。
- 异步更新 `p1` 底层数据文件缓存元数据的时间间隔（由 `remote_file_cache_refresh_interval_sec` 参数指定）为 60 秒。
- 自动丢弃 `p1` 的缓存元数据的时间间隔（由 `metastore_cache_ttl_sec` 参数指定）为 24 小时。
- 自动丢弃 `p1` 底层数据文件缓存元数据的时间间隔（由 `remote_file_cache_ttl_sec` 参数指定）为 36 小时。

下图显示了时间轴上的时间间隔，以便于理解。

![更新和丢弃缓存元数据的时间轴](../../assets/catalog_timeline.png)

然后 StarRocks 按照以下规则更新或丢弃元数据：

- 如果另一个查询再次命中 `p1`，并且从上次更新到现在的时间小于 60 秒，StarRocks 不会更新 `p1` 的缓存元数据或 `p1` 底层数据文件的缓存元数据。
- 如果另一个查询再次命中 `p1`，并且从上次更新到现在的时间超过 60 秒，StarRocks 会更新 `p1` 底层数据文件的缓存元数据。
- 如果另一个查询再次命中 `p1`，并且从上次更新到现在的时间超过 2 小时，StarRocks 会更新 `p1` 的缓存元数据。
- 如果 `p1` 在上次更新后的 24 小时内没有被访问，StarRocks 会丢弃 `p1` 的缓存元数据。下次查询时将重新缓存元数据。
- 如果 `p1` 在上次更新后的 36 小时内没有被访问，StarRocks 会丢弃 `p1` 底层数据文件的缓存元数据。下次查询时将重新缓存元数据。
```
- 如果另一次查询再次命中`p1`，且当前时间距离上次更新小于60秒，StarRocks不会更新`p1`的缓存元数据或`p1`底层数据文件的缓存元数据。
- 如果另一次查询再次命中`p1`，且当前时间距离上次更新超过60秒，StarRocks将更新`p1`底层数据文件的缓存元数据。
- 如果另一次查询再次命中`p1`，且当前时间距离上次更新超过2小时，StarRocks将更新`p1`的缓存元数据。
- 如果自上次更新后24小时内没有访问`p1`，StarRocks将自动丢弃`p1`的缓存元数据。下一次查询时将重新缓存元数据。