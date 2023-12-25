---
displayed_sidebar: Chinese
---
# CBO統計情報

本文では、StarRocks CBOオプティマイザー（Cost-based Optimizer）の基本概念と、CBOオプティマイザーのための統計情報を収集してクエリプランを最適化する方法について説明します。StarRocks 2.4バージョンでは、統計情報としてヒストグラムが導入され、より正確なデータ分布の統計を提供しています。

3.2以前のバージョンでは、StarRocks内部テーブルの統計情報の収集をサポートしていました。3.2バージョンからは、Hive、Iceberg、Hudiテーブルの統計情報を収集することができ、他のシステムに依存することなく行えます。

## CBOオプティマイザーとは

CBOオプティマイザーは、クエリ最適化の鍵です。SQLクエリがStarRocksに到達すると、論理実行プランに解析され、CBOオプティマイザーが論理プランを書き換えて変換し、複数の物理実行プランを生成します。プラン内の各オペレーターの実行コスト（CPU、メモリ、ネットワーク、I/Oなどのリソース消費）を推定し、最もコストが低いクエリパスを最終的な物理クエリプランとして選択します。

StarRocks CBOオプティマイザーはCascadesフレームワークを採用し、多様な統計情報に基づいてコストを推定し、数万に及ぶ実行プランの中から最もコストが低いプランを選択し、複雑なクエリの効率と性能を向上させます。StarRocks 1.16.0バージョンで独自のCBOオプティマイザーが導入されました。1.19バージョン以降、この機能はデフォルトで有効になっています。

統計情報はCBOオプティマイザーの重要な構成要素であり、統計情報の正確さがコスト推定の正確さを決定し、CBOが最適なプランを選択する上で非常に重要であり、CBOオプティマイザーの性能の良し悪しを決定します。以下で、統計情報の種類、収集戦略、および収集タスクの作成方法と統計情報の確認方法について詳しく説明します。

## 統計情報のデータタイプ

StarRocksは、クエリ最適化のためのコスト推定の参考として、多種多様な統計情報を収集します。

### 基本統計情報

StarRocksはデフォルトで定期的に以下の基本情報をテーブルとカラムから収集します：

- row_count: テーブルの総行数
- data_size: カラムのデータサイズ
- ndv: カラムの基数、つまりdistinct valueの数
- null_count: カラム内のNULL値の数
- min: カラムの最小値
- max: カラムの最大値

全量の統計情報はStarRocksクラスターの`_statistics_`データベースの`column_statistics`テーブルに保存されており、`_statistics_`データベースで確認できます。クエリすると、以下のような情報が返されます：

```sql
SELECT * FROM _statistics_.column_statistics\G
*************************** 1. row ***************************
      table_id: 10174
  partition_id: 10170
   column_name: is_minor
         db_id: 10169
    table_name: zj_test.source_wiki_edit
partition_name: p06
     row_count: 2
     data_size: 2
           ndv: NULL
    null_count: 0
           max: 1
           min: 0
```

### ヒストグラム

2.4バージョンから、StarRocksはヒストグラム（Histogram）を導入しました。ヒストグラムはデータ分布の推定によく使用され、データに偏りがある場合、ヒストグラムは基本統計情報の推定に存在する偏差を補うことができ、より正確なデータ分布情報を出力します。

StarRocksは等深ヒストグラム（Equi-height Histogram）を採用し、つまりいくつかのbucketを選定し、各bucket内のデータ量がほぼ等しくなるようにします。選択率（selectivity）に大きな影響を与える頻度の高い値については、StarRocksは個別のバケットを割り当てて保存します。バケットの数が多いほど、ヒストグラムの推定精度が高くなりますが、統計情報のメモリ使用量も増加します。ビジネスの状況に応じて、ヒストグラムのバケット数と個々の収集タスクのMCV（most common value）の数を調整できます。

**ヒストグラムは、明らかなデータの偏りがあり、頻繁にクエリされるカラムに適しています。テーブルのデータ分布が比較的均一である場合は、ヒストグラムを使用する必要はありません。ヒストグラムは数値型、DATE、DATETIME、または文字列型のカラムをサポートしています。**

現在、ヒストグラムは**手動サンプリング**のみで収集をサポートしています。ヒストグラム統計情報はStarRocksクラスターの`_statistics_`データベースの`histogram_statistics`テーブルに保存されています。クエリすると、以下のような情報が返されます：

```sql
SELECT * FROM _statistics_.histogram_statistics\G
*************************** 1. row ***************************
   table_id: 10174
column_name: added
      db_id: 10169
 table_name: zj_test.source_wiki_edit
    buckets: NULL
        mcv: [["23","1"],["5","1"]]
update_time: 2023-12-01 15:17:10.274000
```

## 収集タイプ

テーブルのインポートや削除操作に伴い、テーブルのサイズとデータ分布が頻繁に更新されるため、統計情報を定期的に更新して、テーブル内の実際のデータを正確に反映させる必要があります。収集タスクを作成する前に、どの収集タイプと収集方法を使用するかをビジネスシナリオに基づいて決定する必要があります。

StarRocksは全量収集（full collection）とサンプリング収集（sampled collection）をサポートしています。全量収集とサンプリング収集は、自動および手動の収集方法をサポートしています。

| **収集タイプ** | **収集方法** | **収集手順**                                                                                                                                                | **利点と欠点**                                                                          |
|----------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| 全量収集     | 自動または手動    | 全テーブルをスキャンして、実際の統計情報値を収集します。パーティション（Partition）レベルで収集し、基本統計情報の対応するカラムは、各パーティションごとに1つの統計情報を収集します。対応するパーティションにデータ更新がない場合、次回の自動収集時にそのパーティションの統計情報を再収集せず、リソースの使用を減らします。全量統計情報は`_statistics_.column_statistics`テーブルに保存されます。   | 利点：統計情報が正確で、オプティマイザーが実行プランをより正確に推定するのに役立ちます。欠点：大量のシステムリソースを消費し、速度が遅い。2.4.5バージョンから、自動収集の時間帯をユーザーが設定できるようになり、リソースの消費を減らすことができます。 |
| サンプリング収集     | 自動または手動    | 各パーティションから均等にN行のデータを抽出して、統計情報を計算します。サンプリング統計情報はテーブルレベルで収集され、基本統計情報の各カラムは1つの統計情報を保存します。カラムの基数情報（ndv）は、サンプリングサンプルに基づいて推定されたグローバル基数であり、実際の基数情報とは若干の差異があります。サンプリング統計情報は`_statistics_.table_statistic_v1`テーブルに保存されます。 | 利点：システムリソースの消費が少なく、速度が速い。欠点：統計情報に一定の誤差があり、オプティマイザーが実行プランを評価する正確性に影響を与える可能性があります。                              |

## 統計情報の収集

StarRocksは柔軟な情報収集方法を提供しており、ビジネスシナリオに応じて自動収集、手動収集を選択することも、自動収集タスクをカスタマイズすることもできます。

デフォルトでは、StarRocksは定期的にテーブルの全量統計情報を自動的に収集します。デフォルトの更新チェック時間は5分ごとで、データの更新比率が条件を満たすと自動的に収集がトリガーされます。**全量収集は大量のシステムリソースを消費する可能性があります**。自動全量収集を使用したくない場合は、FEの設定項目`enable_collect_full_statistic`を`false`に設定することで、システムは自動全量収集を停止し、カスタマイズされたタスクに基づいて収集を行います。

### 自動収集（オートコレクション）

基本統計情報については、StarRocksはデフォルトで全テーブルの全量統計情報を自動的に収集し、手動操作は必要ありません。統計情報を一度も収集していないテーブルについては、スケジュールサイクル内で自動的に統計情報を収集します。既に統計情報を収集しているテーブルについては、StarRocksはテーブルの総行数と変更された行数を自動的に更新し、これらの情報を定期的に永続化して、自動収集をトリガーするかどうかの条件として使用します。

2.4.5バージョンからは、自動全量収集の時間帯をユーザーが設定できるようになり、集中収集によるクラスターのパフォーマンスの変動を防ぐことができます。収集時間帯は、`statistic_auto_analyze_start_time`と`statistic_auto_analyze_end_time`の2つのFE設定項目で設定できます。

スケジュールサイクル内で新しい自動収集タスクをトリガーする条件は以下の通りです：

- 前回の統計情報収集後に、テーブルにデータ変更があったかどうか。
- 収集が設定された自動収集時間帯内に行われる（デフォルトは終日収集で、変更可能）。
- 最新の統計情報収集タスクの更新時間が、パーティションデータの更新時間よりも前である。
- テーブルの統計情報の健康度（`statistic_auto_collect_ratio`）が設定された閾値よりも低い。

> 健康度の計算式：
>
> 1. 更新されたパーティションの数が10未満の場合：1 - (前回の統計情報収集後の更新行数/総行数)
>
> 2. 更新されたパーティションの数が10以上の場合：1 - MIN(前回の統計情報収集後の更新行数/総行数, 前回の統計情報収集後の更新されたパーティション数/総パーティション数)

同時に、StarRocksは異なる更新頻度やサイズのテーブルに対して、詳細な設定戦略を行っています。

- データ量が少ないテーブルについては、**StarRocksはデフォルトで制限を設けず、更新頻度が高くてもリアルタイムで収集します**。`statistic_auto_collect_small_table_size`で小規模テーブルのサイズ閾値を設定したり、`statistic_auto_collect_small_table_interval`で収集間隔を設定できます。

- データ量が大きいテーブルについては、StarRocksは以下の戦略で制限を設けます：

  - デフォルトの収集間隔は12時間未満に設定されており、`statistic_auto_collect_large_table_interval`で設定できます。

  - 収集間隔の条件を満たした上で、ヘルスチェックがサンプリング収集の閾値を下回った場合、サンプリング収集をトリガーします。これは`statistic_auto_collect_sample_threshold`で設定します。

  - 収集間隔の条件を満たした上で、ヘルスチェックがサンプリング収集の閾値を上回り、収集閾値を下回る場合、フル収集をトリガーします。これは`statistic_auto_collect_ratio`で設定します。

  - 収集する最大パーティションサイズが100GBを超える場合、サンプリング収集をトリガーします。これは`statistic_max_full_collect_data_size`で設定します。

  - 収集タスクは、前回の収集タスクの時間よりも後に更新されたパーティションのみを収集し、変更されていないパーティションは収集しません。

:::tip

特に注意が必要なのは、あるテーブルのデータが変更された後に、手動でサンプリング収集タスクをトリガーすると、サンプリングタスクの更新時間がデータ更新時間よりも遅れるため、そのスケジュールサイクルでフル収集タスクは生成されません。

:::

自動フル収集タスクはシステムによって自動的に実行され、デフォルトの設定は以下の通りです。[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)コマンドを使用して変更できます。

設定項目:

| **FE設定項目**                                   | **タイプ**  | **デフォルト値**      | **説明**                                              |
|---------------------------------------------|---------|--------------|-----------------------------------------------------|
| enable_statistic_collect                    | BOOLEAN | TRUE         | 統計情報を収集するかどうか。このパラメータはデフォルトでオンです。                                   |
| enable_collect_full_statistic               | BOOLEAN | TRUE         | 自動フル統計情報収集を有効にするかどうか。このパラメータはデフォルトでオンです。                             |
| statistic_collect_interval_sec              | LONG    | 300          | 自動定期タスクで、データ更新を検出する間隔。デフォルトは5分です。単位は秒。                     |
| statistic_auto_analyze_start_time           | STRING  | 00:00:00     | 自動フル収集の開始時間を設定するために使用します。範囲は`00:00:00`から`23:59:59`です。       |
| statistic_auto_analyze_end_time             | STRING  | 23:59:59     | 自動フル収集の終了時間を設定するために使用します。範囲は`00:00:00`から`23:59:59`です。       |
| statistic_auto_collect_small_table_size     | LONG    | 5368709120   | 自動フル収集タスクの小規模テーブルの閾値。デフォルトは5GBです。単位はByte。                         |
| statistic_auto_collect_small_table_interval | LONG    | 0            | 自動フル収集タスクの小規模テーブルの収集間隔。単位は秒。                               |
| statistic_auto_collect_large_table_interval | LONG    | 43200        | 自動フル収集タスクの大規模テーブルの収集間隔。デフォルトは12時間です。単位は秒。                               |
| statistic_auto_collect_ratio                | DOUBLE  | 0.8          | 自動統計情報収集をトリガーするヘルスチェックの閾値。統計情報のヘルスチェックがこの閾値を下回る場合、自動収集がトリガーされます。           |
| statistic_auto_collect_sample_threshold     | DOUBLE  | 0.3          | 自動統計情報サンプリング収集をトリガーするヘルスチェックの閾値。統計情報のヘルスチェックがこの閾値を下回る場合、自動サンプリング収集がトリガーされます。       |
| statistic_max_full_collect_data_size        | LONG    | 107374182400 | 自動統計情報収集の最大パーティションサイズ。デフォルトは100GBです。単位はByte。この値を超える場合、フル収集を放棄し、そのテーブルに対してサンプリング収集を行います。 |
| statistic_full_collect_buffer               | LONG    | 20971520     | 自動フル収集タスクの書き込みバッファサイズ。単位はByte。デフォルト値は20971520（20MB）。                              |
| statistic_collect_max_row_count_per_query   | LONG    | 5000000000   | 統計情報収集で一度に最大で問い合わせる行数。統計情報タスクはこの設定に従って複数のタスクに自動的に分割されて実行されます。 |
| statistic_collect_too_many_version_sleep    | LONG    | 600000       | 統計情報テーブルの書き込みバージョンが多すぎる場合（Too many tablet例外）、自動収集タスクのスリープ時間。単位はミリ秒。デフォルト値は600000（10分）。 |

### 手動収集 (Manual Collection)

ANALYZE TABLE文を使用して手動収集タスクを作成できます。**手動収集はデフォルトで同期操作です。手動タスクを非同期に設定することもでき、コマンドを実行した後、システムはすぐにコマンドの状態を返しますが、統計情報収集タスクはバックグラウンドで非同期に実行されます。非同期収集は大規模なテーブルデータに適しており、同期収集は小規模なテーブルデータに適しています。手動タスクは作成後に一度だけ実行され、手動で削除する必要はありません。実行状態はSHOW ANALYZE STATUSで確認できます**。

#### 手動で基本統計情報を収集する

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

パラメータ説明：

- 収集タイプ
  - FULL：フル収集。
  - SAMPLE：サンプリング収集。
  - 収集タイプを指定しない場合、デフォルトはフル収集です。

- `WITH SYNC | ASYNC MODE`：指定しない場合、デフォルトは同期収集です。

- `col_name`：統計情報を収集する列。複数の列はカンマで区切ります。指定しない場合、テーブル全体の情報を収集します。

- `PROPERTIES`：収集タスクのカスタムパラメータ。指定しない場合、`fe.conf`のデフォルト設定が使用されます。

| **プロパティ**                | **タイプ** | **デフォルト値** | **説明**                           |
|-------------------------------|--------|---------|----------------------------------|
| statistic_sample_collect_rows | INT    | 200000  | 最小サンプリング行数。パラメータの値が実際のテーブル行数を超える場合、デフォルトでフル収集が行われます。 |

**例**

- 手動でフル収集 (Manual full collection)

```SQL
-- 手動で指定されたテーブルの統計情報をデフォルト設定でフル収集します。
ANALYZE TABLE tbl_name;

-- 手動で指定されたテーブルの統計情報をデフォルト設定でフル収集します。
ANALYZE FULL TABLE tbl_name;

-- 手動で指定されたテーブルの特定の列の統計情報をデフォルト設定でフル収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

- 手動でサンプリング収集 (Manual sampled collection)

```SQL
-- 手動で指定されたテーブルの統計情報をデフォルト設定でサンプリング収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 手動で指定されたテーブルの特定の列の統計情報をサンプリング行数を設定してサンプリング収集します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### 手動でヒストグラム統計情報を収集する

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
[PROPERTIES (property [,property])]
```

パラメータ説明：

- `col_name`：統計情報を収集する列。複数の列はカンマで区切ります。このパラメータは必須です。

- `WITH SYNC | ASYNC MODE`：指定しない場合、デフォルトは同期収集です。

- `WITH N BUCKETS`：`N`はヒストグラムのバケット数です。指定しない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES：収集タスクのカスタムパラメータ。指定しない場合、`fe.conf`のデフォルト設定が使用されます。

| **プロパティ**                 | **タイプ** | **デフォルト値**  | **説明**                           |
|--------------------------------|--------|----------|----------------------------------|
| statistic_sample_collect_rows  | INT    | 200000   | 最小サンプリング行数。パラメータの値が実際のテーブル行数を超える場合、デフォルトでフル収集が行われます。 |
| histogram_mcv_size             | INT    | 100      | ヒストグラムのMost Common Value（MCV）の数です。 |
| histogram_sample_ratio         | FLOAT  | 0.1      | ヒストグラムのサンプリング比率です。                         |
| histogram_buckets_size         | LONG   | 64       | ヒストグラムのデフォルトバケット数です。                                                                                             |
| histogram_max_sample_row_count | LONG   | 10000000 | ヒストグラムの最大サンプリング行数です。                       |


直方グラフのサンプル行数は複数のパラメータによって制御され、`statistic_sample_collect_rows` と表の総行数 `histogram_sample_ratio` のうちの大きい方を取ります。最大で `histogram_max_sample_row_count` に指定された行数を超えません。超える場合は、そのパラメータで定義された上限行数で収集を行います。

直方グラフタスクが実際に実行中に使用する **PROPERTIES** は、SHOW ANALYZE STATUS の **PROPERTIES** 列で確認できます。

**例**

```SQL
-- v1列の直方グラフ情報を手動で収集し、デフォルト設定を使用します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- v1列の直方グラフ情報を手動で収集し、32のバケットを指定し、mcvを32に、サンプル比率を50%に設定します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### カスタム自動収集 (Custom Collection)

#### 自動収集タスクの作成

CREATE ANALYZE ステートメントを使用してカスタム自動収集タスクを作成できます。

カスタム自動収集タスクを作成する前に、自動全量収集 `enable_collect_full_statistic=false` をオフにする必要があります。そうしないとカスタム収集タスクは有効になりません。`enable_collect_full_statistic=false` をオフにした後、**StarRocks は自動的にカスタム収集タスクを作成し、デフォルトで全てのテーブルを収集します。**

```SQL
-- 定期的に全てのデータベースの統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [,property])]

-- 定期的に指定されたデータベースの全てのテーブルの統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name [PROPERTIES (property [,property])]

-- 定期的に指定されたテーブル、列の統計情報を収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name]) [PROPERTIES (property [,property])]
```

パラメータ説明：

- 収集タイプ
  - FULL：全量収集。
  - SAMPLE：サンプル収集。
  - 収集タイプを指定しない場合、デフォルトは全量収集です。

- `col_name`: 統計情報を収集する列。複数の列はコンマ (,) で区切ります。指定しない場合は、テーブル全体の情報を収集します。

- `PROPERTIES`: 収集タスクのカスタムパラメータ。設定しない場合は、`fe.conf` のデフォルト設定を使用します。

| **PROPERTIES**                        | **タイプ** | **デフォルト値** | **説明**                                                                                                                           |
|---------------------------------------|--------|---------|----------------------------------------------------------------------------------------------------------------------------------|
| statistic_auto_collect_ratio          | FLOAT  | 0.8     | 自動統計情報の健康度閾値。統計情報の健康度がこの閾値未満の場合、自動収集がトリガーされます。                                                                                            |
| statistics_max_full_collect_data_size | INT    | 100     | 自動統計情報収集の最大パーティションサイズ。単位: GB。あるパーティションがこの値を超える場合、全量統計情報収集を放棄し、そのテーブルに対してサンプル統計情報収集を行います。                                                                  |
| statistic_sample_collect_rows         | INT    | 200000  | サンプル収集のサンプル行数。このパラメータの値が実際のテーブル行数を超える場合、全量収集を行います。                                                                                              |
| statistic_exclude_pattern             | STRING | NULL    | 自動収集時に除外するデータベースとテーブル。正規表現によるマッチングがサポートされており、マッチングの形式は `database.table` です。                                                                                  |
| statistic_auto_collect_interval       | LONG   |  0      | 自動収集の間隔時間。単位：秒。StarRocks はデフォルトでテーブルのサイズに基づいて `statistic_auto_collect_small_table_interval` または `statistic_auto_collect_large_table_interval` を使用します。PROPERTIES で `statistic_auto_collect_interval` を指定した場合、その値に基づいて Analyze タスクを実行し、`statistic_auto_collect_small_table_interval` または `statistic_auto_collect_large_table_interval` のパラメータは使用しません。|

**例**

**自動全量収集**

```SQL
-- 定期的に全てのデータベースの統計情報を全量収集します。
CREATE ANALYZE ALL;

-- 定期的に指定されたデータベースの全てのテーブルの統計情報を全量収集します。
CREATE ANALYZE DATABASE db_name;

-- 定期的に指定されたデータベースの全てのテーブルの統計情報を全量収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- 定期的に指定されたテーブル、列の統計情報を全量収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 
    
-- 定期的に全てのデータベースの統計情報を全量収集し、`db_name` データベースは収集しません。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);    
```

**自動サンプル収集**

```SQL
-- 定期的に指定されたデータベースの全てのテーブルの統計情報をサンプル収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 定期的に指定されたテーブル、列の統計情報をサンプル収集し、サンプル行数と健康度閾値を設定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES(
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);
    
-- 自動的に全てのデータベースの統計情報を収集し、`db_name.tbl_name` テーブルは収集しません。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);    
```

#### 自動収集タスクの確認

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE 句を使用してフィルタ条件を設定し、結果を絞り込むことができます。このステートメントは以下の列を返します。

| **列名**       | **説明**                                                         |
|--------------|----------------------------------------------------------------|
| Id           | 収集タスクのID。                                                       |
| Database     | データベース名。                                                          |
| Table        | テーブル名。                                                            |
| Columns      | 列名リスト。                                                          |
| Type         | 統計情報のタイプ。値は FULL または SAMPLE。                                       |
| Schedule     | スケジュールのタイプ。自動収集タスクは `SCHEDULE` に固定されています。                                    |
| Properties   | カスタムパラメータ情報。                                                       |
| Status       | タスクの状態。PENDING（待機中）、RUNNING（実行中）、SUCCESS（成功）、FAILED（失敗）を含みます。 |
| LastWorkTime | 最後に収集された時間。                                                      |
| Reason       | タスクが失敗した理由。成功した場合は `NULL`。                                       |

**例**

```SQL
-- クラスターの全てのカスタム収集タスクを確認します。
SHOW ANALYZE JOB

-- test データベースのカスタム収集タスクを確認します。
SHOW ANALYZE JOB where `database` = 'test';
```

#### 自動収集タスクの削除

```SQL
DROP ANALYZE <ID>
```

**例**

```SQL
DROP ANALYZE 266030;
```

## 収集タスクの状態を確認

SHOW ANALYZE STATUS ステートメントを使用して、現在の全ての収集タスクの状態を確認できます。このステートメントはカスタム収集タスクの状態を確認することはできません。カスタム収集タスクの状態を確認する場合は SHOW ANALYZE JOB を使用してください。

```SQL
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

`Like` または `Where` を使用して、返される情報をフィルタリングできます。

現在 SHOW ANALYZE STATUS は以下の列を返します。

| **列名**   | **説明**                                                     |
| ---------- | ------------------------------------------------------------ |
| Id         | 収集タスクのID。                                               |
| Database   | データベース名。                                                   |
| Table      | テーブル名。                                                       |
| Columns    | 列名リスト。                                                   |
| Type       | 統計情報のタイプ。FULL、SAMPLE、HISTOGRAM を含みます。               |
| Schedule   | スケジュールのタイプ。`ONCE` は手動、`SCHEDULE` は自動を表します。             |
| Status     | タスクの状態。RUNNING（実行中）、SUCCESS（成功）、FAILED（失敗）を含みます。 |
| StartTime  | タスクが開始された時間。                                         |
| EndTime    | タスクが終了した時間。                                         |
| Properties | カスタムパラメータ情報。                                             |
| Reason     | タスクが失敗した理由。成功した場合は NULL。                      |

## 統計情報を確認

### 基本統計情報メタデータ

```SQL
SHOW STATS META [WHERE predicate]
```

このステートメントは以下の列を返します。

| **列名**   | **説明**                                            |
| ---------- | --------------------------------------------------- |
| Database   | データベース名。                                          |
| Table      | テーブル名。                                              |
| Columns    | 列名。                                              |
| Type       | 統計情報のタイプ。`FULL` は全量、`SAMPLE` はサンプルを表します。 |
| UpdateTime | テーブルの最新統計情報更新時間。                      |
| Properties | カスタムパラメータ情報。                                    |
| Healthy    | 統計情報の健康度。                                    |

### 直方グラフ統計情報メタデータ

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

このステートメントは以下の列を返します。

| **列名**   | **説明**                                  |
| ---------- | ----------------------------------------- |
| Database   | データベース名。                                |
| Table      | テーブル名。                                    |
| Column     | 列名。                                    |
| Type       | 統計情報のタイプ。直方グラフは `HISTOGRAM` に固定されています。 |
| UpdateTime | テーブルの最新統計情報更新時間。            |

| プロパティ | カスタムパラメータ情報。                          |

## 統計情報の削除

StarRocks は統計情報を手動で削除する機能をサポートしています。統計情報を手動で削除すると、統計データと統計メタデータが削除され、期限切れの統計情報キャッシュもメモリから削除されます。自動収集タスクが存在する場合、削除された統計情報が再収集される可能性がある点に注意が必要です。統計情報の収集履歴は `SHOW ANALYZE STATUS` で確認できます。

### 基本統計情報の削除

```SQL
DROP STATS tbl_name
```

このコマンドは `default_catalog._statistics_` に保存されている統計情報を削除し、FE cache の対応する表の統計情報も無効になります。

### ヒストグラム統計情報の削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 収集タスクのキャンセル

`KILL ANALYZE` ステートメントを使用して、実行中（Running）の統計情報収集タスクをキャンセルできます。これには手動収集タスクとカスタム自動収集タスクが含まれます。

手動収集タスクのタスクIDは `SHOW ANALYZE STATUS` で確認できます。カスタム自動収集タスクのタスクIDは `SHOW ANALYZE JOB` で確認できます。

```SQL
KILL ANALYZE <ID>
```

## その他の FE 設定項目

| **FE 設定項目**        | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_collect_concurrency               | INT     | 3            | 手動収集タスクの最大同時実行数。デフォルトは 3 で、最大で 3 つの手動収集タスクが同時に実行されます。<br />それを超えるタスクは PENDING 状態となり、スケジュールを待ちます。                                              |
| statistic_manager_sleep_time_sec            | LONG    | 60           | 統計情報関連のメタデータスケジュール間隔。単位は秒です。この間隔に基づいて、以下の操作が実行されます：<ul><li>統計情報テーブルの作成；</li><li>削除されたテーブルの統計情報の削除；</li><li>期限切れの統計情報履歴の削除。</li></ul> |
| statistic_analyze_status_keep_second        | LONG    | 259200       | 収集タスク記録の保持時間。デフォルトは 3 日間です。単位は秒です。                                                                                          |

## システム変数

`statistic_collect_parallel` は BE 上で同時に実行できる統計情報収集タスクの数を調整するために使用されます。デフォルト値は 1 ですが、この値を増やすことで収集タスクの実行速度を速めることができます。

## Hive/Iceberg/Hudi テーブルの統計情報の収集

バージョン 3.2 以降、Hive、Iceberg、Hudi テーブルの統計情報の収集がサポートされています。**収集の構文は内部テーブルと同じですが、手動全量収集と自動全量収集のみをサポートし、サンプリング収集とヒストグラム収集はサポートしていません**。収集された統計情報は `_statistics_` データベースの `external_column_statistics` テーブルに書き込まれますが、Hive Metastore には書き込まれないため、他のクエリエンジンと共有することはできません。`default_catalog._statistics_.external_column_statistics` テーブルにテーブルの統計情報が書き込まれているかどうかを確認できます。

クエリすると、以下の情報が返されます：

```sql
SELECT * FROM _statistics_.external_column_statistics\G
*************************** 1. row ***************************
    table_uuid: hive_catalog.tn_test.ex_hive_tbl.1673596430
partition_name: 
   column_name: k1
  catalog_name: hive_catalog
       db_name: tn_test
    table_name: ex_hive_tbl
     row_count: 3
     data_size: 12
           ndv: NULL
    null_count: 0
           max: 3
           min: 2
   update_time: 2023-12-01 14:57:27.137000
```

### 使用上の制限

Hive、Iceberg、Hudi テーブルの統計情報を収集する際には、以下の制限があります：

1. 現在、Hive、Iceberg、Hudi テーブルの統計情報の収集のみをサポートしています。
2. 現在、全量収集のみをサポートし、サンプリング収集とヒストグラム収集はサポートしていません。
3. 全量自動収集には、収集タスクを作成する必要があります。システムは外部データソースの統計情報をデフォルトで自動収集しません。
4. 自動収集タスクでは、指定されたテーブルの統計情報のみを収集し、すべてのデータベースやデータベース内のすべてのテーブルの統計情報を収集することはサポートしていません。
5. 自動収集タスクでは、現在、Hive と Iceberg テーブルのみがデータ更新を検出し、データが更新された場合にのみ収集タスクを実行し、更新されたパーティションのみを収集します。Hudi テーブルはデータ更新を検出することができないため、収集間隔に基づいて定期的に全テーブルを収集します。

以下の例では、External Catalog で指定されたデータベース内のテーブルの統計情報を収集することを前提としています。`default_catalog` で External Catalog のテーブルの統計情報を収集する場合は、テーブル名を `[catalog_name.][database_name.]<table_name>` の形式で参照できます。

### 手動収集

#### 手動収集タスクの作成

構文：

```sql
ANALYZE [FULL] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES(property [,property])]
```

例：

```sql
ANALYZE TABLE ex_hive_tbl(k1);
+----------------------------------+---------+----------+----------+
| Table                            | Op      | Msg_type | Msg_text |
+----------------------------------+---------+----------+----------+
| hive_catalog.tn_test.ex_hive_tbl | analyze | status   | OK       |
+----------------------------------+---------+----------+----------+
```

#### Analyze タスクの実行状態の確認

構文：

```sql
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

例：

```sql
SHOW ANALYZE STATUS where `table` = 'ex_hive_tbl';
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| Id    | Database             | Table       | Columns | Type | Schedule | Status  | StartTime           | EndTime             | Properties | Reason |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
| 16400 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:31:42 | 2023-12-04 16:31:42 | {}         |        |
| 16465 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:35 | 2023-12-04 16:37:35 | {}         |        |
| 16467 | hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | ONCE     | SUCCESS | 2023-12-04 16:37:46 | 2023-12-04 16:37:46 | {}         |        |
+-------+----------------------+-------------+---------+------+----------+---------+---------------------+---------------------+------------+--------+
```

#### 統計情報メタデータの確認

テーブルの統計情報メタデータを確認します。

構文：

```sql
SHOW STATS META [WHERE predicate]
```

例：

```sql
SHOW STATS META where `table` = 'ex_hive_tbl';
+----------------------+-------------+---------+------+---------------------+------------+---------+
| Database             | Table       | Columns | Type | UpdateTime          | Properties | Healthy |
+----------------------+-------------+---------+------+---------------------+------------+---------+
| hive_catalog.tn_test | ex_hive_tbl | k1      | FULL | 2023-12-04 16:37:46 | {}         |         |
+----------------------+-------------+---------+------+---------------------+------------+---------+
```

#### 収集タスクのキャンセル

実行中（Running）の統計情報収集タスクをキャンセルします。

構文：

```sql
KILL ANALYZE <ID>
```

タスクIDは `SHOW ANALYZE STATUS` で確認できます。

### 自動収集

外部データソース内のテーブルに対しては、自動収集タスクを作成する必要があります。StarRocks は収集タスクに指定された属性に基づいて、タスクが実行されるべきかどうかを定期的にチェックします。デフォルトのチェック時間は 5 分です。Hive と Iceberg はデータ更新が検出された場合にのみ、自動的に一度収集タスクを実行します。

Hudi はデータ更新を検出する機能をサポートしていないため、周期的に収集する必要があります（収集周期は収集スレッドの時間間隔とユーザーが設定した収集間隔によって決まります。以下の属性を参照して調整してください）。

- statistic_collect_interval_sec

  自動定期タスクで、統計情報を収集する必要があるかどうかをチェックする間隔。デフォルトは 5 分です。

- statistic_auto_collect_small_table_rows

  自動収集で、外部データソース下のテーブル（Hive、Iceberg、Hudi）が小規模テーブルかどうかを判断するための行数の閾値。デフォルトは 10000000 行です。このパラメータはバージョン 3.2 で導入されました。

- statistic_auto_collect_small_table_interval

  自動収集で小規模テーブルの収集間隔。単位は秒です。デフォルト値は 0 です。

- statistic_auto_collect_large_table_interval

  自動収集で大規模テーブルの収集間隔。単位は秒です。デフォルト値は 43200（12 時間）です。

自動収集スレッドは `statistic_collect_interval_sec` の間隔ごとにタスクチェックを行い、実行すべきタスクがあるかどうかを確認します。テーブルの行数が `statistic_auto_collect_small_table_rows` 未満の場合は、小規模テーブルの収集間隔が設定されます。それ以外の場合は、大規模テーブルの収集間隔が設定されます。`最後の更新時間 + 収集間隔 > 現在の時間` の場合は更新が必要ですが、そうでなければ更新は不要です。これにより、大規模テーブルに対して頻繁に Analyze タスクを実行することを避けることができます。

#### 自動収集タスクの作成

構文：

```sql
CREATE ANALYZE TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]

```

例：

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

クエリ OK、0行が影響を受けました (0.01秒)
```

例：

#### Analyze タスクの実行状態を確認する

手動での収集と同じです。

#### 統計情報メタデータを確認する

手動での収集と同じです。

#### 自動収集タスクを確認する

構文：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

空のセット (0.00秒)
```

#### 収集タスクをキャンセルする

手動での収集と同じです。

#### Hive/Iceberg/Hudi テーブルの統計情報を削除する

```sql
DROP STATS tbl_name
```

## 詳細情報

- FE の設定項目の値を確認するには、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) を実行します。

- FE の設定項目の値を変更するには、[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) を実行します。

