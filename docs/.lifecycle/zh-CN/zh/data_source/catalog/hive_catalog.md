```yaml
      + {T}
      + {T}
    + {T}
  + {T}
```

```yaml
      + {T}
      + {T}
    + {T}
  + {T}
```
> Before querying Hive data, the mapping relationship between the hostnames and IP addresses of all HMS nodes must be added to the **/etc/hosts** path. Otherwise, StarRocks may not be able to access HMS when initiating a query.

`MetastoreParams` contains the following parameters.

| Parameter             | Mandatory | Explanation                                                  |
| -------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type  | Yes      | The type of metadata service used by the Hive cluster. Set to `hive`. |
| hive.metastore.uris  | Yes      | The URI of HMS. Format: `thrift://<HMS IP address>:<HMS port>`.<br />If your HMS is in high availability mode, you can specify multiple HMS addresses separated by commas, for example: `"thrift://<HMS IP address 1>:<HMS port 1>,thrift://<HMS IP address 2>:<HMS port 2>,thrift://<HMS IP address 3>:<HMS port 3>"`. |

##### AWS Glue

If you choose AWS Glue as the metadata service for the Hive cluster (supported only when using AWS S3 as the storage system), please configure `MetastoreParams` as follows:

- Authentication and authorization based on Instance Profile

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Authentication and authorization based on Assumed Role

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "true",
  "aws.glue.iam_role_arn" = "<iam_role_arn>",
  "aws.glue.region" = "<aws_glue_region>"
  ```

- Authentication and authorization based on IAM User

  ```SQL
  "hive.metastore.type" = "glue",
  "aws.glue.use_instance_profile" = "false",
  "aws.glue.access_key" = "<iam_user_access_key>",
  "aws.glue.secret_key" = "<iam_user_secret_key>",
  "aws.glue.region" = "<aws_s3_region>"
  ```

`MetastoreParams` contains the following parameters.

| Parameter                      | Mandatory | Explanation                                                  |
| ----------------------------- | -------- | ------------------------------------------------------------ |
| hive.metastore.type           | Yes      | The type of metadata service used by the Hive cluster. Set to `glue`. |
| aws.glue.use_instance_profile | Yes      | Specifies whether to enable Instance Profile and Assumed Role authentication. Possible values are: `true` and `false`. Default value: `false`. |
| aws.glue.iam_role_arn         | No       | The ARN of the IAM Role with permissions to access AWS Glue Data Catalog. This parameter must be specified when accessing AWS Glue using Assumed Role authentication. |
| aws.glue.region               | Yes      | The region where the AWS Glue Data Catalog is located. Example: `us-west-1`. |
| aws.glue.access_key           | No       | The Access Key of the IAM User. This parameter must be specified when accessing AWS Glue using IAM User authentication. |
| aws.glue.secret_key           | No       | The Secret Key of the IAM User. This parameter must be specified when accessing AWS Glue using IAM User authentication. |

For information on how to choose the authentication method for accessing AWS Glue and how to configure access control policies in the AWS IAM console, refer to [Authentication Parameters for Accessing AWS Glue](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-glue).

#### StorageCredentialParams

Parameters for StarRocks to access the file storage of the Hive cluster.

If you are using HDFS as the storage system, `StorageCredentialParams` does not need to be configured.

If you are using AWS S3, another S3-compatible object storage, Microsoft Azure Storage, or GCS, `StorageCredentialParams` must be configured.

##### AWS S3

If you choose AWS S3 as the file storage for the Hive cluster, please configure `StorageCredentialParams` as follows:

- Authentication and authorization based on Instance Profile

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Authentication and authorization based on Assumed Role

  ```SQL
  "aws.s3.use_instance_profile" = "true",
  "aws.s3.iam_role_arn" = "<iam_role_arn>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

- Authentication and authorization based on IAM User

  ```SQL
  "aws.s3.use_instance_profile" = "false",
  "aws.s3.access_key" = "<iam_user_access_key>",
  "aws.s3.secret_key" = "<iam_user_secret_key>",
  "aws.s3.region" = "<aws_s3_region>"
  ```

`StorageCredentialParams` contains the following parameters.

| Parameter                 | Mandatory | Explanation                                                  |
| ------------------------ | -------- | ------------------------------------------------------------ |
| aws.s3.use_instance_profile | Yes      | Specifies whether to enable Instance Profile and Assumed Role authentication. Possible values are: `true` and `false`. Default value: `false`. |
| aws.s3.iam_role_arn       | No       | The ARN of the IAM Role with permissions to access the AWS S3 Bucket. This parameter must be specified when accessing AWS S3 using Assumed Role authentication. |
| aws.s3.region             | Yes      | The region where the AWS S3 Bucket is located. Example: `us-west-1`. |
| aws.s3.access_key         | No       | The Access Key of the IAM User. This parameter must be specified when accessing AWS S3 using IAM User authentication. |
| aws.s3.secret_key         | No       | The Secret Key of the IAM User. This parameter must be specified when accessing AWS S3 using IAM User authentication. |

For information on how to choose the authentication method for accessing AWS S3 and how to configure access control policies in the AWS IAM console, refer to [Authentication Parameters for Accessing AWS S3](../../integrations/authenticate_to_aws_resources.md#authentication-parameters-for-accessing-aws-s3).

##### Alibaba Cloud OSS

If you choose Alibaba Cloud OSS as the file storage for the Hive cluster, the following authentication parameters need to be configured in `StorageCredentialParams`:

```SQL
"aliyun.oss.access_key" = "<user_access_key>",
"aliyun.oss.secret_key" = "<user_secret_key>",
"aliyun.oss.endpoint" = "<oss_endpoint>" 
```

| Parameter                | Mandatory | Explanation                                                  |
| ----------------------- | -------- | ------------------------------------------------------------ |
| aliyun.oss.endpoint     | Yes      | The endpoint of Alibaba Cloud OSS, such as `oss-cn-beijing.aliyuncs.com`. You can find the correspondence between the endpoint and region, please refer to [Domains and Data Centers](https://help.aliyun.com/document_detail/31837.html). |
| aliyun.oss.access_key   | Yes      | The AccessKey ID of the Alibaba Cloud account or RAM user. For how to obtain it, please refer to [Obtaining AccessKey](https://help.aliyun.com/document_detail/53045.html). |
| aliyun.oss.secret_key   | Yes      | The AccessKey Secret of the Alibaba Cloud account or RAM user. For how to obtain it, please refer to [Obtaining AccessKey](https://help.aliyun.com/document_detail/53045.html). |

##### S3-compatible Object Storage

Hive Catalog 从 2.5 版本起支持兼容 S3 协议的对象存储。

如果选择兼容 S3 协议的对象存储（如 MinIO）作为 Hive 集群的文件存储，请按如下配置 `StorageCredentialParams`：

```SQL
"aws.s3.enable_ssl" = "false",
"aws.s3.enable_path_style_access" = "true",
"aws.s3.endpoint" = "<s3_endpoint>",
"aws.s3.access_key" = "<iam_user_access_key>",
"aws.s3.secret_key" = "<iam_user_secret_key>"
```

`StorageCredentialParams` 包含如下参数。

| 参数                             | 是否必须   | 说明                                                  |
| -------------------------------- | -------- | ------------------------------------------------------------ |
| aws.s3.enable_ssl                | 是      | 是否开启 SSL 连接。<br />取值范围：`true` 和 `false`。默认值：`true`。 |
| aws.s3.enable_path_style_access  | 是      | 是否开启路径类型访问 (Path-Style Access)。<br />取值范围：`true` 和 `false`。默认值：`false`。对于 MinIO，必须设置为 `true`。<br />路径类型 URL 使用如下格式：`https://s3.<region_code>.amazonaws.com/<bucket_name>/<key_name>`。例如，如果您在美国西部（俄勒冈）区域中创建一个名为 `DOC-EXAMPLE-BUCKET1` 的存储桶，并希望访问该存储桶中的 `alice.jpg` 对象，则可使用以下路径类型 URL：`https://s3.us-west-2.amazonaws.com/DOC-EXAMPLE-BUCKET1/alice.jpg`。 |
| aws.s3.endpoint                  | 是      | 用于访问兼容 S3 协议的对象存储的 Endpoint。 |
| aws.s3.access_key                | 是      | IAM 用户的访问密钥。 |
| aws.s3.secret_key                | 是      | IAM 用户的秘钥。 |

##### Microsoft Azure 存储

Hive Catalog 从 3.0 版本起支持 Microsoft Azure 存储。

###### Azure Blob 存储

如果选择 Blob 存储作为 Hive 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 Shared Key 进行认证和授权

  ```SQL
  "azure.blob.storage_account" = "<blob_storage_account_name>",
  "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                   | **是否必须** | **说明**                         |
  | -------------------------- | ------------ | -------------------------------- |
  | azure.blob.storage_account | 是           | Blob 存储账户的名称。      |
  | azure.blob.shared_key      | 是           | Blob 存储账户的共享密钥。 |

- 基于 SAS Token 进行认证和授权

  ```SQL
  "azure.blob.account_name" = "<blob_storage_account_name>",
  "azure.blob.container_name" = "<blob_container_name>",
  "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                  | **是否必须** | **说明**                                 |
  | ------------------------- | ------------ | ---------------------------------------- |
  | azure.blob.account_name   | 是           | Blob 存储账户的名称。              |
  | azure.blob.container_name | 是           | 数据所在 Blob 容器的名称。               |
  | azure.blob.sas_token      | 是           | 用于访问 Blob 存储账户的 SAS 令牌。 |

###### Azure Data Lake Storage Gen1

如果选择 Data Lake Storage Gen1 作为 Hive 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于托管服务身份进行认证和授权

  ```SQL
  "azure.adls1.use_managed_service_identity" = "true"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                 | **是否必须** | **说明**                                                     |
  | ---------------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.use_managed_service_identity | 是           | 指定是否开启托管服务身份授权方式。设置为 `true`。 |

- 基于服务主体进行认证和授权

  ```SQL
  "azure.adls1.oauth2_client_id" = "<application_client_id>",
  "azure.adls1.oauth2_credential" = "<application_client_credential>",
  "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **Parameter**                 | **是否必须** | **说明**                                              |
  | ----------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls1.oauth2_client_id  | 是           | 服务主体的客户端（应用程序）ID。            |
  | azure.adls1.oauth2_credential | 是           | 新创建的客户端（应用程序）密钥。                          |
  | azure.adls1.oauth2_endpoint   | 是           | 服务主体或应用程序的 OAuth 2.0 令牌终结点（v1）。 |

###### Azure Data Lake Storage Gen2

如果选择 Data Lake Storage Gen2 作为 Hive 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于托管身份进行认证和授权

  ```SQL
  "azure.adls2.oauth2_use_managed_identity" = "true",
  "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
  "azure.adls2.oauth2_client_id" = "<service_client_id>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                | **是否必须** | **说明**                                                |
  | --------------------------------------- | ------------ | ------------------------------------------------------- |
  | azure.adls2.oauth2_use_managed_identity | 是           | 指定是否开启托管身份授权方式。设置为 `true`。 |
  | azure.adls2.oauth2_tenant_id            | 是           | 数据所属租户的ID。                                 |
  | azure.adls2.oauth2_client_id            | 是           | 托管身份的客户端（应用程序）ID。           |

- 基于 Shared Key 进行认证和授权

  ```SQL
  "azure.adls2.storage_account" = "<storage_account_name>",
  "azure.adls2.shared_key" = "<shared_key>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                    | **是否必须** | **说明**                                   |
  | --------------------------- | ------------ | ------------------------------------------ |
  | azure.adls2.storage_account | 是           | Data Lake Storage Gen2 账户的名称。      |
  | azure.adls2.shared_key      | 是           | Data Lake Storage Gen2 账户的共享密钥。 |

- 基于服务主体进行认证和授权

  ```SQL
  "azure.adls2.oauth2_client_id" = "<service_client_id>",
  "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
  "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                           | **是否必须** | **说明**                                                     |
  | ---------------------------------- | ------------ | ------------------------------------------------------------ |
  | azure.adls2.oauth2_client_id       | 是           | 服务主体的客户端（应用程序）ID。            |
  | azure.adls2.oauth2_client_secret   | 是           | 新创建的客户端（应用程序）密钥。                          |
  | azure.adls2.oauth2_client_endpoint | 是           | 服务主体或应用程序的 OAuth 2.0 令牌终结点（v1）。 |

##### Google GCS

Hive Catalog 从 3.0 版本起支持 Google GCS。

如果选择 Google GCS 作为 Hive 集群的文件存储，请按如下配置 `StorageCredentialParams`：

- 基于 VM 进行认证和授权

  ```SQL
  "gcp.gcs.use_compute_engine_service_account" = "true"
  ```

  `StorageCredentialParams` 包含如下参数。

  | **参数**                                   | **默认值** | **取值样例** | **说明**                                                 |
  | ------------------------------------------ | ---------- | ------------ | -------------------------------------------------------- |
  | gcp.gcs.use_compute_engine_service_account | false      | true         | 是否直接使用 Compute Engine 上绑定的服务账户。 |

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
    | gcp.gcs.use_compute_engine_service_account | false      | true         | 是否直接使用 Compute Engine 上面绑定的 Service Account。     |
    | gcp.gcs.impersonation_service_account      | ""         | "hello"      | 需要模拟的目标 Service Account。 |

  - 使用一个 Service Account（暂时命名为“Meta Service Account”）模拟另一个 Service Account（暂时命名为“Data Service Account”）

    ```SQL
    "gcp.gcs.service_account_email" = "<google_service_account_email>",
    "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",
    "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",
    "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"
    ```

    `StorageCredentialParams` 包含如下参数。
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

- If the Amazon EMR Hive cluster uses AWS Glue as the metadata service, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "glue",
      "aws.glue.use_instance_profile" = "true",
      "aws.glue.iam_role_arn" = "arn:aws:iam::081976408565:role/test_glue_role",
      "aws.glue.region" = "us-west-2",
      "aws.s3.use_instance_profile" = "true",
      "aws.s3.iam_role_arn" = "arn:aws:iam::081976408565:role/test_s3_role",
      "aws.s3.region" = "us-west-2"
  );
  ```

##### If using IAM User for authentication and authorization

- If the Hive cluster uses HMS as the metadata service, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://xx.xx.xx:9083",
      "aws.s3.use_instance_profile" = "false",
      "aws.s3.access_key" = "<iam_user_access_key>",
      "aws.s3.secret_key" = "<iam_user_access_key>",
      "aws.s3.region" = "us-west-2"
  );
  ```

- If the Amazon EMR Hive cluster uses AWS Glue as the metadata service, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_glue
  PROPERTIES
  (
      "type" = "hive",
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

#### Compatible object storage that supports S3 protocol

Using MinIO as an example, you can create a Hive Catalog as follows:

```SQL
CREATE EXTERNAL CATALOG hive_catalog_hms
PROPERTIES
(
    "type" = "hive",
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

- If using Shared Key for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.storage_account" = "<blob_storage_account_name>",
      "azure.blob.shared_key" = "<blob_storage_account_shared_key>"
  );
  ```

- If using SAS Token for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.blob.account_name" = "<blob_storage_account_name>",
      "azure.blob.container_name" = "<blob_container_name>",
      "azure.blob.sas_token" = "<blob_storage_account_SAS_token>"
  );
  ```

##### Azure Data Lake Storage Gen1

- If using Managed Service Identity for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.use_managed_service_identity" = "true"    
  );
  ```

- If using Service Principal for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls1.oauth2_client_id" = "<application_client_id>",
      "azure.adls1.oauth2_credential" = "<application_client_credential>",
      "azure.adls1.oauth2_endpoint" = "<OAuth_2.0_authorization_endpoint_v2>"
  );
  ```

##### Azure Data Lake Storage Gen2

- If using Managed Identity for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_use_managed_identity" = "true",
      "azure.adls2.oauth2_tenant_id" = "<service_principal_tenant_id>",
      "azure.adls2.oauth2_client_id" = "<service_client_id>"
  );
  ```

- If using Shared Key for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.storage_account" = "<storage_account_name>",
      "azure.adls2.shared_key" = "<shared_key>"     
  );
  ```

- If using Service Principal for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "azure.adls2.oauth2_client_id" = "<service_client_id>",
      "azure.adls2.oauth2_client_secret" = "<service_principal_client_secret>",
      "azure.adls2.oauth2_client_endpoint" = "<service_principal_client_endpoint>" 
  );
  ```

#### Google GCS

- If using VM for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.use_compute_engine_service_account" = "true"    
  );
  ```

- If using Service Account for authentication and authorization, you can create a Hive Catalog as follows:

  ```SQL
  CREATE EXTERNAL CATALOG hive_catalog_hms
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.type" = "hive",
      "hive.metastore.uris" = "thrift://34.132.15.127:9083",
      "gcp.gcs.service_account_email" = "<google_service_account_email>",
      "gcp.gcs.service_account_private_key_id" = "<google_service_private_key_id>",
```plaintext
      "gcp.gcs.service_account_private_key" = "<google_service_private_key>"    
  );

  ```

- If impersonation-based authentication and authorization are employed

  - To simulate a Service Account using a VM instance, you can create a Hive Catalog as follows:

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.use_compute_engine_service_account" = "true",
        "gcp.gcs.impersonation_service_account" = "<assumed_google_service_account_email>"    

    );

    ```

  - To simulate one Service Account with another Service Account, you can create a Hive Catalog as follows:

    ```SQL
    CREATE EXTERNAL CATALOG hive_catalog_hms
    PROPERTIES
    (
        "type" = "hive",
        "hive.metastore.type" = "hive",
        "hive.metastore.uris" = "thrift://34.132.15.127:9083",
        "gcp.gcs.service_account_email" = "<google_service_account_email>",
        "gcp.gcs.service_account_private_key_id" = "<meta_google_service_account_email>",

        "gcp.gcs.service_account_private_key" = "<meta_google_service_account_email>",

        "gcp.gcs.impersonation_service_account" = "<data_google_service_account_email>"    

    );
    ```


## Viewing Hive Catalogs

You can use [SHOW CATALOGS](../../sql-reference/sql-statements/data-manipulation/SHOW_CATALOGS.md) to query all the catalogs in the current StarRocks cluster:

```SQL

SHOW CATALOGS;

```

You can also use [SHOW CREATE CATALOG](../../sql-reference/sql-statements/data-manipulation/SHOW_CREATE_CATALOG.md) to query the creation statements of a specific External Catalog. For example, to query the creation statement of the Hive Catalog `hive_catalog_glue`, you can use the following command:

```SQL
SHOW CREATE CATALOG hive_catalog_glue;
```

## Switching Hive Catalogs and Databases


You can switch to the target Hive Catalog and database using the following methods:

- First use [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) to specify the Hive Catalog that will take effect in the current session, and then use [USE](../../sql-reference/sql-statements/data-definition/USE.md) to specify the database:

  ```SQL

  -- Switch the catalog that takes effect in the current session:

  SET CATALOG <catalog_name>

  -- Specify the database that takes effect in the current session:

  USE <db_name>
  ```


- Directly switch the session to the specified database under the target Hive Catalog through [USE](../../sql-reference/sql-statements/data-definition/USE.md):

  ```SQL

  USE <catalog_name>.<db_name>

  ```

## Dropping Hive Catalogs

You can use [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) to delete a specific External Catalog.

For example, to drop the Hive Catalog `hive_catalog_glue`, you can use the following command:

```SQL

DROP Catalog hive_catalog_glue;

```

## Viewing Hive Table Structure

You can view the table structure of a Hive table using the following methods:

- View the table structure

  ```SQL

  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```


- View the table structure and location of table files from the CREATE command

  ```SQL

  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```


## Querying Hive Table Data

1. Use [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) to view the databases in a specific Catalog of the Hive cluster:

   ```SQL

   SHOW DATABASES FROM <catalog_name>
   ```


2. [Switch to the target Hive Catalog and database](#switching-hive-catalogs-and-databases).

3. Use [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) to query the target table in the target database:

   ```SQL

   SELECT count(*) FROM <table_name> LIMIT 10

   ```

## Importing Hive Data

Assuming there is an OLAP table named `olap_tbl`, you can transform the data in the table and import it into StarRocks as follows:


```SQL
INSERT INTO default_catalog.olap_db.olap_tbl SELECT * FROM hive_table

```

## Granting Permissions on Hive Tables and Views

You can use [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) to grant a role the permission to query all tables or views in a specific Hive Catalog.

- Grant the role permission to query all tables in a specific Hive Catalog:

  ```SQL
  GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE <role_name>
  ```

- Grant the role permission to query all views in a specific Hive Catalog:

  ```SQL
  GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE <role_name>
  ```


For example, you can create the role `hive_role_table`, switch to the Hive Catalog `hive_catalog`, and then grant `hive_role_table` the permission to query all tables and views in the `hive_catalog` as follows:

```SQL
-- Create the role hive_role_table.

CREATE ROLE hive_role_table;

-- Switch to the data directory hive_catalog.
SET CATALOG hive_catalog;

-- Grant the permission to query all tables in hive_catalog to hive_role_table.
GRANT SELECT ON ALL TABLES IN ALL DATABASES TO ROLE hive_role_table;

-- Grant the permission to query all views in hive_catalog to hive_role_table.
GRANT SELECT ON ALL VIEWS IN ALL DATABASES TO ROLE hive_role_table;
```

| AWS S3 and other S3-compatible storages (such as MinIO) | `s3`                                                         |

## Drop Hive Database

Consistent with the internal database of StarRocks, if you have the [DROP](../../administration/privilege_item.md#database-privilege-database) permission of the Hive database, you can use [DROP DATABASE](../../sql-reference/sql-statements/data-definition/DROP_DATABASE.md) to delete the Hive database. This feature has been supported since version 3.2. Only empty databases are supported for deletion.

> **Note**
>
> You can grant and revoke permissions to users and roles through [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) and [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) operations.

Deleting a database does not delete the corresponding file path on HDFS or object storage.

[Switch to the target Hive Catalog](#switch-hive-catalog-and-database), and then use the following statement to delete the Hive database:

```SQL
DROP DATABASE <database_name>
```

## Create Hive Table

Consistent with the internal database of StarRocks, if you have the [CREATE TABLE](../../administration/privilege_item.md#database-privilege-database) permission of the Hive database, you can use [CREATE TABLE](../../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) or [CREATE TABLE AS SELECT (CTAS)](../../sql-reference/sql-statements/data-definition/CREATE_TABLE_AS_SELECT.md) to create a Managed Table in the Hive database. This feature has been supported since version 3.2.

> **Note**
>
> You can grant and revoke permissions to users and roles through [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) and [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) operations.

[Switch to the target Hive Catalog and database](#switch-hive-catalog-and-database). Then use the following syntax to create a Managed Table in Hive:

### Syntax

```SQL
CREATE TABLE [IF NOT EXISTS] [database.]table_name
(column_definition1[, column_definition2, ...
partition_column_definition1,partition_column_definition2...])
[partition_desc]
[PROPERTIES ("key" = "value", ...)]
[AS SELECT query]
```

### Parameter Description

#### column_definition

The syntax of `column_definition` is as follows:

```SQL
col_name col_type [COMMENT 'comment']
```

Parameter description:

| Parameter | Description                                                  |
| --------- | ------------------------------------------------------------ |
| col_name  | Column name.                                                 |
| col_type  | Column data type. The following data types are currently supported: TINYINT, SMALLINT, INT, BIGINT, FLOAT, DOUBLE, DECIMAL, DATE, DATETIME, CHAR, VARCHAR[(length)], ARRAY, MAP, STRUCT. LARGEINT, HLL, and BITMAP types are not supported. |

> **Note**
>
> All non-partition columns have `NULL` as the default value (i.e., specify `DEFAULT "NULL"` in the create table statement). Partition columns must be declared last and cannot be `NULL`.

#### partition_desc

The syntax of `partition_desc` is as follows:

```SQL
PARTITION BY (par_col1[, par_col2...])
```

Currently, StarRocks only supports Identity Transforms. That is, a partition is created for each unique partition value.

> **Note**
>
> Partition columns must be declared last, support data types other than FLOAT, DOUBLE, DECIMAL, DATETIME, and do not support `NULL` values. Additionally, the order of partition columns declared in `partition_desc` must be consistent with the order of columns defined in `column_definition`.

#### PROPERTIES

The properties of the Hive table can be declared in `PROPERTIES` using the form `"key" = "value"`.

Here are a few common properties:

| **Property**       | **Description**                                               |
| ------------------ | ------------------------------------------------------------ |
| location           | The file path where the Managed Table is located. When using HMS as the metadata service, you do not need to specify the `location` parameter. When using AWS Glue as the metadata service:<ul><li>If the `location` parameter is specified when creating the current database, then you do not need to specify the `location` parameter when creating a table under the current database, StarRocks defaults to creating the table under the file path where the current database is located.</li><li>If the `location` parameter is not specified when creating the current database, then you must specify the `location` parameter when creating a table in the current database.</li></ul> |
| file_format        | The file format of the Managed Table. Currently, only Parquet format is supported. Default value: `parquet`. |
| compression_codec  | The compression format of the Managed Table. Currently supports SNAPPY, GZIP, ZSTD, and LZ4. Default value: `gzip`. |

### Examples

1. Create a non-partition table `unpartition_tbl`, which includes two columns `id` and `score`, as follows:

   ```SQL
   CREATE TABLE unpartition_tbl
   (
       id int,
       score double
   );
   ```

2. Create a partition table `partition_tbl_1`, which includes `action`, `id`, and `dt` three columns, and defines `id` and `dt` as partition columns, as follows:

   ```SQL
   CREATE TABLE partition_tbl_1
   (
       action varchar(20),
       id int,
       dt date
   )
   PARTITION BY (id,dt);
   ```

3. Query the data of the original table `partition_tbl_1` and create a partition table `partition_tbl_2` based on the query result, defining `id` and `dt` as the partition columns of `partition_tbl_2`:

   ```SQL
   CREATE TABLE partition_tbl_2
   PARTITION BY (id, dt)
   AS SELECT * from partition_tbl_1;
   ```

## Insert Data into Hive Table

Consistent with the internal table of StarRocks, if you have the [INSERT](../../administration/privilege_item.md#table-privilege-table) permission of the Hive table (Managed Table or External Table), you can use [INSERT](../../sql-reference/sql-statements/data-manipulation/INSERT.md) to write StarRocks table data into the Hive table (currently only supported for writing to Parquet format Hive table). This feature has been supported since version 3.2. It should be noted that the function of writing data to External Table is disabled by default, and you can enable it through the [system variable ENABLE_WRITE_HIVE_EXTERNAL_TABLE](../../reference/System_variable.md).

> **Note**
>
> You can grant and revoke permissions to users and roles through [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) and [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) operations.

[Switch to the target Hive Catalog and database](#switch-hive-catalog-and-database), and then use the following syntax to write StarRocks table data into a Parquet format Hive table:

### Syntax

```SQL
INSERT {INTO | OVERWRITE} <table_name>
( column_name [, ...] )
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }

-- Write data to the specified partition.
INSERT {INTO | OVERWRITE} <table_name>
PARTITION (par_col1=<value> [, par_col2=<value>...])
{ VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

> **Note**
>
> Partition columns are not allowed to be `NULL`, so when importing data, make sure that the partition columns have values.

### Parameter Description

| Parameter   | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| INTO        | Append data to the target table.                             |
| OVERWRITE   | Overwrite data into the target table.                         |
| column_name | 导入的目标列。可以指定一个或多个列。指定多个列时，必须用逗号 (`,`) 分隔。指定的列必须是目标表中存在的列，并且必须包含分区列。该参数可以与源表中的列名称不同，但顺序需一一对应。如果不指定该参数，则默认导入数据到目标表中的所有列。如果源表中的某个非分区列在目标列不存在，则写入默认值 `NULL`。如果查询语句的结果列类型与目标列的类型不一致，会进行隐式转化，如果不能进行转化，那么 INSERT INTO 语句会报语法解析错误。 |
| expression  | 表达式，用以为对应列赋值。                                   |
| DEFAULT     | 为对应列赋予默认值。                                         |
| query       | 查询语句，查询的结果会导入至目标表中。查询语句支持任意 StarRocks 支持的 SQL 查询语法。 |
| PARTITION   | 导入的目标分区。需要指定目标表的所有分区列，指定的分区列的顺序可以与建表时定义的分区列的顺序不一致。指定分区时，不允许通过列名 (`column_name`) 指定导入的目标列。 |

### 示例

1. 向表 `partition_tbl_1` 中插入如下三行数据：

   ```SQL
   INSERT INTO partition_tbl_1
   VALUES
       ("buy", 1, "2023-09-01"),
       ("sell", 2, "2023-09-02"),
       ("buy", 3, "2023-09-03");
   ```

2. 向表 `partition_tbl_1` 按指定列顺序插入一个包含简单计算的 SELECT 查询的结果数据：

   ```SQL
   INSERT INTO partition_tbl_1 (id, action, dt) SELECT 1+1, 'buy', '2023-09-03';
   ```

3. 向表 `partition_tbl_1` 中插入一个从其自身读取数据的 SELECT 查询的结果数据：

   ```SQL
   INSERT INTO partition_tbl_1 SELECT 'buy', 1, date_add(dt, INTERVAL 2 DAY)
   FROM partition_tbl_1
   WHERE id=1;
   ```

4. 向表 `partition_tbl_2` 中 `dt='2023-09-01'`、`id=1` 的分区插入一个 SELECT 查询的结果数据：

   ```SQL
   INSERT INTO partition_tbl_2 SELECT 'order', 1, '2023-09-01';
   ```

   Or

   ```SQL
   INSERT INTO partition_tbl_2 partition(dt='2023-09-01',id=1) SELECT 'order';
   ```

5. 将表 `partition_tbl_1` 中 `dt='2023-09-01'`、`id=1` 的分区下所有 `action` 列值全部覆盖为 `close`：

   ```SQL
   INSERT OVERWRITE partition_tbl_1 SELECT 'close', 1, '2023-09-01';
   ```

   Or

   ```SQL
   INSERT OVERWRITE partition_tbl_1 partition(dt='2023-09-01',id=1) SELECT 'close';
   ```

## 删除 Hive 表

同 StarRocks 内表一致，如果您拥有 Hive 表的 [DROP](../../administration/privilege_item.md#表权限-table) 权限，那么您可以使用 [DROP TABLE](../../sql-reference/sql-statements/data-definition/DROP_TABLE.md) 来删除该 Hive 表。本功能自 3.2 版本起开始支持。注意当前只支持删除 Hive 的 Managed Table。

> **说明**
>
> 您可以通过 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 操作对用户和角色进行权限的赋予和收回。

执行删除表的操作时，您必须在 DROP TABLE 语句中指定 `FORCE` 关键字。该操作不会删除表对应的文件路径，但是会删除 HDFS 或对象存储上的表数据。请您谨慎执行该操作。

[切换至目标 Hive Catalog 和数据库](#切换-hive-catalog-和数据库)，然后通过如下语句删除 Hive 表：

```SQL
DROP TABLE <table_name> FORCE
```

## 手动或自动更新元数据缓存

### 手动更新

默认情况下，StarRocks 会缓存 Hive 的元数据、并以异步模式自动更新缓存的元数据，从而提高查询性能。此外，在对 Hive 表做了表结构变更或其他表更新后，您也可以使用 [REFRESH EXTERNAL TABLE](../../sql-reference/sql-statements/data-definition/REFRESH_EXTERNAL_TABLE.md) 手动更新该表的元数据，从而确保 StarRocks 第一时间生成合理的查询计划：

```SQL
REFRESH EXTERNAL TABLE <table_name>
```

以下情况适用于执行手动更新元数据：

- 已有分区内的数据文件发生变更，如执行过 `INSERT OVERWRITE ... PARTITION ...` 命令。
- Hive 表有 Schema 变更。
- Hive 表被 DROP 后重建一个同名 Hive 表。
- 创建 Hive Catalog 时在 `PROPERTIES` 中指定了 `"enable_cache_list_names" = "true"`。在 Hive 侧新增分区后，需要查询新增分区。
  > **说明**
  >
  > 自 2.5.5 版本起，StarRocks 支持周期性刷新 Hive 元数据缓存。参见本文下面”[周期性刷新元数据缓存](#周期性刷新元数据缓存)“小节。开启 Hive 元数据缓存周期性刷新功能以后，默认情况下 StarRocks 每 10 分钟刷新一次 Hive 元数据缓存。因此，一般情况下，无需执行手动更新。您只有在新增分区后，需要立即查询新增分区的数据时，才需要执行手动更新。

注意 REFRESH EXTERNAL TABLE 只会更新 FE 中已缓存的表和分区。

### 自动增量更新

与自动异步更新策略不同，在自动增量更新策略下，FE 可以定时从 HMS 读取各种事件，进而感知 Hive 表元数据的变更情况，如增减列、增减分区和更新分区数据等，无需手动更新 Hive 表的元数据。
此功能会对 HMS 产生较大压力，请慎重使用。推荐使用[周期性刷新元数据缓存](#周期性刷新元数据缓存) 来感知数据变更。

开启自动增量更新策略的步骤如下：

#### 步骤 1：在 HMS 上配置事件侦听器

HMS 2.x 和 3.x 版本均支持配置事件侦听器。这里以配套 HMS 3.1.2 版本的事件侦听器配置为例。将以下配置项添加到 **$HiveMetastore/conf/hive-site.xml** 文件中，然后重启 HMS：

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
```xml
    <value>858993459</value>
</property>
```

配置完成后，您可以在 FE 日志文件中搜索 `event id`，然后通过查看事件 ID 来检查事件监听器是否配置成功。如果配置失败，则所有 `event id` 均为 `0`。

#### 步骤 2：在 StarRocks 上开启自动增量更新策略

您可以给 StarRocks 集群中某一个 Hive Catalog 开启自动增量更新策略，也可以给 StarRocks 集群中所有 Hive Catalog 开启自动增量更新策略。

- 如果要给单个 Hive Catalog 开启自动增量更新策略，则需要在创建该 Hive Catalog 时把 `PROPERTIES` 中的 `enable_hms_events_incremental_sync` 参数设置为 `true`，如下所示：

  ```SQL
  CREATE EXTERNAL CATALOG <catalog_name>
  [COMMENT <comment>]
  PROPERTIES
  (
      "type" = "hive",
      "hive.metastore.uris" = "thrift://102.168.xx.xx:9083",
       ....
      "enable_hms_events_incremental_sync" = "true"
  );
  ```
  
- 如果要给所有 Hive Catalog 开启自动增量更新策略，则需要把 `enable_hms_events_incremental_sync` 参数添加到每个 FE 的 **$FE_HOME/conf/fe.conf** 文件中，并设置为 `true`，然后重启 FE，使参数配置生效。

您还可以根据业务需求在每个 FE 的 **$FE_HOME/conf/fe.conf** 文件中对以下参数进行调优，然后重启 FE，使参数配置生效。

| Parameter                         | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| hms_events_polling_interval_ms    | StarRocks 从 HMS 中读取事件的时间间隔。默认值：`5000`。单位：毫秒。 |
| hms_events_batch_size_per_rpc     | StarRocks 每次读取事件的最大数量。默认值：`500`。            |
| enable_hms_parallel_process_evens | 指定 StarRocks 在读取事件时是否并行处理读取的事件。取值范围：`true` 和 `false`。默认值：`true`。取值为 `true` 则开启并行机制，取值为 `false` 则关闭并行机制。 |
| hms_process_events_parallel_num   | StarRocks 每次处理事件的最大并发数。默认值：`4`。            |

## 周期性刷新元数据缓存

自 2.5.5 版本起，StarRocks 可以周期性刷新经常访问的 Hive 外部数据目录的元数据缓存，达到感知数据更新的效果。您可以通过以下 [FE 参数](../../administration/Configuration.md#fe-配置项)配置 Hive 元数据缓存周期性刷新：

| 配置名称                                                      | 默认值                        | 说明                                  |
| ------------------------------------------------------------ | ---------------------------- | ------------------------------------ |
| enable_background_refresh_connector_metadata                 | v3.0 为 true，v2.5 为 false   | 是否开启 Hive 元数据缓存周期性刷新。开启后，StarRocks 会轮询 Hive 集群的元数据服务（HMS 或 AWS Glue），并刷新经常访问的 Hive 外部数据目录的元数据缓存，以感知数据更新。`true` 代表开启，`false` 代表关闭。[FE 动态参数](../../administration/Configuration.md#配置-fe-动态参数)，可以通过 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令设置。 |
| background_refresh_metadata_interval_millis                  | 600000（10 分钟）             | 接连两次 Hive 元数据缓存刷新之间的间隔。单位：毫秒。[FE 动态参数](../../administration/Configuration.md#配置-fe-动态参数)，可以通过 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令设置。 |
| background_refresh_metadata_time_secs_since_last_access_secs | 86400（24 小时）              | Hive 元数据缓存刷新任务过期时间。对于已被访问过的 Hive Catalog，如果超过该时间没有被访问，则停止刷新其元数据缓存。对于未被访问过的 Hive Catalog，StarRocks 不会刷新其元数据缓存。单位：秒。[FE 动态参数](../../administration/Configuration.md#配置-fe-动态参数)，可以通过 [ADMIN SET FRONTEND CONFIG](../../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 命令设置。 |

元数据缓存周期性刷新与元数据自动异步更新策略配合使用，可以进一步加快数据访问速度，降低从外部数据源读取数据的压力，提升查询性能。

## 附录：理解元数据自动异步更新策略

自动异步更新策略是 StarRocks 用于更新 Hive Catalog 中元数据的默认策略。

默认情况下（即当 `enable_metastore_cache` 参数和 `enable_remote_file_cache` 参数均设置为 `true` 时），如果一个查询命中 Hive 表的某个分区，则 StarRocks 会自动缓存该分区的元数据、以及该分区下数据文件的元数据。缓存的元数据采用懒更新 (Lazy Update) 策略。

例如，有一张名为 `table2` 的 Hive 表，该表的数据分布在四个分区：`p1`、`p2`、`p3` 和 `p4`。当一个查询命中 `p1` 时，StarRocks 会自动缓存 `p1` 的元数据、以及 `p1` 下数据文件的元数据。假设当前缓存元数据的更新和淘汰策略设置如下：

- 异步更新 `p1` 的缓存元数据的时间间隔（通过 `metastore_cache_refresh_interval_sec` 参数指定）为 2 小时。
- 异步更新 `p1` 下数据文件的缓存元数据的时间间隔（通过 `remote_file_cache_refresh_interval_sec` 参数指定）为 60 秒。
- 自动淘汰 `p1` 的缓存元数据的时间间隔（通过 `metastore_cache_ttl_sec` 参数指定）为 24 小时。
- 自动淘汰 `p1` 下数据文件的缓存元数据的时间间隔（通过 `remote_file_cache_ttl_sec` 参数指定）为 36 小时。

如下图所示。

![Update policy on timeline](../../assets/catalog_timeline_zh.png)

StarRocks 采用如下策略更新和淘汰缓存的元数据：

- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔不超过 60 秒，则 StarRocks 既不会更新 `p1` 的缓存元数据，也不会更新 `p1` 下数据文件的缓存元数据。
- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔超过 60 秒，则 StarRocks 会更新 `p1` 下数据文件的缓存元数据。
- 如果另有查询再次命中 `p1`，并且当前时间距离上次更新的时间间隔超过 2 小时，则 StarRocks 会更新 `p1` 的缓存元数据。
- 如果继上次更新结束后，`p1` 在 24 小时内未被访问，则 StarRocks 会淘汰 `p1` 的缓存元数据。后续有查询再次命中 `p1` 时，会重新缓存 `p1` 的元数据。
- 如果继上次更新结束后，`p1` 在 36 小时内未被访问，则 StarRocks 会淘汰 `p1` 下数据文件的缓存元数据。后续有查询再次命中 `p1` 时，会重新缓存 `p1` 下数据文件的元数据。