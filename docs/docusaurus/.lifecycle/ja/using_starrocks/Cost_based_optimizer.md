---
displayed_sidebar: "Japanese"
---

# CBOの統計情報を収集する

このトピックでは、StarRocksコストベース最適化（CBO）の基本的な概念と、CBOが最適なクエリプランを選択するためにCBOの統計情報を収集する方法について説明します。StarRocks 2.4では、正確なデータ分布統計を収集するためにヒストグラムが導入されています。v3.2.0から、StarRocksはHive、Iceberg、Hudiテーブルから統計情報を収集することがサポートされており、他のメタストアシステムへの依存が低減されています。この構文は、StarRocks内部テーブルの収集と類似しています。

## CBOとは

CBOはクエリの最適化において重要です。SQLクエリがStarRocksに到着すると、それは論理実行プランに解析されます。CBOはその論理プランを複数の物理実行プランに書き換え、変換します。次に、CBOは計画内の各演算子の実行コスト（CPU、メモリー、ネットワーク、I/Oなど）を推定し、最終的な物理プランとして最も低いコストのクエリパスを選択します。

StarRocks CBOはStarRocks 1.16.0で開始され、1.19以降はデフォルトで有効になっています。Cascadesフレームワークをベースに開発されたStarRocks CBOは、さまざまな統計情報に基づいてコストを推定します。数万の実行計画から低いコストの実行計画を選択でき、複雑なクエリの効率とパフォーマンスを大幅に向上させることができます。

統計情報はCBOにとって重要です。これによってコストの推定が正確で役立つかどうかが決まります。以下では、統計情報の種類、収集ポリシー、統計情報の収集方法、および統計情報の表示方法について詳しく説明します。

## 統計情報の種類

StarRocksはコスト推定のための様々な統計情報を収集します。

### 基本統計情報

デフォルトで、StarRocksは定期的にテーブルと列の以下の基本統計情報を収集します。

- row_count: テーブル内の合計行数

- data_size: 列のデータサイズ

- ndv: 列のカーディナリティ、つまり列の中の一意な値の数

- null_count: 列内のNULL値のデータ量

- min: 列内の最小値

- max: 列内の最大値

完全な統計情報は`_statistics_.column_statistics`の`column_statistics`に格納されており、このテーブルを表示してテーブルの統計情報をクエリできます。以下は、このテーブルから統計データをクエリする例です。

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

StarRocks 2.4では、基本統計情報に加えてヒストグラムが導入されています。ヒストグラムはデータ表現の効果的な手段と考えられています。データがスキューしているテーブルにおいては、ヒストグラムがデータ分布を正確に反映できます。

StarRocksは均一な高さのヒストグラムを使用し、それぞれのバケツに一定量のデータが含まれるように構築されています。クエリが頻繁に行われ、選択度に大きな影響を与えるデータ値に対しては、それぞれ別のバケツが割り当てられます。バケツが多いほど推定が正確になりますが、若干のメモリ使用量の増加も引き起こす可能性があります。ヒストグラムの収集タスクでは、バケツの数や最も一般的な値（MCV）を調整することができます。

**ヒストグラムはデータが高度にスキューしており、頻繁にクエリされる列に適しています。テーブルデータが均一に分布している場合は、ヒストグラムを作成する必要はありません。ヒストグラムは、数値、DATE、DATETIME、または文字列型の列にのみ作成できます。**

現在のところ、StarRocksはヒストグラムの手動収集のみをサポートしています。ヒストグラムは`_statistics_.histogram_statistics`テーブルに格納されており、このテーブルから統計情報データをクエリする例は次のとおりです。

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

テーブル内のデータサイズやデータ分布は変化する可能性があります。それらのデータ変更を表すために定期的に統計情報を更新する必要があります。統計情報の収集タスクを作成する前に、ビジネス要件に最も適した収集種類と方法を選択する必要があります。

StarRocksは完全な収集とサンプリング収集の両方を自動的および手動でサポートしています。デフォルトでStarRocksはテーブルの完全な統計情報を自動的に収集します。データ更新を5分ごとにチェックします。データ変更が検出された場合、データの収集が自動的にトリガーされます。自動完全収集を使用しない場合は、FE構成項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズすることができます。

| **収集タイプ** | **収集方法** | **説明**                                              | **利点と欠点**                               |
| ------------------- | --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 完全収集     | 自動/手動      | テーブルの完全な統計情報を収集するためにテーブル全体をスキャンします。統計情報はパーティションで収集されます。パーティションにデータ変更がない場合、そのパーティションからデータは収集されません。完全な統計情報は`_statistics_.column_statistics`テーブルに格納されます。 | 利点: 統計情報は正確であり、これによりCBOが正確な推定を行うのに役立ちます。欠点: システムリソースを消費し、処理が遅くなります。2.5以降では、StarRocksでは自動収集期間を指定することが可能で、リソースの消費を削減できます。 |
| サンプリング収集  | 自動/手動      | テーブルの各パーティションから均等に`N`行のデータを抽出します。統計情報はテーブルごとに収集されます。各列の基本的な統計情報が1レコードとして格納されます。列のカーディナリティ情報（ndv）はサンプリングデータに基づいて推定されるため、正確ではありません。サンプリング統計情報は`_statistics_.table_statistic_v1`テーブルに格納されます。 | 利点: システムリソースを少なく消費し、処理が速くなります。欠点: 統計情報が完全ではないため、コストの推定の正確性に影響を与える可能性があります。 |

## 統計情報の収集

StarRocksでは柔軟な統計情報の収集方法を提供しています。ビジネスシナリオに応じて自動、手動、カスタムの収集方法を選択することができます。

### 自動完全収集

基本統計情報については、StarRocksはデフォルトで自動的にテーブルの完全な統計情報を収集します。統計情報が収集されていないテーブルに対しては、統計情報はスケジュールされた期間内に自動的に収集されます。統計情報が収集されているテーブルに対しては、StarRocksは定期的にテーブル内の合計行数と変更行数を更新し、自動的に収集をトリガーするかどうかを定期的に判断します。

2.4.5以降、StarRocksでは自動完全収集のための収集期間を指定することが可能で、自動完全収集によるクラスタの性能の揺れが防止されます。この期間は、FEパラメータ`statistic_auto_analyze_start_time`および`statistic_auto_analyze_end_time`で指定されます。

自動収集がトリガーされる条件：

- 前回の統計情報収集以降にテーブルデータが変更されている場合。

- テーブルの統計情報の健全性が指定されたしきい値（`statistic_auto_collect_ratio`）以下である場合。

> 統計情報の健康度を計算するための式: 1 - 前回の統計情報収集以降の追加行数/最小のパーティション内の合計行数

- パーティションデータが変更されている。データが変更されていないパーティションでは再収集されません。

- 収集時間が構成された収集期間の範囲内である。（デフォルトの収集期間は一日中です。）

自動完全収集はデフォルトで有効になっており、システムによってデフォルトの設定で実行されます。

以下の表はデフォルトの設定を説明しています。これを変更する必要がある場合は、**ADMIN SET CONFIG**コマンドを実行してください。

| **FE構成項目**         | **タイプ** | **デフォルト値** | **説明**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| enable_statistic_collect              | BOOLEAN  | TRUE              | 統計情報の収集を行うかどうか。このスイッチはデフォルトでオンになっています。 |
| enable_collect_full_statistic         | BOOLEAN  | TRUE              | 自動的完全収集を有効にするかどうか。このスイッチはデフォルトでオンになっています。 |
| statistic_collect_interval_sec        | LONG     | 300               | 自動収集中のデータ更新をチェックする間隔。単位: 秒。 |
| statistic_auto_analyze_start_time | STRING      | 00:00:00   | 自動収集の開始時間。値の範囲: `00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | STRING      | 23:59:59  | 自動収集の終了時間。値の範囲: `00:00:00` - `23:59:59`。 |
| statistic_auto_collect_small_table_size     | LONG    | 5368709120   | 自動的完全収集のための小さいテーブルの判定しきい値。この値より大きいサイズのテーブルは大きなテーブルと見なされ、この値以下のサイズのテーブルは小さなテーブルと見なされます。単位: バイト。デフォルト値: 5368709120 (5 GB)。                         |
| statistic_auto_collect_small_table_interval | LONG    | 0         | 小さいテーブルの完全な統計情報を自動収集する間隔。単位: 秒。                              |
| statistic_auto_collect_large_table_interval | LONG | 43200 | 大きなテーブルの統計情報を自動的に収集する間隔。単位: 秒。デフォルト値: 43200 (12時間)。 |
| statistic_auto_collect_ratio | FLOAT | 0.8 | 統計情報を自動的に収集するかどうかを判断するための閾値。統計情報の健全性がこの閾値以下の場合、自動収集がトリガーされます。 |
| statistic_max_full_collect_data_size | LONG | 107374182400 | 自動収集のためにデータを収集する最大パーティションのサイズ。単位: バイト。デフォルト値: 107374182400 (100GB)。パーティションがこの値を超えると、フルコレクションは破棄され、代わりにサンプルコレクションが実行されます。 |
| statistic_collect_max_row_count_per_query | INT | 5000000000 | 単一の分析タスクにクエリする最大行数。この値を超える場合、分析タスクは複数のクエリに分割されます。 |

ほとんどの統計情報収集には自動ジョブを信頼できますが、特定の統計情報要件がある場合は、ANALYZE TABLEステートメントを実行してタスクを手動で作成したり、CREATE ANALYZEステートメントを実行して自動タスクをカスタマイズすることができます。

### 手動収集

ANALYZE TABLEを使用して手動収集タスクを作成できます。デフォルトでは、手動収集は同期操作です。非同期操作にも設定できます。非同期モードでは、ANALYZE TABLEを実行した後、システムはすぐにこのステートメントが成功したかどうかを返します。ただし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。SHOW ANALYZE STATUSを実行することでタスクの状態を確認できます。非同期収集はデータ量が多いテーブルに適しており、同期収集はデータ量が少ないテーブルに適しています。**手動収集タスクは作成後に1回だけ実行されます。手動収集タスクを削除する必要はありません。**

#### 基本統計情報の手動収集

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES (property [,property])]
```

パラメータの説明:

- 収集タイプ
  - FULL: フルコレクションを指定します。
  - SAMPLE: サンプルコレクションを指定します。
  - 収集タイプが指定されていない場合、デフォルトでフルコレクションが使用されます。

- `col_name`: 統計情報を収集する列。複数の列をカンマ(,)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]: 手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

| **PROPERTIES**                | **Type** | **Default value** | **Description**                                              |
| ----------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows | INT      | 200000            | サンプル収集のために収集する最小行数。このパラメータの値がテーブルの実際の行数を超えると、フルコレクションが実行されます。 |

例

手動フルコレクション

```SQL
-- デフォルトの設定を使用してテーブルのフル統計情報を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルトの設定を使用してテーブルのフル統計情報を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- デフォルトの設定を使用してテーブルの指定された列の統計情報を手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

手動サンプルコレクション

```SQL
-- デフォルトの設定を使用してテーブルの部分的な統計情報を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- デフォルトの設定を使用してテーブルの指定された列の統計情報を指定された行数と共に手動で収集します。
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

パラメータの説明:

- `col_name`: 統計情報を収集する列。複数の列をカンマ(,)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。ヒストグラムの場合、このパラメータは必須です。

- [WITH SYNC | ASYNC MODE]: ヒストグラムの手動収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、デフォルトで同期収集が使用されます。

- `WITH N BUCKETS`: `N` はヒストグラムの収集のためのバケット数です。指定しない場合、`fe.conf`のデフォルト値が使用されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。

| **PROPERTIES**                 | **Type** | **Default value** | **Description**                                              |
| ------------------------------ | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_sample_collect_rows  | INT      | 200000            | 収集するための最小行数。このパラメータの値がテーブルの実際の行数を超えると、フルコレクションが実行されます。 |
| histogram_buckets_size         | LONG     | 64                | ヒストグラムのデフォルトのバケット数。                   |
| histogram_mcv_size             | INT      | 100               | ヒストグラムの最頻値(MCV)の数。                          |
| histogram_sample_ratio         | FLOAT    | 0.1               | ヒストグラムのサンプリング比率。                          |
| histogram_max_sample_row_count | LONG     | 10000000          | ヒストグラムの収集のために収集する最大行数。             |

ヒストグラムのための収集する行数は複数のパラメータで制御されます。`statistic_sample_collect_rows` とテーブルの行数 * `histogram_sample_ratio` の大きい方をとります。この数値は`histogram_max_sample_row_count`で指定された値を超えることはできません。この値を超えた場合は、`histogram_max_sample_row_count`が優先されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で確認できます。

例

```SQL
-- デフォルトの設定を使用してv1のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32バケット、32 MCV、50%のサンプリング比率でv1とv2のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### カスタム収集

#### 自動収集タスクをカスタマイズ

CREATE ANALYZEステートメントを使用して自動収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動フルコレクションを無効にする必要があります (`enable_collect_full_statistic = false`)。そうしないと、カスタムタスクが有効になりません。

```SQL
-- すべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL [PROPERTIES (property [,property])]

-- データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
[PROPERTIES (property [,property])]

-- テーブルの指定された列の統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

パラメータの説明:

- 収集タイプ
  - FULL: フルコレクションを指定します。
  - SAMPLE: サンプルコレクションを指定します。
  - 収集タイプが指定されていない場合、デフォルトでフルコレクションが使用されます。

- `col_name`: 統計情報を収集する列。複数の列をカンマ(,)で区切ります。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`: カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。

| **PROPERTIES**                        | **Type** | **Default value** | **Description**                                              |
| ------------------------------------- | -------- | ----------------- | ------------------------------------------------------------ |
| statistic_auto_collect_ratio          | FLOAT    | 0.8               | 自動収集の統計情報の健全性を判断するための閾値。統計情報の健全性がこの閾値以下の場合、自動収集がトリガーされます。 |
| statistics_max_full_collect_data_size | INT      | 100               | 自動収集のためにデータを収集する最大パーティションのサイズ。単位: GB。パーティションがこの値を超えると、フルコレクションは破棄され、サンプルコレクションが実行されます。 |
| statistic_sample_collect_rows         | INT      | 200000            | 収集するための最小行数。このパラメータの値がテーブルの実際の行数を超えると、フルコレクションが実行されます。 |
| statistic_exclude_pattern             | String   | null              | ジョブで統計を収集しないデータベースやテーブルの名前。ジョブで統計情報を収集しないデータベースやテーブルを指定できます。これは正規表現パターンであり、一致する内容は `database.table` です。 |
| statistic_auto_collect_interval       | LONG   |  0      | 自動収集の間隔。単位：秒。デフォルトでは、StarRocksはテーブルのサイズに基づいて`statistic_auto_collect_small_table_interval`または`statistic_auto_collect_large_table_interval`を収集間隔として選択します。分析ジョブを作成する際に`statistic_auto_collect_interval`プロパティを指定した場合、この設定は`statistic_auto_collect_small_table_interval`および`statistic_auto_collect_large_table_interval`より優先されます。 |

例

自動フルコレクション

```SQL
-- すべてのデータベースのフル統計を自動収集します。
CREATE ANALYZE ALL;

-- データベースのフル統計を自動収集します。
CREATE ANALYZE DATABASE db_name;

-- データベース内のすべてのテーブルのフル統計を自動収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- テーブルの指定された列の統計を自動収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 指定されたデータベース 'db_name' を除くすべてのデータベースの統計を自動収集します。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自動サンプル収集

```SQL
-- デフォルト設定でデータベース内のすべてのテーブルの統計を自動収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- データベース内のすべてのテーブルの統計を自動収集しますが、テーブル 'db_name.tbl_name' を除外します。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- テーブルの指定された列の統計を収集し、統計の健全性と収集する行数を指定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### カスタムコレクションタスクを表示

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE句を使用して結果をフィルタリングできます。このステートメントは、次の列を返します。

| **列名**      | **説明**                         |
| ------------- | -------------------------------- |
| Id           | コレクションタスクのID。 |
| Database     | データベース名。 |
| Table        | テーブル名。 |
| Columns      | 列名。 |
| Type         | 統計のタイプ。「FULL」と「SAMPLE」を含む。 |
| Schedule     | スケジューリングのタイプ。自動タスクの場合は`SCHEDULE`です。 |
| Properties   | カスタムパラメータ。 |
| Status       | タスクのステータス。「PENDING」、「RUNNING」、「SUCCESS」、および「FAILED」を含む。 |
| LastWorkTime | 前回の収集時間。 |
| Reason       | タスクの失敗理由。「NULL」は、タスクの実行が成功した場合に返されます。 |

例

```SQL
-- すべてのカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタムコレクションタスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

#### カスタムコレクションタスクを削除

```SQL
DROP ANALYZE <ID>
```

タスクIDはSHOW ANALYZE JOBステートメントを使用して取得できます。

例

```SQL
DROP ANALYZE 266030;
```

## コレクションタスクのステータスを表示

`SHOW ANALYZE STATUS`ステートメントを実行して、現在のすべてのタスクのステータスを表示できます。このステートメントを使用してカスタムコレクションタスクのステータスを表示することはできません。カスタムコレクションタスクのステータスを表示するには、`SHOW ANALYZE JOB`を使用してください。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

`LIKE`または`WHERE`を使用して情報をフィルタリングできます。

このステートメントを使用して、次の列を返します。

| **リスト名** | **説明**                         |
| ------------- | -------------------------------- |
| Id           | コレクションタスクのID。 |
| Database     | データベース名。 |
| Table        | テーブル名。 |
| Columns      | 列名。 |
| Type         | 統計のタイプ。「FULL」、「SAMPLE」、「HISTOGRAM」を含む。 |
| Schedule     | スケジューリングのタイプ。`ONCE`は手動、`SCHEDULE`は自動です。 |
| Status       | タスクのステータス。 |
| StartTime     | タスクの開始時間。 |
| EndTime       | タスクの終了時間。 |
| Properties    | カスタムパラメータ。 |
| Reason        | タスクの失敗理由。実行が成功した場合は「NULL」が返されます。 |

## 統計を表示

### 基本統計のメタデータを表示

```SQL
SHOW STATS META [WHERE predicate]
```

このステートメントを使用して、次の列を返します。

| **列名** | **説明**                         |
| ---------- | -------------------------------- |
| Database   | データベース名。 |
| Table      | テーブル名。 |
| Columns    | 列名。 |
| Type       | 統計のタイプ。`FULL`はフルコレクション、「SAMPLE」はサンプルコレクションを意味します。 |
| UpdateTime | 現在のテーブルの最新の統計更新時間。 |
| Properties | カスタムパラメータ。 |
| Healthy    | 統計情報の健全性。 |

### ヒストグラムのメタデータを表示

```SQL
SHOW HISTOGRAM META [WHERE predicate]
```

このステートメントを使用して、次の列を返します。

| **列名** | **説明**                         |
| ---------- | -------------------------------- |
| Database   | データベース名。 |
| Table      | テーブル名。 |
| Column     | 列。 |
| Type       | 統計のタイプ。「HISTOGRAM」の値がヒストグラムです。 |
| UpdateTime | 現在のテーブルの最新の統計更新時間。 |
| Properties | カスタムパラメータ。 |

## 統計を削除

不要な統計情報を削除できます。統計を削除すると、統計データとメタデータが削除されます。また、期限切れのキャッシュ内の統計も削除されます。自動収集タスクが進行中の場合、以前に削除された統計が再収集される場合があります。コレクションタスクの履歴を表示するには`SHOW ANALYZE STATUS`を使用できます。

### 基本統計を削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムを削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name]
```

## コレクションタスクのキャンセル

**実行中の**コレクションタスク、手動およびカスタムタスクをキャンセルするには、`KILL ANALYZE`ステートメントを使用できます。

```SQL
KILL ANALYZE <ID>
```

手動コレクションタスクのタスクIDは`SHOW ANALYZE STATUS`から取得できます。カスタムコレクションタスクのタスクIDは`SHOW ANALYZE SHOW ANALYZE JOB`から取得できます。

## その他のFE構成項目

| **FE構成項目**        | **タイプ** | **デフォルト値** | **説明**                         |
| ------------------------------------ | -------- | ----------------- | -------------------------------- |
| statistic_collect_concurrency        | INT      | 3                 | 実行できる並列マニュアルコレクションタスクの最大数。デフォルト値は3であり、最大3つのマニュアルコレクションタスクを並列に実行できます。この値を超えると、着信タスクは予約中の状態になり、スケジュール待ちになります。 |
| statistic_manager_sleep_time_sec     | LONG     | 60                | メタデータのスケジュール間隔。単位：秒。システムはこの間隔に基づいて以下の操作を実行します：統計情報を保存するテーブルを作成します。削除された統計情報を削除します。期限切れの統計情報を削除します。 |
| statistic_analyze_status_keep_second | LONG     | 259200            | コレクションタスクの履歴を保持する期間。単位：秒。デフォルト値：259200（3日）。 |

## セッション変数

`statistic_collect_parallel`：BEで実行できる統計収集タスクの並列性を調整するために使用されます。デフォルト値は`1`です。この値を増やすと、コレクションタスクの実行が速くなります。

## Hive/Iceberg/Hudiテーブルの統計を収集

v3.2.0から、StarRocksはHive、Iceberg、およびHudiテーブルの統計を収集できるようになりました。構文は、StarRocks内部テーブルの統計と同様です。**ただし、手動および自動のフルコレクションのみをサポートします。サンプルコレクションやヒストグラムコレクションはサポートされていません。** 収集された統計情報は、`default_catalog`の`_statistics_`内の`external_column_statistics`テーブルに保存されます。これらはHiveメタストアに保存されず、他の検索エンジンで共有することはできません。Hive/Iceberg/Hudiテーブルの統計が収集されているかどうかを確認するには、`default_catalog._statistics_.external_column_statistics`テーブルからデータをクエリできます。

以下は`external_column_statistics`から統計データをクエリする例です。

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

Hive、Iceberg、Hudiテーブルの統計を収集する際の制限事項は次のとおりです：
1. 完全なコレクションのみサポートされています。サンプリングコレクションやヒストグラムコレクションはサポートされていません。
2. Hive、Iceberg、Hudi テーブルの統計情報の収集が可能です。
3. 自動収集タスクでは、特定のテーブルの統計情報のみを収集できます。データベース内のすべてのテーブルの統計情報や外部カタログ内のすべてのデータベースの統計情報を収集することはできません。
4. 自動収集タスクでは、StarRocks はHive と Iceberg テーブルのデータが更新されたかどうかを検知し、更新されたデータのみのパーティションの統計情報を収集します。StarRocks は Hudi テーブルのデータが更新されたかどうかを検知できず、定期的な完全収集のみが行えます。

以下の例は、Hive 外部カタログ内のデータベースで発生します。`default_catalog` から Hive テーブルの統計情報を収集する場合は、`[catalog_name.][database_name.]<table_name>` 形式でテーブルを参照してください。

### 手動収集

#### 手動収集タスクの作成

構文:

```sql
ANALYZE [FULL] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
[PROPERTIES(property [,property])]
```

例:

```sql
ANALYZE TABLE ex_hive_tbl(k1);
+----------------------------------+---------+----------+----------+
| Table                            | Op      | Msg_type | Msg_text |
+----------------------------------+---------+----------+----------+
| hive_catalog.tn_test.ex_hive_tbl | analyze | status   | OK       |
+----------------------------------+---------+----------+----------+
```

#### タスクのステータスの表示

構文:

```sql
SHOW ANALYZE STATUS [LIKE | WHERE predicate]
```

例:

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

#### 統計情報のメタデータの表示

構文:

```sql
SHOW STATS META [WHERE predicate]
```

例:

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

構文:

```sql
KILL ANALYZE <ID>
```

SHOW ANALYZE STATUS の出力でタスクIDを確認できます。

### 自動収集

自動収集タスクを作成すると、StarRocks はデフォルトの5分間隔でタスクを実行するかどうかを自動的にチェックします。StarRocks は Hive と Iceberg テーブルのデータが更新された場合にのみ収集タスクを実行します。ただし、Hudi テーブルのデータ変更は検知できず、指定されたチェック間隔と収集間隔に基づいて StarRocks は統計情報を定期的に収集します。以下のFEパラメータが使用可能です:

- statistic_collect_interval_sec

  自動収集時のデータ更新をチェックする間隔。単位: 秒。デフォルト: 5分。

- statistic_auto_collect_small_table_rows（v3.2以降）

  自動収集時に外部データソース（Hive、Iceberg、Hudi）のテーブルが小規模かどうかを判断するための閾値。デフォルト: 10000000。

- statistic_auto_collect_small_table_interval

  小規模テーブルの統計情報を収集する間隔。単位: 秒。デフォルト: 0。

- statistic_auto_collect_large_table_interval

  大規模テーブルの統計情報を収集する間隔。単位: 秒。デフォルト: 43200 (12時間)。

自動収集スレッドは `statistic_collect_interval_sec` で指定された間隔でデータ更新をチェックします。テーブルの行数が `statistic_auto_collect_small_table_rows` よりも少ない場合は、そのようなテーブルの統計情報を `statistic_auto_collect_small_table_interval` に基づいて収集します。

テーブルの行数が `statistic_auto_collect_small_table_rows` を超える場合は、そのようなテーブルの統計情報を `statistic_auto_collect_large_table_interval` に基づいて収集します。大規模テーブルの統計情報の更新は、`最終テーブル更新時刻 + 収集間隔 > 現在時刻` の場合にのみ行われます。これにより、大規模テーブルの頻繁な分析タスクが防止されます。

#### 自動収集タスクの作成

構文:

```sql
CREATE ANALYZE TABLE tbl_name (col_name [,col_name])
[PROPERTIES (property [,property])]
```

例:

```sql
CREATE ANALYZE TABLE ex_hive_tbl (k1)
PROPERTIES ("statistic_auto_collect_interval" = "5");

Query OK, 0 rows affected (0.01 sec)
```

#### 自動収集タスクのステータスの表示

手動収集と同じです。

#### 統計情報のメタデータの表示

手動収集と同じです。

#### 自動収集タスクの表示

構文:

```sql
SHOW ANALYZE JOB [WHERE predicate]
```

例:

```sql
SHOW ANALYZE JOB WHERE `id` = '17204';

Empty set (0.00 sec)
```

#### 収集タスクのキャンセル

手動収集と同じです。

### 統計情報の削除

```sql
DROP STATS tbl_name
```

## 参照

- FEの構成項目をクエリするには、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md) を実行してください。

- FEの構成項目を変更するには、[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) を実行してください。