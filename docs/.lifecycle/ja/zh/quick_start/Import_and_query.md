---
displayed_sidebar: Chinese
---

# データのインポートとクエリ

この記事では、StarRocks でデータをインポートおよびクエリする方法について説明します。

## データのインポート

異なるデータインポートニーズに対応するため、StarRocks システムは5種類の異なるインポート方法を提供しており、異なるデータソースやインポート方法をサポートしています。

### Broker Load

Broker Load（参照：[HDFS からのインポート](../loading/hdfs_load.md)および[クラウドストレージからのインポート](../loading/cloud_storage_load.md)）は、非同期データインポートモードであり、Broker プロセスを通じて外部データソースにアクセスして読み取り、MySQL プロトコルを使用して StarRocks にインポートジョブを作成します。

Broker Load モードは、Broker プロセスがアクセス可能なストレージシステム（例：HDFS、S3）にあるソースデータに適しており、数百GBのデータインポートジョブをサポートできます。このインポート方法がサポートするデータソースには Apache Hive™ などがあります。

### Spark Load

[Spark Load](../loading/SparkLoad.md) は、非同期データインポートモードであり、外部の Apache Spark™ リソースを利用してインポートデータの前処理を行い、StarRocks の大量データインポート性能を向上させ、StarRocks クラスターの計算リソースを節約します。

Spark Load モードは、TB レベルの大量データを StarRocks に初めて移行するシナリオに適しています。このインポート方法がサポートするデータソースは Apache Spark™ がアクセス可能なストレージシステム（例：HDFS）に位置している必要があります。

Spark Load を使用すると、Apache Hive™ テーブルに基づいて [グローバル辞書](../loading/SparkLoad.md) のデータ構造を実現し、入力データの型変換を行い、元の値からエンコード値へのマッピングを保存することができます。例えば、文字列型を整数型にマッピングするなどです。

### Stream Load

[Stream Load](../loading/StreamLoad.md) は、同期データインポートモードです。ユーザーは HTTP プロトコルを使用してリクエストを送信し、ローカルファイルまたはデータストリームを StarRocks にインポートし、システムからインポート結果のステータスを待って、インポートが成功したかどうかを判断します。

Stream Load モードは、ローカルファイルをインポートする場合や、プログラムを通じてデータストリームからデータをインポートする場合に適しています。このインポート方法がサポートするデータソースには Apache Flink®、CSV ファイルなどがあります。

### Routine Load

[Routine Load](../loading/RoutineLoad.md)（定期インポート）は、指定されたデータソースから自動的にデータをインポートする機能を提供します。MySQL プロトコルを使用して定期インポートジョブを提出し、常駐スレッドを生成し、データソース（例：Apache Kafka®）からデータを途切れることなく読み取り、StarRocks にインポートします。

### Insert Into

[Insert Into](../loading/InsertInto.md) インポートモードは、同期データインポートモードであり、MySQL の Insert ステートメントに似ています。StarRocks は `INSERT INTO tbl SELECT ...;` の形式で StarRocks のテーブルからデータを読み取り、別のテーブルにインポートすることをサポートしています。また、`INSERT INTO tbl VALUES(...);` を使用して単一のデータを挿入することもできます。このインポート方法がサポートするデータソースには DataX/DTS、Kettle/Informatica、および StarRocks 自体があります。

具体的なインポート方法の詳細については、[データインポート](../loading/Loading_intro.md) を参照してください。

### Stream Load を使用してデータをインポートする

以下の例では、Stream Load インポート方法を使用して、[テーブルの作成](../quick_start/Create_table.md) セクションで作成した `detailDemo` テーブルにファイル内のデータをインポートします。

ローカルにデータファイル **detailDemo_data** を作成し、カンマをデータの区切り文字として、2つのデータを挿入します。具体的な内容は以下の通りです：

```Plain Text
2022-03-13,1,1212,1231231231,123412341234,123452342342343324,hello,welcome,starrocks,2022-03-15 12:21:32,123.04,21.12345,123456.123456,true
2022-03-14,2,1212,1231231231,123412341234,123452342342343324,hello,welcome,starrocks,2022-03-15 12:21:32,123.04,21.12345,123456.123456,false
```

次に、"streamDemo" をラベルとして、curl コマンドを使用して HTTP リクエストを作成し、ローカルファイル **detailDemo_data** を `detailDemo` テーブルにインポートします。

```bash
curl --location-trusted -u <username>:<password> -T detailDemo_data -H "label: streamDemo" \
-H "column_separator:," \
http://127.0.0.1:8030/api/example_db/detailDemo/_stream_load
```

> **注意**
>
> - アカウントにパスワードが設定されていない場合は、`<username>:` のみを入力する必要があります。
> - HTTP アドレスの IP は FE ノードの IP で、ポートは **fe.conf** で設定された `http port` です。

## クエリ

StarRocks は MySQL プロトコルと互換性があり、そのクエリステートメントは基本的に SQL-92 標準に準拠しています。

### シンプルクエリ

MySQL クライアントを使用して StarRocks にログインし、テーブル内のすべてのデータをクエリします。

```sql
use example_db;
select * from detailDemo;
```

### 標準 SQL でクエリする

クエリ結果を `region_num` フィールドで降順に並べ替えます。

```sql
select * from detailDemo order by region_num desc;
```

StarRocks は、[Join](../sql-reference/sql-statements/data-manipulation/SELECT.md)、[サブクエリ](../sql-reference/sql-statements/data-manipulation/SELECT.md#サブクエリ)、[With 句](../sql-reference/sql-statements/data-manipulation/SELECT.md#with) など、さまざまな select の使用法をサポートしています。詳細は [クエリセクション](../sql-reference/sql-statements/data-manipulation/SELECT.md) を参照してください。

## 拡張サポート

StarRocks は、さまざまな関数、ビュー、および外部テーブルを拡張サポートしています。

### 関数

StarRocks は、[日付関数](../sql-reference/sql-functions/date-time-functions/convert_tz.md)、[地理位置関数](../sql-reference/sql-functions/spatial-functions/st_astext.md)、[文字列関数](../sql-reference/sql-functions/string-functions/append_trailing_char_if_absent.md)、[集計関数](../sql-reference/sql-functions/aggregate-functions/approx_count_distinct.md)、[Bitmap 関数](../sql-reference/sql-functions/bitmap-functions/bitmap_and.md)、[配列関数](../sql-reference/sql-functions/array-functions/array_append.md)、[cast 関数](../sql-reference/sql-functions/cast.md)、[hash 関数](../sql-reference/sql-functions/hash-functions/murmur_hash3_32.md)、[暗号化関数](../sql-reference/sql-functions/crytographic-functions/md5.md)、[ウィンドウ関数](../sql-reference/sql-functions/Window_function.md) など、多くの関数をサポートしています。

### ビュー

StarRocks は、[論理ビュー](../sql-reference/sql-statements/data-definition/CREATE_VIEW.md#機能) と [物化ビュー](../using_starrocks/Materialized_view.md) の作成をサポートしています。具体的な使用方法は、対応するセクションを参照してください。

### 外部テーブル

StarRocks は、[MySQL 外部テーブル](../data_source/External_table.md#deprecated-mysql-外部テーブル)、[Elasticsearch 外部テーブル](../data_source/External_table.md#deprecated-elasticsearch-外部テーブル)、[Apache Hive™ 外部テーブル](../data_source/External_table.md#deprecated-hive-外部テーブル)、[StarRocks 外部テーブル](../data_source/External_table.md#starrocks-外部テーブル)、[Apache Iceberg 外部テーブル](../data_source/External_table.md#deprecated-iceberg-外部テーブル)、[Apache Hudi 外部テーブル](../data_source/External_table.md#deprecated-hudi-外部テーブル) など、さまざまな種類の外部テーブルをサポートしています。外部テーブルを正常に作成した後、他のデータソースにアクセスするために外部テーブルをクエリすることができます。

## 遅いクエリの分析

StarRocks は、さまざまな方法でクエリのボトルネックを分析し、クエリ効率を最適化することをサポートしています。

### 並列度を調整してクエリ効率を最適化する

Pipeline 実行エンジン変数の設定をお勧めします。また、`set parallel_fragment_exec_instance_num = 8;` を設定して [Fragment](../introduction/Features.md#mpp-分布式执行框架) インスタンスの並列数を調整し、CPU リソースの利用率とクエリ効率を向上させることができます。詳細なパラメータの紹介と設定については、[クエリ並列度に関するパラメータ](../administration/Query_management.md) を参照してください。

### Profile を表示してクエリのボトルネックを分析する

- クエリプランを表示します。

  ```sql
  explain costs select * from detailDemo;
  ```

  > **注意**
  >
  > StarRocks 1.19 以前のバージョンでは `explain sql` を使用してクエリプランを表示する必要があります。

- Profile のレポートを有効にします。

  ```sql
  set enable_profile = true;
  ```

  > **注意**
  >
  > この方法で Profile のレポートを設定すると、現在のセッションでのみ有効になります。

- コミュニティ版のユーザーは `http://FE_IP:FE_HTTP_PORT/query` を通じて現在のクエリと Profile 情報を表示できます。
- エンタープライズ版のユーザーは StarRocks Manager のクエリページでグラフィカルな Profile 表示を見ることができ、クエリリンクをクリックすると **実行時間** ページでツリー形式の表示を見ることができ、**実行詳細** ページで完全な Profile の詳細情報を見ることができます。上記の方法で期待通りの結果が得られない場合は、実行詳細ページのテキストをコミュニティや技術サポートグループに送信して、支援を求めることができます。

Plan と Profile の詳細な紹介については、[クエリ分析](../administration/Query_planning.md) と [パフォーマンス最適化](../administration/Profiling.md) のセクションを参照してください。
