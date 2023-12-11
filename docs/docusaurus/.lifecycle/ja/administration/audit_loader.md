---
displayed_sidebar: "Japanese"
---

# 監査ローダーを介してStarRocksで監査ログを管理する

このトピックでは、プラグインである監査ローダーを介して、StarRocks内の監査ログをテーブル内で管理する方法について説明します。

StarRocksは、監査ログを内部データベースではなく、ローカルファイル**fe/log/fe.audit.log**に保存します。監査ローダープラグインを使用すると、クラスタ内で監査ログを直接管理できます。監査ローダーは、ファイルからログを読み取り、HTTP PUTを介してStarRocksにロードします。

## 監査ログを格納するためのテーブルを作成する

StarRocksクラスタにデータベースとテーブルを作成し、その監査ログを格納します。詳しい手順については、[CREATE DATABASE](../sql-reference/sql-statements/data-definition/CREATE_DATABASE.md)と[CREATE TABLE](../sql-reference/sql-statements/data-definition/CREATE_TABLE.md)を参照してください。

監査ログのフィールドは、異なるStarRocksバージョン間で異なるため、StarRocksと互換性のあるテーブルを作成するために、以下の例から選択する必要があります。

> **注意**
>
> 例でテーブルスキーマを変更しないでください。そうしないと、ログのロードに失敗します。

- StarRocks v2.4、v2.5、v3.0、v3.1、およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;

CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時間",
  `queryType`      VARCHAR(12)            COMMENT "クエリタイプ (クエリ、slow_query)",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始するユーザー",
  `authorizedUser` VARCHAR(64)            COMMENT "user_identity",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `catalog`        VARCHAR(32)            COMMENT "カタログ名",
  `db`             VARCHAR(96)            COMMENT "クエリをスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態 (EOF、ERR、OK)",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリの遅延時間(ミリ秒)",
  `scanBytes`      BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンしたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "増分SQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0と1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLフィンガープリント",
  `planCpuCosts`   DOUBLE                 COMMENT "プランニングのCPUリソース消費時間（ナノ秒）",
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

- StarRocks v2.3.0 およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__ (
  `queryId`        VARCHAR(48)            COMMENT "ユニークなクエリID",
  `timestamp`      DATETIME     NOT NULL  COMMENT "クエリ開始時間",
  `clientIp`       VARCHAR(32)            COMMENT "クライアントIPアドレス",
  `user`           VARCHAR(64)            COMMENT "クエリを開始するユーザー",
  `resourceGroup`  VARCHAR(64)            COMMENT "リソースグループ名",
  `db`             VARCHAR(96)            COMMENT "クエリをスキャンするデータベース",
  `state`          VARCHAR(8)             COMMENT "クエリ状態 (EOF、ERR、OK)",
  `errorCode`      VARCHAR(96)            COMMENT "エラーコード",
  `queryTime`      BIGINT                 COMMENT "クエリの遅延時間（ミリ秒）",
  `scanBytes`      BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト単位）",
  `scanRows`       BIGINT                 COMMENT "スキャンしたデータの行数",
  `returnRows`     BIGINT                 COMMENT "結果の行数",
  `cpuCostNs`      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
  `memCostBytes`   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
  `stmtId`         INT                    COMMENT "増分SQLステートメントID",
  `isQuery`        TINYINT                COMMENT "SQLがクエリかどうか（0と1）",
  `feIp`           VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
  `stmt`           STRING                 COMMENT "SQLステートメント",
  `digest`         VARCHAR(32)            COMMENT "SQLフィンガープリント"
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

- StarRocks v2.2.1 およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id         VARCHAR(48)            COMMENT "ユニークなクエリID",
    time             DATETIME     NOT NULL  COMMENT "クエリ開始時間",
    client_ip        VARCHAR(32)            COMMENT "クライアントIPアドレス",
    user             VARCHAR(64)            COMMENT "クエリを開始するユーザー",
    db               VARCHAR(96)            COMMENT "クエリをスキャンするデータベース",
    state            VARCHAR(8)             COMMENT "クエリ状態 (EOF、ERR、OK)",
    query_time       BIGINT                 COMMENT "クエリの遅延時間（ミリ秒）",
    scan_bytes       BIGINT                 COMMENT "スキャンしたデータのサイズ（バイト単位）",
    scan_rows        BIGINT                 COMMENT "スキャンしたデータの行数",
    return_rows      BIGINT                 COMMENT "結果の行数",
    cpu_cost_ns      BIGINT                 COMMENT "クエリのCPUリソース消費時間（ナノ秒）",
    mem_cost_bytes   BIGINT                 COMMENT "クエリのメモリコスト（バイト単位）",
    stmt_id          INT                    COMMENT "増分SQLステートメントID",
    is_query         TINYINT                COMMENT "SQLがクエリかどうか（0と1）",
    frontend_ip      VARCHAR(32)            COMMENT "SQLを実行するFEのIPアドレス",
    stmt             STRING                 COMMENT "SQLステートメント",
    digest           VARCHAR(32)            COMMENT "SQLフィンガープリント"
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

- StarRocks v2.2.0、v2.1.0 およびそれ以降のマイナーバージョン：

```SQL
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時間",
    client_ip       VARCHAR(32)           COMMENT "クライアントIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始するユーザー",
```sql
CREATE DATABASE starrocks_audit_db__;
CREATE TABLE starrocks_audit_db__.starrocks_audit_tbl__
(
    query_id        VARCHAR(48)           COMMENT "ユニークなクエリID",
    time            DATETIME    NOT NULL  COMMENT "クエリ開始時刻",
    client_ip       VARCHAR(32)           COMMENT "クライアントIPアドレス",
    user            VARCHAR(64)           COMMENT "クエリを開始したユーザー",
    db              VARCHAR(96)           COMMENT "クエリがスキャンするデータベース",
    state           VARCHAR(8)            COMMENT "クエリの状態 (EOF, ERR, OK)",
    query_time      BIGINT                COMMENT "クエリの遅延時間（ミリ秒）",
    scan_bytes      BIGINT                COMMENT "スキャンされたデータのサイズ（バイト）",
    scan_rows       BIGINT                COMMENT "スキャンされたデータの行数",
    return_rows     BIGINT                COMMENT "結果の行数",
    stmt_id         INT                   COMMENT "インクリメンタルSQLステートメントID",
    is_query        TINYINT               COMMENT "SQLがクエリかどうか (0 および 1)",
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

starrocks_audit_tbl__ は動的パーティションで作成されます。デフォルトでは、テーブル作成後に最初の動的パーティションが10分後に作成されます。監査ログはその後、テーブルにロードできます。次のステートメントを使用して、テーブルのパーティションを確認できます：

```sql
SHOW PARTITIONS FROM starrocks_audit_db__.starrocks_audit_tbl__;
```

パーティションが作成されたら、次のステップに進むことができます。

## 監査ローダーのダウンロードと構成

1. [Audit Loader](https://releases.starrocks.io/resources/AuditLoader.zip) インストールパッケージをダウンロードします。パッケージには、異なるStarRocksバージョン用の複数のディレクトリが含まれています。対応するディレクトリに移動し、StarRocksと互換性のあるパッケージをインストールする必要があります。

    - **2.4**: StarRocks v2.4.0 およびそれ以降のマイナーバージョン
    - **2.3**: StarRocks v2.3.0 およびそれ以降のマイナーバージョン
    - **2.2.1+**: StarRocks v2.2.1 およびそれ以降のマイナーバージョン
    - **2.1-2.2.0**: StarRocks v2.2.0、StarRocks v2.1.0 およびそれ以降のマイナーバージョン
    - **1.18.2-2.0**: StarRocks v2.0.0 およびそれ以降のマイナーバージョン、StarRocks v1.19.0 およびそれ以降のマイナーバージョン

2. インストールパッケージを解凍します。

    ```shell
    unzip auditloader.zip
    ```

    以下のファイルが展開されます：

    - **auditloader.jar**: Audit LoaderのJARファイル。
    - **plugin.properties**: Audit Loaderのプロパティファイル。
    - **plugin.conf**: Audit Loaderの構成ファイル。

3. **plugin.conf** を修正して、Audit Loaderを構成する必要があります。Audit Loaderが正しく動作するようにするには、以下の項目を構成する必要があります：

    - `frontend_host_port`: FEのIPアドレスとHTTPポート。形式は `<fe_ip>:<fe_http_port>` です。デフォルト値は `127.0.0.1:8030` です。
    - `database`: 監査ログをホストするデータベースの名前。
    - `table`: 監査ログをホストするテーブルの名前。
    - `user`: クラスターユーザー名。テーブルにデータをロードする権限（LOAD_PRIV）を持っている必要があります。
    - `password`: ユーザーパスワード。

4. ファイルをパッケージに再度圧縮します。

    ```shell
    zip -q -m -r auditloader.zip auditloader.jar plugin.conf plugin.properties
    ```

5. パッケージを全てのFEノードをホストするすべてのマシンに配布します。インストールに失敗しないように、すべてのパッケージを同一のパスに保存する必要があります。パッケージを配布した後に、パッケージの絶対パスをコピーすることをお忘れなく。

## 監査ローダーのインストール

以下のステートメントを実行し、Audit LoaderをStarRocksのプラグインとしてインストールします：

```sql
INSTALL PLUGIN FROM "<absolute_path_to_package>";
```

詳細な手順については[INSTALL PLUGIN](../sql-reference/sql-statements/Administration/INSTALL_PLUGIN.md)を参照してください。

## インストールの確認とクエリの監査ログ

1. [SHOW PLUGINS](../sql-reference/sql-statements/Administration/SHOW_PLUGINS.md) を使用して、インストールが成功したかどうかを確認できます。

    次の例では、プラグイン `AuditLoader` の `Status` が `INSTALLED` であり、インストールが成功していることが示されています。

    ```Plain
    mysql> SHOW PLUGINS\G
    *************************** 1. row ***************************
        Name: __builtin_AuditLogBuilder
        Type: AUDIT
        Description: ビルトイン監査ログビルダー
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
        Description: 監査ログをOLAPロードにロードし、ユーザーはクエリの統計を表示できます
        Version: 1.0.1
        JavaVersion: 1.8.0
        ClassName: com.starrocks.plugin.audit.AuditLoaderPlugin
        SoName: NULL
        Sources: /x/xx/xxx/xxxxx/auditloader.zip
        Status: INSTALLED
        Properties: {}
    2 rows in set (0.01 sec)
    ```

2. いくつかのランダムなSQLを実行して監査ログを生成し、[max_batch_interval_sec] を設定した場合は60秒（または構成した時間）待って、Audit Loaderが監査ログをStarRocksにロードするのを許可します。

3. テーブルをクエリして監査ログを確認できます。

    ```SQL
    SELECT * FROM starrocks_audit_db__.starrocks_audit_tbl__;
    ```

    以下の例は、監査ログがテーブルに正常にロードされたことを示しています：

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

動的パーティションが作成され、プラグインがインストールされた後でも、監査ログがテーブルにロードされない場合は、**plugin.conf** が正しく構成されているかどうかを確認できます。修正するには、まずプラグインをアンインストールする必要があります：

```SQL
UNINSTALL PLUGIN AuditLoader;
```

すべての構成が正しく設定されたら、上記の手順に従って監査ローダーを再びインストールできます。