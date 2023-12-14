---
displayed_sidebar: "Chinese"
---

# 统一目录

从v3.2版开始，星罗提供了一种称为统一目录的外部目录类型，用于处理来自各种数据源（包括Apache Hive™、Apache Iceberg、Apache Hudi和Delta Lake）的表，并将其作为统一数据源提供，无需摄入。使用统一目录，您可以：

- 直接查询存储在Hive、Iceberg、Hudi和Delta Lake中的数据，无需手动创建表。
- 使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)或异步物化视图（从v2.5版开始支持）处理存储在Hive、Iceberg、Hudi和Delta Lake中的数据，并将数据加载到StarRocks中。
- 在StarRocks上执行操作以创建或删除Hive和Iceberg数据库和表。

为确保在统一数据源上成功执行SQL工作负载，您的StarRocks集群需集成两个重要组件：

- 分布式文件系统（HDFS）或像AWS S3、微软Azure存储、谷歌GCS或其他兼容S3的对象存储系统

- 像Hive元存储或AWS Glue的元存储

  > **注意**
  >
  > 如果选择AWS S3作为存储系统，则可以使用HMS或AWS Glue作为元存储。如果选择其他任何存储系统，则只能使用HMS作为元存储。

## 限制

一个统一目录仅支持与单一存储系统和单一元存储服务的集成。因此，请确保您希望作为StarRocks统一数据源集成的所有数据源均使用相同的存储系统和元存储服务。

## 使用说明

- 参见[Hive目录](../../data_source/catalog/hive_catalog.md)、[Iceberg目录](../../data_source/catalog/iceberg_catalog.md)、[Hudi目录](../../data_source/catalog/hudi_catalog.md)和[Delta Lake目录](../../data_source/catalog/deltalake_catalog.md)中的"使用说明"部分，了解支持的文件格式和数据类型。

- 仅支持特定表格式的特定格式操作。例如，[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)和[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)仅支持Hive和Iceberg，而[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)仅支持Hive和Hudi。

  使用[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)语句在统一目录中创建表时，使用`ENGINE`参数指定表格式（Hive或Iceberg）。

## 集成准备

在创建统一目录之前，请确保您的StarRocks集群可以与统一数据源的存储系统和元存储集成。

### AWS IAM

如果您将AWS S3作为存储或AWS Glue作为元存储，选择合适的身份验证方法并做好必要准备，以确保您的StarRocks集群能够访问相关的AWS云资源。有关更多信息，请参见[身份验证到AWS资源 - 准备工作](../../integrations/authenticate_to_aws_resources.md#preparations)。

### HDFS

如果选择HDFS作为存储，请按照以下方式配置StarRocks集群：

- （可选）设置用于访问HDFS集群和Hive元存储的用户名。默认情况下，StarRocks使用FE和BE进程的用户名来访问HDFS集群和Hive元存储。您还可以通过在每个FE的**fe/conf/hadoop_env.sh**文件的开头和每个BE的**be/conf/hadoop_env.sh**文件的开头添加`export HADOOP_USER_NAME="<user_name>"`来设置用户名。在这些文件中设置完用户名后，重新启动每个FE和每个BE以使参数设置生效。每个StarRocks集群只能设置一个用户名。
- 查询数据时，您的StarRocks集群的FE和BE使用HDFS客户端访问HDFS集群。在大多数情况下，您不需要配置StarRocks集群即可实现此目的，StarRocks会使用默认配置启动HDFS客户端。仅在以下情况下需要配置StarRocks集群：
  - 您的HDFS集群启用了高可用性（HA）：将HDFS集群的**hdfs-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。
  - 您的HDFS集群启用了View File System（ViewFs）：将HDFS集群的**core-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。

> **注意**
>
> 如果在发送查询时返回了指示未知主机的错误，则必须将HDFS集群节点的主机名与IP地址的映射添加到**/etc/hosts**路径。

### Kerberos认证

如果您的HDFS集群或Hive元存储启用了Kerberos认证，请按照以下方式配置StarRocks集群：

- 在每个FE和每个BE上运行`kinit -kt keytab_path principal`命令，以从密钥分发中心（KDC）获取票证授权票（TGT）。运行此命令时，您必须具有访问您的HDFS集群和Hive元存储的权限。请注意，使用此命令访问KDC是时间敏感的。因此，您需要使用cron定期运行此命令。
- 在每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf`是krb5.conf文件的保存路径。您可以根据需要修改路径。

## 创建统一目录

### 语法

```SQL
CREATE EXTERNAL CATALOG <catalog_name>
[COMMENT <comment>]
PROPERTIES
(
    "type" = "unified",
    MetastoreParams,
    StorageCredentialParams,
    MetadataUpdateParams
)
```

### 参数

#### catalog_name

统一目录的名称。命名约定如下：

- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称区分大小写，且长度不能超过1023个字符。

#### comment

统一目录的描述。此参数是可选的。

#### type

您的数据源类型。将该值设置为`unified`。

#### MetastoreParams

有关StarRocks如何与您的元存储集成的一组参数。

##### Hive元存储

如果选择Hive元存储作为统一数据源的元存储，请按以下方式配置`MetastoreParams`：

```SQL
"unified.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询数据之前，您必须将Hive元存储节点的主机名和IP地址的映射添加到**/etc/hosts**路径。否则，当您启动查询时，StarRocks可能无法访问Hive元存储。

以下表格描述了在`MetastoreParams`中需要配置的参数。

| 参数                  | 是否必填 | 描述                                                      |
| --------------------- | -------- | --------------------------------------------------------- |
| unified.metastore.type | 是       | 您用于统一数据源的元存储类型。将该值设置为`hive`。       |
| hive.metastore.uris   | 是       | 您的Hive元存储的URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。如果您的Hive元存储启用了高可用性（HA），您可以指定多个元存储URI，并用逗号（`,`）分隔它们，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue

如果选择AWS Glue作为数据源的元存储，仅当选择AWS S3作为存储时才支持。请执行以下操作之一：

- 要选择基于实例配置文件的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择基于假定角色的身份验证方法，请按以下方式配置`MetastoreParams`：

  ```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 要选择基于IAM用户的身份验证方法，请按以下方式配置`MetastoreParams`：

```SQL
  "unified.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

  以下表格描述了您需要在`MetastoreParams`中配置的参数。

  | 参数                             | 必填项 | 描述                                                         |
  | ----------------------------- | -------- | ------------------------------------------------------------ |
  | unified.metastore.type        | 是      | 您用于统一数据源的元存储类型。将该值设置为 `glue`。 |
  | aws.glue.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证。 有效值：`true` 和 `false`。默认值：`false`。 |
  | aws.glue.iam_role_arn         | 否      | 具有对您的AWS Glue数据目录具有权限的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS Glue，则必须指定此参数。 |
  | aws.glue.region               | 是      | 您的AWS Glue数据目录所在的地区。 示例：`us-west-1`。 |
  | aws.glue.access_key           | 否      | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，则必须指定此参数。 |
  | aws.glue.secret_key           | 否      | 您的AWS IAM用户的密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，则必须指定此参数。 |

  有关如何选择访问AWS Glue的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS Glue的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

  #### StorageCredentialParams

  有关StarRocks与您的存储系统集成的一组参数。此参数集是可选的。

  如果您使用HDFS作为存储，您不需要配置`StorageCredentialParams`。

  如果您使用AWS S3、其他兼容的S3存储系统、Microsoft Azure存储或Google GCS作为存储，则必须配置`StorageCredentialParams`。

  ##### AWS S3

  如果您选择AWS S3作为存储系统，请执行以下操作之一：

  - 若要选择基于实例配置文件的身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.region" = "<aws_s3_region>"
    ```

  - 若要选择基于假定角色的身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "aws.s3.use_instance_profile" = "true",
    "aws.s3.iam_role_arn" = "<iam_role_arn>",
    "aws.s3.region" = "<aws_s3_region>"
    ```

  - 若要选择基于IAM用户的身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "aws.s3.use_instance_profile" = "false",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>",
    "aws.s3.region" = "<aws_s3_region>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | 参数                   | 必填项 | 描述                                                  |
  | ----------------------- | -------- | ------------------------------------------------------------ |
  | aws.s3.use_instance_profile | 是      | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：`true` 和 `false`。默认值：`false`。 |
  | aws.s3.iam_role_arn         | 否      | 具有对您的AWS S3存储桶具有权限的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS S3，则必须指定此参数。  |
  | aws.s3.region               | 是      | 您的AWS S3存储桶所在的地区。 示例：`us-west-1`。 |
  | aws.s3.access_key           | 否      | 您的IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。 |
  | aws.s3.secret_key           | 否      | 您的IAM用户的密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，则必须指定此参数。 |

  有关如何选择访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参见[访问AWS S3的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

  ##### 兼容S3存储系统

  如果您选择像MinIO这样的兼容S3存储系统作为存储，请使用以下方式配置`StorageCredentialParams`以确保成功集成：

  ```SQL
  "aws.s3.enable_ssl" = "false",
  "aws.s3.enable_path_style_access" = "true",
  "aws.s3.endpoint" = "<s3_endpoint>",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | 参数                        | 必填项 | 描述                                                  |
  | --------------------------- | -------- | ------------------------------------------------------------ |
  | aws.s3.enable_ssl           | 是      | 指定是否启用SSL连接。<br />有效值：`true` 和 `false`。默认值：`true`。 |
  | aws.s3.enable_path_style_access | 是      | 指定是否启用路径样式访问。<br />有效值：`true` 和 `false`。默认值：`false`。对于MinIO，您必须将值设置为`true`。<br />路径样式URL使用以下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈州）地区创建了一个名为`DOC-EXAMPLE-BUCKET1`的桶，并且要访问该桶中的`alice.jpg`对象，您可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
  | aws.s3.endpoint             | 是      | 用于连接到您的兼容S3存储系统的终端。 |
  | aws.s3.access_key           | 是      | 您的IAM用户的访问密钥。 |
  | aws.s3.secret_key           | 是      | 您的IAM用户的密钥。 |

  ##### Microsoft Azure存储

  ###### Azure Blob存储

  如果您选择Blob存储作为存储，请执行以下操作之一：

  - 要选择共享密钥身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "azure.blob.storage_account" = "<blob_storage_account_name>",
    "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | 参数             | 必填项 | 描述                               |
  | ----------------- | -------- | -------------------------------------- |
  | azure.blob.storage_account | 是      | 您Blob存储帐户的用户名。   |
  | azure.blob.shared_key      | 是      | 您Blob存储帐户的共享密钥。 |

  - 要选择SAS令牌身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "azure.blob.account_name" = "<blob_storage_account_name>",
    "azure.blob.container_name" = "<blob_container_name>",
    "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | 参数      | 必填项 | 描述                                              |
  | --------- | -------- | ----------------------------------------- |
  | azure.blob.account_name   | 是      | 您的Blob存储帐户的用户名。                   |
  | azure.blob.container_name | 是      | 存储您数据的blob容器的名称。         |
  | azure.blob.sas_token      | 是      | 用于访问您的Blob存储帐户的SAS令牌。  |

  ###### Azure数据湖存储Gen1

  如果您选择Data Lake存储Gen1作为存储，请执行以下操作之一：

  - 要选择托管服务标识身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "azure.adls1.use_managed_service_identity" = "true"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | 参数                            | 必填项 | 描述                                              |
  | -------------------------------- | -------- | ----------------------------------------- |
  | azure.adls1.use_managed_service_identity | 是      | 指定是否启用托管服务标识身份验证方法。将值设置为 `true`。 |

  - 要选择服务主体身份验证方法，请配置`StorageCredentialParams`如下：

    ```SQL
    "azure.adls1.oauth2_client_id" = "<application_client_id>",
    "azure.adls1.oauth2_credential" = "<application_client_credential>",
    "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
    ```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                             | **必需** | **描述**                                                  |
| ------------------------------------ | -------- | --------------------------------------------------------- |
| azure.adls1.oauth2_client_id         | 是       | 服务主体的客户端（应用程序）ID。                          |
| azure.adls1.oauth2_credential        | 是       | 创建的新客户端（应用程序）秘密的值。                      |
| azure.adls1.oauth2_endpoint          | 是       | 服务主体或应用程序的OAuth 2.0令牌端点（v1）。            |

###### Azure Data Lake Storage Gen2

如果选择Data Lake Storage Gen2作为存储解决方案，请执行以下操作之一：

- 选择托管标识身份验证方法，请按如下方式配置`StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                                     | **必需** | **描述**                                                  |
  | -------------------------------------------- | -------- | --------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity      | 是       | 指定是否启用托管标识身份验证方法。将值设置为`true`。    |
  | azure.adls2.oauth2_tenant_id                 | 是       | 您要访问其数据的租户ID。                                  |
  | azure.adls2.oauth2_client_id                 | 是       | 托管标识的客户端（应用程序）ID。                          |

- 选择共享密钥身份验证方法，请按如下方式配置`StorageCredentialParams`：

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                        | **必需** | **描述**                                                  |
  | ------------------------------- | -------- | --------------------------------------------------------- |
  | azure.adls2.storage_account     | 是       | 您的Data Lake Storage Gen2存储账户的用户名。               |
  | azure.adls2.shared_key          | 是       | 您的Data Lake Storage Gen2存储账户的共享密钥。             |

- 选择服务主体身份验证方法，请按如下方式配置`StorageCredentialParams`：

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                         | **必需** | **描述**                                                  |
  | -------------------------------- | -------- | --------------------------------------------------------- |
  | azure.adls2.oauth2_client_id      | 是       | 服务主体的客户端（应用程序）ID。                          |
  | azure.adls2.oauth2_client_secret  | 是       | 创建的新客户端（应用程序）秘密的值。                      |
  | azure.adls2.oauth2_client_endpoint | 是       | 服务主体或应用程序的OAuth 2.0令牌端点（v1）。            |

##### Google GCS

如果选择Google GCS作为存储解决方案，请执行以下操作之一：

- 选择基于VM的身份验证方法，请按如下方式配置`StorageCredentialParams`：

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                                  | **默认值** | **值示例** | **描述**                                                  |
  | ----------------------------------------- | ---------- | ---------- | --------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true       | 指定是否直接使用绑定到您的Compute Engine的服务账号。     |

- 选择基于服务账号的身份验证方法，请按如下方式配置`StorageCredentialParams`：

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                       | **默认值** | **值示例**                                                   | **描述**                                                  |
  | ------------------------------ | ---------- | ----------------------------------------------------------- | --------------------------------------------------------- |
  | gcp.gcs.service_account_email  | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建服务账号时生成的JSON文件中的电子邮件地址。        |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | 在创建服务账号时生成的JSON文件中的私有密钥ID。         |
  | gcp.gcs.service_account_private_key | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建服务账号时生成的JSON文件中的私有密钥。            |

- 选择基于模拟的身份验证方法，请按如下方式配置`StorageCredentialParams`：

  - 使VM实例模拟服务账号：
  
    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                                  | **默认值** | **值示例** | **描述**                                                  |
    | ----------------------------------------- | ---------- | ---------- | --------------------------------------------------------- |
    | gcp.gcs.use_compute_engine_service_account | false      | true       | 指定是否直接使用绑定到您的Compute Engine的服务账号。    |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"    | 您要模拟的服务账号。                                      |

  - 使服务账号（临时命名为元服务账号）模拟另一个服务账号（临时命名为数据服务账号）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                               | **默认值** | **值示例**                                                   | **描述**                                                  |
    | -------------------------------------- | ---------- | ----------------------------------------------------------- | --------------------------------------------------------- |
    | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务账号时生成的JSON文件中的电子邮件地址。     |
    | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | 在创建元服务账号时生成的JSON文件中的私有密钥ID。        |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 在创建元服务账号时生成的JSON文件中的私有密钥。           |
    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 您要模拟的数据服务账号。                                 |

#### MetadataUpdateParams

一组关于StarRocks如何更新Hive、Hudi和Delta Lake的缓存元数据的参数。此参数集是可选的。有关来自Hive、Hudi和Delta Lake的缓存元数据更新策略的更多信息，请参见[Hive目录](../../data_source/catalog/hive_catalog.md)、[Hudi目录](../../data_source/catalog/hudi_catalog.md)和 [Delta Lake目录](../../data_source/catalog/deltalake_catalog.md)。

在大多数情况下，您可以忽略`MetadataUpdateParams`，不需要调整其中的策略参数，因为这些参数的默认值已经为您提供了即插即用的性能。

但是，如果Hive、Hudi或Delta Lake中数据更新的频率很高，您可以调整这些参数以进一步优化自动异步更新的性能。

| 参数                     | 必需  | 描述                                                  |
| ------------------------ | ----- | ----------------------------------------------------- |
| enable_metastore_cache   | 否    | 指定StarRocks是否缓存Hive、Hudi或Delta Lake表的元数据。有效值：`true`和`false`。默认值：`true`。`true`表示启用缓存，`false`表示禁用缓存。 |
| enable_remote_file_cache               | 否         | 指定StarRocks是否缓存Hive、Hudi或Delta Lake表或分区的底层数据文件的元数据。有效值为`true`和`false`。默认值为`true`。值为true时启用缓存，值为false时禁用缓存。 |
| metastore_cache_refresh_interval_sec   | 否         | StarRocks异步更新其自身缓存的Hive、Hudi或Delta Lake表或分区的元数据的时间间隔。单位：秒。默认值为`7200`，即2小时。 |
| remote_file_cache_refresh_interval_sec | 否         | StarRocks异步更新其自身缓存的Hive、Hudi或Delta Lake表或分区的底层数据文件的元数据的时间间隔。单位：秒。默认值为`60`。 |
| metastore_cache_ttl_sec                | 否         | StarRocks自动丢弃其自身缓存的Hive、Hudi或Delta Lake表或分区的元数据的时间间隔。单位：秒。默认值为`86400`，即24小时。 |
| remote_file_cache_ttl_sec              | 否         | StarRocks自动丢弃其自身缓存的Hive、Hudi或Delta Lake表或分区的底层数据文件的元数据的时间间隔。单位：秒。默认值为`129600`，即36小时。 |

### 示例

以下示例创建一个名为`unified_catalog_hms`或`unified_catalog_glue`的统一目录，具体取决于您使用的元数据存储类型，以从统一数据源查询数据。

#### HDFS

如果您使用HDFS作为存储，运行以下命令：

```SQL
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### 基于实例配置文件的身份验证

- 如果您使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您使用Amazon EMR的AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 基于假定角色的身份验证

- 如果您使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您使用Amazon EMR的AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### 基于IAM用户的身份验证

- 如果您使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您使用Amazon EMR的AWS Glue，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_glue
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "glue",
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
CREATE EXTERNAL CATALOG unified_catalog_hms
PROPERTIES
(
    "type" = "unified",
    "unified.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "aws.s3.enable_ssl" = "true",
    "aws.s3.enable_path_style_access" = "true",
    "aws.s3.endpoint" = "<s3_endpoint>",
    "aws.s3.access_key" = "<iam_user_access_key>",
    "aws.s3.secret_key" = "<iam_user_secret_key>"
);
```

#### 微软Azure存储

##### Azure Blob 存储

- 如果您选择共享密钥验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果您选择SAS令牌验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure 数据湖存储 Gen1

- 如果您选择托管服务标识身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果您选择服务主体身份验证方法，请运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure 数据湖存储 Gen2

- 如果选择托管身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择服务主体身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果选择基于VM的身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务账号的身份验证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG unified_catalog_hms
  PROPERTIES
  (
      "type" = "unified",
      "unified.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于模拟的身份验证方法：

  - 如果让VM实例模拟服务账号，请运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果让服务账号模拟另一个服务账号，请运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG unified_catalog_hms
    PROPERTIES
    (
        "type" = "unified",
        "unified.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    
    );
    ```

## 查看统一目录

您可以使用[SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md)来查询当前StarRocks集群中的所有目录：

```SQL
SHOW CATALOGS;
```

您也可以使用[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)来查询外部目录的创建语句。以下示例查询名为 `unified_catalog_glue` 的统一目录的创建语句：

```SQL
SHOW CREATE CATALOG unified_catalog_glue;
```

## 切换到统一目录及其中的数据库

您可以使用以下方法之一切换到统一目录及其中的数据库：

- 使用[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)来指定当前会话中的统一目录，然后使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来指定活动数据库：

  ```SQL
  -- 切换到当前会话中的指定目录：
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库：
  USE <db_name>
  ```

- 直接使用[USE](../../sql-reference/sql-statements/data-definition/USE.md)来切换到统一目录及其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除统一目录

您可以使用[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)来删除外部目录。

以下示例删除名为 `unified_catalog_glue` 的统一目录：

```SQL
DROP CATALOG unified_catalog_glue;
```

## 查看统一目录中表的模式

您可以使用以下语法之一来查看统一目录中表的模式：

- 查看模式

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 查看通过CREATE语句的模式和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 从统一目录查询数据

要从统一目录查询数据，请执行以下步骤：

1. 使用[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)来查看统一数据源中的数据库，这些数据库与统一目录相关联：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到Hive Catalog及其中的数据库](#switch-to-a-unified-catalog-and-a-database-in-it)。

3. 使用[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)来查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从Hive、Iceberg、Hudi或Delta Lake加载数据

您可以使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)将Hive、Iceberg、Hudi或Delta Lake表的数据加载到统一目录中创建的StarRocks表中。

以下示例将Hive表 `hive_table` 的数据加载到属于统一目录 `unified_catalog` 的数据库 `test_database` 中创建的名为 `test_tbl` 的StarRocks表中：

```SQL
INSERT INTO unified_catalog.test_database.test_table SELECT * FROM hive_table
```

## 在统一目录中创建数据库

与StarRocks的内部目录类似，如果您对统一目录拥有[CREATE DATABASE](../../administration/privilege_item.md#catalog)权限，可以使用[CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)语句在该目录中创建数据库。

> **注意**
>
> 您可以通过使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

StarRocks仅支持在统一目录中创建Hive和Iceberg数据库。

[切换到统一目录](#switch-to-a-unified-catalog-and-a-database-in-it)，然后使用以下语句在该目录中创建数据库：

```SQL
CREATE DATABASE <database_name>
[properties ("location" = "<prefix>://<path_to_database>/<database_name.db>")]
```

`location` 参数指定要创建数据库的文件路径，可以是HDFS或云存储中的文件路径。

- 当您使用Hive元存储作为数据源的元存储时，如果在数据库创建时未指定该参数，则`location`参数默认为`<warehouse_location>/<database_name.db>`，Hive元存储支持此参数，如果在数据库创建时未指定该参数。
- 当您将AWS Glue用作数据源的元存储时，`location`参数没有默认值，因此您必须在创建数据库时指定该参数。

`前缀`根据您使用的存储系统而变化：

| **存储系统**        | **`前缀`值**  |
| ----------------- | ------------- |
| HDFS              | `hdfs`        |
| Google GCS        | `gs`          |
| Azure Blob 存储   | <ul><li>如果您的存储账户允许通过HTTP访问，则`前缀`是`wasb`。</li><li>如果您的存储账户允许通过HTTPS访问，则`前缀`是`wasbs`。</li></ul> |
| Azure Data Lake 存储 Gen1 | `adl`  |
| Azure Data Lake 存储 Gen2 | <ul><li>如果您的存储账户允许通过HTTP访问，则`前缀`是`abfs`。</li><li>如果您的存储账户允许通过HTTPS访问，则`前缀`是`abfss`。</li></ul> |
| AWS S3或其他兼容S3的存储（例如，MinIO）| `s3` |

## 从统一目录中删除数据库

与StarRocks的内部数据库类似，如果您具有[DROP](../../administration/privilege_item.md#database)权限，则可以使用[DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md)语句删除统一目录中创建的数据库。您只能删除空数据库。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

StarRocks仅支持从统一目录中删除Hive和Iceberg数据库。

从统一目录中删除数据库时，数据库在HDFS群集或云存储上的文件路径不会随着数据库一起被删除。

[切换到统一目录](#切换到统一目录及其中的数据库)，然后使用以下语句在该目录中删除数据库：

```SQL
DROP DATABASE <database_name>
```

## 在统一目录中创建表

与StarRocks的内部数据库类似，如果您具有在统一目录中创建表的[CREATE TABLE](../../administration/privilege_item.md#database)权限，则可以使用[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)或[CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md)语句在该数据库中创建表。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

StarRocks仅支持在统一目录中创建Hive和Iceberg表。

[切换到Hive目录及其中的数据库](#切换到统一目录及其中的数据库)，然后使用[CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)在该数据库中创建Hive或Iceberg表：

```SQL
CREATE TABLE <table_name>
(column_definition1[, column_definition2, ...]
ENGINE = {|hive|iceberg}
[partition_desc]
```

更多信息，请参见[创建Hive表](../catalog/hive_catalog.md#create-a-hive-table)和[创建Iceberg表](../catalog/iceberg_catalog.md#create-an-iceberg-table)。

以下示例创建一个名为`hive_table`的Hive表。该表由三列`action`、`id`和`dt`组成，其中`id`和`dt`为分区列。

```SQL
CREATE TABLE hive_table
(
    action varchar(65533),
    id int,
    dt date
)
ENGINE = hive
PARTITION BY (id,dt);
```

## 将数据写入统一目录中的表

与StarRocks的内部表类似，如果您具有在统一目录中创建表的[INSERT](../../administration/privilege_item.md#table)权限，则可以使用[INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md)语句将StarRocks表的数据写入统一目录表（目前仅支持Parquet格式的统一目录表）。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

StarRocks仅支持将数据写入统一目录中的Hive和Iceberg表。

[切换到Hive目录及其中的数据库](#切换到统一目录及其中的数据库)，然后使用[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)将数据插入该数据库中的Hive或Iceberg表：

```SQL
INSERT {INTO | OVERWRITE} <table_name>
[ (column_name [, ...]) ]
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- 如果您要将数据写入指定的分区，请使用以下语法：
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

更多信息，请参见[将数据写入Hive表](../catalog/hive_catalog.md#sink-data-to-a-hive-table)和[将数据写入Iceberg表](../catalog/iceberg_catalog.md#sink-data-to-an-iceberg-table)。

以下示例向名为`hive_table`的Hive表插入三行数据：

```SQL
INSERT INTO hive_table
VALUES
    ("buy", 1, "2023-09-01"),
    ("sell", 2, "2023-09-02"),
    ("buy", 3, "2023-09-03");
```

## 从统一目录中删除表

与StarRocks的内部表类似，如果您具有在统一目录中创建表的[DROP](../../administration/privilege_item.md#table)权限，则可以使用[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)语句删除该表。

> **注意**
>
> 您可以使用[GRANT](../../sql-reference/sql-statements/account-management/GRANT.md)和[REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md)来授予和撤销权限。

StarRocks仅支持从统一目录中删除Hive和Iceberg表。

[切换到Hive目录及其中的数据库](#切换到统一目录及其中的数据库)，然后使用[DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md)删除该数据库中的Hive或Iceberg表：

```SQL
DROP TABLE <table_name>
```

更多信息，请参见[删除Hive表](../catalog/hive_catalog.md#drop-a-hive-table)和[删除Iceberg表](../catalog/iceberg_catalog.md#drop-an-iceberg-table)。

以下示例删除名为`hive_table`的Hive表：

```SQL
DROP TABLE hive_table FORCE
```