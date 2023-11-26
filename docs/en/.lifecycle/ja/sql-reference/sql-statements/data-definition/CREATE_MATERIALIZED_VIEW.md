---
displayed_sidebar: "Japanese"
---

# MATERIALIZED VIEWの作成

## 説明

MATERIALIZED VIEWを作成します。MATERIALIZED VIEWの使用方法については、[同期MATERIALIZED VIEW](../../../using_starrocks/Materialized_view-single_table.md)および[非同期MATERIALIZED VIEW](../../../using_starrocks/Materialized_view.md)を参照してください。

> **注意**
>
> ベーステーブルが存在するデータベースでCREATE MATERIALIZED VIEW権限を持つユーザーのみがMATERIALIZED VIEWを作成できます。

MATERIALIZED VIEWの作成は非同期の操作です。このコマンドが正常に実行されると、MATERIALIZED VIEWの作成タスクが正常に送信されたことを示します。データベース内の同期MATERIALIZED VIEWのビルド状態は[SHOW ALTER MATERIALIZED VIEW](../data-manipulation/SHOW_ALTER_MATERIALIZED_VIEW.md)コマンドで表示でき、非同期MATERIALIZED VIEWのビルド状態は[Information Schema](../../../reference/information_schema/information_schema.md)の`tasks`および`task_runs`メタデータビューをクエリすることで表示できます。

StarRocksはv2.4から非同期MATERIALIZED VIEWをサポートしています。以前のバージョンの同期MATERIALIZED VIEWと非同期MATERIALIZED VIEWの主な違いは次のとおりです。

|                       | **単一テーブルの集計** | **複数テーブルの結合** | **クエリの書き換え** | **リフレッシュ戦略** | **ベーステーブル** |
| --------------------- | ---------------------------- | -------------------- | ----------------- | -------------------- | -------------- |
| **非同期MV** | はい | はい | はい | <ul><li>非同期リフレッシュ</li><li>手動リフレッシュ</li></ul> | デフォルトカタログからの複数のテーブル:<ul><li>デフォルトカタログ</li><li>外部カタログ（v2.5）</li><li>既存のMATERIALIZED VIEW（v2.5）</li><li>既存のビュー（v3.1）</li></ul> |
| **同期MV（ロールアップ）**  | 集約関数の選択肢が制限される | いいえ | はい | データのロード中に同期リフレッシュ | デフォルトカタログの単一テーブル |

## 同期MATERIALIZED VIEW

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

角括弧[]内のパラメータはオプションです。

### パラメータ

**mv_name**（必須）

MATERIALIZED VIEWの名前です。名前の要件は次のとおりです。

- 名前は文字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（_）で構成する必要があり、最初の文字は文字でなければなりません。
- 名前の長さは64文字を超えることはできません。
- 名前は大文字と小文字を区別します。

**COMMENT**（オプション）

MATERIALIZED VIEWに関するコメントです。`COMMENT`は`mv_name`の後に配置する必要があります。そうしないと、MATERIALIZED VIEWを作成できません。

**query_statement**（必須）

MATERIALIZED VIEWを作成するためのクエリ文です。その結果はMATERIALIZED VIEWのデータです。構文は次のとおりです。

```SQL
SELECT select_expr[, select_expr ...]
[WHERE where_expr]
[GROUP BY column_name[, column_name ...]]
[ORDER BY column_name[, column_name ...]]
```

- select_expr（必須）

  クエリ文のすべての列、つまりMATERIALIZED VIEWスキーマのすべての列です。このパラメータは次の値をサポートします。

  - ベーステーブルの列名である`SELECT a, abs(b), min(c) FROM table_a`などの単純な列または集計列。MATERIALIZED VIEWの列に列名を指定しない場合、StarRocksは自動的に列に名前を割り当てます。
  - ベーステーブルの列を参照する式である`SELECT a+1 AS x, b+2 AS y, c*c AS z FROM table_a`などの式。`a+1`、`b+2`、`c*c`はベーステーブルの列を参照する式であり、`x`、`y`、`z`はMATERIALIZED VIEWの列に割り当てられるエイリアスです。

  > **注意**
  >
  > - `select_expr`には少なくとも1つの列を指定する必要があります。
  > - 集約関数を使用して同期MATERIALIZED VIEWを作成する場合、GROUP BY句を指定し、`select_expr`で少なくとも1つのGROUP BY列を指定する必要があります。
  > - 同期MATERIALIZED VIEWはJOIN、WHERE、およびGROUP BYのHAVING句をサポートしていません。
  > - v3.1以降、同期MATERIALIZED VIEWでは、各列のベーステーブルの複数の集計関数をサポートできます。たとえば、`select b, sum(a), min(a) from table group by b`などのクエリ文が使用できます。
  > - v3.1以降、同期MATERIALIZED VIEWはSELECTおよび集計関数に複雑な式をサポートできます。たとえば、`select b, sum(a + 1) as sum_a1, min(cast (a as bigint)) as min_a from table group by b`や`select abs(b) as col1, a + 1 as col2, cast(a as bigint) as col3 from table`などのクエリ文が使用できます。同期MATERIALIZED VIEWで使用される複雑な式には次の制限があります。
  >   - 各複雑な式にはエイリアスが必要であり、すべての同期MATERIALIZED VIEWの複雑な式に異なるエイリアスを割り当てる必要があります。たとえば、クエリ文`select b, sum(a + 1) as sum_a from table group by b`と`select b, sum(a) as sum_a from table group by b`は同じベーステーブルの同期MATERIALIZED VIEWを作成するために使用できません。複雑な式には異なるエイリアスを設定できます。
  >   - 同期MATERIALIZED VIEWで使用される複雑な式によってクエリが書き換えられたかどうかは、`EXPLAIN <sql_statement>`を実行して確認できます。詳細については、[クエリ解析](../../../administration/Query_planning.md)を参照してください。

- WHERE（オプション）

  v3.2以降、同期MATERIALIZED VIEWは、MATERIALIZED VIEWに使用される行をフィルタリングするWHERE句をサポートしています。

- GROUP BY（オプション）

  クエリのGROUP BY列です。このパラメータが指定されていない場合、データはデフォルトでグループ化されません。

- ORDER BY（オプション）

  クエリのORDER BY列です。

  - ORDER BY句の列は、`select_expr`の列と同じ順序で宣言する必要があります。
  - クエリ文にGROUP BY句が含まれている場合、ORDER BY列はGROUP BY列と同じでなければなりません。
  - このパラメータが指定されていない場合、システムは次のルールに従って自動的にORDER BY列を補完します：
    - MATERIALIZED VIEWがAGGREGATEタイプの場合、すべてのGROUP BY列が自動的にソートキーとして使用されます。
    - MATERIALIZED VIEWがAGGREGATEタイプでない場合、StarRocksはプレフィックス列に基づいてソートキーを自動的に選択します。

### 同期MATERIALIZED VIEWのクエリ

同期MATERIALIZED VIEWは、物理テーブルです。通常のテーブルと同様に操作できますが、**同期MATERIALIZED VIEWに直接データをロードすることはできません**。

### 同期MATERIALIZED VIEWによる自動クエリ書き換え

同期MATERIALIZED VIEWのパターンに従うクエリが実行されると、元のクエリ文が自動的に書き換えられ、MATERIALIZED VIEWに格納された中間結果が使用されます。

次の表は、元のクエリの集計関数とMATERIALIZED VIEWの構築に使用される集計関数の対応関係を示しています。ビジネスシナリオに応じて、対応する集計関数を選択してMATERIALIZED VIEWを構築できます。

| **元のクエリの集計関数**           | **MATERIALIZED VIEWの集計関数** |
| ---------------------------------- | ------------------------------- |
| sum                                | sum                             |
| min                                | min                             |
| max                                | max                             |
| count                              | count                           |
| bitmap_union、bitmap_union_count、count(distinct) | bitmap_union                    |
| hll_raw_agg、hll_union_agg、ndv、approx_count_distinct | hll_union                       |
| percentile_approx、percentile_union | percentile_union                |

## 非同期MATERIALIZED VIEW

### 構文

```SQL
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [database.]<mv_name>
[COMMENT ""]
-- distribution_desc
[DISTRIBUTED BY HASH(<bucket_key>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]]
-- refresh_desc
[REFRESH 
-- refresh_moment
    [IMMEDIATE | DEFERRED]
-- refresh_scheme
    [ASYNC [START (<start_time>)] [EVERY (INTERVAL <refresh_interval>)] | MANUAL]
]
-- partition_expression
[PARTITION BY 
    {<date_column> | date_trunc(fmt, <date_column>)}
]
-- order_by_expression
[ORDER BY (<sort_key>)]
[PROPERTIES ("key"="value", ...)]
AS 
<query_statement>
```

角括弧[]内のパラメータはオプションです。

### パラメータ

**mv_name**（必須）

MATERIALIZED VIEWの名前です。名前の要件は次のとおりです。

- 名前は文字（a-zまたはA-Z）、数字（0-9）、またはアンダースコア（_）で構成する必要があり、最初の文字は文字でなければなりません。
- 名前の長さは64文字を超えることはできません。
- 名前は大文字と小文字を区別します。

> **注意**
>
> 同じデータベース内のMATERIALIZED VIEWの名前は重複してはなりません。

**COMMENT**（オプション）

MATERIALIZED VIEWに関するコメントです。`COMMENT`は`mv_name`の後に配置する必要があります。そうしないと、MATERIALIZED VIEWを作成できません。

**distribution_desc**（オプション）

非同期MATERIALIZED VIEWのバケット分割戦略です。StarRocksはハッシュ分割とランダム分割（v3.1以降）をサポートしています。このパラメータを指定しない場合、StarRocksはランダム分割戦略を使用し、バケットの数を自動的に設定します。

> **注意**
>
> 非同期MATERIALIZED VIEWを作成する場合、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

- **ハッシュ分割**：

  構文

  ```SQL
  DISTRIBUTED BY HASH (<bucket_key1>[,<bucket_key2> ...]) [BUCKETS <bucket_number>]
  ```

  詳細については、[データ分散](../../../table_design/Data_distribution.md#data-distribution)を参照してください。

  > **注意**
  >
  > v2.5.7以降、StarRocksはテーブルの作成またはパーティションの追加時にバケットの数（BUCKETS）を自動的に設定できます。バケットの数を手動で設定する必要はありません。詳細については、[バケットの数を決定する](../../../table_design/Data_distribution.md#determine-the-number-of-buckets)を参照してください。

- **ランダム分割**：

  ランダム分割戦略を選択し、StarRocksにバケットの数を自動的に設定させる場合は、`distribution_desc`を指定する必要はありません。ただし、バケットの数を手動で設定する場合は、次の構文を参照してください。

  ```SQL
  DISTRIBUTED BY RANDOM BUCKETS <bucket_number>
  ```

  > **注意**
  >
  > ランダム分割戦略を使用する非同期MATERIALIZED VIEWは、コロケーショングループに割り当てることはできません。

  詳細については、[ランダム分割](../../../table_design/Data_distribution.md#random-bucketing-since-v31)を参照してください。

**refresh_moment**（オプション）

MATERIALIZED VIEWのリフレッシュタイミングです。デフォルト値は`IMMEDIATE`です。有効な値は次のとおりです。

- `IMMEDIATE`：MATERIALIZED VIEWが作成された直後に非同期MATERIALIZED VIEWをリフレッシュします。
- `DEFERRED`：非同期MATERIALIZED VIEWは作成後にリフレッシュされません。MATERIALIZED VIEWを手動でリフレッシュするか、定期的なリフレッシュタスクをスケジュールできます。

**refresh_scheme**（オプション）

> **注意**
>
> 非同期MATERIALIZED VIEWを作成する場合、`distribution_desc`または`refresh_scheme`、またはその両方を指定する必要があります。

非同期MATERIALIZED VIEWのリフレッシュ戦略です。有効な値は次のとおりです。

- `ASYNC`：非同期リフレッシュモード。ベーステーブルのデータが変更されるたびに、事前に定義されたリフレッシュ間隔に従ってMATERIALIZED VIEWが自動的にリフレッシュされます。リフレッシュの開始時刻を`START('yyyy-MM-dd hh:mm:ss')`として指定し、リフレッシュ間隔を`EVERY (interval n day/hour/minute/second)`として指定できます。使用できる単位は`DAY`、`HOUR`、`MINUTE`、`SECOND`です。例：`ASYNC START ('2023-09-12 16:30:25') EVERY (INTERVAL 5 MINUTE)`。間隔を指定しない場合、デフォルト値の`10 MINUTE`が使用されます。
- `MANUAL`：手動リフレッシュモード。非同期MATERIALIZED VIEWは自動的にリフレッシュされません。リフレッシュタスクはユーザーによって手動でトリガーすることができます。

このパラメータが指定されていない場合、デフォルト値の`MANUAL`が使用されます。

**partition_expression**（オプション）

非同期MATERIALIZED VIEWのパーティショニング戦略です。現在のバージョンのStarRocksでは、非同期MATERIALIZED VIEWを作成する際には1つのパーティション式のみがサポートされています。

> **注意**
>
> 現在、非同期MATERIALIZED VIEWはリストパーティショニング戦略をサポートしていません。

有効な値は次のとおりです。

- `column_name`：パーティショニングに使用する列の名前です。`PARTITION BY dt`という式は、`dt`列に基づいてMATERIALIZED VIEWをパーティション分割することを意味します。
- `date_trunc`関数：時間単位を切り捨てるために使用される関数です。`PARTITION BY date_trunc("MONTH", dt)`は、`dt`列を月単位で切り捨ててパーティション分割することを意味します。`date_trunc`関数は、`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`などの単位で時間を切り捨てることができます。
- `str2date`関数：ベーステーブルの文字列型パーティションをMATERIALIZED VIEWのパーティションに分割するために使用される関数です。`PARTITION BY str2date(dt, "%Y%m%d")`は、`dt`列が文字列の日付型であり、日付形式が`"%Y%m%d"`であることを意味します。`str2date`関数は多くの日付形式をサポートしており、詳細については[str2date](../../sql-functions/date-time-functions/str2date.md)を参照してください。v3.1.4以降でサポートされています。
- `time_slice`または`date_slice`関数：v3.1以降、これらの関数を使用して指定された時間を指定された時間粒度に基づいて時間間隔の始まりまたは終わりに変換できます。たとえば、`PARTITION BY date_trunc("MONTH", time_slice(dt, INTERVAL 7 DAY))`のように使用できます。time_sliceとdate_sliceは、date_truncよりも細かい粒度であるGROUP BY列を指定するために使用できます。たとえば、`GROUP BY time_slice(dt, INTERVAL 1 MINUTE) PARTITION BY date_trunc('DAY', ts)`のように使用できます。

このパラメータが指定されていない場合、デフォルトではパーティショニング戦略は採用されません。

**order_by_expression**（オプション）

非同期MATERIALIZED VIEWのソートキーです。ソートキーを指定しない場合、StarRocksはSELECT列の一部のプレフィックス列をソートキーとして選択します。たとえば、`select a, b, c, d`では、ソートキーは`a`と`b`になります。このパラメータはStarRocks v3.0以降でサポートされています。

**PROPERTIES**（オプション）

非同期MATERIALIZED VIEWのプロパティです。[ALTER MATERIALIZED VIEW](./ALTER_MATERIALIZED_VIEW.md)を使用して既存のMATERIALIZED VIEWのプロパティを変更できます。

- `session.`：MATERIALIZED VIEWのセッション変数関連のプロパティを変更する場合は、プロパティの前に`session.`接頭辞を追加する必要があります。たとえば、`session.query_timeout`というプロパティの場合、非セッションプロパティの場合は接頭辞を指定する必要はありません。たとえば、`mv_rewrite_staleness_second`です。
- `replication_num`：作成するMATERIALIZED VIEWのレプリカ数です。
- `storage_medium`：ストレージメディアのタイプです。有効な値は`HDD`および`SSD`です。
- `storage_cooldown_time`：パーティションのストレージの冷却時間です。HDDとSSDのストレージ両方を使用する場合、このプロパティで指定された時間が経過すると、SSDストレージのデータはHDDストレージに移動されます。形式：「yyyy-MM-dd HH:mm:ss」。指定した時間は現在の時間よりも後である必要があります。このプロパティが明示的に指定されていない場合、デフォルトではストレージの冷却は実行されません。
- `partition_ttl`：パーティションの有効期限（TTL）です。指定された時間範囲内のデータを持つパーティションが保持されます。期限が切れたパーティションは自動的に削除されます。単位：`YEAR`、`MONTH`、`DAY`、`HOUR`、`MINUTE`。たとえば、このプロパティを`2 MONTH`と指定できます。このプロパティは`partition_ttl_number`よりも推奨されます。v3.1.5以降でサポートされています。
- `partition_ttl_number`：保持する最新のMATERIALIZED VIEWパーティションの数です。現在の時間よりも前の開始時間を持つパーティションについて、これらのパーティションの数がこの値を超えると、より古いパーティションが削除されます。StarRocksは、FE設定項目`dynamic_partition_check_interval_seconds`で指定された時間間隔で定期的にMATERIALIZED VIEWパーティションをチェックし、期限切れのパーティションを自動的に削除します。[動的パーティショニング](../../../table_design/dynamic_partitioning.md)戦略を有効にした場合、事前に作成されたパーティションはカウントされません。値が`-1`の場合、MATERIALIZED VIEWのすべてのパーティションが保持されます。デフォルト：`-1`。
- `partition_refresh_number`：単一のリフレッシュでリフレッシュする最大のパーティション数です。リフレッシュするパーティションの数がこの値を超える場合、StarRocksはリフレッシュタスクを分割してバッチで完了します。以前のバッチのパーティションが正常にリフレッシュされた場合のみ、StarRocksは次のバッチのパーティションをリフレッシュし続け、すべてのパーティションがリフレッシュされるまで続けます。パーティションのリフレッシュに失敗した場合、以降のリフレッシュタスクは生成されません。値が`-1`の場合、リフレッシュタスクは分割されません。デフォルト：`-1`。
- `excluded_trigger_tables`：MATERIALIZED VIEWのベーステーブルがリストされている場合、ベーステーブルのデータが変更されたときに自動リフレッシュタスクはトリガーされません。このパラメータはロードトリガーによるリフレッシュ戦略にのみ適用され、通常はプロパティ`auto_refresh_partitions_limit`と一緒に使用されます。形式：`[db_name.]table_name`。値が空の文字列の場合、すべてのベーステーブルのデータ変更は対応するMATERIALIZED VIEWのリフレッシュをトリガーします。デフォルト値は空の文字列です。
- `auto_refresh_partitions_limit`：MATERIALIZED VIEWリフレッシュがトリガーされたときにリフレッシュする必要のある最新のMATERIALIZED VIEWパーティションの数です。このプロパティを使用してリフレッシュ範囲を制限し、リフレッシュコストを削減できます。ただし、すべてのパーティションがリフレッシュされないため、MATERIALIZED VIEWのデータはベーステーブルと一致しない場合があります。デフォルト：`-1`。値が`-1`の場合、すべてのパーティションがリフレッシュされます。正の整数Nの場合、StarRocksは既存のパーティションを時系列順にソートし、現在のパーティションとN-1つの最新のパーティションをリフレッシュします。パーティションの数がNよりも少ない場合、StarRocksはすべての既存のパーティションをリフレッシュします。事前に作成されたパーティションがある場合、StarRocksはすべての事前に作成されたパーティションをリフレッシュします。
- `mv_rewrite_staleness_second`：MATERIALIZED VIEWの最後のリフレッシュがこのプロパティで指定された時間間隔内にある場合、ベーステーブルのデータが変更されたかどうかに関係なく、このMATERIALIZED VIEWはクエリ書き換えに使用できます。最後のリフレッシュがこの時間間隔よりも前の場合、StarRocksはベーステーブルが更新されたかどうかをチェックして、MATERIALIZED VIEWがクエリ書き換えに使用できるかどうかを判断します。単位：秒。v3.0以降でサポートされています。
- `colocate_with`：非同期MATERIALIZED VIEWのコロケーショングループです。詳細については、[コロケーション結合](../../../using_starrocks/Colocate_join.md)を参照してください。v3.0以降でサポートされています。
- `unique_constraints`および`foreign_key_constraints`：View Delta Joinシナリオでクエリ書き換えのために非同期MATERIALIZED VIEWを作成する場合のUnique Key制約およびForeign Key制約です。詳細については、[非同期MATERIALIZED VIEW - View Delta Joinシナリオでのクエリ書き換え](../../../using_starrocks/Materialized_view.md#rewrite-queries-in-view-delta-join-scenario)を参照してください。v3.0以降でサポートされています。
- `resource_group`：MATERIALIZED VIEWのリフレッシュタスクが属するリソースグループです。リソースグループについての詳細は[リソースグループ](../../../administration/resource_group.md)を参照してください。
- `query_rewrite_consistency`：非同期MATERIALIZED VIEWのクエリ書き換えルールです。v3.2以降でサポートされています。有効な値は次のとおりです。
  - `disable`：非同期MATERIALIZED VIEWの自動クエリ書き換えを無効にします。
  - `checked`（デフォルト値）：MATERIALIZED VIEWがタイムリネス要件を満たす場合にのみ、自動クエリ書き換えを有効にします。タイムリネス要件とは次の場合です。
    - `mv_rewrite_staleness_second`が指定されていない場合、MATERIALIZED VIEWのデータがすべてのベーステーブルのデータと一致する場合のみ、クエリ書き換えにMATERIALIZED VIEWを使用できます。
    - `mv_rewrite_staleness_second`が指定されている場合、MATERIALIZED VIEWの最後のリフレッシュがタイムリネス時間間隔内にある場合、クエリ書き換えにMATERIALIZED VIEWを使用できます。
  - `loose`：直接に自動クエリ書き換えを有効にし、一貫性チェックは必要ありません。
- `force_external_table_query_rewrite`：外部カタログベースのMATERIALIZED VIEWのクエリ書き換えを有効にするかどうか。v3.2以降でサポートされています。有効な値は次のとおりです。
  - `true`：外部カタログベースのMATERIALIZED VIEWのクエリ書き換えを有効にします。
  - `false`（デフォルト値）：外部カタログベースのMATERIALIZED VIEWのクエリ書き換えを無効にします。

  ベーステーブルと外部カタログベースのMATERIALIZED VIEWの間にはデータの強力な一貫性が保証されないため、この機能はデフォルトで`false`に設定されています。この機能が有効になっている場合、MATERIALIZED VIEWは`query_rewrite_consistency`で指定されたルールに従ってクエリ書き換えに使用されます。

> **注意**
>
> Unique Key制約とForeign Key制約はクエリ書き換えにのみ使用されます。データがテーブルにロードされるときにForeign Key制約のチェックは保証されません。テーブルにロードされるデータが制約を満たしていることを確認する必要があります。

**query_statement**（必須）

非同期MATERIALIZED VIEWを作成するためのクエリ文です。

> **注意**
>
> 現在、StarRocksはリストパーティショニング戦略で作成されたベーステーブルで非同期MATERIALIZED VIEWを作成することはサポートしていません。

### 非同期MATERIALIZED VIEWのクエリ

非同期MATERIALIZED VIEWは物理テーブルです。通常のテーブルと同様に操作できますが、**非同期MATERIALIZED VIEWに直接データをロードすることはできません**。

### 非同期MATERIALIZED VIEWによる自動クエリ書き換え

StarRocks v2.5は、SPJGタイプの非同期MATERIALIZED VIEWに基づく自動および透過的なクエリ書き換えをサポートしています。SPJGタイプのMATERIALIZED VIEWは、スキャン、フィルタ、プロジェクト、および集計のタイプの演算子のみを含むプランを持つMATERIALIZED VIEWを指します。SPJGタイプのMATERIALIZED VIEWのクエリ書き換えには、単一テーブルのクエリ書き換え、結合クエリの書き換え、集計クエリの書き換え、UNIONクエリの書き換え、およびネストされたMATERIALIZED VIEWに基づくクエリ書き換えが含まれます。

詳細については、[非同期MATERIALIZED VIEW - 非同期MATERIALIZED VIEWによるクエリ書き換え](../../../using_starrocks/Materialized_view.md#rewrite_queries_with_the_asynchronous_materialized_view)を参照してください。

### サポートされるデータ型

- StarRocksのデフォルトカタログを基に作成された非同期MATERIALIZED VIEWは、次のデータ型をサポートしています。

  - **日付**：DATE、DATETIME
  - **文字列**：CHAR、VARCHAR
  - **数値**：BOOLEAN、TINYINT、SMALLINT、INT、BIGINT、LARGEINT、FLOAT、DOUBLE、DECIMAL、PERCENTILE
  - **半構造化**：ARRAY、JSON、MAP（v3.1以降）、STRUCT（v3.1以降）
  - **その他**：BITMAP、HLL

> **注意**
>
> BITMAP、HLL、およびPERCENTILEはv2.4.5以降でサポートされています。

- StarRocksの外部カタログを基に作成された非同期MATERIALIZED VIEWは、次のデータ型をサポートしています。

  - Hiveカタログ

    - **数値**：INT/INTEGER、BIGINT、DOUBLE、FLOAT、DECIMAL
    - **日付**：TIMESTAMP
    - **文字列**：STRING、VARCHAR、CHAR
    - **半構造化**：ARRAY

  - Hudiカタログ

    - **数値**：BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL
    - **日付**：DATE、TimeMillis/TimeMicros、TimestampMillis/TimestampMicros
    - **文字列**：STRING
    - **半構造化**：ARRAY

  - Icebergカタログ

    - **数値**：BOOLEAN、INT、LONG、FLOAT、DOUBLE、DECIMAL(P, S)
    - **日付**：DATE、TIME、TIMESTAMP
    - **文字列**：STRING、UUID、FIXED(L)、BINARY
    - **半構造化**：LIST

## 使用上の注意

- 現在のバージョンのStarRocksでは、複数のMATERIALIZED VIEWを同時に作成することはサポートされていません。新しいMATERIALIZED VIEWは、前のMATERIALIZED VIEWが完了した後にのみ作成できます。

- 同期MATERIALIZED VIEWについて：

  - 同期MATERIALIZED VIEWは、単一の列に対する集計関数のみをサポートしています。`sum(a+b)`のような形式のクエリ文はサポートされていません。
  - 同期MATERIALIZED VIEWは、ベーステーブルの各列に対して1つの集計関数のみをサポートしています。`select sum(a), min(a) from table`のようなクエリ文はサポートされていません。
  - 集計関数を使用して同期MATERIALIZED VIEWを作成する場合、GROUP BY句を指定し、SELECTで少なくとも1つのGROUP BY列を指定する必要があります。
  - 同期MATERIALIZED VIEWはJOIN、WHERE、およびGROUP BYのHAVING句をサポートしていません。
  - ALTER TABLE DROP COLUMNを使用してベーステーブルの特定の列を削除する場合、削除される列を含む同期MATERIALIZED VIEWが存在しないことを確認する必要があります。列を削除する前に、削除される列を含むすべての同期MATERIALIZED VIEWを最初に削除する必要があります。
  - テーブルに対して同期MATERIALIZED VIEWを多数作成すると、データのロード効率に影響を与えます。ベーステーブルにデータがロードされている間、同期MATERIALIZED VIEWとベーステーブルのデータは同期して更新されます。ベーステーブルに`n`個の同期MATERIALIZED VIEWが含まれる場合、ベーステーブルにデータをロードする効率は、`n`個のテーブルにデータをロードする効率とほぼ同じです。

- ネストされた非同期MATERIALIZED VIEWについて：

  - 各MATERIALIZED VIEWのリフレッシュ戦略は、対応するMATERIALIZED VIEWにのみ適用されます。
  - 現在、StarRocksはネストのレベル数を制限していません。本番環境では、ネストのレイヤー数が3を超えないように推奨します。

- 外部カタログ非同期マテリアライズドビューについて：

  - 外部カタログマテリアライズドビューは、非同期の固定間隔リフレッシュと手動リフレッシュのみをサポートしています。
  - 外部カタログのマテリアライズドビューとベーステーブルの間には厳密な一貫性が保証されません。
  - 現在、外部リソースを基にしたマテリアライズドビューの構築はサポートされていません。
  - 現在、StarRocksは外部カタログのベーステーブルのデータが変更されたかどうかを認識することができないため、ベーステーブルがリフレッシュされるたびにすべてのパーティションがデフォルトでリフレッシュされます。[REFRESH MATERIALIZED VIEW](../data-manipulation/REFRESH_MATERIALIZED_VIEW.md)を使用して一部のパーティションのみを手動でリフレッシュすることができます。

## 例

### 同期マテリアライズドビューの例

ベーステーブルのスキーマは次のようになります：

```Plain Text
mysql> desc duplicate_table;
+-------+--------+------+------+---------+-------+
| Field | Type   | Null | Key  | Default | Extra |
+-------+--------+------+------+---------+-------+
| k1    | INT    | Yes  | true | N/A     |       |
| k2    | INT    | Yes  | true | N/A     |       |
| k3    | BIGINT | Yes  | true | N/A     |       |
| k4    | BIGINT | Yes  | true | N/A     |       |
+-------+--------+------+------+---------+-------+
```

例1：元のテーブル（k1、k2）の列のみを含む同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2 as
select k1, k2 from duplicate_table;
```

マテリアライズドビューには、集計なしでk1とk2の2つの列のみが含まれています。

```plain text
+-----------------+-------+--------+------+------+---------+-------+
| IndexName       | Field | Type   | Null | Key  | Default | Extra |
+-----------------+-------+--------+------+------+---------+-------+
| k1_k2           | k1    | INT    | Yes  | true | N/A     |       |
|                 | k2    | INT    | Yes  | true | N/A     |       |
+-----------------+-------+--------+------+------+---------+-------+
```

例2：k2でソートされた同期マテリアライズドビューを作成します。

```sql
create materialized view k2_order as
select k2, k1 from duplicate_table order by k2;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューには、集計なしでk2とk1の2つの列のみが含まれており、k2列はソート列です。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k2_order        | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k1    | INT    | Yes  | false | N/A     | NONE  |
+-----------------+-------+--------+------+-------+---------+-------+
```

例3：k1とk2でグループ化され、k3に対するSUM集計を行う同期マテリアライズドビューを作成します。

```sql
create materialized view k1_k2_sumk3 as
select k1, k2, sum(k3) from duplicate_table group by k1, k2;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューには、k1、k2の2つのグループ化列と、k1とk2に基づいてグループ化されたk3列の合計が含まれています。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| k1_k2_sumk3     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | false | N/A     | SUM   |
+-----------------+-------+--------+------+-------+---------+-------+
```

マテリアライズドビューはソート列を宣言していないため、および集計関数を採用しているため、StarRocksはデフォルトでグループ化列k1とk2を補完します。

例4：重複行を削除する同期マテリアライズドビューを作成します。

```sql
create materialized view deduplicate as
select k1, k2, k3, k4 from duplicate_table group by k1, k2, k3, k4;
```

マテリアライズドビューのスキーマは以下の通りです。マテリアライズドビューにはk1、k2、k3、k4の列が含まれており、重複する行はありません。

```plain text
+-----------------+-------+--------+------+-------+---------+-------+
| IndexName       | Field | Type   | Null | Key   | Default | Extra |
+-----------------+-------+--------+------+-------+---------+-------+
| deduplicate     | k1    | INT    | Yes  | true  | N/A     |       |
|                 | k2    | INT    | Yes  | true  | N/A     |       |
|                 | k3    | BIGINT | Yes  | true  | N/A     |       |
|                 | k4    | BIGINT | Yes  | true  | N/A     |       |
+-----------------+-------+--------+------+-------+---------+-------+
```

例5：ソート列を宣言しない非集計の同期マテリアライズドビューを作成します。

ベーステーブルのスキーマは以下の通りです：

```plain text
+-------+--------------+------+-------+---------+-------+
| Field | Type         | Null | Key   | Default | Extra |
+-------+--------------+------+-------+---------+-------+
| k1    | TINYINT      | Yes  | true  | N/A     |       |
| k2    | SMALLINT     | Yes  | true  | N/A     |       |
| k3    | INT          | Yes  | true  | N/A     |       |
| k4    | BIGINT       | Yes  | true  | N/A     |       |
| k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
| k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
| k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
+-------+--------------+------+-------+---------+-------+
```

マテリアライズドビューにはk3、k4、k5、k6、k7の列が含まれており、ソート列は宣言されていません。次のステートメントでマテリアライズドビューを作成します。

```sql
create materialized view mv_1 as
select k3, k4, k5, k6, k7 from all_type_table;
```

StarRocksはデフォルトでk3、k4、k5をソート列として使用します。これらの3つの列の占有バイト数の合計は4（INT）+ 8（BIGINT）+ 16（DECIMAL）= 28 < 36です。したがって、これらの3つの列がソート列として追加されます。

マテリアライズドビューのスキーマは以下の通りです。

```plain text
+----------------+-------+--------------+------+-------+---------+-------+
| IndexName      | Field | Type         | Null | Key   | Default | Extra |
+----------------+-------+--------------+------+-------+---------+-------+
| mv_1           | k3    | INT          | Yes  | true  | N/A     |       |
|                | k4    | BIGINT       | Yes  | true  | N/A     |       |
|                | k5    | DECIMAL(9,0) | Yes  | true  | N/A     |       |
|                | k6    | DOUBLE       | Yes  | false | N/A     | NONE  |
|                | k7    | VARCHAR(20)  | Yes  | false | N/A     | NONE  |
+----------------+-------+--------------+------+-------+---------+-------+
```

k3、k4、k5列の`key`フィールドが`true`であることがわかります。これは、それらがソートキーであることを示しています。k6、k7列のキーフィールドは`false`であり、ソートキーではないことを示しています。

例6：WHERE句と複雑な式を含む同期マテリアライズドビューを作成します。

```SQL
-- ベーステーブル：user_eventを作成します。
CREATE TABLE user_event (
      ds date   NOT NULL,
      id  varchar(256)    NOT NULL,
      user_id int DEFAULT NULL,
      user_id1    varchar(256)    DEFAULT NULL,
      user_id2    varchar(256)    DEFAULT NULL,
      column_01   int DEFAULT NULL,
      column_02   int DEFAULT NULL,
      column_03   int DEFAULT NULL,
      column_04   int DEFAULT NULL,
      column_05   int DEFAULT NULL,
      column_06   DECIMAL(12,2)   DEFAULT NULL,
      column_07   DECIMAL(12,3)   DEFAULT NULL,
      column_08   JSON   DEFAULT NULL,
      column_09   DATETIME    DEFAULT NULL,
      column_10   DATETIME    DEFAULT NULL,
      column_11   DATE    DEFAULT NULL,
      column_12   varchar(256)    DEFAULT NULL,
      column_13   varchar(256)    DEFAULT NULL,
      column_14   varchar(256)    DEFAULT NULL,
      column_15   varchar(256)    DEFAULT NULL,
      column_16   varchar(256)    DEFAULT NULL,
      column_17   varchar(256)    DEFAULT NULL,
      column_18   varchar(256)    DEFAULT NULL,
      column_19   varchar(256)    DEFAULT NULL,
      column_20   varchar(256)    DEFAULT NULL,
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
  PARTITION BY date_trunc("day", ds)
  DISTRIBUTED BY hash(id);

  -- WHERE句と複雑な式を含むマテリアライズドビューを作成します。
  CREATE MATERIALIZED VIEW test_mv1
  AS 
  SELECT
  ds,
  column_19,
  column_36,
  sum(column_01) as column_01_sum,
  bitmap_union(to_bitmap( user_id)) as user_id_dist_cnt,
  bitmap_union(to_bitmap(case when column_01 > 1 and column_34 IN ('1','34')   then user_id2 else null end)) as filter_dist_cnt_1,
  bitmap_union(to_bitmap( case when column_02 > 60 and column_35 IN ('11','13') then  user_id2 else null end)) as filter_dist_cnt_2,
  bitmap_union(to_bitmap(case when column_03 > 70 and column_36 IN ('21','23') then  user_id2 else null end)) as filter_dist_cnt_3,
  bitmap_union(to_bitmap(case when column_04 > 20 and column_27 IN ('31','27') then  user_id2 else null end)) as filter_dist_cnt_4,
  bitmap_union(to_bitmap( case when column_05 > 90 and column_28 IN ('41','43') then  user_id2 else null end)) as filter_dist_cnt_5
  FROM user_event
  WHERE ds >= '2023-11-02'
  GROUP BY
  ds,
  column_19,
  column_36;
 ```

### 非同期マテリアライズドビューの例

以下の例は、次のベーステーブルに基づいています：

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
DUPLICATE KEY(`lo_orderkey`)
COMMENT "OLAP"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [("-2147483648"), ("19930101")),
PARTITION p2 VALUES [("19930101"), ("19940101")),
PARTITION p3 VALUES [("19940101"), ("19950101")),
PARTITION p4 VALUES [("19950101"), ("19960101")),
PARTITION p5 VALUES [("19960101"), ("19970101")),
PARTITION p6 VALUES [("19970101"), ("19980101")),
PARTITION p7 VALUES [("19980101"), ("19990101")))
DISTRIBUTED BY HASH(`lo_orderkey`);

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
DUPLICATE KEY(`c_custkey`)
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
DUPLICATE KEY(`d_datekey`)
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
DUPLICATE KEY(`s_suppkey`)
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
DUPLICATE KEY(`p_partkey`)
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
PARTITION BY RANGE(`dt`) 
( PARTITION p20210820 VALUES [('2021-08-20'), ('2021-08-21')), 
PARTITION p20210821 VALUES [('2021-08-21'), ('2021-08-22')) ) 
DISTRIBUTED BY HASH(order_id)
PROPERTIES (
    "replication_num" = "3", 
    "enable_persistent_index" = "true"
);
```

例1：パーティションされていないマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv1
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC
AS
select
    lo_orderkey, 
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_custkey 
order by lo_orderkey;
```

例2：パーティションされたマテリアライズドビューを作成します。

```SQL
CREATE MATERIALIZED VIEW lo_mv2
PARTITION BY `lo_orderdate`
DISTRIBUTED BY HASH(`lo_orderkey`)
REFRESH ASYNC START('2023-07-01 10:00:00') EVERY (interval 1 day)
AS
select
    lo_orderkey,
    lo_orderdate,
    lo_custkey, 
    sum(lo_quantity) as total_quantity, 
    sum(lo_revenue) as total_revenue, 
    count(lo_shipmode) as shipmode_count
from lineorder 
group by lo_orderkey, lo_orderdate, lo_custkey
order by lo_orderkey;

-- date_trunc()関数を使用してマテリアライズドビューを月ごとにパーティション分割します。
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

例3：非同期マテリアライズドビューを作成します。

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
    p.P_CONTAINER AS P_CONTAINER FROM lineorder AS l 
INNER JOIN customer AS c ON c.C_CUSTKEY = l.LO_CUSTKEY
INNER JOIN supplier AS s ON s.S_SUPPKEY = l.LO_SUPPKEY
INNER JOIN part AS p ON p.P_PARTKEY = l.LO_PARTKEY;
```

例4：パーティションされたマテリアライズドビューを作成し、ベーステーブルのSTRING型のパーティションキーをマテリアライズドビューの日付型に変換するために`str2date`を使用します。

``` SQL

-- 文字列のパーティション列を持つHiveテーブルを作成します。
CREATE TABLE `part_dates` (
  `d_date` varchar(20) DEFAULT NULL,
  `d_dayofweek` varchar(10) DEFAULT NULL,
  `d_month` varchar(11) DEFAULT NULL,
  `d_year` int(11) DEFAULT NULL,
  `d_yearmonthnum` int(11) DEFAULT NULL,
  `d_yearmonth` varchar(9) DEFAULT NULL,
  `d_daynuminweek` int(11) DEFAULT NULL,
  `d_daynuminmonth` int(11) DEFAULT NULL,
  `d_daynuminyear` int(11) DEFAULT NULL,
  `d_monthnuminyear` int(11) DEFAULT NULL,
  `d_weeknuminyear` int(11) DEFAULT NULL,
  `d_sellingseason` varchar(14) DEFAULT NULL,
  `d_lastdayinweekfl` int(11) DEFAULT NULL,
  `d_lastdayinmonthfl` int(11) DEFAULT NULL,
  `d_holidayfl` int(11) DEFAULT NULL,
  `d_weekdayfl` int(11) DEFAULT NULL,
  `d_datekey` varchar(11) DEFAULT NULL
) partition by (d_datekey);


-- `str2date`を使用してマテリアライズドビューの日付型にベーステーブルのSTRING型のパーティションキーを変換して作成します。
CREATE MATERIALIZED VIEW IF NOT EXISTS `test_mv` 
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
