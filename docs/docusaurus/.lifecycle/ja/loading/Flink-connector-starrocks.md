```yaml
---
displayed_sidebar: "Japanese"
---
# Apache Flink®からデータを連続して読み込む

StarRocksでは、Apache Flink®（以下、Flinkコネクタ）のための自社開発コネクタであるFlinkコネクタを使用して、データを蓄積し、それをすべて[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を介してStarRocksにロードすることができます。FlinkコネクタはDataStream API、Table API & SQL、Python APIをサポートしており、Apache Flink®が提供する[flink-connector-jdbc](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/table/jdbc/)よりも高い安定性とパフォーマンスがあります。

> **注意**
>
> Flinkコネクタを使用してStarRocksテーブルにデータをロードするには、SELECTおよびINSERTの権限が必要です。これらの権限がない場合は、[GRANT](../sql-reference/sql-statements/account-management/GRANT.md)に記載されている手順に従い、StarRocksクラスタに接続するために使用するユーザーにこれらの権限を付与してください。

## バージョン要件

| コネクタ | Flink                    | StarRocks     | Java | Scala     |
|-----------|--------------------------|---------------| ---- |-----------|
| 1.2.8     | 1.13,1.14,1.15,1.16,1.17 | 2.1 以降     | 8    | 2.11,2.12 |
| 1.2.7     | 1.11,1.12,1.13,1.14,1.15 | 2.1 以降     | 8    | 2.11,2.12 |

## Flinkコネクタの取得

FlinkコネクタのJARファイルを以下の方法で取得できます。

- コンパイルされたFlinkコネクタのJARファイルを直接ダウンロードします。
- FlinkコネクタをMavenプロジェクトの依存関係として追加し、その後JARファイルをダウンロードします。
- Flinkコネクタのソースコードを自分でコンパイルしてJARファイルを作成します。

FlinkコネクタのJARファイルの命名形式は以下の通りです。

- Flink 1.15以降では、`flink-connector-starrocks-${connector_version}_flink-${flink_version}.jar` です。例えば、Flink 1.15をインストールし、Flinkコネクタ1.2.7を使用したい場合は`flink-connector-starrocks-1.2.7_flink-1.15.jar`を使用できます。

- Flink 1.15より前では、`flink-connector-starrocks-${connector_version}_flink-${flink_version}_${scala_version}.jar` です。例えば、環境にFlink 1.14とScala 2.12をインストールしており、Flinkコネクタ1.2.7を使用したい場合は`flink-connector-starrocks-1.2.7_flink-1.14_2.12.jar`を使用できます。

> **注意**
>
> 一般的に、Flinkコネクタの最新バージョンは、Flinkの最新3バージョンとの互換性を維持します。

### コンパイルされたJarファイルをダウンロード

[Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks) から対応するFlinkコネクタのJarファイルを直接ダウンロードしてください。

### Mavenの依存関係

Mavenプロジェクトの`pom.xml`ファイルに、以下の形式に従ってFlinkコネクタを依存関係として追加してください。`flink_version`、`scala_version`、`connector_version`をそれぞれのバージョンに置き換えてください。

- Flink 1.15以降

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}</version>
    </dependency>
    ```

- Flink 1.15より前のバージョン

    ```xml
    <dependency>
        <groupId>com.starrocks</groupId>
        <artifactId>flink-connector-starrocks</artifactId>
        <version>${connector_version}_flink-${flink_version}_${scala_version}</version>
    </dependency>
    ```

### 自分でコンパイル

1. [Flinkコネクタパッケージ](https://github.com/StarRocks/starrocks-connector-for-apache-flink)をダウンロードします。
2. 次のコマンドを実行して、FlinkコネクタのソースコードをJARファイルにコンパイルします。`flink_version`は対応するFlinkバージョンに置き換えることに注意してください。

      ```bash
      sh build.sh <flink_version>
      ```

   例えば、環境のFlinkバージョンが1.15である場合は、次のコマンドを実行する必要があります。

      ```bash
      sh build.sh 1.15
      ```

3. `target/`ディレクトリに移動し、コンパイル後に生成された`flink-connector-starrocks-1.2.7_flink-1.15-SNAPSHOT.jar`などのFlinkコネクタのJARファイルを見つけてください。

> **注意**
>
> 正式にリリースされていないFlinkコネクタの名前には、`SNAPSHOT`接尾辞が含まれます。

## オプション

| **Option**                        | **Required** | **Default value** | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|-----------------------------------|--------------|-------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| connector                         | はい          | NONE              | 使用するコネクタ。値は「starrocks」である必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| jdbc-url                          | はい          | NONE              | FEのMySQLサーバに接続するために使用されるアドレスです。複数のアドレスを指定することができ、コンマ（,）で区切られている必要があります。形式：`jdbc:mysql://<fe_host1>:<fe_query_port1>,<fe_host2>:<fe_query_port2>,<fe_host3>:<fe_query_port3>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| load-url                          | はい          | NONE              | StarRocksクラスタ内のFEのHTTP URLです。複数のURLを指定することができ、セミコロン（;）で区切られている必要があります。形式：`<fe_host1>:<fe_http_port1>;<fe_host2>:<fe_http_port2>`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| database-name                     | はい          | NONE              | データをロードするStarRocksデータベースの名前です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| table-name                        | はい          | NONE              | StarRocksにデータをロードするテーブルの名前です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| username                          | はい          | NONE              | StarRocksにデータをロードするために使用するアカウントのユーザー名。アカウントには[SELECTおよびINSERTの権限](../sql-reference/sql-statements/account-management/GRANT.md)が必要です。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| password                          | はい          | NONE              | 前述のアカウントのパスワード。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.semantic                     | いいえ           | at-least-once     | シンクによって保証されるセマンティック。有効な値：**at-least-once**および**exactly-once**。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.version                      | いいえ           | AUTO              | データをロードするために使用されるインタフェース。このパラメータはFlinkコネクタのバージョン1.2.4以降でサポートされています。<ul><li>`V1`: [Stream Load](../loading/StreamLoad.md) インタフェースを使用してデータをロードします。1.2.4より前のコネクタはこのモードのみをサポートしています。 </li> <li>`V2`: [Stream Load transaction](../loading/Stream_Load_transaction_interface.md) インタフェースを使用してデータをロードします。StarRocksがバージョン2.4以上である必要があります。メモリ使用量を最適化し、より安定したexactly-onceの実装を提供しますので、`V2`を推奨します。 </li> <li>`AUTO`: StarRocksのバージョンがトランザクションStream Loadをサポートしている場合は自動的に`V2`を選択し、それ以外の場合は`V1`を選択します </li></ul> |
| sink.label-prefix | いいえ | NONE | Stream Loadで使用されるラベルの接頭辞。コネクタ1.2.8以降をexactly-onceで使用する場合には、この設定を推奨します。 [exactly-onceの使用に関する注意事項](#exactly-once)をご覧ください。 |
| sink.buffer-flush.max-bytes       | いいえ           | 94371840(90M)     | StarRocksに送信される前にメモリに蓄積されるデータの最大サイズ。最大値は64 MBから10 GBまでです。このパラメータをより大きな値に設定すると、ロードのパフォーマンスが向上するが、ロードの遅延が増加する可能性があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| sink.buffer-flush.max-rows        | いいえ           | 500000            | 一度にStarRocksに送信される前にメモリに蓄積される行の最大数。このパラメータは、`sink.version`を`V1`に、`sink.semantic`を`at-least-once`に設定した場合にのみ使用できます。有効な値：64000から5000000。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.buffer-flush.interval-ms     | いいえ           | 300000            | データをフラッシュする間隔。このパラメータは、`sink.semantic`を`at-least-once`に設定した場合にのみ使用できます。有効な値: 1000 ～ 3600000。単位: ミリ秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| sink.max-retries                  | いいえ           | 3                 | システムがストリームロードジョブを実行し直す回数。このパラメータは、`sink.version`を`V1`に設定した場合にのみ使用できます。有効な値: 0 ～ 10。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| sink.connect.timeout-ms           | いいえ           | 1000              | HTTP接続を確立するためのタイムアウト。有効な値: 100 ～ 60000。単位: ミリ秒。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| sink.wait-for-continue.timeout-ms | いいえ           | 10000             | 1.2.7以降でサポートされています。FEからHTTP 100-continueのレスポンスを待機するためのタイムアウト。有効な値: `3000` ～ `60000`。単位: ミリ秒                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.ignore.update-before         | いいえ           | true              | 1.2.8以降でサポートされています。主キーテーブルにデータをロードする際に、Flinkからの`UPDATE_BEFORE`レコードを無視するかどうか。このパラメータがfalseに設定されている場合、レコードはStarRocksテーブルに対する削除操作として扱われます。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.*                 | いいえ           | NONE              | ストリームロードの動作を制御するために使用されるパラメータ。たとえば、パラメータ`sink.properties.format`は、ストリームロードに使用する形式（CSVまたはJSONなど）を指定します。サポートされているパラメータとその説明のリストについては、[STREAM LOAD](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。                                                                                                                                                                                                                                                                                                                                 |
| sink.properties.format            | いいえ           | csv               | ストリームロードに使用する形式。Flinkコネクタは、データの各バッチをStarRocksに送信する前にその形式に変換します。有効な値: `csv` および `json`。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.row_delimiter     | いいえ           | \n                | CSV形式のデータの行区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| sink.properties.column_separator  | いいえ           | \t                | CSV形式のデータの列区切り文字。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| sink.properties.max_filter_ratio  | いいえ           | 0                 | ストリームロードの最大エラートレランス。不適切なデータ品質のためにフィルタリングされるデータレコードの最大パーセンテージ。有効な値: `0` ～ `1`。デフォルト値: `0`。詳細については[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。                                                                                                                                                                                                                                                                                                                                                                      |
| sink.parallelism                  | いいえ           | NONE              | コネクタの並列性。Flink SQL専用の機能です。設定しない場合、Flinkプランナーが並列性を決定します。複数の並列性のシナリオでは、データが正しい順序で書き込まれることを保証する必要があります。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| sink.properties.strict_mode | いいえ | false | ストリームロードの厳密モードを有効にするかどうかを指定します。不適格な行（一貫性のある列値など）がある場合のロード動作に影響します。有効な値: `true` および `false`。デフォルト値: `false`。詳細については[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を参照してください。 |

## FlinkとStarRocks間のデータ型マッピング

| Flinkのデータ型                   | StarRocksのデータ型   |
|-----------------------------------|-----------------------|
| BOOLEAN                           | BOOLEAN               |
| TINYINT                           | TINYINT               |
| SMALLINT                          | SMALLINT              |
| INTEGER                           | INTEGER               |
| BIGINT                            | BIGINT                |
| FLOAT                             | FLOAT                 |
| DOUBLE                            | DOUBLE                |
| DECIMAL                           | DECIMAL               |
| BINARY                            | INT                   |
| CHAR                              | STRING                |
| VARCHAR                           | STRING                |
| STRING                            | STRING                |
| DATE                              | DATE                  |
| TIMESTAMP_WITHOUT_TIME_ZONE(N)    | DATETIME              |
| TIMESTAMP_WITH_LOCAL_TIME_ZONE(N) | DATETIME              |
| ARRAY&lt;T&gt;                        | ARRAY&lt;T&gt;              |
| MAP&lt;KT,VT&gt;                        | JSON STRING           |
| ROW&lt;arg T...&gt;                     | JSON STRING           |

## 使用上の注意

### 正確一度

- シンクが正確に一度のセマンティクスを保証する場合は、StarRocksを2.5以上にアップグレードし、Flinkコネクタを1.2.4以上にアップグレードすることをお勧めします。
  - Flinkコネクタ1.2.4以降は、StarRocksが2.4以降で提供する[Stream Loadトランザクションインターフェース](https://docs.starrocks.io/en-us/latest/loading/Stream_Load_transaction_interface)に基づいて正確一度を再設計しました。非トランザクショナルインターフェースに基づく以前の実装と比較して、新しい実装はメモリ使用量とチェックポイントのオーバーヘッドを削減し、リアルタイム性能と安定性を向上させています。

  - StarRocksのバージョンが2.4未満であるか、Flinkコネクタのバージョンが1.2.4未満である場合、シンクは自動的に非トランザクショナルなインターフェースに基づく実装を選択します。

- 正確一度を保証するための設定

  - `sink.semantic`の値を`exactly-once`にする必要があります。

  - Flinkコネクタのバージョンが1.2.8以上の場合、`sink.label-prefix`の値を指定することをお勧めします。ラベルプレフィックスはStarRocksでのすべてのタイプのロード（Flinkジョブ、ルーチンロード、ブローカーロードなど）の間で一意である必要があります。

    - ラベルプレフィックスが指定されている場合、Flinkコネクタは、Flinkジョブのチェックポイントが進行中で失敗した場合などのいくつかのFlinkの障害シナリオで生成される残留トランザクションをクリーンアップするためにラベルプレフィックスを使用します。これらの残留トランザクションは、通常`PREPARED`の状態にあります。Flinkジョブがチェックポイントから復元されると、Flinkコネクタはこれらの残留トランザクションをラベルプレフィックスとチェックポイント内のいくつかの情報に基づいて見つけ、取り消します。 Flinkジョブが正常に終了した後は、Flinkコネクタは2相コミットメカニズムを介して正確に一度を実現するため、これらのトランザクションを中止することができません。Flinkジョブが終了すると、Flinkコネクタはまだトランザクションコーディネーターから成功したチェックポイントに含めるべきトランザクションかどうかの通知を受け取っていない可能性があり、これらのトランザクションを中止してもデータの損失につながる可能性があります。Flinkでエンドツーエンドの正確一度を実現する方法について詳細は、この[ブログポスト](https://flink.apache.org/2018/02/28/an-overview-of-end-to-end-exactly-once-processing-in-apache-flink-with-apache-kafka-too/)を参照してください。

    - ラベルプレフィックスが指定されていない場合、残留トランザクションはStarRocksによってタイムアウト後にクリーンアップされます。ただし、Flinkジョブが頻繁に失敗する場合、実行中のトランザクション数がStarRocksの`max_running_txn_num_per_db`の制限に達する可能性があります。タイムアウトの長さは、StarRocks FEの構成`prepared_transaction_default_timeout_second`によって制御され、デフォルト値は`86400`（1日）です。ラベルプレフィックスが指定されていない場合にトランザクションをより速く期限切れにするためにこの設定に小さい値を設定できます。

- 長時間のダウンタイムの後、Flinkジョブが最終的にチェックポイントまたはセーブポイントから回復することを確信している場合は、以下のStarRocks構成を調整して、データ損失を回避してください。

  - `prepared_transaction_default_timeout_second`：StarRocks FEの構成で、デフォルト値は`86400`です。この構成の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうしないと、実際のチェックポイントに含まれる残留トランザクションが再起動する前にタイムアウトによって中止される可能性があり、これによりデータが失われることになります。

    この構成に大きな値を設定する場合は、ラベルプレフィックスの値を指定することをお勧めします。なぜなら、残留トランザクションについての情報を検索し、それらを中止するために、StarRocksはチェックポイント内のラベルプレフィックスといくつかの情報を使用します。

  - `label_keep_max_second`および`label_keep_max_num`：StarRocks FEの構成で、デフォルト値はそれぞれ`259200`および`1000`です。詳細については、[FE configurations](../loading/Loading_intro.md#fe-configurations)を参照してください。`label_keep_max_second`の値は、Flinkジョブのダウンタイムよりも大きくする必要があります。そうしないと、FlinkコネクタはFlinkのセーブポイントまたはチェックポイントに保存されたトランザクションラベルを使用してStarRocksでトランザクションの状態をチェックし、これらのトランザクションがコミットされているかどうかを確認することができません。その結果、最終的にデータ損失を招く可能性があります。

  これらの構成は変更可能で、`ADMIN SET FRONTEND CONFIG`を使用して変更できます：

  ```SQL
    ADMIN SET FRONTEND CONFIG ("prepared_transaction_default_timeout_second" = "3600");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_second" = "259200");
    ADMIN SET FRONTEND CONFIG ("label_keep_max_num" = "1000");
  ```

### フラッシュポリシー
Flinkコネクタはデータをメモリにバッファリングし、それらをバッチでStarRocksにStream Load経由でフラッシュします。フラッシュがトリガーされる方法は、少なくとも一度はおおよそ一度であるのとまさに一度で異なります。

少なくとも一度の場合、フラッシュは次の条件のいずれかが満たされた時にトリガーされます。

- バッファリングされた行のバイト数が`sink.buffer-flush.max-bytes`の制限に達した場合
- バッファリングされた行の数が`sink.buffer-flush.max-rows`の制限に達した場合（SinkバージョンV1のみ有効）
- 最後にフラッシュしてからの経過時間が`sink.buffer-flush.interval-ms`の制限に達した場合
- チェックポイントがトリガーされた場合

まさに一度の場合、フラッシュはチェックポイントがトリガーされたときのみ発生します。

### ロードメトリクスのモニタリング

Flinkコネクタは、以下のメトリクスを提供してロードをモニタリングします。

| メトリクス             | タイプ    | 説明                             |
|------------------------|---------|--------------------------------|
| totalFlushBytes        | カウンタ  | フラッシュされたバイト数             |
| totalFlushRows         | カウンタ  | 正常にフラッシュされた行数                 |
| totalFlushSucceededTimes | カウンタ  | データが正常にフラッシュされた回数  |
| totalFlushFailedTimes  | カウンタ  | データがフラッシュに失敗した回数       |
| totalFilteredRows      | カウンタ  | フィルタリングされた行数。totalFlushRowsにも含まれます。         |

## 例

次の例は、Flinkコネクタを使用してFlink SQLまたはFlink DataStreamを使用してデータをStarRocksテーブルにロードする方法を示しています。

### 準備

#### StarRocksテーブルの作成

データベース`test`を作成し、プライマリキーのテーブル`score_board`を作成します。

```sql
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### Flink環境のセットアップ

- [Flink 1.15.2](https://archive.apache.org/dist/flink/flink-1.15.2/flink-1.15.2-bin-scala_2.12.tgz)のバイナリをダウンロードし、それをディレクトリ`flink-1.15.2`に展開します。
- [Flink connector 1.2.7](https://repo1.maven.org/maven2/com/starrocks/flink-connector-starrocks/1.2.7_flink-1.15/flink-connector-starrocks-1.2.7_flink-1.15.jar)をダウンロードし、それをディレクトリ`flink-1.15.2/lib`に配置します。
- 次のコマンドを実行して、Flinkクラスタを起動します。

    ```shell
    cd flink-1.15.2
    ./bin/start-cluster.sh
    ```

### Flink SQLで実行

- 次のコマンドを実行して、Flink SQLクライアントを起動します。

    ```shell
    ./bin/sql-client.sh
    ```

- Flinkテーブル`score_board`を作成し、Flink SQLクライアントを介してテーブルに値を挿入します。
StarRocksのプライマリキーのテーブルにデータをロードする場合は、Flink DDLでプライマリキーを定義する必要があります。他のタイプのStarRocksテーブルにデータをロードする場合は任意です。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = ''
    );

    INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    ```

### Flink DataStreamで実行

入力レコードのタイプに応じて、CSV Javaの`String`、JSON Javaの`String`、またはカスタムJavaオブジェクトを使用してFlink DataStreamジョブを実装する方法がいくつかあります。

- 入力レコードはCSV形式の`String`です。完全な例については、[LoadCsvRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCsvRecords.java)を参照してください。

    ```java
    /**
     * CSV形式のレコードを生成します。各レコードは"\t"で区切られた3つの値を持っています。 
     * これらの値は、StarRocksテーブルの列`id`、`name`、`score`にロードされます。
     */
    String[] records = new String[]{
            "1\tstarrocks-csv\t100",
            "2\tflink-csv\t100"
    };
    DataStream<String> source = env.fromElements(records);

    /**
     * 必要なプロパティでコネクタを構成します。
     * コネクタにプロパティ"dashboard.properties.format"と"dashborad.properties.column_separator"を追加する必要があります。 
     * これによって、入力レコードがCSV形式であること、および列の区切りが"\t"であることをコネクタに伝えます。
     * CSV形式のレコードで他の列区切りを使用することもできますが、"dashborad.properties.column_separator"を適切に変更する必要があります。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "csv")
            .withProperty("sink.properties.column_separator", "\t")
            .build();
    // オプションを使用してシンクを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードはJSON形式の`String`です。完全な例については、[LoadJsonRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadJsonRecords.java)を参照してください。

    ```java
    /**
     * JSON形式のレコードを生成します。 
     * 各レコードは、StarRocksテーブルの列`id`、`name`、`score`に対応する3つのキーと値のペアを持っています。
     */
    String[] records = new String[]{
            "{\"id\":1, \"name\":\"starrocks-json\", \"score\":100}",
            "{\"id\":2, \"name\":\"flink-json\", \"score\":100}",
    };
    DataStream<String> source = env.fromElements(records);

    /** 
     * 必要なプロパティでコネクタを構成します。
     * コネクタにプロパティ"dashborad.properties.format"と"dashborad.properties.strip_outer_array"を追加する必要があります。 
     * これによって、入力レコードがJSON形式であり、最外の配列構造を削除する必要があることをコネクタに伝えます。
     */
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
            .withProperty("sink.properties.format", "json")
            .withProperty("sink.properties.strip_outer_array", "true")
            .build();
    // オプションを使用してシンクを作成します。
    SinkFunction<String> starRockSink = StarRocksSink.sink(options);
    source.addSink(starRockSink);
    ```

- 入力レコードはカスタムJavaオブジェクトです。完全な例については、[LoadCustomJavaRecords](https://github.com/StarRocks/starrocks-connector-for-apache-flink/tree/main/examples/src/main/java/com/starrocks/connector/flink/examples/datastream/LoadCustomJavaRecords.java)を参照してください。

  - この例では、入力レコードは単純なPOJO `RowData`です。

      ```java
      public static class RowData {
              public int id;
              public String name;
              public int score;
    
              public RowData() {}
    
              public RowData(int id, String name, int score) {
                  this.id = id;
                  this.name = name;
                  this.score = score;
              }
          }
      ```

  - メインプログラムは次のようになります:

    ```java
    // RowDataをコンテナとして使用するレコードを生成します。
    RowData[] records = new RowData[]{
            new RowData(1, "starrocks-rowdata", 100),
            new RowData(2, "flink-rowdata", 100),
        };
    DataStream<RowData> source = env.fromElements(records);

    // 必要なプロパティでコネクタを構成します。
    StarRocksSinkOptions options = StarRocksSinkOptions.builder()
            .withProperty("jdbc-url", jdbcUrl)
            .withProperty("load-url", loadUrl)
            .withProperty("database-name", "test")
            .withProperty("table-name", "score_board")
            .withProperty("username", "root")
            .withProperty("password", "")
```java
      .build();

/**
 * Flinkコネクターは、StarRocksテーブルにロードされる行を表すためにJavaオブジェクト配列（Object[]）を使用し、各要素は列の値です。
 * StarRocksテーブルと一致するObject[]のスキーマを定義する必要があります。
 */
TableSchema schema = TableSchema.builder()
        .field("id", DataTypes.INT().notNull())
        .field("name", DataTypes.STRING())
        .field("score", DataTypes.INT())
        // StarRocksテーブルがプライマリキーテーブルの場合、例えば、プライマリキー `id` に対して DataTypes.INT().notNull() のように、notNull() を指定する必要があります。
        .primaryKey("id")
        .build();
// RowDataをスキーマに従ってObject[]に変換します。
RowDataTransformer transformer = new RowDataTransformer();
// スキーマ、オプション、およびトランスフォーマを使用してシンクを作成します。
SinkFunction<RowData> starRockSink = StarRocksSink.sink(schema, options, transformer);
source.addSink(starRockSink);
```

- メインプログラム内の `RowDataTransformer` は次のように定義されます。

```java
private static class RowDataTransformer implements StarRocksSinkRowBuilder<RowData> {

    /**
     * 入力されたRowDataに応じて、オブジェクト配列の各要素を設定します。
     * 配列のスキーマはStarRocksテーブルと一致します。
     */
    @Override
    public void accept(Object[] internalRow, RowData rowData) {
        internalRow[0] = rowData.id;
        internalRow[1] = rowData.name;
        internalRow[2] = rowData.score;
        // StarRocksテーブルがプライマリキーテーブルの場合、最後の要素を設定して、データのロードがUPSERTまたはDELETE操作なのかを示す必要があります。
        internalRow[internalRow.length - 1] = StarRocksSinkOP.UPSERT.ordinal();
    }
}
```

## ベストプラクティス

### プライマリキーテーブルへのデータのロード

このセクションでは、部分的な更新と条件付き更新を実現するために、StarRocksのプライマリキーテーブルにデータをロードする方法を示します。
これらの機能の紹介については、[データのロードを通じた変更](https://docs.starrocks.io/en-us/latest/loading/Load_to_Primary_Key_tables)をご覧ください。
これらの例では、Flink SQLを使用します。

#### 準備

StarRocksでデータベース `test` を作成し、プライマリキーテーブル `score_board` を作成します。

```SQL
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`);
```

#### 部分的な更新

この例では、データを列 `id` と `name` にのみロードする方法を示します。

1. MySQLクライアントで、二つのデータ行をStarRocksテーブル `score_board` に挿入します。

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);

    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. Flink SQLクライアントで `score_board` というFlinkテーブルを作成します。

   - 列 `id` と `name` のみを含むDDLを定義します。
   - 部分的な更新を実行するために、Flinkコネクターに `sink.properties.partial_update` オプションを `true` に設定します。
   - Flinkコネクターのバージョンが `<=` 1.2.7 の場合、`sink.properties.columns` オプションを `id,name,__op` に設定する必要があります。これにより、更新する必要のある列をFlinkコネクターに伝えることができます。`__op` フィールドを末尾に追加する必要があります。 `__op` フィールドは、データのロードがUPSERTまたはDELETE操作なのかを示し、その値はコネクターによって自動的に設定されます。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.partial_update' = 'true',
        -- Flinkコネクターのバージョンが <= 1.2.7 の場合のみ
        'sink.properties.columns' = 'id,name,__op'
    ); 
    ```

3. Flinkテーブルに二つのデータ行を挿入します。データ行のプライマリキーはStarRocksテーブルの行と同じですが、列 `name` の値が変更されています。

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update'), (2, 'flink-update');
    ```

4. MySQLクライアントでStarRocksテーブルをクエリします。
  
    ```SQL
    mysql> select * from score_board;
    +------+------------------+-------+
    | id   | name             | score |
    +------+------------------+-------+
    |    1 | starrocks-update |   100 |
    |    2 | flink-update     |   100 |
    +------+------------------+-------+
    2 rows in set (0.02 sec)
    ```

    列 `name` の値のみが変更され、列 `score` の値は変更されていないことがわかります。

#### 条件付き更新

この例では、列 `score` の値に応じて条件付き更新を行う方法を示します。新しい `score` の値が古い値以上の場合のみ、 `id` の更新が有効となります。

1. MySQLクライアントで、二つのデータ行をStarRocksテーブルに挿入します。

    ```SQL
    mysql> INSERT INTO `score_board` VALUES (1, 'starrocks', 100), (2, 'flink', 100);
    
    mysql> select * from score_board;
    +------+-----------+-------+
    | id   | name      | score |
    +------+-----------+-------+
    |    1 | starrocks |   100 |
    |    2 | flink     |   100 |
    +------+-----------+-------+
    2 rows in set (0.02 sec)
    ```

2. 次の方法で `score_board` というFlinkテーブルを作成します。
  
    - すべての列を含むDDLを定義します。
    - 更新条件として列 `score` を使用するために、`sink.properties.merge_condition` オプションを `score` に設定します。
    - ストリームロードを使用することをコネクタに伝えるために、`sink.version` オプションを `V1` に設定します。

    ```SQL
    CREATE TABLE `score_board` (
        `id` INT,
        `name` STRING,
        `score` INT,
        PRIMARY KEY (id) NOT ENFORCED
    ) WITH (
        'connector' = 'starrocks',
        'jdbc-url' = 'jdbc:mysql://127.0.0.1:9030',
        'load-url' = '127.0.0.1:8030',
        'database-name' = 'test',
        'table-name' = 'score_board',
        'username' = 'root',
        'password' = '',
        'sink.properties.merge_condition' = 'score',
        'sink.version' = 'V1'
        );
    ```

3. Flinkテーブルに二つのデータ行を挿入します。データ行のプライマリキーはStarRocksテーブルの行と同じです。最初のデータ行は列 `score` で小さい値を持ち、二つめのデータ行は列 `score` で大きい値を持ちます。

    ```SQL
    INSERT INTO `score_board` VALUES (1, 'starrocks-update', 99), (2, 'flink-update', 101);
    ```

4. MySQLクライアントでStarRocksテーブルをクエリします。

    ```SQL
    mysql> select * from score_board;
    +------+--------------+-------+
    | id   | name         | score |
    +------+--------------+-------+
    |    1 | starrocks    |   100 |
    |    2 | flink-update |   101 |
    +------+--------------+-------+
    2 rows in set (0.03 sec)
    ```

   二つ目のデータ行のみが変更され、一つ目のデータ行は変更されていないことがわかります。

### BITMAPタイプの列にデータをロード

[`BITMAP`](https://docs.starrocks.io/en-us/latest/sql-reference/sql-statements/data-types/BITMAP)は、UVの計算など、正確なCount Distinctの加速によく使用されます。詳細については、[Use Bitmap for exact Count Distinct](https://docs.starrocks.io/en-us/latest/using_starrocks/Using_bitmap)をご覧ください。
```SQL
      + {T}
      + {T}
    + {T}
  + {T}
```
```SQL
      + {T}
      + {T}
    + {T}
  + {T}
```