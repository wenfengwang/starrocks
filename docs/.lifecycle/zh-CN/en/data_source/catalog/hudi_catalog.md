---
displayed_sidebar: "Chinese"
---

# Hudi目录

Hudi目录是一种外部目录，可以使您在不进行摄取的情况下查询来自Apache Hudi的数据。

此外，您还可以直接通过使用基于Hudi目录的[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)来转换和加载来自Hudi的数据。StarRocks从v2.4开始支持Hudi目录。

为了确保在您的Hudi群集上成功执行SQL工作负载，您的StarRocks集群需要与两个重要组件集成：

- 分布式文件系统（HDFS）或对象存储（如AWS S3、Microsoft Azure Storage、Google GCS或其他兼容S3的存储系统（例如MinIO））

  

- Hive Metastore或AWS Glue等元数据存储

  > **注意**

  >

  > 如果您选择AWS S3作为存储系统，则可以将HMS或AWS Glue作为元数据存储。如果选择其他存储系统，则只能使用HMS作为元数据存储。


## 使用说明

- StarRocks支持的Hudi文件格式为Parquet。Parquet文件支持以下压缩格式：SNAPPY、LZ4、ZSTD、GZIP和NO_COMPRESSION。

- StarRocks完全支持Hudi的COW（复制写）表和MOR（读取时合并）表。

## 集成准备

在创建Hudi目录之前，请确保您的StarRocks集群能够与Hudi集群的存储系统和元数据存储集成。

### AWS IAM

如果您的Hudi集群使用AWS S3作为存储系统或AWS Glue作为元数据存储，请选择合适的认证方法并进行必要的准备工作，以确保您的StarRocks集群能够访问相关的AWS云资源。

推荐以下认证方法：

- 实例配置文件

- 假定角色

- IAM用户

在上述三种认证方法中，实例文件配置是最常用的。


有关更多信息，请参阅[AWS IAM认证准备](../../integrations/authenticate_to_aws_resources.md#preparations)。


### HDFS

如果您选择HDFS作为存储系统，请按照以下方式配置您的StarRocks集群：

- （可选）设置用于访问HDFS集群和Hive元数据存储的用户名。默认情况下，StarRocks使用FE和BE进程的用户名来访问您的HDFS集群和Hive元数据存储。 您还可以通过在每个FE的**fe/conf/hadoop_env.sh**文件的开头和每个BE的**be/conf/hadoop_env.sh**文件的开头添加`export HADOOP_USER_NAME="<user_name>"`来设置用户名。在这些文件中设置了用户名后，请重新启动每个FE和每个BE以使参数设置生效。您只能为每个StarRocks集群设置一个用户名。

- 当查询Hudi数据时，您的StarRocks集群的FE和BE使用HDFS客户端访问您的HDFS集群。在大多数情况下，您不需要配置StarRocks集群来实现该目的，StarRocks会使用默认配置启动HDFS客户端。您只需要在以下情况下配置StarRocks集群：

  - 您的HDFS集群启用了高可用性（HA）：将您的HDFS集群的**hdfs-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。
  - 您的HDFS集群启用了View File System（ViewFs）：将您的HDFS集群的**core-site.xml**文件添加到每个FE的**$FE_HOME/conf**路径和每个BE的**$BE_HOME/conf**路径。

> **注意**

>

> 如果发送查询时返回未知主机错误，则必须将您的HDFS集群节点的主机名和IP地址的映射添加到**/etc/hosts**路径。

### Kerberos认证

如果您的HDFS集群或Hive元数据存储启用了Kerberos认证，请按照以下方式配置您的StarRocks集群：

- 在每个FE和每个BE上运行`kinit -kt keytab_path principal`命令，以从密钥分发中心（KDC）获取TGT（票据授予票据）。要运行此命令，您必须具有访问HDFS集群和Hive元数据存储的权限。请注意，使用此命令访问KDC是时间敏感的。因此，您需要使用cron定期运行此命令。
- 在每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。在此示例中，`/etc/krb5.conf`是**krb5.conf**文件的保存路径。您可以根据实际需要修改路径。

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


- 名称可以包含字母、数字（0-9）和下划线（_）。必须以字母开头。
- 名称大小写敏感，长度不能超过1023个字符。


#### comment

Hudi目录的描述。此参数是可选的。

#### type


您的数据源类型。将值设置为`hudi`。

#### MetastoreParams

关于StarRocks如何与您的数据源的元数据存储集成的一组参数。

##### Hive元数据存储

如果您选择Hive元数据存储作为您的数据源的元数据存储，请按照以下方式配置`MetastoreParams`：

```SQL

"hive.metastore.type" = "hive",

"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **注意**
>
> 在查询Hudi数据之前，您必须将您的Hive元数据存储节点的主机名和IP地址的映射添加到**/etc/hosts**路径。否则，在启动查询时，StarRocks可能无法访问您的Hive元数据存储。

以下表格描述了您需要在`MetastoreParams`中配置的参数。

| 参数                | 是否必填 | 描述                                         |
| ------------------- | -------- | -------------------------------------------- |
| hive.metastore.type | 是       | 您用于Hudi集群的元数据存储的类型。将值设置为`hive`。 |
| hive.metastore.uris | 是       | 您的Hive元数据存储的URI。格式：`thrift://<metastore_IP_address>:<metastore_port>`。<br />如果您的Hive元数据存储启用了高可用性（HA），则可以指定多个元数据存储URI，并使用逗号（`,`）将它们分隔，例如，`"thrift://<metastore_IP_address_1>:<metastore_port_1>,thrift://<metastore_IP_address_2>:<metastore_port_2>,thrift://<metastore_IP_address_3>:<metastore_port_3>"`。 |

##### AWS Glue


如果您选择AWS Glue作为您的数据源的元数据存储（仅当您选择了AWS S3作为存储系统时支持），请执行以下操作之一：

- 要选择基于实例配置文件的认证方法，请按照以下方式配置`MetastoreParams`：

  ```SQL
  "hive.metastore.type" = "glue",
| aws.glue.iam_role_arn         | 否       | 拥有权限访问您的AWS Glue数据目录的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS Glue，您必须指定此参数。 |
| aws.glue.region               | 是       | 您的AWS Glue数据目录所在的区域。示例：`us-west-1`。 |
| aws.glue.access_key           | 否       | 您的AWS IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，您必须指定此参数。 |
| aws.glue.secret_key           | 否       | 您的AWS IAM用户的秘密密钥。如果您使用基于IAM用户的身份验证方法来访问AWS Glue，您必须指定此参数。 |

有关如何选择访问AWS Glue的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS Glue的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue)。

#### StorageCredentialParams

关于StarRocks如何与您的存储系统集成的一组参数。此参数集是可选的。

如果您使用HDFS作为存储，您不需要配置`StorageCredentialParams`。

如果您使用AWS S3、其他兼容S3的存储系统、Microsoft Azure Storage或Google GCS作为存储，您必须配置`StorageCredentialParams`。

##### AWS S3

如果您选择AWS S3作为Hudi集群的存储，请执行以下操作之一：

- 选择基于实例配置文件的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 选择基于假定角色的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- 选择基于IAM用户的身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                         | 必需      | 描述                                            |
| --------------------------- | -------- | ----------------------------------------------------- |
| aws.s3.use_instance_profile | 是       | 指定是否启用基于实例配置文件的身份验证方法和基于假定角色的身份验证方法。有效值：`true`和`false`。默认值：`false`。 |
| aws.s3.iam_role_arn         | 否       | 拥有权限访问您的AWS S3存储桶的IAM角色的ARN。如果您使用基于假定角色的身份验证方法来访问AWS S3，您必须指定此参数。 |
| aws.s3.region               | 是       | 您的AWS S3存储桶所在的区域。示例：`us-west-1`。 |
| aws.s3.access_key           | 否       | 您的IAM用户的访问密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，您必须指定此参数。 |
| aws.s3.secret_key           | 否       | 您的IAM用户的秘密密钥。如果您使用基于IAM用户的身份验证方法来访问AWS S3，您必须指定此参数。 |

有关如何选择访问AWS S3的身份验证方法以及如何在AWS IAM控制台中配置访问控制策略的信息，请参阅[访问AWS S3的身份验证参数](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3)。

##### 兼容S3的存储系统

Hudi目录从v2.5版本开始支持兼容S3的存储系统。

如果您选择兼容S3的存储系统（如MinIO）作为Hudi集群的存储，请配置`StorageCredentialParams`如下，以确保成功集成：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| 参数                        | 必需      | 描述                                            |
| ---------------------------- | -------- | ------------------------------------------------------ |
| aws.s3.enable_ssl            | 是       | 指定是否启用SSL连接。<br />有效值：`true`和`false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是       | 指定是否启用路径样式访问。<br />有效值：`true`和`false`。默认值：`false`。对于MinIO，您必须设置该值为`true`。<br />路径样式URL的格式如下：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域创建了名为`DOC-EXAMPLE-BUCKET1`的存储桶，并且要访问该存储桶中的`alice.jpg`对象，您可以使用以下路径样式URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint              | 是       | 用于连接到兼容S3的存储系统的终结点。 |
| aws.s3.access_key            | 是       | 您的IAM用户的访问密钥。 |
| aws.s3.secret_key            | 是       | 您的IAM用户的秘密密钥。 |

##### Microsoft Azure Storage

Hudi目录从v3.0版本开始支持Microsoft Azure Storage。

###### Azure Blob Storage

如果您选择Blob Storage作为Hudi集群的存储，请执行以下操作之一：

- 选择Shared Key身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**              | **必需** | **描述**                                  |
  | ---------------------- | -------- | -------------------------------------------- |
  | azure.blob.storage_account | 是       | 您的Blob Storage帐户的用户名。  |
  | azure.blob.shared_key | 是       | 您的Blob Storage帐户的共享密钥。 |

- 选择SAS Token身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**             | **必需** | **描述**                                            |
  | --------------------- | -------- | ------------------------------------------------------ |
  | azure.blob.account_name | 是       | 您的Blob Storage帐户的用户名。                     |
  | azure.blob.container_name | 是       | 存储您数据的blob容器的名称。 |
  | azure.blob.sas_token | 是       | 用于访问您的Blob Storage帐户的SAS令牌。 |

###### Azure Data Lake Storage Gen1

如果您选择Data Lake Storage Gen1作为Hudi集群的存储，请执行以下操作之一：

- 选择托管服务标识身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                            | **必需** | **描述**                                            |
  | ------------------------------------ | -------- | ------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是       | 指定是否启用托管服务标识身份验证方法。将值设置为`true`。 |

- 选择服务主体身份验证方法，将`StorageCredentialParams`配置如下：

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

  | **参数**                 | **必需** | **描述**                                           |
  | -------------------------- | -------- | ----------------------------------------------------- |
  | azure.adls1.oauth2_client_id | 是       | 服务主体的客户（应用程序）ID。                        |
  | azure.adls1.oauth2_credential | 是       | 创建的新客户（应用程序）密钥的值。                    |
  | azure.adls1.oauth2_endpoint | 是       | 服务主体或应用程序的OAuth 2.0令牌终结点（v1）。 |

###### Azure Data Lake Storage Gen2

如果您选择将Data Lake Storage Gen2用作Hudi集群的存储，执行以下操作之一：

- 要选择托管标识身份验证方法，请配置`StorageCredentialParams`如下：

```SQL
"azure.adls2.oauth2_use_managed_identity" = "true",
"azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
"azure.adls2.oauth2_client_id" = "<service_client_id>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                                 | **必需** | **描述**                                              |
| ---------------------------------------- | -------- | ------------------------------------------------------------ |
| azure.adls2.oauth2_use_managed_identity | 是       | 指定是否启用托管标识验证方法。将值设置为`true`。 |
| azure.adls2.oauth2_tenant_id            | 是       | 您想要访问其数据的租户的ID。          |
| azure.adls2.oauth2_client_id            | 是       | 托管标识的客户（应用程序）ID。         |

- 要选择共享密钥身份验证方法，请配置`StorageCredentialParams`如下：

```SQL
"azure.adls2.storage_account" = "<storage_account_name>",
"azure.adls2.shared_key" = "<shared_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**              | **必需** | **描述**                                              |
| ---------------------- | -------- | ------------------------------------------------------------ |
| azure.adls2.storage_account | 是       | 您的Data Lake Storage Gen2存储账户的用户名。 |
| azure.adls2.shared_key      | 是       | 您的Data Lake Storage Gen2存储账户的共享密钥。 |

- 要选择服务主体身份验证方法，请配置`StorageCredentialParams`如下：

```SQL
"azure.adls2.oauth2_client_id" = "<service_client_id>",
"azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
"azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                  | **必需** | **描述**                                              |
| -------------------------- | -------- | ------------------------------------------------------------ |
| azure.adls2.oauth2_client_id       | 是       | 服务主体的客户（应用程序）ID。        |
| azure.adls2.oauth2_client_secret   | 是       | 创建的新客户（应用程序）密钥的值。    |
| azure.adls2.oauth2_client_endpoint | 是       | 服务主体或应用程序的OAuth 2.0令牌端点（v1）。|

##### Google GCS

Hudi目录从v3.0开始支持Google GCS。

如果您选择将Google GCS用作Hudi集群的存储，执行以下操作之一：

- 要选择基于VM的身份验证方法，请配置`StorageCredentialParams`如下：

```SQL
"gcp.gcs.use_compute_engine_service_account" = "true"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                          | **默认值** | **示例值** | **描述**                                              |
| ---------------------------------- | --------- | ---------- | ------------------------------------------------------------ |
| gcp.gcs.use_compute_engine_service_account | false     | true       | 指定是否直接使用绑定到您的计算引擎的服务帐号。 |

- 要选择基于服务帐号的身份验证方法，请配置`StorageCredentialParams`如下：

```SQL
"gcp.gcs.service_account_email" = "<google_service_account_email>",
"gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
"gcp.gcs.service_account_private_key" = "<google_service_private_key>"
```

以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

| **参数**                                 | **默认值** | **示例值**                                              | **描述**                                              |
| --------------------------------------  | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建服务帐号时生成的JSON文件中的电子邮件地址。 |
| gcp.gcs.service_account_private_key_id  | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | 创建服务帐号时生成的JSON文件中的私钥ID。 |
| gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建服务帐号时生成的JSON文件中的私钥。 |

- 要选择基于模拟的身份验证方法，请配置`StorageCredentialParams`如下：

  - 使VM实例模拟一个服务帐号：

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                                 | **默认值** | **示例值** | **描述**                                              |
    | --------------------------------------  | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false     | true       | 指定是否直接使用绑定到您的计算引擎的服务帐号。 |
    | gcp.gcs.impersonation_service_account      | ""        | "hello"    | 您要模拟的服务帐号。|

  - 使服务帐号（暂时命名为元服务帐号）模拟另一个服务帐号（暂时命名为数据服务帐号）：

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    以下表格描述了您需要在`StorageCredentialParams`中配置的参数。

    | **参数**                                 | **默认值** | **示例值**                                              | **描述**                                              |
    | --------------------------------------  | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 在创建元服务帐号时生成的JSON文件中的电子邮件地址。 |
    | gcp.gcs.service_account_private_key_id  | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                | 创建元服务帐号时生成的JSON文件中的私钥ID。 |
    | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | 创建元服务帐号时生成的JSON文件中的私钥。 |
    | gcp.gcs.impersonation_service_account  | ""         | "hello"                                                      | 您要模拟的数据服务帐号。 |

#### MetadataUpdateParams

关于StarRocks如何更新Hudi缓存元数据的一组参数。此参数集是可选的。

默认情况下，StarRocks实现了[自动异步更新策略](#附录-了解元数据自动异步更新)。

在大多数情况下，可以忽略`MetadataUpdateParams`，不需要调整其中的策略参数，默认值已经为您提供了即插即用的性能。

但是，如果Hudi中数据的更新频率很高，可以调整这些参数以进一步优化自动异步更新的性能。

> **注意**
>
> 在大多数情况下，如果您的Hudi数据以1小时或更短的间隔更新，数据更新频率被视为很高。

| 参数                                  | 必需     | 描述                                                     |
|----------------------------------------| -------- | ------------------------------------------------------------ |
| enable_metastore_cache                 | 否       | 指定StarRocks是否缓存Hudi表的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| enable_remote_file_cache               | 否       | 指定StarRocks是否缓存Hudi表或分区的底层数据文件的元数据。有效值：`true`和`false`。默认值：`true`。值`true`启用缓存，值`false`禁用缓存。 |
| metastore_cache_refresh_interval_sec   | 否       | StarRocks异步更新其自身中缓存的Hudi表或分区的元数据的时间间隔。单位：秒。默认值：`7200`，即2小时。 |
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(

    "type" = "hudi",

    "hive.metastore.type" = "hive",

    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"

);

```

#### AWS S3

##### 如果您选择基于实例配置文件的凭据

- 如果您在Hudi集群中使用Hive元数据存储，运行以下命令：

  ```SQL

  CREATE EXTERNAL CATALOG hudi_catalog_hms

  PROPERTIES

  (

      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Hudi集群中使用AWS Glue，运行以下命令：

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

##### 如果您选择基于假定角色的凭据

- 如果您在Hudi集群中使用Hive元数据存储，运行以下命令：

  ```SQL

  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```


- 如果您在Amazon EMR Hudi集群中使用AWS Glue，运行以下命令：

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

##### 如果您选择基于IAM用户的凭据

- 如果您在Hudi集群中使用Hive元数据存储，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- 如果您在Amazon EMR Hudi集群中使用AWS Glue，运行以下命令：

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

#### 兼容S3的存储系统

以MinIO为例。运行以下命令：

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
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

- 如果您选择共享密钥身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- 如果您选择SAS令牌身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure数据湖存储Gen1

- 如果您选择托管服务标识身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- 如果您选择服务主体身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure数据湖存储Gen2

- 如果您选择托管标识身份验证方法，运行以下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- 如果选择共享密钥认证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- 如果选择服务主体认证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  );
  ```

#### Google GCS

- 如果选择基于VM的认证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- 如果选择基于服务账户的认证方法，请运行如下命令：

  ```SQL
  CREATE EXTERNAL CATALOG hudi_catalog_hms
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );
  ```

- 如果选择基于冒充的认证方法：

  - 如果让VM实例冒充服务账户，运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    
    );
    ```

  - 如果让服务账户冒充另一个服务账户，运行如下命令：

    ```SQL
    CREATE EXTERNAL CATALOG hudi_catalog_hms
    PROPERTIES
    (
        "type" = "hudi",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
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

您也可以使用 [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) 查询外部目录创建语句。以下示例查询名为 `hudi_catalog_glue` 的 Hudi 目录的创建语句：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## 切换到 Hudi 目录及其内部的数据库

您可以使用以下方法之一切换到 Hudi 目录及其内部的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中指定 Hudi 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定一个活动数据库：

  ```SQL
  -- 切换到当前会话中指定的目录:
  SET CATALOG <catalog_name>
  -- 指定当前会话中的活动数据库:
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到 Hudi 目录及其内部的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```

## 删除 Hudi 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `hudi_catalog_glue` 的 Hudi 目录：

```SQL
DROP Catalog hudi_catalog_glue;
```

## 查看 Hudi 表的模式

您可以使用以下语法之一查看 Hudi 表的模式：

- 查看模式

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 查看 CREATE 语句的模式和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```

## 查询 Hudi 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Hudi 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到 Hudi 目录及其内部的数据库](#switch-to-a-hudi-catalog-and-a-database-in-it).

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```

## 从 Hudi 加载数据

假设您有名为 `olap_tbl` 的 OLAP 表，您可以进行以下转换和加载数据的操作：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks 会缓存 Hudi 的元数据，并以异步方式自动更新元数据以提供更好的性能。此外，在对 Hudi 表进行某些模式更改或表更新后，您还可以使用 [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) 手动更新其元数据，以确保 StarRocks 能够尽快获取最新的元数据并生成适当的执行计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自动增量更新

与自动异步更新策略不同，自动增量更新策略使您的 StarRocks 集群中的 FEs 可以读取事件（例如添加列、删除分区和更新数据）以自动更新 Hudi 表的元数据缓存在 FEs 中。这意味着您无需手动更新 Hudi 表的元数据。

要启用自动增量更新，请执行以下步骤：

#### 步骤 1：为您的 Hive 元存储配置事件监听器

Hive 元存储 v2.x 和 v3.x 都支持配置事件监听器。此步骤以配置 Hive 元存储 v3.1.2 用于事件监听器的配置项为例。将以下配置项目添加到 **$HiveMetastore/conf/hive-site.xml** 文件，然后重启 Hive 元存储：

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
```
    <name>hive.metastore.server.max.message.size</name>
    <value>858993459</value>
</property>
```


您可以在FE日志文件中搜索`event id`，检查事件监听器是否成功配置。如果配置失败，`event id`的值为`0`。

#### 步骤2：在StarRocks上启用自动增量更新

您可以为单个Hudi目录或StarRocks集群中的所有Hudi目录启用自动增量更新。

- 要为单个Hudi目录启用自动增量更新，请在创建Hudi目录时在`PROPERTIES`中设置`enable_hms_events_incremental_sync`参数为`true`，如下所示：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hudi",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```

- 要为所有Hudi目录启用自动增量更新，请将`"enable_hms_events_incremental_sync" = "true"`添加到每个FE的`$FE_HOME/conf/fe.conf`文件中，然后重新启动每个FE以使参数设置生效。

您还可以根据业务需求调整每个FE的`$FE_HOME/conf/fe.conf`文件中的以下参数，然后重新启动每个FE以使参数设置生效。

| 参数                             | 描述                                                      |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks从Hive元存储中读取事件的时间间隔。默认值：`5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks可以一次读取的最大事件数。默认值：`500`。 |
| enable_hms_parallel_process_events | 指定StarRocks是否在读取事件时并行处理事件。有效值：`true`和`false`。默认值：`true`。值`true`启用并行性，值`false`禁用并行性。 |
| hms_process_events_parallel_num   | StarRocks可以并行处理的最大事件数。默认值：`4`。 |

## 附录：理解元数据自动异步更新

自动异步更新是StarRocks用于更新Hudi目录中的元数据的默认策略。

默认情况下（即当`enable_metastore_cache`和`enable_remote_file_cache`参数均设置为`true`时），如果查询命中了Hudi表的一个分区，StarRocks会自动缓存该分区的元数据以及该分区底层数据文件的元数据。缓存的元数据通过延迟更新策略进行更新。

例如，有一个名为`table2`的Hudi表，拥有四个分区：`p1`、`p2`、`p3`和`p4`。查询命中了`p1`，StarRocks缓存了`p1`的元数据以及`p1`的底层数据文件的元数据。假设默认更新和丢弃缓存元数据的时间间隔如下：

- 异步更新`p1`的缓存元数据的时间间隔（由`metastore_cache_refresh_interval_sec`参数指定）为2小时。
- 异步更新`p1`的底层数据文件的缓存元数据的时间间隔（由`remote_file_cache_refresh_interval_sec`参数指定）为60秒。
- 自动丢弃`p1`的缓存元数据的时间间隔（由`metastore_cache_ttl_sec`参数指定）为24小时。
- 自动丢弃`p1`的底层数据文件的缓存元数据的时间间隔（由`remote_file_cache_ttl_sec`参数指定）为36小时。

以下图示显示了时间轴上的时间间隔，以便更好地理解。

![更新和丢弃缓存元数据的时间轴](../../assets/catalog_timeline.png)

然后，StarRocks根据以下规则更新或丢弃元数据：

- 如果另一个查询再次命中了`p1`，并且从上次更新以来的当前时间少于60秒，则StarRocks不会更新`p1`的缓存元数据或`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了`p1`，并且从上次更新以来的当前时间超过60秒，则StarRocks会更新`p1`的底层数据文件的缓存元数据。
- 如果另一个查询再次命中了`p1`，并且从上次更新以来的当前时间超过2小时，则StarRocks会更新`p1`的缓存元数据。
- 如果从上次更新起24小时内未访问过`p1`，StarRocks会丢弃`p1`的缓存元数据。元数据将在下一个查询时缓存。
- 如果从上次更新起36小时内未访问过`p1`，StarRocks会丢弃`p1`的底层数据文件的缓存元数据。元数据将在下一个查询时缓存。