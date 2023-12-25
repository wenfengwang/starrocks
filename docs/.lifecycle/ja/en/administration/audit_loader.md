---
displayed_sidebar: English
---

# StarRocks内での監査ログの管理について

このトピックでは、プラグインAudit Loaderを使用してStarRocks内のテーブルで監査ログを管理する方法について説明します。

StarRocksは内部データベースではなく、ローカルファイル**fe/log/fe.audit.log**に監査ログを保存します。Audit Loaderプラグインを使用すると、クラスタ内で直接監査ログを管理できます。Audit Loaderはファイルからログを読み込み、HTTP PUTを介してStarRocksにロードします。

## 監査ログを保存するためのテーブルを作成する

StarRocksクラスタにデータベースとテーブルを作成し、監査ログを保存します。詳細な手順については、[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)および[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

監査ログのフィールドはStarRocksのバージョンによって異なるため、StarRocksと互換性のあるテーブルを作成するには、以下の例から選択する必要があります。

> **注意**
>
> 例のテーブルスキーマを変更しないでください。変更するとログのロードが失敗する可能性があります。

- StarRocks v2.4、v2.5、v3.0、v3.1、およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `queryType`      VARCHAR(12)            COMMENT "クエリタイプ（query, slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `authorizedUser` VARCHAR(64)            COMMENT "ユーザー識別子",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `catalog`        VARCHAR(32)            COMMENT "カタログ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒）",
  `scanBytes`      BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト）",
  `scanRows`       BIGINT                 COMMENT "スキャンしたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト）",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0または1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "プランニングのCPUリソース消費時間（ナノ秒）",
  `planMemCosts`   DOUBLE                 COMMENT "プランニングのメモリコスト（バイト）"
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

- StarRocks v2.3.0以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始したユーザー",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態（EOF, ERR, OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒）",
  `scanBytes`      BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト）",
  `scanRows`       BIGINT                 COMMENT "スキャンしたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト）",
  `stmtId`         INT                    COMMENT "インクリメンタルSQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0または1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "プランニングのCPUリソース消費時間（ナノ秒）",
  `planMemCosts`   DOUBLE                 COMMENT "プランニングのメモリコスト（バイト）"
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

- StarRocks v2.2.1以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "ユニークなクエリID",
    time             DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
    client_ip        VARCHAR(32)            COMMENT "クライアントIPアドレス",
    user             VARCHAR(64)            COMMENT "クエリを開始したユーザー",
    db               VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
    state            VARCHAR(8)             COMMENT "クエリ状態（EOF, ERR, OK）",
    query_time       BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒）",
    scan_bytes       BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト）",
    scan_rows        BIGINT                 COMMENT "スキャンしたデータの行数",
    return_rows      BIGINT                 COMMENT "結果の行数",
    cpu_cost_ns      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
    mem_cost_bytes   BIGINT                 COMMENT "クエリのメモリコスト（バイト）",
    stmt_id          INT                    COMMENT "インクリメンタルSQLステートメントID",
    is_query         TINYINT                COMMENT "SQLがクエリかどうか（0または1）",
    frontend_ip      VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
    stmt             STRING                 COMMENT "SQLステートメント",
    digest           VARCHAR(32)            COMMENT "SQLフィンガープリント"
) ENGINE=OLAP
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

- StarRocks v2.2.0、v2.1.0以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始したユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリ状態（EOFE, RR, OK）",
    query_time      BIGINT                COMMENT "クエリのレイテンシ（ミリ秒）",
    scan_bytes      BIGINT                COMMENT "スキャンしたデータのサイズ（バイト）",
    scan_rows       BIGINT                COMMENT "スキャンしたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "インクリメンタルSQLステートメントID",
    is_query        TINYINT               COMMENT "SQLがクエリかどうか（0または1）",
    frontend_ip     VARCHAR(32)           COMMENT "SQLを実行するFEのIPアドレス",
    stmt            STRING                COMMENT "SQLステートメント",
    digest          VARCHAR(32)           COMMENT "SQLフィンガープリント"
) ENGINE=OLAP
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

    "replication_num" = "3"
);
```

- StarRocks v2.0.0 およびそれ以降のマイナーバージョン、StarRocks v1.19.0 およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始したユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態 (EOF, ERR, OK)",
    query_time      BIGINT                COMMENT "クエリのレイテンシー（ミリ秒）",
    scan_bytes      BIGINT                COMMENT "スキャンしたデータのサイズ（バイト）",
    scan_rows       BIGINT                COMMENT "スキャンしたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "インクリメンタルSQLステートメントID",
    is_query        TINYINT               COMMENT "SQLがクエリかどうか（0または1）",
    frontend_ip     VARCHAR(32)           COMMENT "SQLを実行するFEのIPアドレス",
    stmt            STRING                COMMENT "SQLステートメント"
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

`starrocks_audit_tbl__` は動的パーティションを使用して作成されます。デフォルトでは、テーブル作成後10分で最初の動的パーティションが作成されます。その後、監査ログをテーブルにロードすることができます。テーブルのパーティションを確認するには、以下のステートメントを使用します。

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

パーティションが作成されたら、次のステップに進むことができます。

## Audit Loader のダウンロードと設定

1. [ダウンロード](https://releases.starrocks.io/resources/AuditLoader.zip) Audit Loader のインストールパッケージ。このパッケージには、StarRocksのバージョンごとに複数のディレクトリが含まれています。対応するディレクトリに移動し、StarRocksと互換性のあるパッケージをインストールする必要があります。

    - **2.4**: StarRocks v2.4.0 以降のマイナーバージョン
    - **2.3**: StarRocks v2.3.0 以降のマイナーバージョン
    - **2.2.1+**: StarRocks v2.2.1 以降のマイナーバージョン
    - **2.1-2.2.0**: StarRocks v2.2.0、StarRocks v2.1.0 以降のマイナーバージョン
    - **1.18.2-2.0**: StarRocks v2.0.0 以降のマイナーバージョン、StarRocks v1.19.0 以降のマイナーバージョン

2. インストールパッケージを解凍します。

    ```shell
    unzip AuditLoader.zip
    ```

    次のファイルが展開されます：

    - **auditloader.jar**: Audit Loader の JAR ファイル。
    - **plugin.properties**: Audit Loader のプロパティファイル。
    - **plugin.conf**: Audit Loader の設定ファイル。

3. **plugin.conf** を編集して Audit Loader を設定します。Audit Loader が正常に動作するためには、以下の項目を設定する必要があります：

    - `frontend_host_port`: FEのIPアドレスとHTTPポート、形式は `<fe_ip>:<fe_http_port>`。デフォルト値は `127.0.0.1:8030` です。
    - `database`: 監査ログを保存するために作成したデータベースの名前。
    - `table`: 監査ログを保存するために作成したテーブルの名前。
    - `user`: クラスタのユーザー名。テーブルにデータをロードする権限（LOAD_PRIV）が必要です。
    - `password`: ユーザーのパスワード。

4. ファイルを再びパッケージに圧縮します。

    ```shell
    zip -q -m -r AuditLoader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. FEノードをホストするすべてのマシンにパッケージを配布します。すべてのパッケージが同一のパスに保存されていることを確認してください。そうでない場合、インストールは失敗します。パッケージを配布した後、パッケージの絶対パスをコピーしておいてください。

## Audit Loader のインストール

コピーしたパスを使用して、次のステートメントを実行し、StarRocks に Audit Loader をプラグインとしてインストールします。

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

詳細な手順については、[INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md) を参照してください。

## インストールの確認と監査ログのクエリ

1. [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) を使用して、インストールが成功したかどうかを確認できます。

    次の例では、プラグイン `AuditLoader` の `Status` が `INSTALLED` であり、インストールが成功したことを意味します。

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
    Description: load audit log to OLAP load, and user can view the statistic of queries
        Version: 1.0.1
    JavaVersion: 1.8.0
    ClassName: com.starrocks.plugin.audit.AuditLoaderPlugin
        SoName: NULL
        Sources: /x/xx/xxx/xxxxx/AuditLoader.zip
        Status: INSTALLED
    Properties: {}
    2 rows in set (0.01 sec)
    ```

2. いくつかのランダムなSQLを実行して監査ログを生成し、60秒間（またはAudit Loaderを設定する際に指定した `max_batch_interval_sec` の時間）待って、Audit Loaderが監査ログをStarRocksにロードするのを許可します。

3. テーブルをクエリして監査ログを確認します。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    次の例は、監査ログがテーブルに正常にロードされたことを示しています。

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

動的パーティションが作成され、プラグインがインストールされた後にテーブルに監査ログがロードされない場合は、**plugin.conf** が適切に設定されているかどうかを確認してください。それを変更するには、まずプラグインをアンインストールする必要があります。

```SQL
UNINSTALL PLUGIN AuditLoader;
```

すべての設定が正しく行われた後、上記のステップに従ってAudit Loaderを再度インストールすることができます。
