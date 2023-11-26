---
displayed_sidebar: "Japanese"
---

# Audit Loaderを使用してStarRocks内の監査ログを管理する

このトピックでは、Audit Loaderプラグインを介してテーブル内でStarRocksの監査ログを管理する方法について説明します。

StarRocksは、内部データベースではなく、ローカルファイル**fe/log/fe.audit.log**に監査ログを保存します。Audit Loaderプラグインを使用すると、クラスタ内で監査ログを直接管理することができます。Audit Loaderはログをファイルから読み取り、HTTP PUTを介してStarRocksにロードします。

## 監査ログを保存するためのテーブルを作成する

StarRocksクラスタ内のデータベースとテーブルを作成して、監査ログを保存します。詳しい手順については、[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)と[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

監査ログのフィールドは、StarRocksのバージョンによって異なるため、StarRocksと互換性のあるテーブルを作成するために、以下の例から選択する必要があります。

> **注意**
>
> 例のテーブルスキーマを変更しないでください。そうしないと、ログのロードが失敗します。

- StarRocks v2.4、v2.5、v3.0、v3.1、およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリの開始時刻",
  `queryType`      VARCHAR(12)            COMMENT "クエリのタイプ（query、slow_query）",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントのIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始するユーザー",
  `authorizedUser` VARCHAR(64)            COMMENT "user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `catalog`        VARCHAR(32)            COMMENT "カタログ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリの状態（EOF、ERR、OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンされたデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンされたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "増分SQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリであるかどうか（0と1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "計画のCPUリソース消費時間（ナノ秒単位）",
  `planMemCosts`   DOUBLE                 COMMENT "計画のメモリコスト（バイト単位）"
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
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリの開始時刻",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントのIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始するユーザー",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `db`             VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリの状態（EOF、ERR、OK）",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒単位）",
  `scanBytes`      BIGINT                 COMMENT "スキャンされたデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンされたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "増分SQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリであるかどうか（0と1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLのフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "計画のCPUリソース消費時間（ナノ秒単位）",
  `planMemCosts`   DOUBLE                 COMMENT "計画のメモリコスト（バイト単位）"
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
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "ユニークなクエリID",
    time             DATETIME     NOT NULL  COMMENT "クエリの開始時刻",
    client_ip        VARCHAR(32)            COMMENT "クライアントのIPアドレス",
    user             VARCHAR(64)            COMMENT "クエリを開始するユーザー",
    db               VARCHAR(96)            COMMENT "クエリがスキャンするデータベース",
    state            VARCHAR(8)             COMMENT "クエリの状態（EOF、ERR、OK）",
    query_time       BIGINT                 COMMENT "クエリのレイテンシ（ミリ秒単位）",
    scan_bytes       BIGINT                 COMMENT "スキャンされたデータのサイズ（バイト単位）",
    scan_rows        BIGINT                 COMMENT "スキャンされたデータの行数",
    return_rows      BIGINT                 COMMENT "結果の行数",
    cpu_cost_ns      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒単位）",
    mem_cost_bytes   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
    stmt_id          INT                    COMMENT "増分SQLステートメントID",
    is_query         TINYINT                COMMENT "SQLがクエリであるかどうか（0と1）",
    frontend_ip      VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
    stmt             STRING                 COMMENT "SQLステートメント",
    digest           VARCHAR(32)            COMMENT "SQLのフィンガープリント"
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

- StarRocks v2.2.0、v2.1.0およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリの開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントのIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始するユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態（EOF、ERR、OK）",
    query_time      BIGINT                COMMENT "クエリのレイテンシ（ミリ秒単位）",
    scan_bytes      BIGINT                COMMENT "スキャンされたデータのサイズ（バイト単位）",
    scan_rows       BIGINT                COMMENT "スキャンされたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "増分SQLステートメントID",
    is_query        TINYINT               COMMENT "SQLがクエリであるかどうか（0と1）",
    frontend_ip     VARCHAR(32)           COMMENT "SQLを実行するFEのIPアドレス",
    stmt            STRING                COMMENT "SQLステートメント",
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

- StarRocks v2.0.0およびそれ以降のマイナーバージョン、StarRocks v1.19.0およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリの開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントのIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始するユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態（EOF、ERR、OK）",
    query_time      BIGINT                COMMENT "クエリのレイテンシ（ミリ秒単位）",
    scan_bytes      BIGINT                COMMENT "スキャンされたデータのサイズ（バイト単位）",
    scan_rows       BIGINT                COMMENT "スキャンされたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "増分SQLステートメントID",
    is_query        TINYINT               COMMENT "SQLがクエリであるかどうか（0と1）",
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

`starrocks_audit_tbl__`は、動的パーティションを持つように作成されます。デフォルトでは、テーブルが作成されてから10分後に最初の動的パーティションが作成されます。その後、監査ログをテーブルにロードすることができます。次のステートメントを使用して、テーブルのパーティションを確認できます。

```SQL
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

パーティションが作成されたら、次のステップに進むことができます。

## Audit Loaderのダウンロードと設定

1. [Audit Loaderインストールパッケージ](https://releases.starrocks.io/resources/AuditLoader.zip)をダウンロードします。パッケージには、さまざまなStarRocksバージョン用の複数のディレクトリが含まれています。StarRocksと互換性のあるパッケージを対応するディレクトリに移動してインストールする必要があります。

    - **2.4**: StarRocks v2.4.0およびそれ以降のマイナーバージョン
    - **2.3**: StarRocks v2.3.0およびそれ以降のマイナーバージョン
    - **2.2.1+**: StarRocks v2.2.1およびそれ以降のマイナーバージョン
    - **2.1-2.2.0**: StarRocks v2.2.0、StarRocks v2.1.0およびそれ以降のマイナーバージョン
    - **1.18.2-2.0**: StarRocks v2.0.0およびそれ以降のマイナーバージョン、StarRocks v1.19.0およびそれ以降のマイナーバージョン

2. インストールパッケージを解凍します。

    ```shell
    unzip auditloader.zip
    ```

    次のファイルが展開されます。

    - **auditloader.jar**: Audit LoaderのJARファイルです。
    - **plugin.properties**: Audit Loaderのプロパティファイルです。
    - **plugin.conf**: Audit Loaderの設定ファイルです。

3. **plugin.conf**を編集して、Audit Loaderを設定します。Audit Loaderが正常に動作するために、次の項目を設定する必要があります。

    - `frontend_host_port`: FEのIPアドレスとHTTPポートです。フォーマットは`<fe_ip>:<fe_http_port>`です。デフォルト値は`127.0.0.1:8030`です。
    - `database`: 監査ログをホストするために作成したデータベースの名前です。
    - `table`: 監査ログをホストするために作成したテーブルの名前です。
    - `user`: クラスタのユーザー名です。テーブルにデータをロードするための権限（LOAD_PRIV）を持っている必要があります。
    - `password`: ユーザーのパスワードです。

4. ファイルをパッケージに戻します。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. パッケージをFEノードをホストするすべてのマシンに配布します。すべてのパッケージが同じパスに保存されていることを確認してください。そうしないと、インストールに失敗します。パッケージを配布した後、パッケージの絶対パスをコピーすることを忘れないでください。

## Audit Loaderをインストールする

次のステートメントを実行し、Audit LoaderをStarRocksのプラグインとしてインストールします。コピーしたパスとともに実行してください。

```SQL
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

詳しい手順については、[INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)を参照してください。

## インストールの確認と監査ログのクエリ

1. [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md)を使用して、インストールが成功したかどうかを確認できます。

    次の例では、プラグイン`AuditLoader`の`Status`が`INSTALLED`であるため、インストールが成功していることを示しています。

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

2. ランダムなSQLを実行して監査ログを生成し、Audit Loaderが監査ログをStarRocksにロードするために60秒（またはAudit Loaderの設定で指定した時間）待ちます。

3. テーブルをクエリして監査ログを確認します。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    次の例は、監査ログが正常にテーブルにロードされた場合の例です。

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

動的パーティションが作成され、プラグインがインストールされた後にテーブルに監査ログがロードされない場合は、**plugin.conf**が正しく設定されているかどうかを確認できます。設定を変更するには、まずプラグインをアンインストールする必要があります。

```SQL
UNINSTALL PLUGIN AuditLoader;
```

すべての設定が正しく設定されたら、Audit Loaderを再インストールするために上記の手順に従うことができます。
