---
displayed_sidebar: English
---

# CBOの統計収集

このトピックでは、StarRocksのコストベース最適化（CBO）の基本概念と、最適なクエリプランを選択するためにCBOが統計を収集する方法について説明します。StarRocks 2.4では、正確なデータ分布統計を収集するためのヒストグラムが導入されました。

バージョン3.2.0以降、StarRocksはHive、Iceberg、Hudiテーブルからの統計収集をサポートし、他のメタストアシステムへの依存を減らしました。構文はStarRocksの内部テーブルの収集に似ています。

## CBOとは何か

CBOはクエリ最適化にとって重要です。SQLクエリがStarRocksに到着すると、論理実行プランに解析されます。CBOは論理プランを書き換えて複数の物理実行プランに変換します。その後、CBOはプラン内の各オペレーター（CPU、メモリ、ネットワーク、I/Oなど）の実行コストを見積もり、最も低いコストのクエリパスを最終的な物理プランとして選択します。

StarRocks CBOはStarRocks 1.16.0で導入され、1.19以降はデフォルトで有効化されています。Cascadesフレームワークを基に開発されたStarRocks CBOは、さまざまな統計情報に基づいてコストを見積もります。数万に及ぶ実行プランの中から最もコストが低いプランを選択する能力を持ち、複雑なクエリの効率とパフォーマンスを大幅に向上させます。

統計はCBOにとって重要です。これらはコスト見積もりが正確かつ有用であるかどうかを決定します。以下のセクションでは、統計情報の種類、収集ポリシー、統計の収集方法、および統計情報の表示方法について詳しく説明します。

## 統計情報の種類

StarRocksは、コスト見積もりのための入力として様々な統計を収集します。

### 基本統計

デフォルトで、StarRocksは定期的にテーブルとカラムの以下の基本統計を収集します：

- row_count：テーブル内の総行数

- data_size：カラムのデータサイズ

- ndv：カラムのカーディナリティ、つまりカラム内の異なる値の数

- null_count：カラム内のNULL値を持つデータの量

- min：カラム内の最小値

- max：カラム内の最大値

完全な統計は`_statistics_`データベースの`column_statistics`テーブルに保存されます。このテーブルを参照してテーブル統計をクエリすることができます。以下はこのテーブルから統計データをクエリする例です。

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

StarRocks 2.4では、基本統計を補完するためにヒストグラムが導入されました。ヒストグラムはデータ表現の効果的な方法とされています。データが偏っているテーブルでは、ヒストグラムはデータ分布を正確に反映することができます。

StarRocksは等高ヒストグラムを使用します。これはいくつかのバケットに基づいて構築され、各バケットには等しい量のデータが含まれます。頻繁にクエリされ、選択性に大きな影響を与えるデータ値については、StarRocksはそれら専用のバケットを割り当てます。バケットが多ければ多いほど、推定が正確になりますが、メモリ使用量が若干増加する可能性もあります。ヒストグラム収集タスクのバケット数と最も一般的な値（MCV）を調整することができます。

**ヒストグラムは、データが大きく偏っており、頻繁にクエリされるカラムに適しています。テーブルデータが均一に分布している場合、ヒストグラムを作成する必要はありません。ヒストグラムは数値型、DATE型、DATETIME型、または文字列型のカラムにのみ作成できます。**

現在、StarRocksはヒストグラムの手動収集のみをサポートしています。ヒストグラムは`_statistics_`データベースの`histogram_statistics`テーブルに保存されます。以下はこのテーブルから統計データをクエリする例です：

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

## 収集タイプと方法

テーブルのデータサイズとデータ分布は常に変化しています。統計は、そのデータ変更を反映するために定期的に更新される必要があります。統計収集タスクを作成する前に、ビジネス要件に最適な収集タイプと方法を選択する必要があります。

StarRocksは完全収集とサンプル収集をサポートしており、どちらも自動的にも手動的にも実行できます。デフォルトでは、StarRocksはテーブルの完全統計を自動的に収集します。5分ごとにデータ更新をチェックし、データ変更が検出された場合、自動的に収集がトリガされます。自動的な完全収集を使用したくない場合は、FE設定項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズできます。

| **収集タイプ** | **収集方法** | **説明**                                              | **長所と短所**                               |
| ------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 完全収集     | 自動/手動      | テーブル全体をスキャンして統計を収集します。統計はパーティションごとに収集されます。パーティションにデータ変更がない場合、そのパーティションからはデータが収集されません。これによりリソース消費が削減されます。完全統計は`_statistics_.column_statistics`テーブルに格納されます。 | 長所: 統計が正確で、CBOが正確な推定を行うのに役立ちます。短所: システムリソースを消費し、時間がかかります。2.5以降、StarRocksでは自動収集期間を指定でき、リソース消費を削減できます。 |
| サンプル収集  | 自動/手動      | テーブルの各パーティションから均等に`N`行のデータを抽出します。統計はテーブルごとに収集されます。各カラムの基本統計は1レコードとして格納されます。カラムのカーディナリティ情報（ndv）はサンプルデータに基づいて推定されますが、これは正確ではありません。サンプル統計は`_statistics_.table_statistic_v1`テーブルに格納されます。| 長所: システムリソースの消費が少なく、速いです。短所: 統計が完全でないため、コスト見積もりの精度に影響を与える可能性があります。 |

## 統計の収集

StarRocksは柔軟な統計収集方法を提供します。自動収集、手動収集、カスタム収集の中から、ビジネスシナリオに合ったものを選択できます。

### 自動収集


基本統計について、StarRocksはデフォルトでテーブルの完全統計を自動的に収集し、手動操作を必要としません。統計が収集されていないテーブルについては、StarRocksはスケジューリング期間内に自動的に統計を収集します。既に統計が収集されているテーブルでは、StarRocksはテーブルの総行数と変更された行数を更新し、定期的にこの情報を保存して自動収集をトリガーするかどうかを判断します。

バージョン2.4.5以降、StarRocksは自動完全収集のための収集期間を指定することができ、自動完全収集によるクラスタのパフォーマンスの変動を防ぐことができます。この期間はFEパラメータ`statistic_auto_analyze_start_time`と`statistic_auto_analyze_end_time`で指定されます。

自動収集をトリガーする条件:

- テーブルデータが前回の統計収集以降に変更されている。

- 収集時間が設定された収集期間内である。（デフォルトの収集期間は一日中です。）

- 前回の収集ジョブの更新時間がパーティションの最新の更新時間よりも前である。

- テーブル統計の健全性が指定された閾値（`statistic_auto_collect_ratio`）を下回っている。

> 統計の健全性を計算する式:
>
> 更新されたデータを持つパーティションの数が10未満の場合、式は`1 - (前回の収集からの更新された行数/総行数)`です。
> 更新されたデータを持つパーティションの数が10以上の場合、式は`1 - MIN(前回の収集からの更新された行数/総行数, 前回の収集からの更新されたパーティション数/総パーティション数)`です。

さらに、StarRocksではテーブルのサイズと更新頻度に基づいて収集ポリシーを設定することができます:

- データ量が少ないテーブルでは、テーブルデータが頻繁に更新されても、**制限なしでリアルタイムに統計が収集されます**。`statistic_auto_collect_small_table_size`パラメータを使用して、テーブルが小さいか大きいかを判断できます。また、`statistic_auto_collect_small_table_interval`を使用して小テーブルの収集間隔を設定することもできます。

- データ量が大きいテーブルには以下の制限が適用されます:

  - デフォルトの収集間隔は12時間未満に設定されておらず、`statistic_auto_collect_large_table_interval`を使用して設定できます。

  - 収集間隔が満たされ、統計の健全性が自動サンプリング収集の閾値（`statistic_auto_collect_sample_threshold`）を下回った場合、サンプリング収集がトリガーされます。

  - 収集間隔が満たされ、統計の健全性が自動サンプリング収集の閾値（`statistic_auto_collect_sample_threshold`）を上回り、自動収集の閾値（`statistic_auto_collect_ratio`）を下回った場合、完全収集がトリガーされます。

  - 収集する最大パーティションのサイズ（`statistic_max_full_collect_data_size`）が100GBを超える場合、サンプリング収集がトリガーされます。

  - 前回の収集タスクの時刻より後に更新されたパーティションの統計のみが収集されます。データ変更がないパーティションの統計は収集されません。

:::tip

テーブルのデータが変更された後、そのテーブルに対してサンプリング収集タスクを手動でトリガーすると、サンプリング収集タスクの更新時刻がデータ更新時刻より後になり、このスケジューリング期間中にそのテーブルに対する自動完全収集はトリガーされません。
:::

自動完全収集はデフォルトで有効になっており、デフォルト設定を使用してシステムによって実行されます。

以下の表はデフォルト設定を説明しています。変更する必要がある場合は、**ADMIN SET CONFIG**コマンドを実行してください。

| **FE構成項目**                      | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| enable_statistic_collect              | BOOLEAN  | TRUE              | 統計を収集するかどうか。このスイッチはデフォルトでオンです。 |
| enable_collect_full_statistic         | BOOLEAN  | TRUE              | 自動完全収集を有効にするかどうか。このスイッチはデフォルトでオンです。 |
| statistic_collect_interval_sec        | LONG     | 300               | 自動収集中にデータ更新をチェックする間隔。単位は秒。 |
| statistic_auto_analyze_start_time | STRING   | 00:00:00          | 自動収集の開始時刻。値の範囲: `00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | STRING   | 23:59:59          | 自動収集の終了時刻。値の範囲: `00:00:00` - `23:59:59`。 |
| statistic_auto_collect_small_table_size     | LONG    | 5368709120        | 自動完全収集のための小テーブルか大テーブルかを判断するしきい値。この値より大きいサイズのテーブルは大テーブルと見なされ、この値以下のサイズのテーブルは小テーブルと見なされます。単位はバイト。デフォルト値は5368709120（5GB）。 |
| statistic_auto_collect_small_table_interval | LONG    | 0                 | 小テーブルの完全統計を自動的に収集する間隔。単位は秒。 |
| statistic_auto_collect_large_table_interval | LONG    | 43200             | 大テーブルの完全統計を自動的に収集する間隔。単位は秒。デフォルト値は43200（12時間）。 |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集のための統計の健全性を判断するしきい値。このしきい値を下回ると自動収集がトリガーされます。 |
| statistic_auto_collect_sample_threshold  | DOUBLE  | 0.3               | 自動サンプリング収集をトリガーする統計の健全性のしきい値。このしきい値を下回ると自動サンプリング収集がトリガーされます。 |
| statistic_max_full_collect_data_size | LONG      | 107374182400      | 自動収集でデータを収集する最大パーティションのサイズ。単位はバイト。デフォルト値は107374182400（100GB）。この値を超えるパーティションは完全収集を行わず、サンプリング収集が行われます。 |
| statistic_full_collect_buffer | LONG | 20971520 | 自動収集タスクによって使用される最大バッファサイズ。単位はバイト。デフォルト値は20971520（20MB）。 |
| statistic_collect_max_row_count_per_query | INT     | 5000000000        | 単一の分析タスクに対してクエリする最大行数。この値を超えると、分析タスクは複数のクエリに分割されます。 |
| statistic_collect_too_many_version_sleep | LONG | 600000 | 収集タスクが実行されるテーブルにデータバージョンが多すぎる場合の自動収集タスクのスリープ時間。単位はミリ秒。デフォルト値は600000（10分）。 |

統計収集の大部分は自動ジョブに依存できますが、特定の要件がある場合は、ANALYZE TABLEステートメントを実行してタスクを手動で作成したり、CREATE ANALYZEステートメントを実行して自動タスクをカスタマイズすることができます。

### 手動収集

ANALYZE TABLEを使用して手動収集タスクを作成できます。デフォルトでは、手動収集は同期操作です。非同期操作に設定することも可能です。非同期モードでは、ANALYZE TABLEを実行した後、システムはすぐにそのステートメントが成功したかどうかを返します。しかし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。タスクのステータスはSHOW ANALYZE STATUSを実行して確認できます。非同期収集は大量のデータを持つテーブルに適しており、同期収集は少量のデータを持つテーブルに適しています。**手動収集タスクは作成後一度だけ実行されます。手動収集タスクを削除する必要はありません。**

#### 基本統計の手動収集

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [, col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [, property])]
```

パラメータの説明:

- 収集タイプ
  - FULL: 全データ収集を示します。
  - SAMPLE: サンプル収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで全データ収集が使用されます。

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータが指定されていない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力にある`Properties`列で確認できます。

| **PROPERTIES**                | **タイプ** | **デフォルト値** | **説明**                                              |
| ----------------------------- | ---------- | ----------------- | ---------------------------------------------------- |
| statistic_sample_collect_rows | INT        | 200000            | サンプル収集で収集する最小行数。パラメータ値がテーブルの実際の行数を超えた場合、全データ収集が実行されます。 |

例

全データの手動収集

```SQL
-- デフォルト設定を使用してテーブルの全統計を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルト設定を使用してテーブルの全統計を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- デフォルト設定を使用してテーブルの特定の列の統計を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

サンプル収集の手動収集

```SQL
-- デフォルト設定を使用してテーブルの部分統計を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 収集する行数を指定して、テーブルの特定の列の統計を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### ヒストグラムの手動収集

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
[PROPERTIES (property [, property])]
```

パラメータの説明:

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムにはこのパラメータが必要です。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータが指定されていない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`: `N`はヒストグラム収集のバケット数です。指定されていない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。

| **PROPERTIES**                 | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------ | ---------- | ----------------- | ---------------------------------------------------- |
| statistic_sample_collect_rows  | INT        | 200000            | 収集する最小行数。パラメータ値がテーブルの実際の行数を超えた場合、全データ収集が実行されます。 |
| histogram_buckets_size         | LONG       | 64                | ヒストグラムのデフォルトバケット数。                 |
| histogram_mcv_size             | INT        | 100               | ヒストグラムで最も一般的な値(MCV)の数。             |
| histogram_sample_ratio         | FLOAT      | 0.1               | ヒストグラムのサンプリング比率。                    |
| histogram_max_sample_row_count | LONG       | 10000000          | ヒストグラムで収集する最大行数。                    |

ヒストグラムで収集する行数は複数のパラメータによって制御されます。`statistic_sample_collect_rows`とテーブル行数×`histogram_sample_ratio`のうち大きい方の値が採用されます。この数は`histogram_max_sample_row_count`によって指定された値を超えることはできません。値を超えた場合は`histogram_max_sample_row_count`が優先されます。

実際に使用されるプロパティはSHOW ANALYZE STATUSの出力にある`Properties`列で確認できます。

例

```SQL
-- デフォルト設定を使用してv1に対するヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32バケット、32MCV、50%のサンプリング比率でv1とv2に対するヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1, v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### カスタム収集

#### 自動収集タスクのカスタマイズ

CREATE ANALYZE文を使用して自動収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動全データ収集(`enable_collect_full_statistic = false`)を無効にする必要があります。そうしないと、カスタムタスクは有効になりません。

```SQL
-- すべてのデータベースの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [, property])]

-- データベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
[PROPERTIES (property [, property])]

-- テーブルの特定の列の統計を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [, col_name])
[PROPERTIES (property [, property])]
```

パラメータの説明:

- 収集タイプ
  - FULL: 全データ収集を示します。
  - SAMPLE: サンプル収集を示します。
  - 収集タイプが指定されていない場合、デフォルトで全データ収集が使用されます。

- `col_name`: 統計を収集する列。複数の列はコンマ(`,`)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。

| **PROPERTIES**                        | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------- | ---------- | ----------------- | ---------------------------------------------------- |
| statistic_auto_collect_ratio          | FLOAT      | 0.8               | 自動収集の統計が健全かどうかを判断するしきい値。統計の健全性がこのしきい値を下回ると、自動収集がトリガされます。 |
| statistics_max_full_collect_data_size | INT        | 100               | 自動収集でデータを収集する最大パーティションのサイズ。単位はGBです。パーティションがこの値を超えると、全データ収集は行われず、サンプル収集が代わりに実行されます。 |
| statistic_sample_collect_rows         | INT        | 200000            | 収集する最小行数。パラメータ値がテーブルの実際の行数を超えた場合、全データ収集が実行されます。 |

| statistic_exclude_pattern             | String   | null              | ジョブで除外する必要があるデータベースまたはテーブルの名前。ジョブで統計を収集しないデータベースとテーブルを指定できます。これは正規表現パターンであり、マッチする内容は `database.table` です。 |
| statistic_auto_collect_interval       | LONG     |  0                | 自動収集の間隔。単位は秒です。デフォルトでは、StarRocksはテーブルサイズに基づいて `statistic_auto_collect_small_table_interval` または `statistic_auto_collect_large_table_interval` を収集間隔として選択します。分析ジョブを作成する際に `statistic_auto_collect_interval` プロパティを指定した場合、この設定が `statistic_auto_collect_small_table_interval` および `statistic_auto_collect_large_table_interval` よりも優先されます。 |

例

自動全体収集

```SQL
-- すべてのデータベースの完全な統計を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルの完全な統計を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- テーブル内の指定された列の完全な統計を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 指定されたデータベース 'db_name' を除外して、すべてのデータベースの統計を自動的に収集します。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自動サンプル収集

```SQL
-- デフォルト設定でデータベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 指定されたテーブル 'db_name.tbl_name' を除外して、データベース内のすべてのテーブルの統計を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- 統計の健全性と収集する行数を指定して、テーブル内の指定された列の統計を自動的に収集します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### カスタム収集タスクの表示

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE句を使用して結果をフィルタリングできます。このステートメントは、次の列を返します。

| **カラム**   | **説明**                                              |
| ------------ | ------------------------------------------------------------ |
| Id           | 収集タスクのID。                               |
| Database     | データベース名。                                           |
| Table        | テーブル名。                                              |
| Columns      | 列名。                                            |
| Type         | 統計のタイプ。`FULL` と `SAMPLE` を含みます。       |
| Schedule     | スケジュールのタイプ。`SCHEDULE` は自動タスクの場合です。 |
| Properties   | カスタムパラメータ。                                           |
| Status       | タスクのステータス。PENDING、RUNNING、SUCCESS、FAILED を含みます。 |
| LastWorkTime | 最後の収集時刻。                             |
| Reason       | タスクが失敗した理由。タスクの実行が成功した場合は、NULLが返されます。 |

例

```SQL
-- すべてのカスタム収集タスクを表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタム収集タスクを表示します。
SHOW ANALYZE JOB WHERE `Database` = 'test';
```

#### カスタム収集タスクの削除

```SQL
DROP ANALYZE <ID>
```

タスクIDは、SHOW ANALYZE JOB ステートメントを使用して取得できます。

例

```SQL
DROP ANALYZE 266030;
```

## 収集タスクのステータスを表示

SHOW ANALYZE STATUS ステートメントを実行することで、現在のすべてのタスクのステータスを表示できます。このステートメントは、カスタム収集タスクのステータスを表示するために使用することはできません。カスタム収集タスクのステータスを表示するには、SHOW ANALYZE JOBを使用します。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

`LIKE` または `WHERE` を使用して、返す情報をフィルタリングできます。

このステートメントは、次の列を返します。

| **リスト名** | **説明**                                              |
| ------------- | ------------------------------------------------------------ |
| Id            | 収集タスクのID。                               |
| Database      | データベース名。                                           |
| Table         | テーブル名。                                              |
| Columns       | 列名。                                            |
| Type          | 統計のタイプ。FULL、SAMPLE、HISTOGRAM を含みます。 |
| Schedule      | スケジュールのタイプ。`ONCE` は手動、`SCHEDULE` は自動を意味します。 |
| Status        | タスクのステータス。                                      |
| StartTime     | タスクが実行を開始する時刻。                     |
| EndTime       | タスクの実行が終了する時刻。                       |
| Properties    | カスタムパラメータ。                                           |
| Reason        | タスクが失敗した理由。実行が成功した場合はNULLが返されます。 |

## 統計の表示

### 基本統計のメタデータの表示

```SQL
SHOW STATS META [WHERE predicate]
```

このステートメントは、次の列を返します。

| **カラム** | **説明**                                              |
| ---------- | ------------------------------------------------------------ |
| Database   | データベース名。                                           |
| Table      | テーブル名。                                              |
| Columns    | 列名。                                            |
| Type       | 統計のタイプ。`FULL` は完全収集、`SAMPLE` はサンプル収集を意味します。 |
| UpdateTime | 現在のテーブルの最新の統計更新時刻。     |
| Properties | カスタムパラメータ。                                           |
| Healthy    | 統計情報の健全性。                       |

### ヒストグラムのメタデータの表示

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

このステートメントは、次の列を返します。

| **カラム** | **説明**                                              |
| ---------- | ------------------------------------------------------------ |
| Database   | データベース名。                                           |
| Table      | テーブル名。                                              |
| Column     | 列。                                                 |
| Type       | 統計のタイプ。`HISTOGRAM` はヒストグラム用です。 |
| UpdateTime | 現在のテーブルの最新の統計更新時刻。     |
| Properties | カスタムパラメータ。                                           |

## 統計情報の削除

不要な統計情報を削除できます。統計を削除すると、統計のデータとメタデータの両方が削除されます。また、期限切れのキャッシュ内の統計も削除されます。自動収集タスクが進行中の場合、以前に削除した統計が再度収集される可能性があることに注意してください。`SHOW ANALYZE STATUS` を使用して、収集タスクの履歴を表示できます。

### 基本統計の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 収集タスクのキャンセル

KILL ANALYZE ステートメントを使用して、**実行中**の収集タスクをキャンセルできます。これには、手動およびカスタムタスクが含まれます。

```SQL
KILL ANALYZE <ID>
```

手動収集タスクのタスクIDは、SHOW ANALYZE STATUS から取得できます。カスタム収集タスクのタスクIDは、SHOW ANALYZE JOB から取得できます。

## その他のFE構成項目

| **FE構成項目**                        | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_collect_concurrency        | INT      | 3                 | 並行して実行できる手動収集タスクの最大数。デフォルト値は3で、最大3つの手動収集タスクを並行して実行できます。この値を超えると、受信タスクはPENDING状態になり、スケジュールされるのを待機します。 |
| statistic_manager_sleep_time_sec     | LONG     | 60                | メタデータがスケジュールされる間隔。単位: 秒。システムは、この間隔に基づいて次の操作を実行します: 統計を格納するテーブルを作成する。削除された統計を削除する。期限切れの統計を削除する。 |
| statistic_analyze_status_keep_second | LONG     | 259200            | 収集タスクの履歴を保持する期間。単位: 秒。デフォルト値: 259200 (3日)。 |

## セッション変数

`statistic_collect_parallel`: BEs上で実行可能な統計収集タスクの並列度を調整するために使用されます。デフォルト値: 1。この値を増やすことで、収集タスクの速度を上げることができます。

## Hive/Iceberg/Hudi テーブルの統計収集

v3.2.0 以降、StarRocks は Hive、Iceberg、Hudi テーブルの統計収集をサポートしています。構文は StarRocks 内部テーブルの収集と似ています。**ただし、手動および自動の完全収集のみがサポートされており、サンプル収集やヒストグラム収集はサポートされていません。** 収集された統計は、`default_catalog` の `_statistics_` スキーマ内の `external_column_statistics` テーブルに保存されます。これらは Hive Metastore には保存されず、他の検索エンジンと共有することはできません。`default_catalog._statistics_.external_column_statistics` テーブルからデータをクエリして、Hive/Iceberg/Hudi テーブルの統計が収集されているかどうかを確認できます。

以下は `external_column_statistics` から統計データをクエリする例です。

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

### 制限事項

Hive、Iceberg、Hudi テーブルの統計を収集する際に適用される制限事項は以下の通りです:

1. Hive、Iceberg、Hudi テーブルの統計のみを収集できます。
2. 完全収集のみがサポートされています。サンプル収集やヒストグラム収集はサポートされていません。
3. システムによる完全な統計の自動収集を行うには、Analyze ジョブを作成する必要があります。これは StarRocks 内部テーブルの統計をシステムがデフォルトでバックグラウンドで収集するのとは異なります。
4. 自動収集タスクでは、特定のテーブルの統計のみを収集できます。データベース内のすべてのテーブルや外部カタログ内のすべてのデータベースの統計を収集することはできません。
5. 自動収集タスクでは、StarRocks は Hive および Iceberg テーブルのデータが更新されているかどうかを検出し、更新されている場合にのみパーティションのデータが更新された部分の統計を収集します。Hudi テーブルのデータが更新されているかどうかは StarRocks には認識できず、指定されたチェック間隔と収集間隔に基づいて定期的に完全収集を行うのみです。

以下の例は、Hive 外部カタログ下のデータベースで行われます。`default_catalog` から Hive テーブルの統計を収集する場合、テーブルは `[catalog_name.][database_name.]<table_name>` 形式で参照します。

### 手動収集

オンデマンドで Analyze ジョブを作成でき、ジョブは作成直後に実行されます。

#### 手動収集タスクを作成する

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

#### タスクのステータスを確認する

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

#### 統計のメタデータを確認する

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

#### 収集タスクをキャンセルする

実行中の収集タスクをキャンセルします。

構文：

```sql
KILL ANALYZE <ID>
```

タスク ID は SHOW ANALYZE STATUS の出力で確認できます。

### 自動収集

外部データソースのテーブルの統計を自動的に収集するために Analyze ジョブを作成できます。StarRocks は、デフォルトのチェック間隔である 5 分ごとにタスクを実行するかどうかを自動的にチェックします。Hive および Iceberg テーブルについては、テーブル内のデータが更新された場合のみ収集タスクを実行します。

しかし、Hudi テーブルのデータ変更は StarRocks には認識されず、指定されたチェック間隔と収集間隔に基づいて定期的に統計を収集します。Analyze ジョブを作成する際には、以下のプロパティを指定できます：

- statistic_collect_interval_sec

  自動収集中にデータ更新をチェックする間隔。単位: 秒。デフォルト: 5 分。

- statistic_auto_collect_small_table_rows (v3.2 以降)

  自動収集中に外部データソース（Hive、Iceberg、Hudi）のテーブルが小さなテーブルであるかどうかを判断するしきい値。デフォルト: 10000000。

- statistic_auto_collect_small_table_interval

  小さなテーブルの統計を収集する間隔。単位: 秒。デフォルト: 0。

- statistic_auto_collect_large_table_interval

  大きなテーブルの統計を収集する間隔。単位: 秒。デフォルト: 43200 (12 時間)。

自動収集スレッドは `statistic_collect_interval_sec` で指定された間隔でデータ更新をチェックします。テーブル内の行数が `statistic_auto_collect_small_table_rows` より少ない場合、`statistic_auto_collect_small_table_interval` に基づいてそのようなテーブルの統計が収集されます。

テーブル内の行数が `statistic_auto_collect_small_table_rows` を超える場合、`statistic_auto_collect_large_table_interval` に基づいてそのようなテーブルの統計が収集されます。大きなテーブルの統計は、`Last table update time + Collection interval > Current time` の場合にのみ更新されます。これにより、大きなテーブルに対する頻繁な分析タスクが防がれます。

#### 自動収集タスクを作成する

構文：

```sql
CREATE ANALYZE TABLE tbl_name (col_name [, col_name])
[PROPERTIES (property [, property])]
```

例：

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

Query OK, 0 rows affected (0.01 sec)
```

#### 自動収集タスクのステータスを確認する

手動収集と同様です。

#### 統計のメタデータを確認する

手動収集と同様です。

#### 自動収集タスクを確認する

構文：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 収集タスクをキャンセルする

手動収集と同様です。

#### 統計情報を削除する

```sql
DROP STATS tbl_name
```

## 参照

- FE 設定項目を照会するには、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)を実行します。

- FE 設定項目を変更するには、[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を実行します。
