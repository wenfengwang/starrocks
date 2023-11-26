---
displayed_sidebar: "Japanese"
---

# CBOのための統計情報を収集する

このトピックでは、StarRocks CBOの基本的な概念とCBOのための統計情報の収集方法について説明します。StarRocks 2.4では、正確なデータ分布統計情報を収集するためにヒストグラムが導入されました。

## CBOとは

コストベースの最適化（CBO）は、クエリの最適化において重要です。SQLクエリがStarRocksに到着すると、論理的な実行計画にパースされます。CBOは、論理計画を複数の物理的な実行計画に書き換え、各オペレータの実行コスト（CPU、メモリ、ネットワーク、I/Oなど）を推定し、最もコストの低いクエリパスを最終的な物理計画として選択します。

StarRocks CBOは、StarRocks 1.16.0で開始され、1.19以降はデフォルトで有効になっています。StarRocks CBOは、Cascadesフレームワークをベースに開発され、さまざまな統計情報に基づいてコストを推定します。数万の実行計画の中から最もコストの低い実行計画を選択することができ、複雑なクエリの効率とパフォーマンスを大幅に向上させることができます。

統計情報はCBOにとって重要です。統計情報が正確で有用かどうかは、コストの推定が正確かどうかを決定します。以下のセクションでは、統計情報の種類、収集ポリシー、および統計情報の収集方法と表示方法について詳しく説明します。

## 統計情報の種類

StarRocksは、コストの推定に必要なさまざまな統計情報を収集します。

### 基本統計情報

デフォルトでは、StarRocksは定期的に次の基本統計情報をテーブルと列に収集します。

- row_count：テーブル内の行の総数

- data_size：列のデータサイズ

- ndv：列のカーディナリティ（列内の一意の値の数）

- null_count：列内のNULL値のデータ量

- min：列内の最小値

- max：列内の最大値

基本統計情報は、`_statistics_.table_statistic_v1`テーブルに格納されます。StarRocksクラスタの`_statistics_`データベースでこのテーブルを表示できます。

### ヒストグラム

StarRocks 2.4では、ヒストグラムが導入され、基本統計情報を補完します。ヒストグラムは、データの分布を正確に反映する効果的な方法とされています。データの偏りがあるテーブルでは、ヒストグラムはデータの分布を正確に反映することができます。

StarRocksでは、等高ヒストグラムが使用されており、複数のバケットに構築されます。各バケットには同じ量のデータが含まれます。頻繁にクエリされ、選択率に大きな影響を与えるデータ値については、別々のバケットが割り当てられます。バケットの数が多いほど、より正確な推定が可能ですが、メモリ使用量がわずかに増加する可能性もあります。ヒストグラムの収集タスクのバケット数と最も一般的な値（MCV）の数を調整することができます。

**ヒストグラムは、データの分布が非常に偏っており、頻繁にクエリされる列に適用されます。テーブルのデータが均一に分布している場合は、ヒストグラムを作成する必要はありません。ヒストグラムは、数値、DATE、DATETIME、または文字列型の列にのみ作成できます。**

現在、StarRocksはヒストグラムの手動収集のみをサポートしています。ヒストグラムは、`_statistics_`データベースの`histogram_statistics`テーブルに格納されます。

## 収集タイプと方法

テーブルのデータサイズとデータ分布は常に変化します。統計情報は、そのデータの変化を表すために定期的に更新する必要があります。統計情報の収集タスクを作成する前に、ビジネス要件に最も適した収集タイプと方法を選択する必要があります。

StarRocksは、完全収集とサンプル収集の両方を自動的および手動でサポートしています。デフォルトでは、StarRocksはテーブルの完全な統計情報を自動的に収集します。データの変更を5分ごとにチェックします。データの変更が検出されると、データ収集が自動的にトリガされます。自動的な完全収集を使用しない場合は、FEの設定項目`enable_collect_full_statistic`を`false`に設定し、収集タスクをカスタマイズすることができます。

| **収集タイプ** | **収集方法** | **説明** | **利点と欠点** |
| --- | --- | --- | --- |
| 完全収集 | 自動/手動 | テーブルの完全な統計情報を収集するためにテーブル全体をスキャンします。統計情報はパーティションごとに収集されます。パーティションにデータの変更がない場合、このパーティションからデータは収集されず、リソースの消費が削減されます。完全な統計情報は`_statistics_.column_statistics`テーブルに格納されます。 | 利点：統計情報が正確であり、CBOが正確な推定を行うのに役立ちます。欠点：システムリソースを消費し、遅いです。2.5以降、StarRocksでは自動収集期間を指定できるようになり、リソースの消費を削減できます。 |
| サンプル収集 | 自動/手動 | テーブルの各パーティションからデータの`N`行を均等に抽出します。統計情報はテーブルごとに収集されます。各列の基本統計情報は1つのレコードとして格納されます。列のカーディナリティ情報（ndv）は、サンプルデータに基づいて推定され、正確ではありません。サンプル統計情報は`_statistics_.table_statistic_v1`テーブルに格納されます。 | 利点：システムリソースを消費せず、高速です。欠点：統計情報が不完全であり、コストの推定の正確性に影響を与える可能性があります。 |

## 統計情報の収集

StarRocksでは、柔軟な統計情報の収集方法を提供しています。ビジネスシナリオに応じて、自動、手動、またはカスタムの収集方法を選択できます。

### 自動完全収集

基本統計情報については、デフォルトでStarRocksは自動的にテーブルの完全な統計情報を収集します。統計情報が収集されていないテーブルでは、StarRocksはスケジューリング期間内に自動的に統計情報を収集します。統計情報が収集されているテーブルでは、StarRocksはテーブルの総行数と変更された行数を更新し、定期的にこの情報を永続化して自動収集をトリガするかどうかを判断します。

2.5以降、StarRocksでは、自動完全収集のための収集期間を指定できるようになり、自動完全収集によるクラスタのパフォーマンスのジッタを防ぐことができます。この期間は、FEパラメータ`statistic_auto_analyze_start_time`および`statistic_auto_analyze_end_time`で指定されます。

自動収集がトリガされる条件は次のとおりです。

- 前回の統計情報の収集以降にテーブルのデータが変更された場合。

- テーブルの統計情報の健全性が指定されたしきい値（`statistic_auto_collect_ratio`）以下の場合。

> 統計情報の健全性の計算式：1 - 前回の統計情報の収集以降に追加された行数/最小のパーティションの総行数

- パーティションのデータが変更された場合。データが変更されていないパーティションは再度収集されません。

- 収集時間が設定された収集期間内にある場合（デフォルトの収集期間は終日です）。

自動完全収集はデフォルトで有効になっており、システムによって実行されるデフォルトの設定で実行されます。

以下の表には、デフォルトの設定が説明されています。これらを変更する必要がある場合は、**ADMIN SET CONFIG**コマンドを実行してください。

| **FE設定項目** | **タイプ** | **デフォルト値** | **説明** |
| --- | --- | --- | --- |
| enable_statistic_collect | BOOLEAN | TRUE | 統計情報を収集するかどうか。このスイッチはデフォルトでオンになっています。 |
| enable_collect_full_statistic | BOOLEAN | TRUE | 自動的な完全収集を有効にするかどうか。このスイッチはデフォルトでオンになっています。 |
| statistic_collect_interval_sec | LONG | 300 | 自動収集中にデータの更新をチェックする間隔。単位：秒。 |
| statistic_auto_collect_ratio | FLOAT | 0.8 | 自動収集の統計情報の健全性を判断するためのしきい値。統計情報の健全性がこのしきい値を下回る場合、自動収集がトリガされます。 |
| statistic_max_full_collect_data_size | LONG | 107374182400 | 自動収集がデータを収集する最大のパーティションのサイズ。単位：バイト。パーティションがこの値を超える場合、完全収集は破棄され、サンプル収集が代わりに実行されます。 |
| statistic_collect_max_row_count_per_query | INT | 5000000000 | 単一の解析タスクに対してクエリする最大行数。この値を超える場合、解析タスクは複数のクエリに分割されます。 |
| statistic_auto_analyze_start_time | STRING | 00:00:00 | 自動収集の開始時間。値の範囲：`00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | STRING | 23:59:59 | 自動収集の終了時間。値の範囲：`00:00:00` - `23:59:59`。 |

ほとんどの統計情報の収集には自動ジョブを頼ることができますが、特定の統計情報の要件がある場合は、ANALYZE TABLEステートメントを実行して手動でタスクを作成するか、CREATE ANALYZEステートメントを実行して自動タスクをカスタマイズすることができます。

### 手動収集

ANALYZE TABLEを使用して手動の収集タスクを作成できます。デフォルトでは、手動の収集は同期操作ですが、非同期操作にも設定できます。非同期モードでは、ANALYZE TABLEを実行した後、システムはこのステートメントが成功したかどうかをすぐに返します。ただし、収集タスクはバックグラウンドで実行され、結果を待つ必要はありません。タスクのステータスは、SHOW ANALYZE STATUSを実行して確認できます。大量のデータを持つテーブルには非同期収集が適しており、データ量の少ないテーブルには同期収集が適しています。**手動収集タスクは作成後に1回だけ実行されます。手動収集タスクを削除する必要はありません。**

#### 基本統計情報の手動収集

```SQL
ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
[WITH SYNC | ASYNC MODE]
PROPERTIES (property [,property]);
```

パラメータの説明：

- 収集タイプ
  - FULL：完全収集を意味します。
  - SAMPLE：サンプル収集を意味します。
  - 収集タイプが指定されていない場合、デフォルトで完全収集が使用されます。

- `col_name`：統計情報を収集する列。複数の列をカンマ（`,`）で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。

- [WITH SYNC | ASYNC MODE]：手動の収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、同期収集がデフォルトで使用されます。

- `PROPERTIES`：カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で表示できます。

| **PROPERTIES** | **タイプ** | **デフォルト値** | **説明** |
| --- | --- | --- | --- |
| statistic_sample_collect_rows | INT | 200000 | サンプル収集のために収集する最小行数。このパラメータの値がテーブルの実際の行数を超える場合、完全収集が実行されます。 |

例

完全収集の手動

```SQL
-- デフォルトの設定を使用してテーブルの完全な統計情報を手動で収集します。
ANALYZE TABLE tbl_name;

-- デフォルトの設定を使用してテーブルの完全な統計情報を手動で収集します。
ANALYZE FULL TABLE tbl_name;

-- 指定された列の統計情報をテーブルで手動で収集します。
ANALYZE TABLE tbl_name(c1, c2, c3);
```

サンプル収集の手動

```SQL
-- デフォルトの設定を使用してテーブルの一部の統計情報を手動で収集します。
ANALYZE SAMPLE TABLE tbl_name;

-- 指定された列の統計情報をテーブルで手動で収集します。収集する行数を指定します。
ANALYZE SAMPLE TABLE tbl_name (v1, v2, v3) PROPERTIES(
    "statistic_sample_collect_rows" = "1000000"
);
```

#### ヒストグラムの手動収集

```SQL
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON col_name [, col_name]
[WITH SYNC | ASYNC MODE]
[WITH N BUCKETS]
PROPERTIES (property [,property]);
```

パラメータの説明：

- `col_name`：統計情報を収集する列。複数の列をカンマ（`,`）で区切って指定します。このパラメータはヒストグラムの場合に必要です。

- [WITH SYNC | ASYNC MODE]：手動の収集タスクを同期モードまたは非同期モードで実行するかどうか。このパラメータを指定しない場合、同期収集がデフォルトで使用されます。

- `WITH N BUCKETS`：`N`はヒストグラムのバケット数です。指定しない場合、`fe.conf`のデフォルト値が使用されます。

- PROPERTIES：カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`のデフォルト設定が使用されます。

| **PROPERTIES** | **タイプ** | **デフォルト値** | **説明** |
| --- | --- | --- | --- |
| statistic_sample_collect_rows | INT | 200000 | 収集する行の最小数。このパラメータの値がテーブルの実際の行数を超える場合、完全収集が実行されます。 |
| histogram_buckets_size | LONG | 64 | ヒストグラムのデフォルトのバケット数。 |
| histogram_mcv_size | INT | 100 | ヒストグラムの最も一般的な値（MCV）の数。 |
| histogram_sample_ratio | FLOAT | 0.1 | ヒストグラムのサンプリング比率。 |
| histogram_max_sample_row_count | LONG | 10000000 | ヒストグラムの収集に使用する最大行数。 |

ヒストグラムの収集のための行数は、複数のパラメータで制御されます。`statistic_sample_collect_rows`とテーブルの行数 * `histogram_sample_ratio`の値の大きい方が使用されます。この値は、`histogram_max_sample_row_count`で指定された値を超えることはできません。値が超える場合は、`histogram_max_sample_row_count`が優先されます。

実際に使用されるプロパティは、SHOW ANALYZE STATUSの出力の`Properties`列で表示できます。

例

```SQL
-- デフォルトの設定を使用してv1にヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1;

-- 32のバケット、32のMCV、および50％のサンプリング比率でv1とv2のヒストグラムを手動で収集します。
ANALYZE TABLE tbl_name UPDATE HISTOGRAM ON v1,v2 WITH 32 BUCKETS 
PROPERTIES(
   "histogram_mcv_size" = "32",
   "histogram_sample_ratio" = "0.5"
);
```

### カスタム収集

#### カスタム自動収集タスクのカスタマイズ

CREATE ANALYZEステートメントを使用してカスタム自動収集タスクをカスタマイズできます。

カスタム自動収集タスクを作成する前に、自動完全収集（`enable_collect_full_statistic = false`）を無効にする必要があります。そうしないと、カスタムタスクは効果を発揮しません。

```SQL
-- すべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] ALL PROPERTIES (property [,property]);

-- データベース内のすべてのテーブルの統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] DATABASE db_name
PROPERTIES (property [,property]);

-- テーブルの指定された列の統計情報を自動的に収集します。
CREATE ANALYZE [FULL|SAMPLE] TABLE tbl_name (col_name [,col_name])
PROPERTIES (property [,property]);
```

パラメータの説明：

- 収集タイプ
  - FULL：完全収集を意味します。
  - SAMPLE：サンプル収集を意味します。
  - 収集タイプが指定されていない場合、デフォルトで完全収集が使用されます。

- `col_name`：統計情報を収集する列。複数の列をカンマ（`,`）で区切って指定します。このパラメータが指定されていない場合、テーブル全体が収集されます。

- `PROPERTIES`：カスタムパラメータ。`PROPERTIES`が指定されていない場合、`fe.conf`ファイルのデフォルト設定が使用されます。

| **PROPERTIES** | **タイプ** | **デフォルト値** | **説明** |
| --- | --- | --- | --- |
| statistic_auto_collect_ratio | FLOAT | 0.8 | 自動収集の統計情報の健全性を判断するためのしきい値。統計情報の健全性がこのしきい値を下回る場合、自動収集がトリガされます。 |
| statistics_max_full_collect_data_size | INT | 100 | 自動収集がデータを収集する最大のパーティションのサイズ。単位：GB。パーティションがこの値を超える場合、完全収集は破棄され、サンプル収集が代わりに実行されます。 |
| statistic_sample_collect_rows | INT | 200000 | 収集する最小行数。このパラメータの値がテーブルの実際の行数を超える場合、完全収集が実行されます。 |
| statistic_exclude_pattern | String | null | ジョブで統計情報を収集しないデータベースまたはテーブルの名前。ジョブで統計情報を収集しないデータベースとテーブルを指定できます。これは正規表現パターンであり、一致する内容は`database.table`です。 |

例

自動完全収集

```SQL
-- すべてのデータベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE ALL;

-- データベースの完全な統計情報を自動的に収集します。
CREATE ANALYZE DATABASE db_name;

-- データベースのすべてのテーブルの完全な統計情報を自動的に収集します。
CREATE ANALYZE FULL DATABASE db_name;

-- テーブルの指定された列の完全な統計情報を自動的に収集します。
CREATE ANALYZE TABLE tbl_name(c1, c2, c3); 

-- 指定されたデータベース 'db_name' を除外してすべてのデータベースの統計情報を自動的に収集します。
CREATE ANALYZE ALL PROPERTIES (
   "statistic_exclude_pattern" = "db_name\."
);
```

自動サンプル収集

```SQL
-- データベースのすべてのテーブルの統計情報をデフォルトの設定で自動的に収集します。
CREATE ANALYZE SAMPLE DATABASE db_name;

-- データベースのすべてのテーブルの統計情報を収集しますが、指定したテーブル 'db_name.tbl_name' を除外します。
CREATE ANALYZE SAMPLE DATABASE db_name PROPERTIES (
   "statistic_exclude_pattern" = "db_name\.tbl_name"
);

-- テーブルの指定された列の統計情報を収集します。統計情報の健全性と収集する行数を指定します。
CREATE ANALYZE SAMPLE TABLE tbl_name(c1, c2, c3) PROPERTIES (
   "statistic_auto_collect_ratio" = "0.5",
   "statistic_sample_collect_rows" = "1000000"
);

```

#### カスタム収集タスクの表示

```SQL
SHOW ANALYZE JOB [WHERE predicate]
```

WHERE句を使用して結果をフィルタリングすることができます。このステートメントは、次の列を返します。

| **列** | **説明** |
| --- | --- |
| Id | 収集タスクのID。 |
| Database | データベース名。 |
| Table | テーブル名。 |
| Columns | 列名。 |
| Type | 統計情報のタイプ。`FULL`と`SAMPLE`があります。 |
| Schedule | スケジューリングのタイプ。自動タスクの場合は`SCHEDULE`です。 |
| Properties | カスタムパラメータ。 |
| Status | タスクのステータス。PENDING、RUNNING、SUCCESS、FAILEDがあります。 |
| LastWorkTime | 最後の収集の時間。 |
| Reason | タスクが失敗した理由。タスクの実行が成功した場合はNULLが返されます。 |

例

```SQL
-- カスタム収集タスクをすべて表示します。
SHOW ANALYZE JOB

-- データベース `test` のカスタム収集タスクを表示します。
SHOW ANALYZE JOB where `database` = 'test';
```

#### カスタム収集タスクの削除

```SQL
DROP ANALYZE <ID>;
```

マニュアル収集タスクのタスクIDは、SHOW ANALYZE JOBステートメントを使用して取得できます。

例

```SQL
DROP ANALYZE 266030;
```

## 収集タスクのステータスを表示

SHOW ANALYZE STATUSステートメントを実行することで、現在のすべてのタスクのステータスを表示できます。このステートメントは、カスタム収集タスクのステータスを表示することはできません。カスタム収集タスクのステータスを表示するには、SHOW ANALYZE JOBを使用してください。

```SQL
SHOW ANALYZE STATUS [WHERE predicate];
```

WHERE句を使用して情報をフィルタリングすることができます。

このステートメントは、次の列を返します。

| **リスト名** | **説明** |
| --- | --- |
| Id | 収集タスクのID。 |
| Database | データベース名。 |
| Table | テーブル名。 |
| Columns | 列名。 |
| Type | 統計情報のタイプ。ヒストグラムの場合は`HISTOGRAM`です。 |
| Schedule | スケジューリングのタイプ。手動の場合は`ONCE`、自動の場合は`SCHEDULE`です。 |
| Status | タスクのステータス。 |
| StartTime | タスクの開始時刻。 |
| EndTime | タスクの終了時刻。 |
| Properties | カスタムパラメータ。 |
| Reason | タスクが失敗した理由。実行が成功した場合はNULLが返されます。 |

## 統計情報の表示

### 基本統計情報のメタデータの表示

```SQL
SHOW STATS META [WHERE];
```

このステートメントは、次の列を返します。

| **列** | **説明** |
| --- | --- |
| Database | データベース名。 |
| Table | テーブル名。 |
| Columns | 列名。 |
| Type | 統計情報のタイプ。`FULL`は完全収集を、`SAMPLE`はサンプル収集を意味します。 |
| UpdateTime | 現在のテーブルの最新の統計情報の更新時刻。 |
| Properties | カスタムパラメータ。 |
| Healthy | 統計情報の健全性。 |

### ヒストグラムのメタデータの表示

```SQL
SHOW HISTOGRAM META [WHERE];
```

このステートメントは、次の列を返します。

| **列** | **説明** |
| --- | --- |
| Database | データベース名。 |
| Table | テーブル名。 |
| Column | 列名。 |
| Type | 統計情報のタイプ。ヒストグラムの場合は`HISTOGRAM`です。 |
| UpdateTime | 現在のテーブルの最新の統計情報の更新時刻。 |
| Properties | カスタムパラメータ。 |

## 統計情報の削除

必要のない統計情報を削除することができます。統計情報を削除すると、データとメタデータの両方が削除されます。また、期限切れのキャッシュ内の統計情報も削除されます。ただし、自動収集タスクが進行中の場合、以前に削除された統計情報が再度収集される可能性があります。コレクションタスクの履歴を表示するには、SHOW ANALYZE STATUSを使用できます。

### 基本統計情報の削除

```SQL
DROP STATS tbl_name
```

### ヒストグラムの削除

```SQL
ANALYZE TABLE tbl_name DROP HISTOGRAM ON col_name [, col_name];
```

## 収集タスクのキャンセル

KILL ANALYZEステートメントを使用して、**実行中の**収集タスク（手動およびカスタムタスクを含む）をキャンセルすることができます。

```SQL
KILL ANALYZE <ID>;
```

マニュアル収集タスクのタスクIDは、SHOW ANALYZE STATUSステートメントを使用して取得できます。

例

```SQL
KILL ANALYZE 266030;
```

## FE設定項目

| **FE設定項目** | **タイプ** | **デフォルト値** | **説明** |
| --- | --- | --- | --- |
| enable_statistic_collect | BOOLEAN | TRUE | 統計情報を収集するかどうか。このパラメータはデフォルトでオンになっています。 |
| enable_collect_full_statistic | BOOLEAN | TRUE | 自動的な完全収集を有効にするかどうか。このパラメータはデフォルトでオンになっています。 |
| statistic_auto_collect_ratio | FLOAT | 0.8 | 自動収集の統計情報の健全性を判断するためのしきい値。統計情報の健全性がこのしきい値を下回る場合、自動収集がトリガされます。 |
| statistic_max_full_collect_data_size | LONG | 107374182400 | 自動収集がデータを収集する最大のパーティションのサイズ。単位：バイト。パーティションがこの値を超える場合、完全収集は破棄され、サンプル収集が代わりに実行されます。 |
| statistic_collect_max_row_count_per_query | INT | 5000000000 | 単一の解析タスクに対してクエリする最大行数。この値を超える場合、解析タスクは複数のクエリに分割されます。 |
| statistic_collect_interval_sec | LONG | 300 | 自動収集中にデータの更新をチェックする間隔。単位：秒。 |
| statistic_auto_analyze_start_time | STRING | 00:00:00 | 自動収集の開始時間。値の範囲：`00:00:00` - `23:59:59`。 |
| statistic_auto_analyze_end_time | STRING | 23:59:59 | 自動収集の終了時間。値の範囲：`00:00:00` - `23:59:59`。 |
| statistic_sample_collect_rows | LONG | 200000 | サンプル収集のために収集する最小行数。このパラメータの値がテーブルの実際の行数を超える場合、完全収集が実行されます。 |
| statistic_collect_concurrency | INT | 3 | 並行して実行できる最大の手動収集タスクの数。この値はデフォルトで3に設定されており、最大で3つの手動収集タスクを並行して実行できます。この値を超える場合、受信タスクはPENDING状態になり、スケジュールされるのを待ちます。 |
| histogram_buckets_size | LONG | 64 | ヒストグラムのデフォルトのバケット数。 |
| histogram_mcv_size | LONG | 100 | ヒストグラムの最も一般的な値（MCV）の数。 |
| histogram_sample_ratio | FLOAT | 0.1 | ヒストグラムのサンプリング比率。 |
| histogram_max_sample_row_count | LONG | 10000000 | ヒストグラムの収集に使用する最大行数。 |
| statistic_manager_sleep_time_sec | LONG | 60 | メタデータのスケジュール間隔。単位：秒。この間隔に基づいてシステムは次の操作を実行します。統計情報を格納するテーブルを作成します。削除された統計情報を削除します。期限切れの統計情報を削除します。 |
| statistic_analyze_status_keep_second | LONG | 259200 | 収集タスクの履歴を保持する期間。デフォルト値は3日です。単位：秒。 |

## 参考文献

- FE設定項目をクエリするには、[ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md)を実行します。

- FE設定項目を変更するには、[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)を実行します。
