---
displayed_sidebar: Chinese
---

# Audit Loader を使用して StarRocks の監査ログを管理する

このドキュメントでは、Audit Loader プラグインを使用して StarRocks 内部で監査ログを管理する方法について説明します。

StarRocks はすべての監査ログをローカルファイル **fe/log/fe.audit.log** に保存しており、システム内部のデータベースからはアクセスできません。Audit Loader プラグインを使用すると、クラスタ内で監査ログを管理できます。Audit Loader はローカルファイルからログを読み取り、HTTP PUT を介してログを StarRocks にインポートすることができます。

## 監査ログ用のデータベースとテーブルを作成する

StarRocks クラスタ内で監査ログ用のデータベースとテーブルを作成します。詳細な操作手順については [CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md) と [CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md) を参照してください。

StarRocks のバージョンごとに監査ログのフィールドが異なるため、対応するテーブル作成ステートメントも異なります。以下の例から、ご使用の StarRocks クラスタのバージョンに対応するテーブル作成ステートメントを選択する必要があります。

> **注意**
>
> 例に示されたテーブルの属性を変更しないでください。そうするとログのインポートに失敗する可能性があります。

- StarRocks v2.4、v2.5、v3.0、v3.1 およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "クエリのユニークID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `queryType`      VARCHAR(12)            COMMENT "クエリタイプ（query, slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIP",
  `user`           VARCHAR(64)            COMMENT "クエリユーザー名",
  `authorizedUser` VARCHAR(64)            COMMENT "ユーザーのユニーク識別子、つまりuser_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `catalog`        VARCHAR(32)            COMMENT "カタログ名",
  `db`             VARCHAR(96)            COMMENT "クエリが実行されたデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態（EOF，ERR，OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリ実行時間（ミリ秒）",
  `scanBytes`      BIGINT                 COMMENT "クエリがスキャンしたバイト数",
  `scanRows`       BIGINT                 COMMENT "クエリがスキャンしたレコード行数",
  `returnRows`     BIGINT                 COMMENT "クエリが返した結果行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPU使用時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリ消費量（バイト）",
  `stmtId`         INT                    COMMENT "SQLステートメントのインクリメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（1または0）",
  `feIp`           VARCHAR(32)            COMMENT "そのステートメントを実行したFE IP",
  `stmt`           STRING                 COMMENT "オリジナルSQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのダイジェスト",
  `planCpuCosts`   DOUBLE                 COMMENT "クエリ計画段階のCPU使用量（ナノ秒）",
  `planMemCosts`   DOUBLE                 COMMENT "クエリ計画段階のメモリ使用量（バイト）"
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
  "dynamic_partition.enable" = "true",
  "replication_num" = "3"
);
```

- StarRocks v2.3.0 およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "クエリのユニークID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIP",
  `user`           VARCHAR(64)            COMMENT "クエリユーザー名",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `db`             VARCHAR(96)            COMMENT "クエリが実行されたデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態（EOF，ERR，OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリ実行時間（ミリ秒）",
  `scanBytes`      BIGINT                 COMMENT "クエリがスキャンしたバイト数",
  `scanRows`       BIGINT                 COMMENT "クエリがスキャンしたレコード行数",
  `returnRows`     BIGINT                 COMMENT "クエリが返した結果行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPU使用時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリ消費量（バイト）",
  `stmtId`         INT                    COMMENT "SQLステートメントのインクリメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（1または0）",
  `feIp`           VARCHAR(32)            COMMENT "そのステートメントを実行したFE IP",
  `stmt`           STRING                 COMMENT "オリジナルSQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのダイジェスト",
  `planCpuCosts`   DOUBLE                 COMMENT "クエリ計画段階のCPU使用量（ナノ秒）",
  `planMemCosts`   DOUBLE                 COMMENT "クエリ計画段階のメモリ使用量（バイト）"
) ENGINE = OLAP
DUPLICATE KEY (`queryId`, `timestamp`, `clientIp`)
COMMENT "監査ログテーブル"
PARTITION BY RANGE (`timestamp`) ()
DISTRIBUTED BY HASH (`queryId`)
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

- StarRocks v2.2.1 およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "クエリのユニークID",
    time             DATETIME     NOT NULL  COMMENT "クエリ開始時刻",
    client_ip        VARCHAR(32)            COMMENT "クライアントIP",
    user             VARCHAR(64)            COMMENT "クエリユーザー名",
    db               VARCHAR(96)            COMMENT "クエリが実行されたデータベース",
    state            VARCHAR(8)             COMMENT "クエリ状態（EOF，ERR，OK）",
    query_time       BIGINT                 COMMENT "クエリ実行時間（ミリ秒）",
    scan_bytes       BIGINT                 COMMENT "クエリがスキャンしたバイト数",
    scan_rows        BIGINT                 COMMENT "クエリがスキャンしたレコード行数",
    return_rows      BIGINT                 COMMENT "クエリが返した結果行数",
    cpu_cost_ns      BIGINT                 COMMENT "クエリのCPU使用時間（ナノ秒）",
    mem_cost_bytes   BIGINT                 COMMENT "クエリのメモリ消費量（バイト）",
    stmt_id          INT                    COMMENT "SQLステートメントのインクリメントID",
    is_query         TINYINT                COMMENT "SQLがクエリかどうか（1または0）",
    frontend_ip      VARCHAR(32)            COMMENT "そのステートメントを実行したFE IP",
    stmt             STRING                 COMMENT "オリジナルSQLステートメント",
    digest           VARCHAR(32)            COMMENT "SQLのダイジェスト"
)   ENGINE = OLAP
DUPLICATE KEY (query_id, time, client_ip)
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

- StarRocks v2.2.0、v2.1.0 およびそれ以降のマイナーバージョン:

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "クエリのユニークID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントIP",
    user            VARCHAR(64)           COMMENT "クエリユーザー名",
    db              VARCHAR(96)           COMMENT "クエリが実行されたデータベース",
    state           VARCHAR(8)            COMMENT "クエリ状態（EOF，ERR，OK）",
    query_time      BIGINT                COMMENT "クエリ実行時間（ミリ秒）",

    scan_bytes      BIGINT                COMMENT "クエリがスキャンしたバイト数",
    scan_rows       BIGINT                COMMENT "クエリがスキャンしたレコード行数",
    return_rows     BIGINT                COMMENT "クエリが返した結果行数",
    stmt_id         INT                   COMMENT "SQLステートメントのインクリメントID",
    is_query        TINYINT               COMMENT "SQLがクエリかどうか（1または0）",
    frontend_ip     VARCHAR(32)           COMMENT "このステートメントを実行したFEのIP",
    stmt            STRING                COMMENT "元のSQLステートメント",
    digest          VARCHAR(32)           COMMENT "SQLのフィンガープリント"
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

- StarRocks v2.0.0 以降のマイナーバージョン、v1.19.0 以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "クエリの一意なID",
    time            DATETIME    NOT NULL  COMMENT "クエリの開始時間",
    client_ip       VARCHAR(32)           COMMENT "クライアントのIP",
    user            VARCHAR(64)           COMMENT "クエリユーザー名",
    db              VARCHAR(96)           COMMENT "クエリが存在するデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態（EOF、ERR、OK）",
    query_time      BIGINT                COMMENT "クエリの実行時間（ミリ秒）",
    scan_bytes      BIGINT                COMMENT "クエリがスキャンしたバイト数",
    scan_rows       BIGINT                COMMENT "クエリがスキャンしたレコード行数",
    return_rows     BIGINT                COMMENT "クエリが返した結果行数",
    stmt_id         INT                   COMMENT "SQLステートメントのインクリメントID",
    is_query        TINYINT               COMMENT "SQLがクエリかどうか（1または0）",
    frontend_ip     VARCHAR(32)           COMMENT "このステートメントを実行したFEのIP",
    stmt            STRING                COMMENT "元のSQLステートメント"
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

`starrocks_audit_tbl__` テーブルは動的パーティションテーブルです。デフォルトでは、最初の動的パーティションはテーブル作成後10分で作成されます。パーティションが作成された後、監査ログをテーブルにインポートできます。以下のステートメントを使用して、テーブルのパーティションが作成されたかどうかを確認できます：

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

パーティションの作成が完了したら、次のステップに進むことができます。

## Audit Loader のダウンロードと設定

1. [ダウンロード](https://releases.mirrorship.cn/resources/AuditLoader.zip) Audit Loader インストールパッケージ。StarRocks の異なるバージョンによって、このインストールパッケージには異なるパスが含まれています。StarRocks のバージョンと互換性のある Audit Loader インストールパッケージをインストールする必要があります。

    - **2.4**：StarRocks v2.4.0 以降のマイナーバージョンに対応するインストールパッケージ
    - **2.3**：StarRocks v2.3.0 以降のマイナーバージョンに対応するインストールパッケージ
    - **2.2.1+**：StarRocks v2.2.1 以降のマイナーバージョンに対応するインストールパッケージ
    - **2.1-2.2.0**：StarRocks v2.2.0、StarRocks v2.1.0 以降のマイナーバージョンに対応するインストールパッケージ
    - **1.18.2-2.0**：StarRocks v2.0.0 以降のマイナーバージョン、v1.19.0 以降のマイナーバージョンに対応するインストールパッケージ

2. インストールパッケージを解凍します。

    ```shell
    unzip auditloader.zip
    ```

    解凍すると以下のファイルが生成されます：

    - **auditloader.jar**：プラグインのコアコードパッケージ。
    - **plugin.properties**：プラグインのプロパティファイル。
    - **plugin.conf**：プラグインの設定ファイル。

3. **plugin.conf** ファイルを編集して Audit Loader を設定します。Audit Loader が正常に動作するためには、以下の項目を設定する必要があります：

    - `frontend_host_port`：FEノードのIPアドレスとHTTPポート。形式は `<fe_ip>:<fe_http_port>` です。デフォルト値は `127.0.0.1:8030` です。
    - `database`：監査ログのデータベース名。
    - `table`：監査ログのテーブル名。
    - `user`：クラスタのユーザー名。このユーザーは対応するテーブルのINSERT権限を持っている必要があります。
    - `password`：クラスタのユーザーパスワード。

4. 上記のファイルを再パッケージ化します。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. 圧縮パッケージをすべてのFEノードが実行されているマシンに配布します。すべての圧縮パッケージが同じパスに保存されていることを確認してください。そうでないとプラグインのインストールに失敗します。配布が完了したら、圧縮パッケージの絶対パスをコピーしてください。

## Audit Loader のインストール

以下のステートメントで Audit Loader プラグインをインストールします：

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

詳細な操作説明は [INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md) を参照してください。

## インストールの確認と監査ログのクエリ

1. [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) ステートメントを使用して、プラグインが正常にインストールされたかどうかを確認できます。

    以下の例では、プラグイン `AuditLoader` の `Status` が `INSTALLED` であり、インストールが成功したことを意味します。

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

2. ランダムにSQLステートメントを実行して監査ログを生成し、Audit Loader が監査ログをStarRocksにバッチインポートするために60秒（または `max_batch_interval_sec` 項目で設定した時間）待ちます。

3. 監査ログテーブルをクエリします。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    以下の例は、監査ログが正常にインポートされた状況を示しています：

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


動的なパーティションの作成とプラグインのインストールが成功した後も、長時間にわたって監査ログがテーブルにインポートされない場合は、**plugin.conf** ファイルが正しく設定されているかを確認する必要があります。設定ファイルを変更する必要がある場合は、まずプラグインをアンインストールする必要があります：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

すべての設定が正しく設定された後、上記の手順に従って Audit Loader を再インストールすることができます。
