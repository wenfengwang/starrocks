---
displayed_sidebar: "Chinese"
---

# Hudi 目录

Hudi目录是一种外部目录。通过Hudi目录，您不需要执行数据导入就可以直接查询Apache Hudi里的数据。

此外，您还可以基于Hudi目录，结合[INSERT INTO](../../sql-reference/sql-statements/data-manipulation/INSERT.md)能力来实现数据转换和导入。StarRocks从2.4版本开始支持Hudi目录。

为保证正常访问Hudi内的数据，StarRocks集群必须集成以下两个关键组件:

- 分布式文件系统(HDFS)或对象存储。当前支持的对象存储包括：AWS S3、Microsoft Azure Storage、Google GCS、其他兼容S3协议的对象存储（如阿里云OSS、MinIO）。

- 元数据服务。当前支持的元数据服务包括：Hive Metastore（以下简称HMS）、AWS Glue。

  > **说明**
  >
  > 如果选择AWS S3作为存储系统，您可以选择HMS或AWS Glue作为元数据服务。如果选择其他存储系统，则只能选择HMS作为元数据服务。

## 使用说明

- StarRocks查询Hudi数据时，支持Parquet文件格式。Parquet文件支持SNAPPY、LZ4、ZSTD、GZIP和NO_COMPRESSION压缩格式。
- StarRocks完整支持了Hudi的Copy On Write（COW）表和Merge On Read（MOR）表。

## 准备工作

在创建Hudi目录之前，请确保StarRocks集群能够正常访问Hudi的文件存储及元数据服务。

### AWS IAM

如果Hudi使用AWS S3作为文件存储或使用AWS Glue作为元数据服务，您需要选择一种合适的认证鉴权方案，确保StarRocks集群可以访问相关的AWS云资源。

您可以选择如下认证鉴权方案：

- 实例配置文件（推荐）
- 假定角色
- IAM用户

有关StarRocks访问AWS认证鉴权的详细内容，参见[配置AWS认证方式 - 准备工作](../../integrations/authenticate_to_aws_resources.md#准备工作)。

### HDFS

如果使用HDFS作为文件存储，则需要在StarRocks集群中做如下配置：

- （可选）设置用于访问HDFS集群和HMS的用户名。 您可以在每个FE的**fe/conf/hadoop_env.sh**文件、以及每个BE的**be/conf/hadoop_env.sh**文件最开头增加`export HADOOP_USER_NAME="<user_name>"`来设置该用户名。配置完成后，需重启各个FE和BE使配置生效。如果不设置该用户名，则默认使用FE和BE进程的用户名进行访问。每个StarRocks集群仅支持配置一个用户名。
- 查询Hudi数据时，StarRocks集群的FE和BE会通过HDFS客户端访问HDFS集群。一般情况下，StarRocks会按照默认配置来启动HDFS客户端，无需手动配置。但在以下场景中，需要进行手动配置：
  - 如果HDFS集群开启了高可用（High Availability，简称为“HA”）模式，则需要将HDFS集群中的**hdfs-site.xml**文件放到每个FE的**$FE_HOME/conf**路径下、以及每个BE的**$BE_HOME/conf**路径下。
  - 如果HDFS集群配置了ViewFs，则需要将HDFS集群中的**core-site.xml**文件放到每个FE的**$FE_HOME/conf**路径下、以及每个BE的**$BE_HOME/conf**路径下。

> **注意**
>
> 如果查询时因为域名无法识别(Unknown Host)而发生访问失败，您需要将HDFS集群中各节点的主机名及IP地址之间的映射关系配置到**/etc/hosts**路径中。

### Kerberos认证

如果HDFS集群或HMS开启了Kerberos认证，则需要在StarRocks集群中做如下配置：

- 在每个FE和每个BE上执行`kinit -kt keytab_path principal`命令，从Key Distribution Center (KDC)获取到Ticket Granting Ticket (TGT)。执行命令的用户必须拥有访问HMS和HDFS的权限。注意，使用该命令访问KDC具有时效性，因此需要使用cron定期执行该命令。
- 在每个FE的**$FE_HOME/conf/fe.conf**文件和每个BE的**$BE_HOME/conf/be.conf**文件中添加`JAVA_OPTS="-Djava.security.krb5.conf=/etc/krb5.conf"`。其中，`/etc/krb5.conf`是**krb5.conf**文件的路径，可以根据文件的实际路径进行修改。

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

### 参数说明

#### catalog_name

Hudi目录的名称。命名要求如下：

- 必须由字母(a-z或A-Z)、数字(0-9)或下划线(_)组成，且只能以字母开头。
- 总长度不能超过1023个字符。
- Catalog名称大小写敏感。

#### comment

Hudi目录的描述。此参数为可选。

#### type

数据源的类型。设置为`hudi`。

#### MetastoreParams

StarRocks访问Hudi集群元数据服务的相关参数配置。

##### HMS

如果选择HMS作为Hudi集群的元数据服务，请按如下配置`MetastoreParams`：

```SQL
"hive.metastore.type" = "hive",
"hive.metastore.uris" = "<hive_metastore_uri>"
```

> **说明**
>
> 在查询Hudi数据之前，必须将所有HMS节点的主机名及IP地址之间的映射关系添加到**/etc/hosts**路径。否则，发起查询时，StarRocks可能无法访问HMS。

`MetastoreParams`包含如下参数。

| 参数                | 是否必须 | 说明                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type | 是       | Hudi集群所使用的元数据服务的类型。设置为`hive`。            |
| hive.metastore.uris | 是       | HMS的URI。格式：`thrift://<HMS IP地址>:<HMS端口号>`。<br />如果您的HMS开启了高可用模式，此处可以填写多个HMS地址并用逗号分隔，例如：`"thrift://<HMS IP地址1>:<HMS端口号1>,thrift://<HMS IP地址2>:<HMS端口号2>,thrift://<HMS IP地址3>:<HMS端口号3>"`。 |

##### AWS Glue

如果选择AWS Glue作为Hudi集群的元数据服务（只有使用AWS S3作为存储系统时支持），请按如下配置`MetastoreParams`：

- 基于实例配置文件进行认证和鉴权

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- 基于 Assumed Role 进行认证和授权

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "true",
"aws.glue.iam_role_arn" = "<iam_role_arn>",
"aws.glue.region" = "<aws_glue_region>"
```

- 基于 IAM User 进行认证和授权

```SQL
"hive.metastore.type" = "glue",
"aws.glue.use_instance_profile" = "false",
"aws.glue.access_key" = "<iam_user_access_key>",
"aws.glue.secret_key" = "<iam_user_secret_key>",
"aws.glue.region" = "<aws_s3_region>"
```

`MetastoreParams` 包含以下参数。

| 参数                        | 是否必需 | 说明                                                     |
| --------------------------- | -------- | -------------------------------------------------------- |
| hive.metastore.type         | 是       | Hudi 集羘所使用的元数据服务的类型。设置为 `glue`。       |
| aws.glue.use_instance_profile | 是       | 指定是否开启 Instance Profile 和 Assumed Role 两种授权方式。取值范围：`true` 和 `false`。默认值：`false`。 |
| aws.glue.iam_role_arn       | 否       | 有权限访问 AWS Glue Data Catalog 的 IAM Role 的 ARN。采用 Assumed Role 授权方式访问 AWS Glue 时，必须指定此参数。 |
| aws.glue.region             | 是       | AWS Glue Data Catalog 所在的地域。示例：`us-west-1`。      |
| aws.glue.access_key         | 否       | IAM User 的 Access Key。采用 IAM User 授权方式访问 AWS Glue 时，必须指定此参数。 |
| aws.glue.secret_key         | 否       | IAM User 的 Secret Key。采用 IAM User 授权方式访问 AWS Glue 时，必须指定此参数。 |

有关如何选择用于访问 AWS Glue 的授权方式、以及如何在 AWS IAM 控制台配置访问控制策略，请参见[访问 AWS Glue 的认证参数](../../integrations/authenticate_to_aws_resources.md#访问-aws-glue-的认证参数)。

#### StorageCredentialParams

StarRocks 访问 Hudi 集群文件存储的相关参数配置。

如果您使用 HDFS 作为存储系统，则不需要配置 `StorageCredentialParams`。

如果您使用 AWS S3、其他兼容 S3 协议的对象存储、Microsoft Azure Storage，或 GCS，则必须配置 `StorageCredentialParams`。

##### AWS S3

如果选择 AWS S3 作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 Instance Profile 进行认证和授权

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.region" = "<aws_s3_region>"
```

- 基于 Assumed Role 进行认证和授权

```SQL
"aws.s3.use_instance_profile" = "true",
"aws.s3.iam_role_arn" = "<iam_role_arn>",
"aws.s3.region" = "<aws_s3_region>"
```

- 基于 IAM User 进行认证和授权

```SQL
"aws.s3.use_instance_profile" = "false",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>",
"aws.s3.region" = "<aws_s3_region>"
```

`StorageCredentialParams` 包含以下参数。

| 参数                        | 是否必需 | 说明                                                     |
| --------------------------- | -------- | -------------------------------------------------------- |
| aws.s3.use_instance_profile | 是       | 指定是否开启 Instance Profile 和 Assumed Role 两种授权方式。取值范围：`true` 和 `false`。默认值：`false`。 |
| aws.s3.iam_role_arn         | 否       | 有权限访问 AWS S3 Bucket 的 IAM Role 的 ARN。采用 Assumed Role 授权方式访问 AWS S3 时，必须指定此参数。 |
| aws.s3.region               | 是       | AWS S3 Bucket 所在的地域。示例：`us-west-1`。             |
| aws.s3.access_key           | 否       | IAM User 的 Access Key。采用 IAM User 授权方式访问 AWS S3 时，必须指定此参数。 |
| aws.s3.secret_key           | 否       | IAM User 的 Secret Key。采用 IAM User 授权方式访问 AWS S3 时，必须指定此参数。 |

有关如何选择用于访问 AWS S3 的授权方式、以及如何在 AWS IAM 控制台配置访问控制策略，请参见[访问 AWS S3 的认证参数](../../integrations/authenticate_to_aws_resources.md#访问-aws-s3-的认证参数)。

##### 阿里云 OSS

如果选择阿里云 OSS 作为 Hudi 集群的文件存储，需要在 `StorageCredentialParams` 中配置如下授权参数：

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| 参数                        | 是否必需 | 说明                                                     |
| --------------------------- | -------- | -------------------------------------------------------- |
| aliyun.oss.endpoint         | 是       | 阿里云 OSS Endpoint，例如 `oss-cn-beijing.aliyuncs.com`。您可根据 Endpoint 和地域的对应关系进行查找，请参见 [访问域名和数据中心](https://help.aliyun.com/document_detail/31837.html)。 |
| aliyun.oss.access_key       | 是       | 指定阿里云账号或 RAM 用户的 AccessKey ID。获取方式，请参见 [获取 AccessKey](https://help.aliyun.com/document_detail/53045.html)。 |
| aliyun.oss.secret_key       | 是       | 指定阿里云账号或 RAM 用户的 AccessKey Secret。获取方式，请参见 [获取 AccessKey](https://help.aliyun.com/document_detail/53045.html)。|

##### 兼容 S3 协议的对象存储

Hudi Catalog 从 2.5 版本起支持兼容 S3 协议的对象存储。

如果选择兼容 S3 协议的对象存储（如 MinIO）作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` 包含以下参数。

| 参数                           | 是否必需 | 说明                                                     |
| ------------------------------ | -------- | -------------------------------------------------------- |
| aws.s3.enable_ssl              |是       | 是否开启 SSL 连接。<br />取值范围：`true` 和 `false`。默认值：`true`。 |
| aws.s3.enable_path_style_access | 是       | 是否开启路径类型访问 (Path-Style Access)。<br />取值范围：`true` 和 `false`。默认值：`false`。对于 MinIO，必须设置为 `true`。<br />路径类型 URL 使用如下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域中创建一个名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并希望访问该存储桶中的 `alice.jpg` 对象，则可使用以下路径类型 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。|
| aws.s3.endpoint                  | Yes      | 用于访问兼容 S3 协议的对象存储的 Endpoint。 |
| aws.s3.access_key                | Yes      | IAM User 的 Access Key。 |
| aws.s3.secret_key                | Yes      | IAM User 的 Secret Key。 |

##### Microsoft Azure Storage

Hudi Catalog 从 3.0 版本起支持 Microsoft Azure Storage。

###### Azure Blob Storage

如果选择 Blob Storage 作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 Shared Key 进行认证和鉴权

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                   | **是否必须** | **说明**                         |
  | -------------------------- | ------------ | -------------------------------- |
  | azure.blob.storage_account | 是           | Blob Storage 账号的用户名。      |
  | azure.blob.shared_key      | 是           | Blob Storage 账号的 Shared Key。 |

- 基于 SAS Token 进行认证和鉴权

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                  | **是否必须** | **说明**                                 |
  | ------------------------- | ------------ | ---------------------------------------- |
  | azure.blob.account_name   | 是           | Blob Storage 账号的用户名。              |
  | azure.blob.container_name | 是           | 数据所在 Blob 容器的名称。               |
  | azure.blob.sas_token      | 是           | 用于访问 Blob Storage 账号的 SAS Token。 |

###### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 Managed Service Identity 进行认证和鉴权

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                 | **是否必须** | **说明**                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是           | 指定是否开启 Managed Service Identity 鉴权方式。设置为 `true`。 |

- 基于 Service Principal 进行认证和鉴权

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **Parameter**                 | **Required** | **Description**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是           | Service Principal 的 Client (Application) ID。               |
  | azure.adls1.oauth2_credential | 是           | 新建的 Client (Application) Secret。                         |
  | azure.adls1.oauth2_endpoint   | 是           | Service Principal 或 Application 的 OAuth 2.0 Token Endpoint (v1)。 |

###### Azure Data Lake Storage Gen2

如果选择 Data Lake Storage Gen2 作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 Managed Identity 进行认证和鉴权

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                | **是否必须** | **说明**                                                |
  | --------------------------------------- | ------------ | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | 是           | 指定是否开启 Managed Identity 鉴权方式。设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是           | 数据所属 Tenant 的 ID。                                 |
  | azure.adls2.oauth2_client_id            | 是           | Managed Identity 的 Client (Application) ID。           |

- 基于 Shared Key 进行认证和鉴权

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                    | **是否必须** | **说明**                                   |
  | --------------------------- | ------------ | ------------------------------------------ |
  | azure.adls2.storage_account | 是           | Data Lake Storage Gen2 账号的用户名。      |
  | azure.adls2.shared_key      | 是           | Data Lake Storage Gen2 账号的 Shared Key。 |

- 基于 Service Principal 进行认证和鉴权

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                           | **是否必须** | **说明**                                                     |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是           | Service Principal 的 Client (Application) ID。               |
  | azure.adls2.oauth2_client_secret   | 是           | 新建的 Client (Application) Secret。                         |
  | azure.adls2.oauth2_client_endpoint | 是           | Service Principal 或 Application 的 OAuth 2.0 Token Endpoint (v1)。 |

##### Google GCS

Hudi Catalog 从 3.0 版本起支持 Google GCS。

如果选择 Google GCS 作为 Hudi 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 VM 进行认证和鉴权

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                   | **默认值** | **取值样例** | **说明**                                                 |
  | ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true         | 是否直接使用 Compute Engine 上面绑定的 Service Account。 |

- 基于 Service Account 进行认证和鉴权

  ```SQL
  "gcp.gcs.service_account_email" = "<google_service_account_email>",
  "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
  "gcp.gcs.service_account_private_key" = "<google_service_private_key>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                               | **默认值** | **取值样例**                                                 | **说明**                                                     |
  | -------------------------------------- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | gcp.gcs.service_account_email          | ""         | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | 创建 Service Account 时生成的 JSON 文件中的 Email。          |
  | gcp.gcs.service_account_private_key_id | ""         | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"                   | 创建 Service Account 时生成的 JSON 文件中的 Private Key ID。 |
  | gcp.gcs.service_account_private_key    | ""         | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n"  | 创建 Service Account 时生成的 JSON 文件中的 Private Key。    |

- 基于 Impersonation 进行认证和鉴权

  - 使用 VM 实例模拟 Service Account

    ```SQL
    "gcp.gcs.use_compute_engine_service_account" = "true",
    "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"
    ```

    `StorageCredentialParams` 包含如下参数。

    | **参数**                                   | **默认值** | **取值样例** | **说明**                                                     |
    | ------------------------------------------ | ---------- | ------------ | ------------------------------------------------------------ |
    | gcp.gcs.use_compute_engine_service_account | false      | true         | Whether to use the Service Account bound on Compute Engine directly. |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"      | The target Service Account that needs to be impersonated. |

  - Use one Service Account (temporarily named "Meta Service Account") to impersonate another Service Account (temporarily named "Data Service Account")

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` contains the following parameters.

    | **Parameter**                           | **Default Value** | **Example Value**                                           | **Description**                                              |
    | ---------------------------------------- | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
    | gcp.gcs.service_account_email            | ""                 | "[user@hello.iam.gserviceaccount.com](mailto:user@hello.iam.gserviceaccount.com)" | The Email generated in the JSON file when creating the Meta Service Account. |
    | gcp.gcs.service_account_private_key_id   | ""                 | "61d257bd8479547cb3e04f0b9b6b9ca07af3b7ea"               | The Private Key ID in the JSON file generated when creating the Meta Service Account. |
    | gcp.gcs.service_account_private_key      | ""                 | "-----BEGIN PRIVATE KEY----xxxx-----END PRIVATE KEY-----\n" | The Private Key in the JSON file generated when creating the Meta Service Account. |
    | gcp.gcs.impersonation_service_account    | ""                 | "hello"                                                    | The target Data Service Account to impersonate. |

#### MetadataUpdateParams

A set of parameters specifying the cache metadata update policy. StarRocks updates the cached Hudi metadata based on this policy. This set of parameters is optional.

By default, StarRocks uses the [automatic asynchronous update policy](#appendix-understand-metadata-automatic-asynchronous-update-policy) out of the box. Therefore, in general, you can ignore `MetadataUpdateParams` and do not need to tune the policy parameters.

If the Hudi data update frequency is high, you can optimize the performance of the automatic asynchronous update policy by tuning these parameters.

| Parameter                           | Required | Description                                                  |
| ----------------------------------- | -------- | ------------------------------------------------------------ |
| enable_metastore_cache            | No       | Specifies whether StarRocks caches the metadata of Hudi tables. Possible values: `true` and `false`. Default value: `true`. Value `true` indicates enabling the cache, and `false` indicates disabling the cache. |
| enable_remote_file_cache           | No       | Specifies whether StarRocks caches the metadata of Hudi table or partition data files. Possible values: `true` and `false`. Default value: `true`. Value `true` indicates enabling the cache, and `false` indicates disabling the cache. |
| metastore_cache_refresh_interval_sec   | No       | The time interval at which StarRocks asynchronously updates the metadata of Hudi tables or partitions. Unit: seconds. Default value: `7200`, which is 2 hours. |
| remote_file_cache_refresh_interval_sec | No       | The time interval at which StarRocks asynchronously updates the metadata of Hudi table or partition data files. Unit: seconds. Default value: `60`. |
| metastore_cache_ttl_sec                | No       | The time interval at which StarRocks automatically evicts the metadata cache of Hudi tables or partitions. Unit: seconds. Default value: `86400`, which is 24 hours. |
| remote_file_cache_ttl_sec              | No       | The time interval at which StarRocks automatically evicts the data file metadata cache of Hudi tables or partitions. Unit: seconds. Default value: `129600`, which is 36 hours. |

### Examples

The following examples create a Hudi Catalog named `hudi_catalog_hms` or `hudi_catalog_glue` for querying data in the Hudi cluster.

#### HDFS

When using HDFS as storage, you can create a Hudi Catalog as follows:

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://xx.xx.xx:9083"
);
```

#### AWS S3

##### If authentication and authorization are based on Instance Profile

- If the Hudi cluster uses HMS as the metadata service, you can create a Hudi Catalog as follows:

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

- If the Amazon EMR Hudi cluster uses AWS Glue as the metadata service, you can create a Hudi Catalog as follows:

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

##### If authentication and authorization are based on Assumed Role

- If the Hudi cluster uses HMS as the metadata service, you can create a Hudi Catalog as follows:

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

- If the Amazon EMR Hudi cluster uses AWS Glue as the metadata service, you can create a Hudi Catalog as follows:

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

##### If authentication and authorization are based on IAM User

- If the Hudi cluster uses HMS as the metadata service, you can create a Hudi Catalog as follows:

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

- If the Amazon EMR Hudi cluster uses AWS Glue as the metadata service, the Hudi Catalog can be created as follows:

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

#### Compatible S3 Protocol Object Storage

Using MinIO as an example, the Hudi Catalog can be created as follows:

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

#### Microsoft Azure Storage

##### Azure Blob Storage

- If authentication and authorization are based on Shared Key, the Hudi Catalog can be created as follows:

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

- If authentication and authorization are based on SAS Token, the Hudi Catalog can be created as follows:

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

##### Azure Data Lake Storage Gen1

- If authentication and authorization are based on Managed Service Identity, the Hudi Catalog can be created as follows:

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

- If authentication and authorization are based on Service Principal, the Hudi Catalog can be created as follows:

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

##### Azure Data Lake Storage Gen2

- If authentication and authorization are based on Managed Identity, the Hudi Catalog can be created as follows:

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

- If authentication and authorization are based on Shared Key, the Hudi Catalog can be created as follows:

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

- If authentication and authorization are based on Service Principal, the Hudi Catalog can be created as follows:

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "azure.adls2.oauth2_client_id" = "<service_client_id>",
    "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
    "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>" 
);
```

#### Google GCS

- If authentication and authorization are based on VM, the Hudi Catalog can be created as follows:

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

- If authentication and authorization are based on Service Account, the Hudi Catalog can be created as follows:

```SQL
CREATE EXTERNAL CATALOG hudi_catalog_hms
PROPERTIES
(
    "type" = "hudi",
    "hive.metastore.type" = "hive",
    "hive.metastore.uris" = "thrift://34.132.15.127:9083",
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
    "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
);
```

- If authentication and authorization are based on Impersonation

  - Simulating Service Account using VM instance, the Hudi Catalog can be created as follows:

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

  - Simulating one Service Account using another Service Account, the Hudi Catalog can be created as follows:

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

## View Hudi Catalog

You can use [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) to query all the catalogs in the current StarRocks cluster:

```SQL
SHOW CATALOGS;
```

您也可以通过[SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md)查询某个External Catalog的创建语句。例如，通过如下命令查询Hudi Catalog`hudi_catalog_glue`的创建语句：

```SQL
SHOW CREATE CATALOG hudi_catalog_glue;
```

## 切换Hudi Catalog和数据库

您可以通过如下方法切换至目标Hudi Catalog和数据库：

- 先通过[SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md)指定当前会话生效的Hudi Catalog，然后再通过[USE](../../sql-reference/sql-statements/data-definition/USE.md)指定数据库：

```SQL
--切换当前会话生效的Catalog：
SET CATALOG <catalog_name>
--指定当前会话生效的数据库：
USE <db_name>
```

- 通过[USE](../../sql-reference/sql-statements/data-definition/USE.md)直接将会话切换到目标Hudi Catalog下的指定数据库：

```SQL
USE <catalog_name>.<db_name>
```

## 删除Hudi Catalog

您可以通过[DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md)删除某个External Catalog。

例如，通过如下命令删除Hudi Catalog `hudi_catalog_glue`：

```SQL
DROP Catalog hudi_catalog_glue;
```

## 查看Hudi表结构

您可以通过如下方法查看Hudi表的表结构：

- 查看表结构

```SQL
DESC[RIBE] <catalog_name>.<database_name>.<table_name>
```

- 从CREATE命令查看表结构和表文件存放位置

```SQL
SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
```

## 查询Hudi表数据

1. 通过[SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md)查看指定Catalog所属的Hudi集群中的数据库：

```SQL
SHOW DATABASES FROM <catalog_name>
```

2. [切换至目标Hudi Catalog和数据库](#切换-hudi-catalog-和数据库)。

3. 通过[SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md)查询目标数据库中的目标表：

```SQL
SELECT count(*) FROM <table_name> LIMIT 10
```

## 导入Hudi数据

假设有一个OLAP表，表名为`olap_tbl`。您可以这样来转换该表中的数据，并把数据导入到StarRocks中：

```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hudi_table
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks会缓存Hudi的元数据、并以异步模式自动更新缓存的元数据，从而提高查询性能。此外，在对Hudi表做了表结构变更或其他表更新后，您也可以使用[REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md)手动更新该表的元数据，从而确保StarRocks第一时间生成合理的查询计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

### 自动增量更新

与自动异步更新策略不同，在自动增量更新策略下，FE可以定时从HMS读取各种事件，进而感知Hudi表元数据的变更情况，如增减列、增减分区和更新分区数据等，无需手动更新Hudi表的元数据。

开启自动增量更新策略的步骤如下：

#### 步骤 1：在HMS上配置事件侦听器

HMS 2.x和3.x版本均支持配置事件侦听器。这里以配套HMS 3.1.2版本的事件侦听器配置为例。将以下配置项添加到**$HiveMetastore/conf/hive-site.xml**文件中，然后重启HMS：

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

配置完成后，可以在FE日志文件中搜索`event id`，然后通过查看事件ID来检查事件监听器是否配置成功。如果配置失败，则所有`event id`均为`0`。

#### 步骤 2：在StarRocks上开启自动增量更新策略

您可以给StarRocks集群中某一个Hudi Catalog开启自动增量更新策略，也可以给StarRocks集群中所有Hudi Catalog开启自动增量更新策略。

- 如果要给单个Hudi Catalog开启自动增量更新策略，则需要在创建该Hudi Catalog时把`PROPERTIES`中的`enable_hms_events_incremental_sync`参数设置为`true`，如下所示：

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

- 如果要给所有Hudi Catalog开启自动增量更新策略，则需要把`enable_hms_events_incremental_sync`参数添加到每个FE的**$FE_HOME/conf/fe.conf**文件中，并设置为`true`，然后重启FE，使参数配置生效。

您还可以根据业务需求在每个FE的**$FE_HOME/conf/fe.conf**文件中对以下参数进行调优，然后重启FE，使参数配置生效。

| Parameter                         | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks从HMS中读取事件的时间间隔。默认值：`5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks每次读取事件的最大数量。默认值：`500`。            |
| enable_hms_parallel_process_evens | 指定StarRocks在读取事件时是否并行处理读取的事件。取值范围：`true`和`false`。默认值：`true`。取值为`true`则开启并行机制，取值为`false`则关闭并行机制。 |
| hms_process_events_parallel_num   | StarRocks每次处理事件的最大并发数。默认值：`4`。            |

## 附录：理解元数据自动异步更新策略

自动异步更新策略是StarRocks用于更新Hudi Catalog中元数据的默认策略。

默认情况下（即当`enable_metastore_cache`参数和`enable_remote_file_cache`参数均设置为`true`时），如果一个查询命中Hudi表的某个分区，则StarRocks会自动缓存该分区的元数据、以及该分区下数据文件的元数据。缓存的元数据采用懒更新(Lazy Update)策略。

例如，有一张名为 `table2` 的 Hudi 表，该表的数据分布在四个分区：`p1`、`p2`、`p3` 和 `p4`。当一个查询命中 `p1` 时，StarRocks 会自动缓存 `p1` 的元数据、以及 `p1` 下数据文件的元数据。假设当前缓存元数据的更新和淘汰策略设置如下：

- 异步更新 `p1` 的缓存元数据的时间间隔（通过 `metastore_cache_refresh_interval_sec` 参数指定）为 2 小时。
- 异步更新 `p1` 下数据文件的缓存元数据的时间间隔（通过 `remote_file_cache_refresh_interval_sec` 参数指定）为 60 秒。
- 自动淘汰 `p1` 的缓存元数据的时间间隔（通过 `metastore_cache_ttl_sec` 参数指定）为 24 小时。
- 自动淘汰 `p1` 下数据文件的缓存元数据的时间间隔（通过 `remote_file_cache_ttl_sec` 参数指定）为 36 小时。

如下图所示。

![时间轴上的更新策略](../../assets/catalog_timeline_zh.png)

StarRocks 采用如下策略更新和淘汰缓存的元数据：

- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔不超过 60 秒，则 StarRocks 既不会更新 `p1` 的缓存元数据，也不会更新 `p1` 下数据文件的缓存元数据。
- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔超过 60 秒，则 StarRocks 会更新 `p1` 下数据文件的缓存元数据。
- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔超过 2 小时，则 StarRocks 会更新 `p1` 的缓存元数据。
- 如果继上次更新结束后，`p1` 在 24 小时内未被访问，则 StarRocks 会淘汰 `p1` 的缓存元数据。后续有查询再次命中 `p1` 时，会重新缓存 `p1` 的元数据。
- 如果继上次更新结束后，`p1` 在 36 小时内未被访问，则 StarRocks 会淘汰 `p1` 下数据文件的缓存元数据。后续有查询再次命中 `p1` 时，会重新缓存 `p1` 下数据文件的元数据。