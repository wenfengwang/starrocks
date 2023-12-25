---
displayed_sidebar: Chinese
---

# リソースの作成

## 機能

リソースを作成します。StarRocksは、Apache Spark™、Apache Hive™、Apache Iceberg、Apache Hudi、およびJDBCのリソースを作成することをサポートしています。Sparkリソースは[Spark Load](../../../loading/SparkLoad.md)で使用され、YARN設定、中間データの保存パス、Broker設定など、データインポートに関連する情報を管理します。Hive、Iceberg、Hudi、JDBCリソースは、[外部テーブル](../../../data_source/External_table.md)をクエリする際にデータソースのアクセス情報を管理するために使用されます。

> 注意：
>
> - SystemレベルのCREATE RESOURCE権限を持つユーザーのみがリソースを作成できます。
> - StarRocks 2.3以降のバージョンでは、JDBCリソースの作成がサポートされています。

## 文法

```SQL
CREATE EXTERNAL RESOURCE "resource_name"
PROPERTIES ("key"="value"[, ...])
```

## パラメータ説明

### resource_name

リソース名。命名規則は以下の通りです：

- 数字(0-9)、アンダースコア(_)、またはアルファベット(a-zまたはA-Z)で構成され、アルファベットで始まる必要があります。
- 全体の長さは64文字を超えることはできません。

### PROPERTIES

リソースの設定項目で、リソースのタイプによって異なる設定が可能です。

#### Sparkリソース

Sparkクラスタの設定によって、追加する必要がある設定項目が異なります。現在のSpark Loadは、Sparkのcluster managerがYARNで、データストレージシステムがHDFSである場合にのみサポートされており、YARNとHDFSはHA（高可用性）をサポートしています。具体的には以下のようなケースがあります：

- Brokerプロセスを使用してインポートする場合

  - Sparkのcluster managerがYARNで、データストレージシステムがHDFSの場合、以下の設定項目を追加する必要があります：

    | **設定項目**                                | **必須** | **説明**                                                     |
    | ----------------------------------------- | -------- | ------------------------------------------------------------ |
    | type                                      | はい       | リソースタイプ。`spark`としてください。                                   |
    | spark.master                              | はい       | Sparkのcluster manager。現在はYARNのみをサポートしているため、`yarn`としてください。 |
    | spark.submit.deployMode                   | はい       | Spark driverのデプロイモード。`cluster`または`client`を指定できます。詳細は[Launching Spark on YARN](https://spark.apache.org/docs/3.3.0/running-on-yarn.html#launching-spark-on-yarn)を参照してください。 |
    | spark.executor.memory                     | いいえ       | Spark executorが使用するメモリ量。KB、MB、GB、またはTBで指定します。       |
    | spark.yarn.queue                          | いいえ       | YARNのキュー名。                                              |
    | spark.hadoop.yarn.resourcemanager.address | はい       | YARN ResourceManagerのアドレス。                                  |
    | spark.hadoop.fs.defaultFS                 | はい       | HDFSのNameNodeのアドレス。形式：`hdfs://namenode_host:port`。 |
    | working_dir                               | はい       | ETLジョブが生成するファイルを保存するためのHDFSファイルパス。例：`hdfs://host:port/tmp/starrocks`。 |
    | broker                                    | はい       | Brokerの名前。現在のすべてのBrokerの名前は[SHOW BROKER](../Administration/SHOW_BROKER.md)コマンドで確認できます。Brokerがまだ追加されていない場合は、[ALTER SYSTEM](../Administration/ALTER_SYSTEM.md)コマンドでBrokerを追加できます。 |
    | broker.username                           | いいえ       | 指定されたHDFSユーザーを使用してHDFS内のファイルにアクセスします。HDFSファイルに特定のユーザーのみがアクセスできる場合は、このパラメータを指定する必要があります。そうでない場合は、指定する必要はありません。 |
    | broker.password                           | いいえ       | HDFSユーザーのパスワード。                                              |

  - Sparkのcluster managerがYARNで、YARN ResourceManagerがHAを有効にしている場合、データストレージシステムがHDFSであれば、以下の設定項目を追加する必要があります：

    | **設定項目**                                       | **必須** | **説明**                                                     |
    | ------------------------------------------------ | -------- | ------------------------------------------------------------ |
    | type                                             | はい       | リソースタイプ。`spark`としてください。                                    |
    | spark.master                                     | はい       | Sparkのcluster manager。現在はYARNのみをサポートしているため、`yarn`としてください。 |
    | spark.submit.deployMode                          | はい       | Sparkドライバのデプロイモード。`cluster`または`client`を指定できます。詳細は[Launching Spark on YARN](https://spark.apache.org/docs/3.3.0/running-on-yarn.html#launching-spark-on-yarn)を参照してください。 |
    | spark.hadoop.yarn.resourcemanager.ha.enabled     | はい       | YARN ResourceManagerのHAが有効かどうか。`true`に設定してHAを有効にしてください。 |
    | spark.hadoop.yarn.resourcemanager.ha.rm-ids      | はい       | YARN ResourceManagerの論理IDリスト。複数の論理IDはコンマ(`,`)で区切ります。 |
    | spark.hadoop.yarn.resourcemanager.hostname.rm-id | はい       | 各rm-idに対して、対応するResourceManagerのホスト名を指定してください。この設定項目が追加されている場合は、`spark.hadoop.yarn.resourcemanager.address.rm-id`を追加する必要はありません。 |
    | spark.hadoop.yarn.resourcemanager.address.rm-id  | はい       | 各rm-idに対して、対応するResourceManagerの`host:port`を指定してください。この設定項目が追加されている場合は、`spark.hadoop.yarn.resourcemanager.hostname.rm-id`を追加する必要はありません。 |
    | spark.hadoop.fs.defaultFS                        | はい       | Sparkが使用するHDFSのNameNodeノードのアドレス。形式：`hdfs://namenode_host:port`。 |
    | working_dir                                      | はい       | ETLジョブが生成する中間データを保存するためのディレクトリ。例：`hdfs://host:port/tmp/starrocks`。 |
    | broker                                           | はい       | Brokerの名前。現在のすべてのBrokerの名前は[SHOW BROKER](../Administration/SHOW_BROKER.md)コマンドで確認できます。Brokerがまだ追加されていない場合は、[ALTER SYSTEM](../Administration/ALTER_SYSTEM.md)コマンドでBrokerを追加できます。 |
    | broker.username                           | いいえ       | 指定されたHDFSユーザーを使用してHDFS内のファイルにアクセスします。HDFSファイルに特定のユーザーのみがアクセスできる場合は、このパラメータを指定する必要があります。そうでない場合は、指定する必要はありません。 |
    | broker.password                           | いいえ       | HDFSユーザーのパスワード。                                              |

  - Sparkのcluster managerがYARNで、データストレージシステムがHDFS HAの場合、以下の設定項目を追加する必要があります：

    | **設定項目**                                                   | **必須** | **説明**                                                     |
    | ------------------------------------------------------------ | -------- | ------------------------------------------------------------ |
    | type                                                         | はい       | リソースタイプ。`spark`としてください。                                    |
    | spark.master                                                 | はい       | Sparkのcluster manager。現在はYARNのみをサポートしているため、`yarn`としてください。 |
    | spark.hadoop.yarn.resourcemanager.address                    | はい       | YARN ResourceManagerのアドレス。                                  |
    | spark.hadoop.fs.defaultFS                                    | はい       | Sparkが使用するHDFSのNameNodeノードのアドレス。形式：`hdfs://namenode_host:port`。 |
    | spark.hadoop.dfs.nameservices                                | はい       | HDFSのnameserviceのID。Sparkが使用するための設定です。              |
    | spark.hadoop.dfs.ha.namenodes.[nameservice ID]               | はい       | HDFSのNameNodeのID。複数のNameNode IDを設定でき、IDはコンマ(`,`)で区切ります。Sparkが使用するための設定です。 |
    | spark.hadoop.dfs.namenode.rpc-address.[nameservice ID].[namenode ID] | はい       | 各HDFS NameNodeがリッスンするRPCアドレス。完全修飾RPCアドレスを設定する必要があります。Sparkが使用するための設定です。 |
    | spark.hadoop.dfs.client.failover.proxy.provider              | はい       | Active状態のNameNodeに接続するためのHDFSのJavaクラス。Sparkが使用するための設定です。 |
    | working_dir                                                  | はい       | ETLジョブが生成する中間データを保存するためのディレクトリ。例：`hdfs://host:port/tmp/starrocks`。 |
    | broker                                                       | はい       | Brokerの名前。現在のすべてのBrokerの名前は[SHOW BROKER](../Administration/SHOW_BROKER.md)コマンドで確認できます。Brokerがまだ追加されていない場合は、[ALTER SYSTEM](../Administration/ALTER_SYSTEM.md)コマンドでBrokerを追加できます。 |
    | broker.username                           | いいえ       | 指定されたHDFSユーザーを使用してHDFS内のファイルにアクセスします。HDFSファイルに特定のユーザーのみがアクセスできる場合は、このパラメータを指定する必要があります。そうでない場合は、指定する必要はありません。 |
    | broker.password                           | いいえ       | HDFSユーザーのパスワード。                                              |
    | broker.dfs.nameservices                                      | はい       | HDFSのnameserviceのID。Brokerが使用するための設定です。             |
    | broker.dfs.ha.namenodes.[nameservice ID]                    | はい       | HDFSのNameNodeのID。複数のNameNode IDを設定でき、IDはコンマ(`,`)で区切ります。Brokerが使用するための設定です。 |

    | broker.dfs.namenode.rpc-address.[nameservice_ID].[name_node_ID] | 必須       | 各HDFS NameNodeがリッスンするRPCアドレスです。完全修飾のRPCアドレスを設定する必要があります。この設定はBrokerが使用します。 |
    | broker.dfs.client.failover.proxy.provider                    | 必須       | アクティブなNameNodeに接続するためのHDFSのJavaクラスです。この設定はBrokerが使用します。 |

- Brokerプロセスを使用せずにインポートする場合、リソースを作成する際のパラメータ設定はBrokerプロセスを使用する場合と若干異なります。具体的な違いは以下の通りです：

  - `broker`を渡す必要はありません。
  - ユーザー認証やNameNodeのHAを設定する必要がある場合は、HDFSクラスタ内の**hdfs-site.xml**ファイルにパラメータを設定する必要があります。具体的なパラメータと説明については、[broker_properties](../data-manipulation/BROKER_LOAD.md#hdfs)を参照してください。そして、**hdfs-site.xml**ファイルを各FEの**$FE_HOME/conf**および各BEの**$BE_HOME/conf**に配置してください。

> **説明**
>
> - Brokerプロセスを使用せずにインポートする場合、HDFSファイルが特定のユーザーのみによってアクセス可能である場合は、HDFSユーザー名`broker.name`とHDFSユーザーパスワード`broker.password`を渡す必要があります。
> - 上記のいずれの場合も、表にない設定項目を追加する場合は、[Spark Configuration](https://spark.apache.org/docs/latest/configuration.html)および[Running Spark on YARN](https://spark.apache.org/docs/latest/running-on-yarn.html)を参照してください。

#### Hiveリソース

Hiveリソースを作成する場合、以下の設定項目を追加する必要があります：

| **設定項目**          | **必須** | **説明**                  |
| ------------------- | -------- | ------------------------- |
| type                | 必須       | リソースタイプで、`hive`を指定します。 |
| hive.metastore.uris | 必須       | HiveメタストアのURIです。   |

#### Icebergリソース

Icebergリソースを作成する場合、以下の設定項目を追加する必要があります：

| **設定項目**                          | **必須** | **説明**                                                     |
| ----------------------------------- | -------- | ------------------------------------------------------------ |
| type                                | 必須       | リソースタイプで、`iceberg`を指定します。                                 |
| starrocks.catalog-type              | 必須       | Icebergのカタログタイプです。StarRocks 2.3以下のバージョンはHiveカタログのみをサポートし、StarRocks 2.3以上のバージョンはHiveカタログとカスタムカタログをサポートします。Hiveカタログを使用する場合は、このパラメータを`HIVE`に設定します。カスタムカタログを使用する場合は、このパラメータを`CUSTOM`に設定します。詳細は[Creating Iceberg Resources](../../../data_source/External_table.md#creating-iceberg-resources)を参照してください。 |
| iceberg.catalog.hive.metastore.uris | 必須       | HiveメタストアのURIです。                                       |

#### Hudiリソース

Hudiリソースを作成する場合、以下の設定項目を追加する必要があります：

| **設定項目**          | **必須** | **説明**                  |
| ------------------- | -------- | ------------------------- |
| type                | 必須       | リソースタイプで、`hudi`を指定します。 |
| hive.metastore.uris | 必須       | HiveメタストアのURIです。   |

#### JDBCリソース

JDBCリソースを作成する場合、以下の設定項目を追加する必要があります：

| **設定項目**   | **必須** | **説明**                                                     |
| ------------ | -------- | ------------------------------------------------------------ |
| type         | 必須       | リソースタイプで、`jdbc`を指定します。                                    |
| user         | 必須       | サポートされているJDBCデータベース（以下「対象データベース」と呼びます）にログインするユーザー名です。  |
| password     | 必須       | 対象データベースのログインパスワードです。                                       |
| jdbc_uri     | 必須       | 対象データベースに接続するためのJDBC URIで、対象データベースのURI構文を満たす必要があります。一般的な対象データベースのURIについては、[MySQL](https://dev.mysql.com/doc/connector-j/8.0/en/connector-j-reference-jdbc-url-format.html)、[Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/data-sources-and-URLs.html#GUID-6D8EFA50-AB0F-4A2B-88A0-45B4A67C361E)、[PostgreSQL](https://jdbc.postgresql.org/documentation/head/connect.html)、[SQL Server](https://docs.microsoft.com/en-us/sql/connect/jdbc/building-the-connection-url?view=sql-server-ver16)の公式ドキュメントを参照してください。 |
| driver_url   | 必須       | JDBCドライバのJARファイルをダウンロードするためのURLで、HTTPプロトコルまたはfileプロトコルを使用できます。例：`https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar`、`file:///home/disk1/postgresql-42.3.3.jar`。 |
| driver_class | 必須       | 対象データベースが使用するJDBCドライバのクラス名です。一般的なクラス名は以下の通りです：<ul><li>MySQL：com.mysql.jdbc.Driver（MySQL 5.x以下のバージョン）およびcom.mysql.cj.jdbc.Driver（MySQL 6.x以上のバージョン）</li><li>SQL Server：com.microsoft.sqlserver.jdbc.SQLServerDriver</li><li>Oracle：oracle.jdbc.driver.OracleDriver</li><li>PostgreSQL：org.postgresql.Driver</li></ul> |

## 例

例1：YARNをSparkのクラスタマネージャとして使用し、HDFSでデータを保存する場合、`spark0`という名前のSparkリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "spark0"
PROPERTIES (
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.executor.memory" = "1g",
    "spark.yarn.queue" = "queue0",
    "spark.hadoop.yarn.resourcemanager.address" = "resourcemanager_host:8032",
    "spark.hadoop.fs.defaultFS" = "hdfs://namenode_host:9000",
    "working_dir" = "hdfs://namenode_host:9000/tmp/starrocks",
    "broker" = "broker0",
    "broker.username" = "user0",
    "broker.password" = "password0"
);
```

例2：YARN HAをSparkのクラスタマネージャとして使用し、HDFSでデータを保存する場合、`spark1`という名前のSparkリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "spark1"
PROPERTIES (
    "type" = "spark",
    "spark.master" = "yarn",
    "spark.submit.deployMode" = "cluster",
    "spark.hadoop.yarn.resourcemanager.ha.enabled" = "true",
    "spark.hadoop.yarn.resourcemanager.ha.rm-ids" = "rm1,rm2",
    "spark.hadoop.yarn.resourcemanager.hostname.rm1" = "host1",
    "spark.hadoop.yarn.resourcemanager.hostname.rm2" = "host2",
    "spark.hadoop.fs.defaultFS" = "hdfs://namenode_host:9000",
    "working_dir" = "hdfs://namenode_host:9000/tmp/starrocks",
    "broker" = "broker1"
);
```

例3：YARNをSparkのクラスタマネージャとして使用し、HDFS HAでデータを保存する場合、`spark2`という名前のSparkリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "spark2"
PROPERTIES (
    "type" = "spark", 
    "spark.master" = "yarn",
    "spark.hadoop.yarn.resourcemanager.address" = "resourcemanager_host:8032",
    "spark.hadoop.fs.defaultFS" = "hdfs://myha",
    "spark.hadoop.dfs.nameservices" = "myha",
    "spark.hadoop.dfs.ha.namenodes.myha" = "mynamenode1,mynamenode2",
    "spark.hadoop.dfs.namenode.rpc-address.myha.mynamenode1" = "nn1_host:rpc_port",
    "spark.hadoop.dfs.namenode.rpc-address.myha.mynamenode2" = "nn2_host:rpc_port",
    "spark.hadoop.dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider",
    "working_dir" = "hdfs://myha/tmp/starrocks",
    "broker" = "broker2",
    "broker.dfs.nameservices" = "myha",
    "broker.dfs.ha.namenodes.myha" = "mynamenode1,mynamenode2",
    "broker.dfs.namenode.rpc-address.myha.mynamenode1" = "nn1_host:rpc_port",
    "broker.dfs.namenode.rpc-address.myha.mynamenode2" = "nn2_host:rpc_port",
    "broker.dfs.client.failover.proxy.provider" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
);
```

例4：`hive0`という名前のHiveリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "hive0"
PROPERTIES (
  "type" = "hive",
  "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

例5：`iceberg0`という名前のIcebergリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "iceberg0" 
PROPERTIES ( 
   "type" = "iceberg", 
   "starrocks.catalog-type"="HIVE", 
   "iceberg.catalog.hive.metastore.uris"="thrift://xx.xx.xx.xx:9083" 
);
```

例6：`hudi0`という名前のHudiリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE "hudi0" 
PROPERTIES ( 
    "type" = "hudi", 
    "hive.metastore.uris" = "thrift://xx.xx.xx.xx:9083"
);
```

例7：`jdbc0`という名前のJDBCリソースを作成するには、以下のステートメントを使用します：

```SQL
CREATE EXTERNAL RESOURCE jdbc0
PROPERTIES (
    "type"="jdbc",
    "user"="postgres",
    "password"="changeme",
    "jdbc_uri"="jdbc:postgresql://127.0.0.1:5432/jdbc_test",
    "driver_url"="https://repo1.maven.org/maven2/org/postgresql/postgresql/42.3.3/postgresql-42.3.3.jar",
    "driver_class"="org.postgresql.Driver"
);
```

## 関連操作

- リソース属性を変更するには、[ALTER RESOURCE](../data-definition/ALTER_RESOURCE.md)を参照してください。
- リソースを削除するには、[DROP RESOURCE](../data-definition/DROP_RESOURCE.md)を参照してください。
- Spark リソースを使用して Spark Load を行うには、[Spark Load](../../../loading/SparkLoad.md)を参照してください。
- Hive、Iceberg、Hudi、JDBC リソースを参照して外部テーブルを作成するには、[外部テーブル](../../../data_source/External_table.md)を参照してください。
