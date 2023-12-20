---
displayed_sidebar: English
toc_max_heading_level: 3
---




```markdown
      "azure.adls1.use_managed_service_identity" = "true"
  );
  ```

- 如果您选择服务主体认证方法，请运行如下命令：

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

- 如果您选择托管身份认证方法，请运行如下命令：

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

- 如果您选择共享密钥认证方法，请运行如下命令：

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

- 如果您选择服务主体认证方法，请运行如下命令：

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

- 如果您选择基于虚拟机的认证方法，请运行如下命令：

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

- 如果您选择基于服务账户的认证方法，请运行如下命令：

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

- 如果您选择基于模拟的认证方法：

-   如果您让 VM 实例模拟服务账户，请运行如下命令：

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

-   如果您让一个服务账户模拟另一个服务账户，请运行如下命令：

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


### 切换到 Iceberg 目录和其中的数据库

您可以使用以下方法之一切换到 Iceberg 目录及其中的数据库：

- 使用 [SET CATALOG](../../sql-reference/sql-statements/data-definition/SET_CATALOG.md) 在当前会话中指定 Iceberg 目录，然后使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 指定活动数据库：

  ```SQL
  -- 在当前会话中切换到指定的目录：
  SET CATALOG <catalog_name>
  -- 在当前会话中指定活动数据库：
  USE <db_name>
  ```

- 直接使用 [USE](../../sql-reference/sql-statements/data-definition/USE.md) 切换到一个 Iceberg 目录和其中的数据库：

  ```SQL
  USE <catalog_name>.<db_name>
  ```


### 删除 Iceberg 目录

您可以使用 [DROP CATALOG](../../sql-reference/sql-statements/data-definition/DROP_CATALOG.md) 删除外部目录。

以下示例删除名为 `iceberg_catalog_glue` 的 Iceberg 目录：

```SQL
DROP CATALOG iceberg_catalog_glue;
```


### 查看 Iceberg 表的架构

您可以使用以下语法之一来查看 Iceberg 表的架构：

- 查看架构

  ```SQL
  DESC[RIBE] <catalog_name>.<database_name>.<table_name>
  ```

- 从 CREATE 语句查看架构和位置

  ```SQL
  SHOW CREATE TABLE <catalog_name>.<database_name>.<table_name>
  ```


### 查询 Iceberg 表

1. 使用 [SHOW DATABASES](../../sql-reference/sql-statements/data-manipulation/SHOW_DATABASES.md) 查看 Iceberg 集群中的数据库：

   ```SQL
   SHOW DATABASES FROM <catalog_name>
   ```

2. [切换到一个 Iceberg 目录和其中的数据库](#switch-to-an-iceberg-catalog-and-a-database-in-it)。

3. 使用 [SELECT](../../sql-reference/sql-statements/data-manipulation/SELECT.md) 查询指定数据库中的目标表：

   ```SQL
   SELECT count(*) FROM <table_name> LIMIT 10
   ```


### 创建 Iceberg 数据库

与 StarRocks 的内部目录类似，如果您拥有 [CREATE DATABASE](../../administration/privilege_item.md#catalog) 权限，则可以使用 [CREATE DATABASE](../../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) 语句在该 Iceberg 目录中创建数据库。从 v3.1 开始支持此功能。

:::tip

您可以使用 [GRANT](../../sql-reference/sql-statements/account-management/GRANT.md) 和 [REVOKE](../../sql-reference/sql-statements/account-management/REVOKE.md) 授予和撤销权限。

:::

[切换到 Iceberg 目录](#switch-to-an-iceberg-catalog-and-a-database-in-it)，然后使用以下语句在该目录中创建 Iceberg 数据库：

```SQL
CREATE DATABASE <database_name>
[PROPERTIES ("location" = "<prefix>://<path_to_database>/<database_name.db>/")]
```

您可以使用 `location` 参数指定要在其中创建数据库的文件路径。支持 HDFS 和云存储。如果不指定 `location` 参数，StarRocks 将在 Iceberg 目录的默认文件路径中创建数据库。

`prefix` 根据您使用的存储系统而有所不同：

#### HDFS

`prefix` 值：hdfs

#### Google GCS

`prefix` 值：gs

#### Azure Blob 存储

`prefix` 值：

- 如果您的存储账户允许通过 HTTP 访问，则 `prefix` 为 wasb。
- 如果您的存储账户允许通过 HTTPS 访问，则 `prefix` 为 wasbs。

#### Azure 数据湖存储 Gen1

`prefix` 值：adl

#### Azure 数据湖存储 Gen2

`prefix` 值：

- 如果您的存储账户允许通过 HTTP 访问，则 `prefix` 为 abfs。
- 如果您的存储账户允许通过 HTTPS 访问，则 `prefix` 为 abfss。

#### AWS S3 或其他 S3 兼容存储（例如 MinIO）

`prefix` 值：s3
```
- StarRocks 首先尝试从内存中检索所请求的元数据。如果内存中未命中元数据，StarRocks 将尝试从磁盘中检索元数据。从磁盘检索的元数据将被加载到内存中。如果磁盘中也未命中元数据，StarRocks 将从远程存储中检索元数据并将检索到的元数据缓存在内存中。
- StarRocks 将从内存中逐出的元数据写入磁盘，但直接丢弃从磁盘中逐出的元数据。

#### Iceberg 元数据缓存参数

##### enable_iceberg_metadata_disk_cache

单位：无
默认值：`false`
描述：指定是否启用磁盘缓存。

##### iceberg_metadata_cache_disk_path

单位：无
默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"`
描述：磁盘上缓存的元数据文件的保存路径。

##### iceberg_metadata_disk_cache_capacity

单位：字节
默认值：`2147483648`，相当于 2 GB
描述：磁盘上允许缓存的元数据的最大大小。

##### iceberg_metadata_memory_cache_capacity

单位：字节
默认值：`536870912`，相当于 512 MB
描述：内存中允许缓存的元数据的最大大小。

##### iceberg_metadata_memory_cache_expiration_seconds

单位：秒
默认值：`86500`
描述：内存中缓存条目自最后访问后过期的时间。

##### iceberg_metadata_disk_cache_expiration_seconds

单位：秒
默认值：`604800`，相当于一周
描述：磁盘上缓存条目自最后访问后过期的时间。

##### iceberg_metadata_cache_max_entry_size

单位：字节
默认值：`8388608`，相当于 8 MB
描述：可以缓存的文件的最大大小。超过此参数值的文件无法缓存。如果查询请求这些文件，StarRocks 会从远程存储中检索它们。
```markdown
- StarRocks 首先尝试从内存中检索请求的元数据。如果元数据在内存中未命中，StarRocks 会尝试从磁盘检索元数据。从磁盘检索到的元数据将被加载到内存中。如果磁盘中也未命中元数据，StarRocks 会从远程存储检索元数据，并将检索到的元数据缓存在内存中。
- StarRocks 将从内存中逐出的元数据写入磁盘，但直接丢弃从磁盘中逐出的元数据。

#### Iceberg 元数据缓存参数

##### enable_iceberg_metadata_disk_cache

单位：无
默认值：`false`
描述：指定是否启用磁盘缓存。

##### iceberg_metadata_cache_disk_path

单位：无
默认值：`StarRocksFE.STARROCKS_HOME_DIR + "/caches/iceberg"`
描述：磁盘上缓存元数据文件的保存路径。

##### iceberg_metadata_disk_cache_capacity

单位：字节
默认值：`2147483648`，相当于 2 GB
描述：磁盘上允许缓存的元数据的最大大小。

##### iceberg_metadata_memory_cache_capacity

单位：字节
默认值：`536870912`，相当于 512 MB
描述：内存中允许缓存的元数据的最大大小。

##### iceberg_metadata_memory_cache_expiration_seconds

单位：秒
默认值：`86500`
描述：内存中缓存条目自上次访问后的过期时间。

##### iceberg_metadata_disk_cache_expiration_seconds

单位：秒
默认值：`604800`，相当于一周
描述：磁盘上缓存条目自最后访问后的过期时间。

##### iceberg_metadata_cache_max_entry_size

单位：字节
默认值：`8388608`，相当于 8 MB
描述：可以缓存的文件的最大大小。超过此参数值的文件无法被缓存。如果查询请求这些文件，StarRocks 会从远程存储检索它们。
```