---
displayed_sidebar: "Japanese"
---

# オーディットローダーを使用してStarRocks内での監査ログの管理

このトピックでは、プラグインであるオーディットローダーを介してテーブル内のStarRocks監査ログを管理する方法について説明します。

StarRocksは、内部データベースではなく、ローカルファイル**fe/log/fe.audit.log**に監査ログを保存します。オーディットローダープラグインを使用すると、クラスタ内で監査ログを直接管理できます。オーディットローダーはファイルからログを読み込み、HTTP PUTを介してStarRocksにロードします。

## 監査ログを保存するテーブルを作成する

StarRocksクラスタ内にデータベースとテーブルを作成して、その監査ログを保存します。詳細な手順については、[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)および[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

監査ログのフィールドは、異なるStarRocksバージョンごとに異なるため、StarRocksと互換性のあるテーブルを作成するために次の例から選択する必要があります。

> **注意**
>
> 例のテーブルスキーマを変更しないでください。さもないと、ログの読み込みが失敗します。

- StarRocks v2.4、v2.5、v3.0、v3.1およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `queryType`      VARCHAR(12)            COMMENT "クエリタイプ（query, slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `authorizedUser` VARCHAR(64)            COMMENT "user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `catalog`        VARCHAR(32)            COMMENT "カタログ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリステート（EOF、ERR、OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリの待ち時間（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0および1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "プランニングのCPUリソース消費時間（ナノ秒単位）",
  `planMemCosts`   DOUBLE                 COMMENT "プランニングのメモリコスト（バイト単位）"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `queryType`)
COMMENT "監査ログテーブル"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.buckets" = "3",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- StarRocks v2.3.0およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリステート（EOF、ERR、OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリの待ち時間（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0および1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "監査ログテーブル"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- StarRocks v2.2.1およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリステート（EOF、ERR、OK）",
  `queryTime`      BIGINT                 COMMENT "クエリの待ち時間（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0および1）",
  `frontendIp`     VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "監査ログテーブル"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- StarRocks v2.2.0、v2.1.0およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリステート（EOFE、RR、OK）",
  `queryTime`      BIGINT                 COMMENT "クエリの待ち時間（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0および1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "監査ログテーブル"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`) BUCKETS 3 
PROPERTIES (
  "dynamic_partition.time_unit" = "DAY",
  "dynamic_partition.start" = "-30",
  "dynamic_partition.end" = "3",
  "dynamic_partition.prefix" = "p",
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```
```sql
    is_query        TINYINT               COMMENT "クエリである場合は (0 と 1)",
    frontend_ip     VARCHAR(32)           COMMENT "SQL を実行する FE の IP アドレス",
    stmt            STRING                COMMENT "SQL ステートメント",
    digest          VARCHAR(32)           COMMENT "SQL フィンガープリント"
) engine=OLAP
DUPLICATE KEY(query_id, time, client_ip)
PARTITION BY RANGE(time) ()
DISTRIBUTED BY HASH(query_id) BUCKETS 3 
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.enable" = "true",
    "replication_num" = "3"
);
```

- StarRocks v2.0.0 以降のマイナー バージョン、StarRocks v1.19.0 以降のマイナー バージョン:

```sql
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリ ID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時間",
    client_ip       VARCHAR(32)           COMMENT "クライアント IP アドレス",
    user            VARCHAR(64)           COMMENT "クエリを初期化するユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態 (EOF、ERR、OK)",
    query_time      BIGINT                COMMENT "クエリのレイテンシ (ミリ秒単位)",
    scan_bytes      BIGINT                COMMENT "スキャンされたデータのサイズ (バイト単位)",
    scan_rows       BIGINT                COMMENT "スキャンされたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "増分 SQL ステートメント ID",
    is_query        TINYINT               COMMENT "SQL がクエリである場合 (0 と 1)",
    frontend_ip     VARCHAR(32)           COMMENT "SQL を実行する FE の IP アドレス",
    stmt            STRING                COMMENT "SQL ステートメント"
) engine=OLAP
DUPLICATE KEY(query_id, time, client_ip)
PARTITION BY RANGE(time) ()
DISTRIBUTED BY HASH(query_id) BUCKETS 3 
PROPERTIES(
    "dynamic_partition.time_unit" = "DAY",
    "dynamic_partition.start" = "-30",
    "dynamic_partition.end" = "3",
    "dynamic_partition.prefix" = "p",
    "dynamic_partition.enable" = "true",
    "replication_num" = "3"
);
```

`starrocks_audit_tbl__` は動的パーティションで作成されます。デフォルトでは、テーブル作成後10分後に最初の動的パーティションが作成されます。その後、監査ログをテーブルにロードできます。次のステートメントを使用して、テーブル内のパーティションを確認できます:

```sql
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

パーティションが作成されたら、次の手順に進むことができます。

## 監査ローダーのダウンロードと構成

1. [Audit Loader インストール パッケージ](https://releases.starrocks.io/resources/AuditLoader.zip)をダウンロードします。パッケージにはさまざまな StarRocks バージョン用の複数のディレクトリが含まれています。対応するディレクトリに移動し、StarRocks と互換性のあるパッケージをインストールする必要があります。

    - **2.4**: StarRocks v2.4.0 以降のマイナー バージョン
    - **2.3**: StarRocks v2.3.0 以降のマイナー バージョン
    - **2.2.1+**: StarRocks v2.2.1 以降のマイナー バージョン
    - **2.1-2.2.0**: StarRocks v2.2.0、StarRocks v2.1.0 以降のマイナー バージョン
    - **1.18.2-2.0**: StarRocks v2.0.0 以降のマイナー バージョン、StarRocks v1.19.0 以降のマイナー バージョン

2. インストール パッケージを解凍します。

    ```shell
    unzip auditloader.zip
    ```

    次のファイルが解凍されます:

    - **auditloader.jar**: Audit Loader の JAR ファイル。
    - **plugin.properties**: Audit Loader のプロパティ ファイル。
    - **plugin.conf**: Audit Loader の構成ファイル。

3. **plugin.conf** を修正して、Audit Loader が正常に機能するように必要な項目を構成する必要があります:

    - `frontend_host_port`: FE の IP アドレスと HTTP ポート。形式は `<fe_ip>:<fe_http_port>` です。デフォルト値は `127.0.0.1:8030` です。
    - `database`: 監査ログをホストするために作成したデータベースの名前。
    - `table`: 監査ログをホストするために作成したテーブルの名前。
    - `user`: クラスタのユーザー名。テーブルにデータをロードする権限 (LOAD_PRIV) を持っている必要があります。
    - `password`: ユーザーのパスワード。

4. ファイルをパッケージに再度圧縮します。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. パッケージを FE ノードをホストするすべてのマシンにディスパッチします。すべてのパッケージが同一のパスに保存されていることを確認してください。そうでない場合、インストールに失敗します。パッケージをディスパッチした後、パッケージの絶対パスをコピーすることを忘れないでください。

## 監査ローダーのインストール

次のステートメントを実行して、コピーしたパスと共に Audit Loader を StarRocks のプラグインとしてインストールします:

```sql
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

詳細な手順については[INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)を参照してください。

## インストールの検証とクエリ監査ログ

1. [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) を使用してインストールが成功したかどうかを確認できます。

    次の例では、プラグイン `AuditLoader` の `Status` が `INSTALLED` であり、インストールが成功していることを示しています。

    ```Plain
    mysql> SHOW PLUGINS\G
    *************************** 1. row ***************************
        Name: __builtin_AuditLogBuilder
        Type: AUDIT
    Description: builtin audit logger
        Version: 0.12.0
    JavaVersion: 1.8.31
    ClassName: com.starrocks.qe.AuditLogBuilder
        SoName: NULL
        Sources: Builtin
        Status: INSTALLED
    Properties: {}
    *************************** 2. row ***************************
        Name: AuditLoader
        Type: AUDIT
    Description: load audit log to olap load, and user can view the statistic of queries
        Version: 1.0.1
    JavaVersion: 1.8.0
    ClassName: com.starrocks.plugin.audit.AuditLoaderPlugin
        SoName: NULL
        Sources: /x/xx/xxx/xxxxx/auditloader.zip
        Status: INSTALLED
    Properties: {}
    2 rows in set (0.01 sec)
    ```

2. いくつかのランダムな SQL を実行して監査ログを生成し、その後60秒（または Audit Loader を構成する際に指定した項目 `max_batch_interval_sec` の時間）待機して、Audit Loader が監査ログを StarRocks にロードできるようにします。

3. テーブルをクエリして監査ログを確認します。

    ```sql
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    次の例では、監査ログがテーブルに正常にロードされたことを示しています:

    ```Plain
    mysql> SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__\G
    *************************** 1. row ***************************
        queryId: 082ddf02-6492-11ed-a071-6ae6b1db20eb
        timestamp: 2022-11-15 11:03:08
        clientIp: xxx.xx.xxx.xx:33544
            user: root
    resourceGroup: default_wg
                db: 
            state: EOF
        errorCode: 
        queryTime: 8
        scanBytes: 0
        scanRows: 0
        returnRows: 0
        cpuCostNs: 62380
    memCostBytes: 14504
            stmtId: 33
        isQuery: 1
            feIp: xxx.xx.xxx.xx
            stmt: SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__
            digest: 
    planCpuCosts: 21
    planMemCosts: 0
    1 row in set (0.01 sec)
    ```

## トラブルシューティング

動的パーティションが作成され、プラグインがインストールされた後に監査ログがテーブルにロードされない場合は、**plugin.conf** が適切に構成されているかどうかを確認できます。それを修正するには、まずプラグインをアンインストールする必要があります:

```sql
UNINSTALL PLUGIN AuditLoader;
```

すべての構成が正しく設定された後、再度 Audit Loader をインストールする手順に従うことができます。