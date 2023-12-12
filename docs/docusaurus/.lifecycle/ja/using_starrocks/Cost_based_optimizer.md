```yaml
displayed_sidebar: "English"
```

# CBOの統計情報を収集

このトピックでは、StarRocksのコストベースの最適化(CBO)の基本的な概念と、CBOのための統計情報を収集して最適なクエリプランを選択する方法について説明します。StarRocks 2.4 では、正確なデータ分散統計情報を収集するためにヒストグラムが導入されます。v3.2.0以降、StarRocksはHive、Iceberg、Hudiテーブルから統計情報を収集することができます。これにより、他のメタストアシステムへの依存が軽減されます。構文は、StarRocksの内部テーブルの収集と類似しています。

## CBOとは

CBOはクエリの最適化に非常に重要です。SQLクエリがStarRocksに到着すると、論理実行プランに解析されます。CBOはこの論理プランを複数の物理実行プランに書き換え、変換します。その後、CBOはプラン内の各オペレータの実行コスト（CPU、メモリ、ネットワーク、I/Oなど）を推定し、最終的な物理プランとして最も低いコストのクエリパスを選択します。

StarRocks CBOは、StarRocks 1.16.0 でリリースされ、1.19以降はデフォルトで有効になっています。Cascadesフレームワークの上に開発されたStarRocks CBOは、さまざまな統計情報に基づいてコストを推定します。これにより、数万の実行プランの中から最も低いコストの実行プランを選択し、複雑なクエリの効率とパフォーマンスを大幅に向上させることができます。

統計情報はCBOにとって重要です。これによってコストの推定が正確かつ有用かどうかが決まります。以下のセクションでは、統計情報の種類、収集方針、統計情報の収集方法、および統計情報の表示方法について詳しく説明します。

## 統計情報の種類

StarRocksは、コスト推定のためのさまざまな統計情報を収集します。

### 基本的な統計情報

デフォルトでは、StarRocksは定期的にテーブルと列の次の基本的な統計情報を収集します。

- row_count：テーブルの行数の合計

- data_size：列のデータサイズ

- ndv：列のカーディナリティ（列内の異なる値の数）

- null_count：列内のNULL値のデータ量

- min：列内の最小値

- max：列内の最大値

全体の統計情報は、`_statistics_.column_statistics`の`column_statistics`に保存されます。このテーブルを見るとテーブルの統計情報をクエリすることができます。次に、このテーブルからの統計情報のクエリを例として示します。

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

StarRocks 2.4 では、ヒストグラムが導入され、基本的な統計情報を補完します。ヒストグラムはデータ表現の効果的な方法と見なされます。データがスキューしているテーブルに対して、ヒストグラムはデータ分散を正確に反映することができます。

StarRocksは等高のヒストグラムを使用します。これはいくつかのバケツに構築されます。各バケツには同じ量のデータが含まれています。頻繁にクエリされるデータ値および選択度に主な影響を与えるデータ値に対して、StarRocksはそれらに対して別々のバケツを割り当てます。より多くのバケツはより正確な推定を意味しますが、わずかなメモリ使用量の増加も起こりえます。ヒストグラムの収集タスクのバケツの数と最頻値（MCV）を調整することができます。 

**ヒストグラムは、データが非常にスキューしており、頻繁にクエリされる列に適用されます。テーブルデータが一様に分布している場合、ヒストグラムは作成する必要がありません。ヒストグラムは、数値、DATE、DATETIME、または文字列型の列にのみ作成することができます。**

現在、StarRocksではヒストグラムの手動収集のみがサポートされています。ヒストグラムは、`_statistics_.histogram_statistics`テーブルに保存されます。次に、このテーブルからの統計情報のクエリを例として示します。

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

## 収集タイプと収集方法

テーブルのデータサイズやデータ分散は常に変化します。統計情報はそのデータ変更を表現するために定期的に更新する必要があります。統計情報収集タスクを作成する前に、ビジネス要件に最適な収集タイプと方法を選択する必要があります。

StarRocksは完全収集およびサンプリング収集の両方を自動および手動で実行できます。デフォルトでは、StarRocksはテーブルの完全な統計情報を自動的に収集します。データの更新を5分ごとにチェックします。データ変更が検出された場合、データの収集が自動的にトリガーされます。自動完全収集を使用したくない場合は、FE設定項目`enable_collect_full_statistic`を`false`に設定し、カスタムの収集タスクをカスタマイズすることができます。

| **収集タイプ** | **収集方法**   | **説明**                                                     | **利点と欠点**                                               |
| -------------- | --------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 完全収集       | 自動/手動       | テーブルの完全な統計情報を収集するためにテーブル全体をスキャンします。統計情報はパーティションごとに収集されます。パーティションがデータ変更がない場合、そのパーティションからのデータは収集されず、リソース消費が削減されます。完全な統計情報は`_statistics_.column_statistics`テーブルに格納されます。 | 利点：統計情報は正確であり、CBOが正確な推定を行うのに役立ちます。欠点：システムリソースを消費し、処理が遅い。2.5 以降、StarRocksでは自動収集期間を指定することができるようになり、リソース消費が削減されました。 |
| サンプリング収集 | 自動/手動       | テーブルの各パーティションからデータを均等に抽出します。各列の基本的な統計情報は1つのレコードとして収集されます。列のカーディナリティ情報（ndv）はサンプリングされたデータに基づいて推定されますが、これは正確ではありません。サンプリングされた統計情報は`_statistics_.table_statistic_v1` テーブルに格納されます。 | 利点：システムリソースを消費し、処理が速い。欠点：統計情報が不完全であり、コストの推定の正確性に影響を与えることがあります。 |

## 統計情報の収集

StarRocksは柔軟な統計情報の収集方法を提供しています。ビジネスシナリオに適した自動、手動、またはカスタム収集を選択することができます。

### 自動完全収集

基本的な統計情報の場合、StarRocksはデフォルトで自動的にテーブルの完全な統計情報を収集します。これには手動操作は不要です。統計情報が収集されていないテーブルについては、スケジューリング期間内にStarRocksは自動的に統計情報を収集します。統計情報が収集されているテーブルについては、StarRocksは定期的にテーブルの総行数と変更された行数を更新し、自動収集をトリガーするかどうかを判断するための情報を保存します。

2.4.5 以降、StarRocksは自動完全収集のための収集期間を指定することができるようになり、これにより自動完全収集によるクラスタのパフォーマンスのジッターが抑制されます。この期間はFEパラメータ `statistic_auto_analyze_start_time` と `statistic_auto_analyze_end_time` で指定されます。

自動収集をトリガーする条件：

- 前回の統計情報収集以降にテーブルデータが変更された場合。

- 表の統計情報の健全性が指定された閾値 (`statistic_auto_collect_ratio`) よりも低い場合。

> 統計情報の健全性を計算するための式：1 - 前回の統計情報収集以降に追加された行の数 / 最小のパーティションの総行数

- パーティションのデータが変更された場合。データが変更されていないパーティションは再び収集されません。

- 収集時間が構成された収集期間の範囲内にある場合（デフォルトの収集期間は終日です）。

自動完全収集はデフォルトで有効になっており、システムによってデフォルトの設定で実行されます。

以下の表にデフォルトの設定を示します。これを変更する必要がある場合は、**ADMIN SET CONFIG**コマンドを実行してください。

| **FE設定項目**                            | **タイプ** | **デフォルト値** | **説明**                                                     |
| ------------------------------------------ | ---------- | ---------------- | ------------------------------------------------------------ |
| enable_statistic_collect                   | BOOLEAN    | TRUE             | 統計情報を収集するかどうか。このスイッチはデフォルトでオンになっています。 |
| enable_collect_full_statistic              | BOOLEAN    | TRUE             | 自動完全収集を有効にするかどうか。このスイッチはデフォルトでオンになっています。 |
| statistic_collect_interval_sec             | LONG       | 300              | 自動収集中のデータ更新をチェックする間隔。単位：秒。         |
| statistic_auto_analyze_start_time | STRING     | 00:00:00 | 自動収集の開始時間。値の範囲：`00:00:00` - `23:59:59`。   |
| statistic_auto_analyze_end_time | STRING       | 23:59:59  | 自動収集の終了時間。値の範囲：`00:00:00` - `23:59:59`。   |
| statistic_auto_collect_small_table_size        | LONG     | 5368709120   | 自動完全収集のためにテーブルが小さいとみなされる閾値。この値より大きいサイズのテーブルは大きいテーブルと見なされ、この値以下のサイズのテーブルは小さいテーブルと見なされます。単位：バイト。 デフォルト値：5368709120（5GB）。 |
| statistic_auto_collect_small_table_interval | LONG     | 0              | 小さいテーブルの完全統計情報を自動的に収集する間隔。単位：秒。 |
| statistic_auto_collect_large_table_interval | LONG    | 43200        | 大規模なテーブルの完全な統計を自動的に収集する間隔。単位：秒。デフォルト値：43200（12時間）。                               |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 統計の自動収集が健全かどうかを判断するための閾値。統計がこの閾値以下の場合、自動収集がトリガーされます。 |
| statistic_max_full_collect_data_size | LONG      | 107374182400      | 自動収集によるデータ収集の最大パーティションのサイズ。単位：バイト。デフォルト値：107374182400（100 GB）。パーティションがこの値を超えると、完全な収集は破棄され、代わりにサンプル収集が実行されます。 |
| statistic_collect_max_row_count_per_query | INT  | 5000000000        | 単一の解析タスクにクエリする行の最大数。この値を超える場合、解析タスクは複数のクエリに分割されます。 |

統計情報の大部分の収集には自動ジョブを利用できますが、特定の統計情報の要件がある場合は、ANALYZE TABLE ステートメントを実行して、タスクを手動で作成するか、CREATE ANALYZE ステートメントを実行して、自動的なタスクをカスタマイズできます。

### 手動収集

ANALYZE TABLE を使用して手動収集タスクを作成できます。デフォルトでは、手動収集は同期操作です。非同期操作にも設定できます。非同期モードでは、ANALYZE TABLE を実行した後、システムはすぐにこのステートメントが成功したかどうかを返します。ただし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。SHOW ANALYZE STATUS を実行して、タスクのステータスを確認できます。非同期収集はデータボリュームが大きいテーブルに適しており、同期収集はデータボリュームが小さいテーブルに適しています。**手動収集タスクは作成後、一度だけ実行されます。手動収集タスクを削除する必要はありません。**

#### 基本統計情報の手動収集

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

パラメータの説明：

- 収集タイプ
  - FULL: 完全な収集を示す。
  - SAMPLE: サンプル収集を示す。
  - 収集タイプが指定されていない場合、デフォルトで完全な収集が使用されます。

- `col_name`: 統計情報を収集する列。複数の列をコンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`: カスタムパラメータ。PROPERTIES が指定されていない場合、`fe.conf` ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUS の出力の `Properties` 列で確認できます。

| **PROPERTIES**                | **Type** | **Default value** | **Description**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプル収集のために収集する最小行数。このパラメータの値が実際のテーブルの行数を超える場合、完全な収集が行われます。 |

例

完全な手動収集

```SQL
-- デフォルトの設定を使用してテーブルの完全な統計を手動で収集。
ANALYZE TABLE tbl_name;

-- デフォルトの設定を使用してテーブルの完全な統計を手動で収集。
ANALYZE FULL TABLE tbl_name;

-- デフォルトの設定を使用して、指定された列のテーブルの統計を手動で収集。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

サンプル収集

```SQL
-- デフォルトの設定を使用してテーブルの一部の統計情報を手動で収集。
ANALYZE SAMPLE TABLE tbl_name;

-- 指定された列のテーブルの統計情報を収集し、収集する行数を指定します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### ヒストグラムの手動収集

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
[PROPERTIES (property [,property])]
```

パラメータの説明：

- `col_name`: 統計情報を収集する列。複数の列をコンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムの場合、このパラメータは必須です。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`: `N` はヒストグラム収集のバケット数です。指定されていない場合、`fe.conf` でのデフォルト値が使用されます。

- PROPERTIES: カスタムパラメータ。PROPERTIES が指定されていない場合、`fe.conf` のデフォルト設定が使用されます。

| **PROPERTIES**                 | **Type** | **Default value** | **Description**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集する最小行数。このパラメータの値がテーブルの実際の行数を超える場合、完全な収集が行われます。|
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトのバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最頻値 (MCV) の数。      |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムの収集の最大行数。       |

ヒストグラムの収集行数は複数のパラメータによって制御されます。`statistic_sample_collect_rows` とテーブルの行数 * `histogram_sample_ratio` の大きい値です。この値は、`histogram_max_sample_row_count` で指定された値を超えることはできません。この値を超える場合は、`histogram_max_sample_row_count` が優先されます。実際に使用されるプロパティは、SHOW ANALYZE STATUS の出力の `Properties` 列で確認できます。

例

```SQL
-- デフォルトの設定を使用して v1 のヒストグラムを手動で収集。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- v1 と v2 のヒストグラムを収集し、32 個のバケット、32 個の MCV、および 50% のサンプリング比率を指定します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### カスタム収集

#### 自動収集タスクのカスタマイズ

CREATE ANALYZE ステートメントを使用して自動収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動完全な収集を無効にする必要があります（`enable_collect_full_statistic = false`）。それ以外の場合、カスタムタスクは有効になりません。

```SQL
-- すべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [,property])]

-- データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
[PROPERTIES (property [,property])]

-- テーブル内の指定された列の統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

パラメータの説明：

- 収集タイプ
  - FULL: 完全な収集を示す。
  - SAMPLE: サンプル収集を示す。
  - 収集タイプが指定されていない場合、デフォルトで完全な収集が使用されます。

- `col_name`: 統計情報を収集する列。複数の列をコンマ（`,`）で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。PROPERTIES が指定されていない場合、`fe.conf` ファイルのデフォルト設定が使用されます。

| **PROPERTIES**                        | **Type** | **Default value** | **Description**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集の統計情報が健全かどうかを判断するための閾値。統計の健全性がこの閾値を下回る場合、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自動収集においてデータを収集する最大パーティションのサイズ。単位：GB。パーティションがこの値を超えると、完全な収集は破棄され、サンプル収集が行われます。 |
| statistic_sample_collect_rows         | INT      | 200000            | 収集する最小行数。このパラメータの値が実際のテーブルの行数を超える場合、完全な収集が行われます。 |
| statistic_exclude_pattern             | String   | null              | ジョブで除外する必要のあるデータベースやテーブルの名前。ジョブで統計情報を収集しないデータベースやテーブルを指定できます。これは正規表現パターンであり、一致する内容は `database.table` となります。 |
| statistic_auto_collect_interval       | LONG   |  0      | 自動収集の間隔。単位：秒。デフォルトでは、StarRocksはテーブルサイズに基づいて `statistic_auto_collect_small_table_interval` または `statistic_auto_collect_large_table_interval` を収集間隔として選択します。分析ジョブを作成する際に `statistic_auto_collect_interval` プロパティを指定した場合、この設定は `statistic_auto_collect_small_table_interval` および `statistic_auto_collect_large_table_interval` より優先されます。 |

例

自動完全収集

```SQL
-- すべてのデータベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルの完全な統計情報を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- テーブル内の指定された列の統計情報を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 指定されたデータベース「db_name」を除くすべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自動サンプル収集

```SQL
-- デフォルトの設定で、データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- 指定されたテーブル「db_name.tbl_name」を除くデータベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- テーブル内の指定された列の統計情報を収集し、統計の健全性と収集する行数を指定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### カスタム収集タスクを表示

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE句を使用して結果をフィルタリングできます。このステートメントは、次の列を返します。

| **列**      | **説明**                                 |
| ----------- | ---------------------------------------- |
| Id          | 収集タスクのID。                         |
| Database    | データベース名。                         |
| Table       | テーブル名。                             |
| Columns     | 列名。                                  |
| Type        | FULLおよびSAMPLEを含む統計の種類。     |
| Schedule    | スケジューリングの種類。自動タスクの場合は`SCHEDULE`です。 |
| Properties  | カスタムパラメータ。                     |
| Status      | タスクのステータス。PENDING、RUNNING、SUCCESS、FAILEDを含みます。 |
| LastWorkTime| 最後の収集時間。                        |
| Reason      | タスクが失敗した理由。タスクの実行が成功した場合はNULLが返されます。 |

例

```SQL
-- すべてのカスタム収集タスクを表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタム収集タスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

#### カスタム収集タスクの削除

```SQL
DROP ANALYZE <ID>
```

タスクIDは、SHOW ANALYZE JOBステートメントを使用して取得できます。

例

```SQL
DROP ANALYZE 266030;
```

## 収集タスクのステータスを表示

SHOW ANALYZE STATUSステートメントを実行して、現在のすべてのタスクのステータスを表示できます。このステートメントは、カスタム収集タスクのステータスを表示するためには使用できません。カスタム収集タスクのステータスを表示するには、SHOW ANALYZE JOBを使用してください。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

`LILEまたはWHERE`を使用して情報をフィルタリングできます。

このステートメントは、次の列を返します。

| **リスト名** | **説明**                   |
| ------------ | -------------------------- |
| Id           | 収集タスクのID。           |
| Database     | データベース名。           |
| Table        | テーブル名。               |
| Columns      | 列名。                    |
| Type         | FULL、SAMPLE、HISTOGRAMを含む統計の形式。 |
| Schedule     | スケジューリングのタイプ。 `ONCE` は手動、 `SCHEDULE` は自動です。 |
| Status       | タスクのステータス。        |
| StartTime    | タスクの開始時間。          |
| EndTime      | タスクの終了時間。          |
| Properties   | カスタムパラメータ。         |
| Reason       | タスクが失敗した理由。実行が成功した場合は、NULLが返されます。 |

## 統計の表示

### 基本統計のメタデータを表示

```SQL
SHOW STATS META [WHERE predicate]
```

このステートメントは、次の列を返します。

| **列**       | **説明**                                |
| ------------ | --------------------------------------- |
| Database     | データベース名。                        |
| Table        | テーブル名。                            |
| Columns      | 列名。                                 |
| Type         | 統計のタイプ。 `FULL` は完全な収集、`SAMPLE` はサンプリング収集を意味します。 |
| UpdateTime   | 現在のテーブルの最新統計更新時間。              |
| Properties   | カスタムパラメータ。                        |
| Healthy      | 統計情報の健全性。                        |

### ヒストグラムのメタデータを表示

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

このステートメントは、次の列を返します。

| **列**       | **説明**                                |
| ------------ | --------------------------------------- |
| Database     | データベース名。                        |
| Table        | テーブル名。                            |
| Column       | 列。                                    |
| Type         | 統計の種類。ヒストグラムの場合は `HISTOGRAM` です。 |
| UpdateTime   | 現在のテーブルの最新統計更新時間。              |
| Properties   | カスタムパラメータ。                        |

## 統計の削除

不要な統計情報を削除できます。統計情報を削除すると、データと統計情報のメタデータ、および期限切れのキャッシュ内の統計情報が削除されます。ただし、自動収集タスクが実行中の場合、以前に削除された統計情報は再収集される場合があります。`SHOW ANALYZE STATUS`を使用して収集タスクの履歴を表示できます。

### 基本統計の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## 収集タスクのキャンセル

KILL ANALYZEステートメントを使用して、**実行中の**収集タスク（手動およびカスタムタスクを含む）をキャンセルできます。

```SQL
KILL ANALYZE <ID>
```

手動収集タスクのタスクIDは、SHOW ANALYZE STATUSから取得できます。カスタム収集タスクのタスクIDは、SHOW ANALYZE SHOW ANALYZE JOBから取得できます。

## その他のFE構成アイテム

| **FE** **構成アイテム**                | **タイプ** | **デフォルト値** | **説明**                                                   |
| ------------------------------------- | ---------- | ---------------- | ------------------------------------------------------------ |
| statistic_collect_concurrency        | INT      | 3                | 並列で実行できる手動収集タスクの最大数。値のデフォルト値は3で、最大で3つの手動収集タスクを並列に実行できます。この値を超過すると、新しいタスクはPENDING状態になり、スケジュールされるのを待ちます。 |
| statistic_manager_sleep_time_sec     | LONG     | 60               | メタデータがスケジュールされる間隔。単位：秒。この間隔に基づいてシステムは次の操作を実行します。統計情報を保存するテーブルを作成します。削除された統計情報を削除します。期限切れの統計情報を削除します。 |
| statistic_analyze_status_keep_second | LONG     | 259200           | 収集タスクの履歴を保持する期間。単位：秒。デフォルト値：259200（3日）。 |

## セッション変数

`statistic_collect_parallel`：BEで実行できる統計収集タスクの並列度を調整するために使用します。デフォルト値：1。この値を増やすと、収集タスクを高速化できます。

## Hive/Iceberg/Hudiテーブルの統計の収集

v3.2.0以降、StarRocksはHive、Iceberg、およびHudiテーブルの統計を収集できます。構文はStarRocksの内部テーブルの統計収集と似ています。しかし、サンプリング収集やヒストグラム収集はサポートされていません。収集された統計情報は、`default_catalog`の`_statistics_`内の`external_column_statistics`テーブルに格納されます。これらはHive Metastoreに保存されず、他の検索エンジンで共有することはできません。`default_catalog._statistics_.external_column_statistics` テーブルから統計情報が収集されているかを確認するために、クエリとして実行できます。

次は、`external_column_statistics`から統計データをクエリする例です。

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

### 制限

Hive、Iceberg、Hudiテーブルの統計を収集する際には、次の制限が適用されます：
1. 完全なコレクションのみがサポートされています。サンプリングされたコレクションやヒストグラムコレクションはサポートされていません。
2. Hive、Iceberg、Hudiテーブルの統計情報のみを収集できます。
3. 自動収集タスクでは、特定のテーブルの統計情報のみを収集できます。データベース内のすべてのテーブルの統計情報や外部カタログ内のすべてのデータベースの統計情報を収集することはできません。
4. 自動収集タスクでは、StarRocksはHiveおよびIcebergテーブルのデータが更新されたかどうかを検出し、更新された場合はデータが更新されたパーティションの統計情報のみを収集できます。ただし、StarRocksはHudiテーブルのデータが更新されたかどうかを感知することができず、定期的な完全なコレクションのみを実行できます。

以下の例は、Hive外部カタログ内のデータベースで発生します。`default_catalog`からHiveテーブルの統計情報を収集したい場合は、`[catalog_name.][database_name.]<table_name>`形式でテーブルを参照してください。

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

#### タスクのステータスを表示

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

#### 統計情報のメタデータを表示

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

実行中の収集タスクをキャンセルします。

構文：

```sql
KILL ANALYZE <ID>
```

SHOW ANALYZE STATUSの出力でタスクIDを表示できます。

### 自動収集

自動収集タスクを作成すると、StarRocksはデフォルトのチェック間隔である5分間隔で自動的にタスクを実行するかどうかを自動的にチェックします。StarRocksはHiveおよびIcebergテーブルのデータが更新された場合にのみ収集タスクを実行します。ただし、Hudiテーブルのデータの変更は感知できず、StarRocksはユーザーが指定したチェック間隔および収集間隔に基づいて統計情報を定期的に収集します。以下のFEパラメータを使用できます：

- statistic_collect_interval_sec

  自動収集中のデータ更新をチェックする間隔。単位：秒。デフォルト：5分間隔。

- statistic_auto_collect_small_table_rows (v3.2以降)

  自動収集中に外部データソース（Hive、Iceberg、Hudi）のテーブルが小規模かどうかを判断するための閾値。デフォルト：10000000。

- statistic_auto_collect_small_table_interval

  小規模なテーブルの統計情報を収集する間隔。単位：秒。デフォルト：0。

- statistic_auto_collect_large_table_interval

  大規模なテーブルの統計情報を収集する間隔。単位：秒。デフォルト：43200（12時間）。

自動収集スレッドは、`statistic_collect_interval_sec`で指定された間隔でデータの更新をチェックします。テーブルの行数が`statistic_auto_collect_small_table_rows`よりも少ない場合、それらのテーブルの統計情報を`statistic_auto_collect_small_table_interval`に基づいて収集します。

テーブルの行数が`statistic_auto_collect_small_table_rows`を超える場合、それらのテーブルの統計情報を`statistic_auto_collect_large_table_interval`に基づいて収集します。大規模なテーブルの統計情報は、`最終テーブル更新時刻 + 収集間隔 > 現在時刻`となる場合にのみ更新されます。これにより、大規模なテーブルに対する頻繁な分析タスクが回避されます。

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

Query OK, 0 rows affected (0.01 sec)
```

#### 自動収集タスクのステータスを表示

手動収集と同様です。

#### 統計情報のメタデータを表示

手動収集と同様です。

#### 自動収集タスクの表示

構文：

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例：

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 収集タスクのキャンセル

手動収集と同様です。

### 統計情報の削除

```sql
DROP STATS tbl_name
```

## 参照

- FE構成項目をクエリするには、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)を実行してください。

- FE構成項目を変更するには、[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を実行してください。