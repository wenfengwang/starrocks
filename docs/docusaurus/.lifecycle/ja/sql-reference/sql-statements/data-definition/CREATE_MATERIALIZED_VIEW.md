---
displayed_sidebar: "Japanese"
---

# マテリアライズド・ビューを作成する

## 説明

マテリアライズド・ビューを作成します。マテリアライズド・ビューの使用方法については、[同期マテリアライズド・ビュー]("../../../using_starrocks/Materialized_view-single_table.md") および [非同期マテリアライズド・ビュー]("../../../using_starrocks/Materialized_view.md") を参照してください。

> **注意**
>
> 基底テーブルが存在するデータベースで CREATE MATERIALIZED VIEW 権限を持つユーザーのみがマテリアライズド・ビューを作成できます。

マテリアライズド・ビューの作成は非同期の操作です。このコマンドを正常に実行すると、マテリアライズド・ビューの作成タスクが正常に送信されたことを示します。同期マテリアライズド・ビューの構築ステータスはデータベースにおいて [SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md) コマンドで、非同期マテリアライズド・ビューの構築ステータスはメタデータビュー [`tasks`](../../../reference/information_schema/tasks.md) および [`task_runs`](../../../reference/information_schema/task_runs.md) をクエリすることで確認できます。[情報スキーマ](../../../reference/overview-pages/information_schema.md) で非同期マテリアライズド・ビューをビューできます。

StarRocks は v2.4 から非同期マテリアライズド・ビューをサポートしています。以前のバージョンの同期マテリアライズド・ビューとの主な違いは以下のとおりです:

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリの再構築** | **リフレッシュ戦略** | **基底テーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **SYNC MV (Rollup)**  | 集約関数の選択肢が限られています | なし | はい | データのローディング中に同期リフレッシュ | デフォルトカタログの複数テーブル:<ul><li>外部カタログ（v2.5）</li><li>既存のマテリアライズド・ビュー（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |

## 同期マテリアライズド・ビュー

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

角かっこ [] 内のパラメータはオプションです。

### パラメータ

**mv_name** (必須)

マテリアライズド・ビューの名前。名前の要件は次のとおりです:

- 名前は、文字（a-z または A-Z）、数字（0-9）、またはアンダースコア (_) で構成されている必要があり、文字で始まる必要があります。
- 名前の長さは 64 文字を超えることはできません。
- 名前は大文字と小文字を区別します。

**COMMENT** (オプション)

マテリアライズド・ビューについてのコメント。`COMMENT` は `mv_name` の後に配置する必要があります。そうでない場合、マテリアライズド・ビューは作成できません。

**query_statement** (必須)

マテリアライズド・ビューを作成するためのクエリ文。その結果はマテリアライズド・ビューのデータです。構文は次のとおりです:

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr (必須)

  クエリ文のすべての列、つまりマテリアライズド・ビュースキーマのすべての列です。このパラメータは次の値をサポートしています:

  - `SELECT a, abs(b), min(c) FROM table_a` のような、`a`、`b`、`c` が基底テーブルの列の名前である単純な列または集約列。マテリアライズド・ビューの列名を指定しない場合、StarRocks は自動的に列に名前を割り当てます。
  - `SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a` のような、基底テーブルの列を参照する式である `a+1`、`b+2`、`c*c` などの式、およびマテリアライズド・ビューの列に割り当てられたエイリアスである `x`、`y`、`z` が含まれる式。 

  > **注意**
  >
  > - 少なくとも 1 つの列を `select_expr` で指定する必要があります。
  > - 集約関数を使用して同期マテリアライズド・ビューを作成する場合、GROUP BY 句を指定する必要があり、少なくとも 1 つの GROUP BY 列を `select_expr` で指定する必要があります。
  > - 同期マテリアライズド・ビューでは、JOIN や GROUP BY の HAVING 句などの句はサポートされていません。
  > - v3.1 以降、各同期マテリアライズド・ビューについて、基底テーブルの各列の複数の集約関数をサポートできます。たとえば、`select b, sum(a), min(a) from table group by b` のようなクエリ文があります。
  > - v3.1 以降、同期マテリアライズド・ビューは SELECT および集約関数に複雑な式をサポートします。たとえば、`select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b` や `select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table` のようなクエリ文があります。同期マテリアライズド・ビューに使用される複雑な式には以下の制限が課されます:
  >   - 各複雑な式にはエイリアスが必要であり、すべての同期マテリアライズド・ビューの異なる複雑な式に異なるエイリアスが割り当てられる必要があります。たとえば、`select b, sum(a + 1) as sum_a from table group by b` と `select b, sum(a) as sum_a from table group by b` は同じ基底テーブルの同期マテリアライズド・ビューを作成するために使用できません。複雑な式には異なるエイリアスを設定できます。
  >   - 複雑な式によって作成された同期マテリアライズド・ビューがクエリによって書き換えられているかどうかは、`EXPLAIN <sql_statement>` を実行して確認できます。詳細については、[クエリ解析](../../../administration/Query_planning.md) を参照してください。
- 名前は、文字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（\_）で構成する必要があり、文字で始まる必要があります。
- 名前の長さは64文字を超えることはできません。
- 名前は大文字と小文字を区別します。

> **注意**
>
> 同じベーステーブルに複数のマテリアライズドビューを作成できますが、同じデータベース内のマテリアライズドビューの名前は重複できません。

**COMMENT**（オプション）

マテリアライズドビューに関するコメント。`mv_name`の後に`COMMENT`を配置しないと、マテリアライズドビューは作成できません。

**distribution_desc**（オプション）

非同期マテリアライズドビューのバケティング戦略。StarRocksはハッシュバケティングとランダムバケティング（v3.1以降）をサポートしています。このパラメータを指定しない場合、StarRocksはランダムバケティング戦略を使用し、バケットの数を自動的に設定します。

> **注意**
>
> 非同期マテリアライズドビューを作成する際には、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

- **ハッシュバケティング**:

  構文

  ```SQL
  DISTRIBUTED BY HASH(<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  詳細については、[データ分散](../../../table_design/Data_distribution.md#data-distribution)を参照してください。

  > **注意**
  >
  > v2.5.7以降、StarRocksはテーブルを作成するかパーティションを追加する際に、バケットの数（BUCKETS）を自動的に設定できます。バケットの数を手動で設定する必要はもうありません。詳細については、[バケットの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- **ランダムバケティング**:

  ランダムバケティング戦略を選択し、StarRocksにバケットの数を自動的に設定させる場合、`distribution_desc`を指定する必要はありません。ただし、バケットの数を手動で設定したい場合は、次の構文を参照してください。

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > ランダムバケティング戦略を持つ非同期マテリアライズドビューは、コロケーショングループに割り当てることはできません。

  詳細については、[ランダムバケティング](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

**refresh_moment**（オプション）

マテリアライズドビューのリフレッシュ時刻。デフォルト値: `IMMEDIATE`。有効な値:

- `IMMEDIATE`：非同期マテリアライズドビューを作成した直後に即座にリフレッシュします。
- `DEFERRED`：非同期マテリアライズドビューは作成後にリフレッシュされません。マテリアライズドビューは手動でリフレッシュするか、定期的なリフレッシュタスクをスケジュールすることができます。

**refresh_scheme**（オプション）

> **注意**
>
> 非同期マテリアライズドビューを作成する際には、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

非同期マテリアライズドビューのリフレッシュ戦略。有効な値:

- `ASYNC`：非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、マテリアライズドビューは事前に定義されたリフレッシュ間隔に基づいて自動的にリフレッシュされます。さらに、リフレッシュ開始時刻を`START('yyyy-MM-dd hh:mm:ss')`として指定し、リフレッシュ間隔を`EVERY (interval n day/hour/minute/second)`で指定できます。ユニット: `DAY`、`HOUR`、`MINUTE`、`SECOND`。例: `ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。間隔を指定しない場合、デフォルト値`10 MINUTE`が使用されます。
- `MANUAL`：手動リフレッシュモード。マテリアライズドビューは自動的にリフレッシュされません。リフレッシュタスクはユーザーによって手動でトリガーされます。

このパラメータが指定されていない場合、デフォルト値は`MANUAL`です。

**partition_expression**（オプション）

非同期マテリアライズドビューのパーティション戦略。現在のStarRocksのバージョンでは、非同期マテリアライズドビューを作成する際には、1つのパーティション式のみがサポートされています。

> **注意**
>
> 現在、非同期マテリアライズドビューはリストパーティション戦略をサポートしていません。

有効な値:

- `column_name`：パーティショニングに使用される列の名前。式`PARTITION BY dt`は、`dt`列に従ってマテリアライズドビューをパーティション分けすることを意味します。
- `date_trunc` 関数：時間単位を切り捨てるために使用される関数。`PARTITION BY date_trunc("MONTH", dt)`は`dt`列を月単位で切り捨ててパーティション分けすることを意味します。`date_trunc` 関数は、`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`などの単位で時間を切り捨てることをサポートしています。
- `str2date` 関数：ベーステーブルの文字列型パーティションをマテリアライズドビューのパーティションに分割するために使用される関数。`PARTITION BY str2date(dt, "%Y%m%d")`は、`dt`列が`"%Y%m%d"`の日付形式である文字列の日付型であることを意味します。`str2date` 関数は多くの日付形式をサポートしており、詳細については、[str2date](../../sql-functions/date-time-functions/str2date.md)を参照してください。v3.1.4以降でサポートされています。
- `time_slice` または `date_slice` 関数：v3.1以降、指定された時間を時間粒度に基づいて時間間隔の先頭または末尾に変換するためにこれらの関数を使用できます。たとえば、`PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))`では、time_sliceとdate_sliceはdate_truncより細かい単位である必要があります。これらの関数を使用して、パーティションキーの時間粒度よりも細かい時間でGROUP BY列を指定できます。例: `GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`。

このパラメータが指定されていない場合、デフォルトでパーティション戦略は採用されません。

**order_by_expression**（オプション）

非同期マテリアライズドビューのソートキー。ソートキーが指定されていない場合、StarRocksはSELECT列のプレフィックス列の一部をソートキーとして選択します。たとえば、`select a, b, c, d`の場合、ソートキーは`a`と`b`になります。このパラメータは、StarRocks v3.0以降でサポートされています。

**PROPERTIES**（オプション）

非同期マテリアライズドビューのプロパティ。既存のマテリアライズドビューのプロパティは、[ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md)を使用して変更できます。

- `session.`: マテリアライズドビューのセッション変数関連プロパティを変更する場合、プロパティに`session.`接頭辞を追加する必要があります。たとえば、`session.query_timeout`のように、非セッションプロパティでは接頭辞を指定する必要はありません。例: `mv_rewrite_staleness_second`。
- `replication_num`：作成するマテリアライズドビューレプリカの数。
- `storage_medium`：ストレージメディアの種類。有効な値: `HDD` および `SSD`。
- `storage_cooldown_time`：パーティションのストレージ冷却時間。HDDおよびSSDストレージメディアが使用される場合、このプロパティで指定した時間後にSSDストレージのデータがHDDストレージに移動されます。形式: "yyyy-MM-dd HH:mm:ss"。指定した時刻は現在時刻より後である必要があります。このプロパティが明示的に指定されていない場合、デフォルトではストレージの冷却は実行されません。
- `partition_ttl`：パーティションの生存期間（TTL）。指定された時間範囲内のデータを持つパーティションは保持され、期限が切れたパーティションは自動的に削除されます。単位: `YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`。例: このプロパティは、`2 MONTH`のように指定できます。このプロパティは`partition_ttl_number`よりも推奨されています。v3.1.5以降でサポートされています。
- `partition_ttl_number`：保持する最新のマテリアライズドビューパーティションの数。現在時刻よりも前の開始時刻を持つパーティションについては、これらのパーティションがこの値を超えると、より新しいではないパーティションが削除されます。StarRocksは、`dynamic_partition_check_interval_seconds`で指定された時間間隔に従って定期的にマテリアライズドビューパーティションをチェックし、自動的に期限切れのパーティションを削除します。[dynamic partitioning](../../../table_design/dynamic_partitioning.md)ストラテジーが有効になっている場合、事前に作成されたパーティションはカウントされません。この値が`-1`の場合、マテリアライズドビューのすべてのパーティションが保持されます。デフォルト値: `-1`。
- `excluded_trigger_tables`: マテリアライズドビューの基になるテーブルがここにリストされている場合、ベーステーブルのデータが変更されたときに自動更新タスクがトリガーされません。このパラメータは、ロードトリガーの更新戦略にのみ適用され、通常、プロパティ `auto_refresh_partitions_limit` と一緒に使用されます。形式：`[db_name.]table_name`。値が空の文字列の場合、すべてのベーステーブルのデータ変更が該当するマテリアライズドビューのリフレッシュをトリガーします。デフォルト値は空の文字列です。
- `auto_refresh_partitions_limit`: マテリアライズドビューのリフレッシュがトリガーされたときに更新する必要のある最新のマテリアライズドビューパーティションの数。このプロパティを使用して、リフレッシュ範囲を制限し、リフレッシュコストを削減できます。ただし、すべてのパーティションが更新されないため、マテリアライズドビューのデータはベーステーブルと一貫性がない場合があります。デフォルト値：`-1`。値が `-1` の場合、すべてのパーティションが更新されます。値が正の整数 N の場合、StarRocks は既存のパーティションを時系列順にソートし、現在のパーティションと過去 N-1 のパーティションを更新します。パーティションの数が N よりも少ない場合、StarRocks は既存のすべてのパーティションを更新します。マテリアライズドビューに事前に作成された動的パーティションがある場合、StarRocks はすべての事前作成されたパーティションを更新します。
- `mv_rewrite_staleness_second`: このプロパティで指定された時間間隔内にマテリアライズドビューの最後のリフレッシュがある場合、ベーステーブルのデータが変更されたかどうかに関係なく、このマテリアライズドビューはクエリのリライトに直接使用できます。最後のリフレッシュがこの時間間隔より前の場合、StarRocks はベーステーブルが更新されたかどうかを確認して、マテリアライズドビューをクエリのリライトに使用できるかどうかを判断します。単位：秒。このプロパティはv3.0からサポートされています。
- `colocate_with`: 非同期マテリアライズドビューの配置グループ。詳細については[配置ジョイン](../../../using_starrocks/Colocate_join.md)を参照してください。このプロパティはv3.0からサポートされています。
- `unique_constraints` および `foreign_key_constraints`: View Delta Join シナリオでクエリのリライトに非同期マテリアライズドビューを作成する場合の一意キー制約と外部キー制約。詳細については[非同期マテリアライズドビュー - View Delta Join シナリオでのクエリのリライト](../../../using_starrocks/query_rewrite_with_materialized_views.md)を参照してください。このプロパティはv3.0からサポートされています。
- `resource_group`: マテリアライズドビューのリフレッシュタスクが所属するリソースグループ。リソースグループの詳細については[リソースグループ](../../../administration/resource_group.md)を参照してください。
- `query_rewrite_consistency`: 非同期マテリアライズドビューのクエリのリライトルール。このプロパティはv3.2からサポートされています。有効な値:
  - `disable`: 非同期マテリアライズドビューの自動クエリリライトを無効にします。
  - `checked` (デフォルト値): マテリアライズドビューがタイムリネスの要件を満たす場合にのみ、自動クエリリライトを有効にします。タイムリネスの要件とは次の場合です。
    - `mv_rewrite_staleness_second` が指定されていない場合、マテリアライズドビューのデータがすべてのベーステーブルのデータと一致している場合にのみ、マテリアライズドビューをクエリのリライトに使用できます。
    - `mv_rewrite_staleness_second` が指定されている場合、マテリアライズドビューの最後のリフレッシュがステイルネス時間間隔内の場合、マテリアライズドビューをクエリのリライトに使用できます。
  - `loose`: 直接自動クエリリライトを有効にし、一貫性チェックは不要です。
- `force_external_table_query_rewrite`: 外部カタログベースのマテリアライズドビューのクエリリライトを有効にするかどうか。このプロパティはv3.2からサポートされています。有効な値:
  - `true`: 外部カタログベースのマテリアライズドビューのクエリリライトを有効にします。
  - `false` (デフォルト値): 外部カタログベースのマテリアライズドビューのクエリリライトを無効にします。

  ベーステーブルと外部カタログベースのマテリアライズドビューの間には、データの強力な一貫性は保証されませんので、この機能はデフォルトで `false` に設定されています。この機能が有効になっている場合は、`query_rewrite_consistency` で指定されたルールに従ってマテリアライズドビューがクエリのリライトに使用されます。

> **注意**
>
> 一意キー制約と外部キー制約はクエリのリライトにのみ使用されます。データをテーブルにロードする際に外部キー制約のチェックは保証されません。テーブルにロードするデータが制約を満たしていることを確認する必要があります。

**query_statement** (必須)

非同期マテリアライズドビューを作成するためのクエリ文。

> **注意**
>
> 現在、StarRocksはリストパーティション化戦略で作成されたベーステーブルで非同期マテリアライズドビューを作成することはサポートしていません。

### 非同期マテリアライズドビューのクエリ

非同期マテリアライズドビューは物理テーブルです。**ただし、直接的に非同期マテリアライズドビューにデータをロードすることはできません**。

### 非同期マテリアライズドビューによる自動クエリリライト

StarRocks v2.5は、SPJGタイプの非同期マテリアライズドビューに基づいた自動かつ透過的なクエリリライトをサポートしています。SPJGタイプのマテリアライズドビューとは、計画にスキャン、フィルタ、プロジェクト、および集計のタイプの演算子のみを含むマテリアライズドビューのことです。SPJGタイプのマテリアライズドビューのクエリリライトには、単一テーブルのクエリリライト、JOINのクエリリライト、集計のクエリリライト、UNIONのクエリリライト、およびネストされたマテリアライズドビューに基づくクエリリライトが含まれます。

詳細については、[非同期マテリアライズドビュー - 非同期マテリアライズドビューによるクエリのリライト](../../../using_starrocks/query_rewrite_with_materialized_views.md)を参照してください。

### サポートされているデータ型

- StarRocksのデフォルトカタログに基づいて作成された非同期マテリアライズドビューは、次のデータ型をサポートしています：

  - **日付**: DATE, DATETIME
  - **文字列**: CHAR, VARCHAR
  - **数値**: BOOLEAN, TINYINT, SMALLINT, INT, BIGINT, LARGEINT, FLOAT, DOUBLE, DECIMAL, PERCENTILE
  - **半構造化**: ARRAY, JSON, MAP (v3.1以降), STRUCT (v3.1以降)
  - **その他**: BITMAP, HLL

> **注記**
>
> BITMAP、HLL、およびPERCENTILEは、v2.4.5以降でサポートされています。

- StarRocksの外部カタログに基づいて作成された非同期マテリアライズドビューは、次のデータ型をサポートしています：

  - Hive カタログ

    - **数値**: INT/INTEGER, BIGINT, DOUBLE, FLOAT, DECIMAL
    - **日付**: TIMESTAMP
    - **文字列**: STRING, VARCHAR, CHAR
    - **半構造化**: ARRAY

  - Hudi カタログ

    - **数値**: BOOLEAN, INT, LONG, FLOAT, DOUBLE, DECIMAL
    - **日付**: DATE, TimeMillis/TimeMicros, TimestampMillis/TimestampMicros
    - **文字列**: STRING
    - **半構造化**: ARRAY

  - Iceberg カタログ

    - **数値**: BOOLEAN, INT, LONG, FLOAT, DOUBLE, DECIMAL(P, S)
    - **日付**: DATE, TIME, TIMESTAMP
    - **文字列**: STRING, UUID, FIXED(L), BINARY
    - **半構造化**: LIST

## 使用上の注意

- 現在のバージョンのStarRocksでは、複数のマテリアライズドビューを同時に作成することはサポートされていません。新しいマテリアライズドビューは、前のマテリアライズドビューが完了した後にのみ作成できます。

- 同期マテリアライズドビューに関する注意事項：

  - 同期マテリアライズドビューは、単一の列に対する集約関数のみをサポートしています。`sum(a+b)` のような形式のクエリ文はサポートされていません。
  - 同期マテリアライズドビューでは、ベーステーブルの各列の集約関数は1つだけサポートされています。`select sum(a), min(a) from table` のようなクエリ文はサポートされていません。
  - 集約関数を使用して同期マテリアライズドビューを作成する場合は、GROUP BY句を指定し、SELECTの中で少なくとも1つのGROUP BY列を指定する必要があります。
  - 同期マテリアライズドビューは、JOINのような句やGROUP BYのHAVING句などの句をサポートしていません。
  - ALTER TABLE DROP COLUMN を使用してベーステーブルの特定の列を削除する場合、削除する列が同期マテリアライズドビューに含まれていないことを確認する必要があります。そうでない場合、削除操作が失敗します。列を削除する前に、列を含むすべての同期マテリアライズドビューを先に削除する必要があります。
  - テーブルに対して同期マテリアライズドビューを多数作成すると、データロードの効率が低下します。ベーステーブルにデータをロードすると、同期マテリアライズドビューとベーステーブルのデータが同期して更新されます。ベーステーブルに `n` 個の同期マテリアライズドビューが含まれると、ベーステーブルへのデータのロード効率は `n` 個のテーブルへのデータのロード効率とほぼ同じになります。

- ネストされた非同期マテリアライズドビューに関する注意事項：

  - 各マテリアライズドビューのリフレッシュ戦略は、対応するマテリアライズドビューにのみ適用されます。
  - 現在、StarRocksはネストレベルの数を制限していません。本番環境では、ネストレベルの数が3を超えないことを推奨します。

- 外部カタログの非同期マテリアライズドビューに関する注意事項：

  - 外部カタログマテリアライズドビューは、固定間隔リフレッシュおよび手動リフレッシュの非同期モデルのみをサポートしています。
  - 外部カタログのマテリアライズドビューでは、マテリアライズドビューと外部カタログのベーステーブルの間の厳密な一貫性は保証されません。
  - 現在、外部リソースを基にしたマテリアライズドビューのビルドはサポートされていません。
  - 現在、StarRocksは外部カタログのベーステーブルのデータが変更されたかどうかを検知することができないため、基本テーブルが更新されるたびにデフォルトですべてのパーティションがリフレッシュされます。一部のパーティションのみを手動でリフレッシュするには、[REFRESH MATERIALIZED VIEW](../data-manipulation/REFRESH_MATERIALIZED_VIEW.md)を使用できます。

## 例

### 同期マテリアライズドビューの例

ベーステーブルのスキーマは次の通りです：

```Plain Text
mysql> desc duplicate_table;
+-------+--------+------+------+---------+-------+
| Field | Type    | Null | Key  | Default | Extra |
|-------|---------|------|------|---------|-------|
| ds    | date    | No   | false| N/A     |       |
| id    | varchar(256) | No | false | N/A |       |
| user_id | int    | Yes  | false| N/A     |       |
| user_id1| varchar(256) | Yes | false| N/A |       |
| user_id2| varchar(256) | Yes | false| N/A |       |
| column_01| int    | Yes  | false| N/A     |       |
| column_02| int    | Yes  | false| N/A     |       |
| column_03| int    | Yes  | false| N/A     |       |
| column_04| int    | Yes  | false| N/A     |       |
| column_05| int    | Yes  | false| N/A     |       |
| column_06| DECIMAL(12,2) | Yes  | false| N/A |       |
| column_07| DECIMAL(12,3) | Yes  | false| N/A |       |
| column_08| JSON   | Yes  | false| N/A     |       |
| column_09| DATETIME | Yes  | false| N/A    |       |
| column_10| DATETIME | Yes  | false| N/A    |       |
| column_11| DATE   | Yes  | false| N/A     |       |
| column_12| varchar(256) | Yes | false| N/A  |       |
| column_13| varchar(256) | Yes | false| N/A  |       |
| column_14| varchar(256) | Yes | false| N/A  |       |
| column_15| varchar(256) | Yes | false| N/A  |       |
| column_16| varchar(256) | Yes | false| N/A  |       |
| column_17| varchar(256) | Yes | false| N/A  |       |
| column_18| varchar(256) | Yes | false| N/A  |       |
| column_19| varchar(256) | Yes | false| N/A  |       |
| column_20| varchar(256) | Yes | false| N/A  |       |
      column_21   varchar(256)    DEFAULT NULL,
      column_22   varchar(256)    DEFAULT NULL,
      column_23   varchar(256)    DEFAULT NULL,
      column_24   varchar(256)    DEFAULT NULL,
      column_25   varchar(256)    DEFAULT NULL,
      column_26   varchar(256)    DEFAULT NULL,
      column_27   varchar(256)    DEFAULT NULL,
      column_28   varchar(256)    DEFAULT NULL,
      column_29   varchar(256)    DEFAULT NULL,
      column_30   varchar(256)    DEFAULT NULL,
      column_31   varchar(256)    DEFAULT NULL,
      column_32   varchar(256)    DEFAULT NULL,
      column_33   varchar(256)    DEFAULT NULL,
      column_34   varchar(256)    DEFAULT NULL,
      column_35   varchar(256)    DEFAULT NULL,
      column_36   varchar(256)    DEFAULT NULL,
      column_37   varchar(256)    DEFAULT NULL
  )
  日付でのPARTITION
  表示された "day", ds からPARTITION BY
  idによるHASHでDISTRIBUTED BY;

  -- 複雑な式とWHERE句を用いたマテリアライズド・ビューを作成します。
  test_mv1という名前のMATERIALIZED VIEWを作成
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  user_idをbitmapに変換してunionする as user_id_dist_cnt,
  column_01が1より大きく、かつcolumn_34が('1','34')に含まれる場合にuser_id2を追加してnullでないものをbitmapに変換してunionする as filter_dist_cnt_1,
  column_02が60より大きく、かつcolumn_35が('11','13')に含まれる場合にuser_id2を追加してnullでないものをbitmapに変換してunionする as filter_dist_cnt_2,
  column_03が70より大きく、かつcolumn_36が('21','23')に含まれる場合にuser_id2を追加してnullでないものをbitmapに変換してunionする as filter_dist_cnt_3,
  column_04が20より大きく、かつcolumn_27が('31','27')に含まれる場合にuser_id2を追加してnullでないものをbitmapに変換してunionする as filter_dist_cnt_4,
  column_05が90より大きく、かつcolumn_28が('41','43')に含まれる場合にuser_id2を追加してnullでないものをbitmapに変換してunionする as filter_dist_cnt_5
  FROM user_event
  dsが'2023-11-02'以降のものを対象として、
  GROUP BY
  ds,
  column_19,
  column_36;
 ```

### 非同期マテリアライズド・ビューの例

以下の例は、次のベーステーブルに基づいています:

```SQL
CREATE TABLE `lineorder` (
  `lo_orderkey` int(11) NOT NULL COMMENT "",
  `lo_linenumber` int(11) NOT NULL COMMENT "",
  `lo_custkey` int(11) NOT NULL COMMENT "",
  `lo_partkey` int(11) NOT NULL COMMENT "",
  `lo_suppkey` int(11) NOT NULL COMMENT "",
  `lo_orderdate` int(11) NOT NULL COMMENT "",
  `lo_orderpriority` varchar(16) NOT NULL COMMENT "",
  `lo_shippriority` int(11) NOT NULL COMMENT "",
  `lo_quantity` int(11) NOT NULL COMMENT "",
  `lo_extendedprice` int(11) NOT NULL COMMENT "",
  `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
  `lo_discount` int(11) NOT NULL COMMENT "",
  `lo_revenue` int(11) NOT NULL COMMENT "",
  `lo_supplycost` int(11) NOT NULL COMMENT "",
  `lo_tax` int(11) NOT NULL COMMENT "",
  `lo_commitdate` int(11) NOT NULL COMMENT "",
  `lo_shipmode` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
`lo_orderkey`に重複するキーを持つ
COMMENT "OLAP"
`lo_orderdate`によってRANGEでPARTITIONを行い、
("day", ds)によるPARTITION BYでDISTRIBUTED BY HASH(`lo_orderkey`)でDISTRIBUTED BY;

CREATE TABLE IF NOT EXISTS `customer` (
  `c_custkey` int(11) NOT NULL COMMENT "",
  `c_name` varchar(26) NOT NULL COMMENT "",
  `c_address` varchar(41) NOT NULL COMMENT "",
  `c_city` varchar(11) NOT NULL COMMENT "",
  `c_nation` varchar(16) NOT NULL COMMENT "",
  `c_region` varchar(13) NOT NULL COMMENT "",
  `c_phone` varchar(16) NOT NULL COMMENT "",
  `c_mktsegment` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
`c_custkey`に重複するキーを持つ
COMMENT "OLAP"
DISTRIBUTED BY HASH(`c_custkey`);

CREATE TABLE IF NOT EXISTS `dates` (
  `d_datekey` int(11) NOT NULL COMMENT "",
  `d_date` varchar(20) NOT NULL COMMENT "",
  `d_dayofweek` varchar(10) NOT NULL COMMENT "",
  `d_month` varchar(11) NOT NULL COMMENT "",
  `d_year` int(11) NOT NULL COMMENT "",
  `d_yearmonthnum` int(11) NOT NULL COMMENT "",
  `d_yearmonth` varchar(9) NOT NULL COMMENT "",
  `d_daynuminweek` int(11) NOT NULL COMMENT "",
  `d_daynuminmonth` int(11) NOT NULL COMMENT "",
  `d_daynuminyear` int(11) NOT NULL COMMENT "",
  `d_monthnuminyear` int(11) NOT NULL COMMENT "",
  `d_weeknuminyear` int(11) NOT NULL COMMENT "",
  `d_sellingseason` varchar(14) NOT NULL COMMENT "",
  `d_lastdayinweekfl` int(11) NOT NULL COMMENT "",
  `d_lastdayinmonthfl` int(11) NOT NULL COMMENT "",
  `d_holidayfl` int(11) NOT NULL COMMENT "",
  `d_weekdayfl` int(11) NOT NULL COMMENT ""
) ENGINE=OLAP
`d_datekey`に重複するキーを持つ
COMMENT "OLAP"
DISTRIBUTED BY HASH(`d_datekey`);

CREATE TABLE IF NOT EXISTS `supplier` (
  `s_suppkey` int(11) NOT NULL COMMENT "",
  `s_name` varchar(26) NOT NULL COMMENT "",
  `s_address` varchar(26) NOT NULL COMMENT "",
  `s_city` varchar(11) NOT NULL COMMENT "",
  `s_nation` varchar(16) NOT NULL COMMENT "",
  `s_region` varchar(13) NOT NULL COMMENT "",
  `s_phone` varchar(16) NOT NULL COMMENT ""
) ENGINE=OLAP
`s_suppkey`に重複するキーを持つ
COMMENT "OLAP"
DISTRIBUTED BY HASH(`s_suppkey`);

CREATE TABLE IF NOT EXISTS `part` (
  `p_partkey` int(11) NOT NULL COMMENT "",
  `p_name` varchar(23) NOT NULL COMMENT "",
  `p_mfgr` varchar(7) NOT NULL COMMENT "",
  `p_category` varchar(8) NOT NULL COMMENT "",
  `p_brand` varchar(10) NOT NULL COMMENT "",
  `p_color` varchar(12) NOT NULL COMMENT "",
  `p_type` varchar(26) NOT NULL COMMENT "",
  `p_size` int(11) NOT NULL COMMENT "",
  `p_container` varchar(11) NOT NULL COMMENT ""
) ENGINE=OLAP
`p_partkey`に重複するキーを持つ
COMMENT "OLAP"
DISTRIBUTED BY HASH(`p_partkey`);

create table orders ( 
    dt date NOT NULL, 
    order_id bigint NOT NULL, 
    user_id int NOT NULL, 
    merchant_id int NOT NULL, 
    good_id int NOT NULL, 
    good_name string NOT NULL, 
    price int NOT NULL, 
    cnt int NOT NULL, 
    revenue int NOT NULL, 
    state tinyint NOT NULL 
) 
PRIMARY KEY (dt, order_id) 
RANGE(`dt`)でPARTITIONを行い、
( PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')), 
PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')) ) 
DISTRIBUTED BY HASH(order_id)
"replication_num" = "3", 
"enable_persistent_index" = "true"
というPROPERTIESを指定します。
```

Example 1: 非パーティション化されたマテリアライズド・ビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv1
`lo_orderkey`によるHASHでDISTRIBUTED BY
ASYNCを指定してREFRESH ASYNC
AS
select
    lo_orderkey, 
    lo_custkey, 
    lo_quantityの合計値としてtotal_quantity, 
    lo_revenueの合計値としてtotal_revenue, 
    lo_shipmodeの数としてshipmode_count
lineorderから取得し、
lo_orderkey, lo_custkeyでGROUP BYします。
lo_orderkeyでソートします。
```

Example 2: パーティション化されたマテリアライズド・ビューを作成します。

```SQL
`lo_orderdate`によってPARTITIONされ、`lo_orderkey`によるHASHでDISTRIBUTED BY
```SQL
CREATE MATERIALIZED VIEW flat_lineorder
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH MANUAL
AS
SELECT
    l.LO_ORDERKEY AS LO_ORDERKEY,
    l.LO_LINENUMBER AS LO_LINENUMBER,
    l.LO_CUSTKEY AS LO_CUSTKEY,
    l.LO_PARTKEY AS LO_PARTKEY,
    l.LO_SUPPKEY AS LO_SUPPKEY,
    l.LO_ORDERDATE AS LO_ORDERDATE,
    l.LO_ORDERPRIORITY AS LO_ORDERPRIORITY,
    l.LO_SHIPPRIORITY AS LO_SHIPPRIORITY,
    l.LO_QUANTITY AS LO_QUANTITY,
    l.LO_EXTENDEDPRICE AS LO_EXTENDEDPRICE,
    l.LO_ORDTOTALPRICE AS LO_ORDTOTALPRICE,
    l.LO_DISCOUNT AS LO_DISCOUNT,
    l.LO_REVENUE AS LO_REVENUE,
    l.LO_SUPPLYCOST AS LO_SUPPLYCOST,
    l.LO_TAX AS LO_TAX,
    l.LO_COMMITDATE AS LO_COMMITDATE,
    l.LO_SHIPMODE AS LO_SHIPMODE,
    c.C_NAME AS C_NAME,
    c.C_ADDRESS AS C_ADDRESS,
    c.C_CITY AS C_CITY,
    c.C_NATION AS C_NATION,
    c.C_REGION AS C_REGION,
    c.C_PHONE AS C_PHONE,
    c.C_MKTSEGMENT AS C_MKTSEGMENT,
    s.S_NAME AS S_NAME,
    s.S_ADDRESS AS S_ADDRESS,
    s.S_CITY AS S_CITY,
    s.S_NATION AS S_NATION,
    s.S_REGION AS S_REGION,
    s.S_PHONE AS S_PHONE,
    p.P_NAME AS P_NAME,
    p.P_MFGR AS P_MFGR,
    p.P_CATEGORY AS P_CATEGORY,
    p.P_BRAND AS P_BRAND,
    p.P_COLOR AS P_COLOR,
    p.P_TYPE AS P_TYPE,
    p.P_SIZE AS P_SIZE,
    p.P_CONTAINER FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

```SQL
CREATE MATERIALIZED VIEW order_mv1
PARTITION BY date_trunc('month', `dt`)
DISTRIBUTED BY HASH(`order_id`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    dt,
    order_id,
    user_id,
    sum(cnt) as total_cnt,
    sum(revenue) as total_revenue, 
    count(state) as state_count
from orders
group by dt, order_id, user_id;
```

```SQL
CREATE MATERIALIZED VIEW test_mv
PARTITION BY str2date(`d_datekey`,'%Y%m%d')
DISTRIBUTED BY HASH(`d_date`, `d_month`, `d_month`) 
REFRESH MANUAL 
AS
SELECT
`d_date` ,
  `d_dayofweek`,
  `d_month` ,
  `d_yearmonthnum` ,
  `d_yearmonth` ,
  `d_daynuminweek`,
  `d_daynuminmonth`,
  `d_daynuminyear` ,
  `d_monthnuminyear` ,
  `d_weeknuminyear` ,
  `d_sellingseason`,
  `d_lastdayinweekfl`,
  `d_lastdayinmonthfl`,
  `d_holidayfl` ,
  `d_weekdayfl`,
   `d_datekey`
FROM
 `hive_catalog`.`ssb_1g_orc`.`part_dates` ;
```