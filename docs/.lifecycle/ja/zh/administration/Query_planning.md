---
displayed_sidebar: Chinese
---

# クエリ分析

この文書では、StarRocks内のクエリを分析する方法について説明します。

StarRocksクラスターのパフォーマンスを最適化するために、管理者は定期的に遅いクエリを分析し、最適化する必要があります。そうしないと、遅いクエリがクラスター全体のサービス能力に影響を与え、結果としてユーザーエクスペリエンスに影響を及ぼす可能性があります。

**fe/log/fe.audit.log** で、すべてのクエリと遅いクエリの情報を見ることができます。各クエリにはQueryIDが対応しています。ログまたはページで、クエリに対応するQuery PlanとProfileを見つけることができます。Query PlanはFEがSQLを解析して生成した実行計画であり、ProfileはBEがクエリを実行した後の結果で、各ステップの時間とデータ処理量などのデータが含まれています。エンタープライズ版ユーザーの場合、StarRocksManagerのグラフィカルインターフェースを通じて、視覚化されたProfile実行ツリーを見ることができます。

また、StarRocksは遅いクエリ内のSQL文を分類し、各種SQL文のSQLフィンガープリントを計算することもサポートしています。

## Query Planの確認と分析

StarRocks内のSQL文のライフサイクルは、クエリ解析（Query Parsing）、計画（Query Plan）、実行（Query Execution）の3つの段階に分けることができます。通常、StarRocksにおいては、クエリ解析がクエリパフォーマンスのボトルネックになることはありません。なぜなら、分析型の要求のQPSは一般的に高くないからです。

StarRocks内のクエリパフォーマンスを決定する鍵は、クエリ計画（Query Plan）とクエリ実行（Query Execution）にあります。これらの関係は、Query Planがオペレーター（Join/Order/Aggregation）間の関係を組織することを担当し、Query Executionが具体的なオペレーターを実行すると説明できます。

Query Planは、データベース管理者にマクロな視点を提供し、クエリ実行に関連する情報を取得することができます。優れたQuery Planは、クエリのパフォーマンスを大きく決定するため、データベース管理者はQuery Planを頻繁に確認し、適切に生成されているかを確認する必要があります。この章では、TPC-DSのquery96.sqlを例に、StarRocksのQuery Planを確認する方法を示します。

以下の例は、TPC-DSのquery96.sqlを例に、StarRocksのQuery Planを確認し分析する方法を示しています。

```SQL
-- query96.sql
select count(*)
from store_sales, 
    household_demographics, 
    time_dim, 
    store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
```

### Query Planの確認

Query Planは、論理実行計画（Logical Query Plan）と物理実行計画（Physical Query Plan）に分けることができます。このセクションで説明するQuery Planは、通常、論理実行計画を指します。

以下のコマンドでQuery Planを確認します。

```sql
EXPLAIN sql_statement;
```

TPC-DSのquery96.sqlに対応するQuery Planは以下のように表示されます：

```plain text
mysql> EXPLAIN select count(*)
from store_sales, 
    household_demographics, 
    time_dim, 
    store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;

+------------------------------------------------------------------------------+
| Explain String                                                               |
+------------------------------------------------------------------------------+
| PLAN FRAGMENT 0                                                              |
|  OUTPUT EXPRS:<slot 11>                                                      |
|   PARTITION: UNPARTITIONED                                                   |
|   RESULT SINK                                                                |
|   12:MERGING-EXCHANGE                                                        |
|      limit: 100                                                              |
|      tuple ids: 5                                                            |
|                                                                              |
| PLAN FRAGMENT 1                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 12                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   8:TOP-N                                                                    |
|   |  order by: <slot 11> ASC                                                 |
|   |  offset: 0                                                               |
|   |  limit: 100                                                              |
|   |  tuple ids: 5                                                            |
|   |                                                                          |
|   7:AGGREGATE (update finalize)                                              |
|   |  output: count(*)                                                        |
|   |  group by:                                                               |
|   |  tuple ids: 4                                                            |
|   |                                                                          |
|   6:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_store_sk` = `s_store_sk`                       |
|   |  tuple ids: 0 2 1 3                                                      |
|   |                                                                          |
|   |----11:EXCHANGE                                                           |
|   |       tuple ids: 3                                                       |
|   |                                                                          |
|   4:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: left hash join node can not do colocate        |
|   |  equal join conjunct: `ss_hdemo_sk`=`household_demographics`.`hd_demo_sk`|
|   |  tuple ids: 0 2 1                                                        |
|   |                                                                          |
|   |----10:EXCHANGE                                                           |
|   |       tuple ids: 1                                                       |
|   |                                                                          |
|   2:HASH JOIN                                                                |
|   |  join op: INNER JOIN (BROADCAST)                                         |
|   |  hash predicates:                                                        |
|   |  colocate: false, reason: table not in same group                        |
|   |  equal join conjunct: `ss_sold_time_sk` = `time_dim`.`t_time_sk`         |
|   |  tuple ids: 0 2                                                          |
|   |                                                                          |
|   |----9:EXCHANGE                                                            |
|   |       tuple ids: 2                                                       |
|   |                                                                          |
|   0:OlapScanNode                                                             |
|      TABLE: store_sales                                                      |
|      PREAGGREGATION: OFF. Reason: `ss_sold_time_sk` is value column          |
|      partitions=1/1                                                          |
|      rollup: store_sales                                                     |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 0                                                            |
|                                                                              |
| PLAN FRAGMENT 2                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|                                                                              |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 11                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   5:OlapScanNode                                                             |
|      TABLE: store                                                            |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `store`.`s_store_name` = 'ese'                              |
|      partitions=1/1                                                          |
|      rollup: store                                                           |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 3                                                            |
|                                                                              |
| PLAN FRAGMENT 3                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 10                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   3:OlapScanNode                                                             |
|      TABLE: household_demographics                                           |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `household_demographics`.`hd_dep_count` = 5                 |
|      partitions=1/1                                                          |
|      rollup: household_demographics                                          |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 1                                                            |
|                                                                              |
| PLAN FRAGMENT 4                                                              |
|  OUTPUT EXPRS:                                                               |
|   PARTITION: RANDOM                                                          |
|   STREAM DATA SINK                                                           |
|     EXCHANGE ID: 09                                                          |
|     UNPARTITIONED                                                            |
|                                                                              |
|   1:OlapScanNode                                                             |
|      TABLE: time_dim                                                         |
|      PREAGGREGATION: OFF. Reason: null                                       |
|      PREDICATES: `time_dim`.`t_hour` = 8, `time_dim`.`t_minute` >= 30        |
|      partitions=1/1                                                          |
|      rollup: time_dim                                                        |
|      tabletRatio=0/0                                                         |
|      tabletList=                                                             |
|      cardinality=-1                                                          |
|      avgRowSize=0.0                                                          |
|      numNodes=0                                                              |
|      tuple ids: 2                                                            |
+------------------------------------------------------------------------------+
128 rows in set (0.02 sec)
```

### Query Planの分析


分析 Query Plan には以下の概念が関連します。

|名称|解説|
|---|---|
|avgRowSize|スキャンされるデータ行の平均サイズ。|
|cardinality|スキャンされるテーブルの総行数。|
|colocate|テーブルがColocate形式を採用しているかどうか。|
|numNodes|スキャンに関わるノード数。|
|rollup|マテリアライズドビュー。|
|preaggregation|プリアグリゲーション。|
|predicates|述語、つまりクエリのフィルタ条件。|
|partitions|パーティション。|
|table|テーブル。|

query96.sql の Query Plan は 5 つの Plan Fragment に分かれており、番号は 0 から 4 までです。下から上に向かって Query Plan を確認することができます。

上記の例では、最も下部の Plan Fragment は Fragment 4 で、`time_dim` テーブルのスキャンを担当し、関連するクエリ条件 `time_dim.t_hour = 8 and time_dim.t_minute >= 30` を事前に実行します。これは、述語のプッシュダウンです。集約キーに対して、StarRocks は異なるクエリに基づいてプリアグリゲーション PREAGGREGATION を有効にするかどうかを選択します。上記の例での `time_dim` テーブルのプリアグリゲーションはオフになっており、この状態では StarRocks は `time_dim` のすべての次元列を読み取ります。もし現在のテーブルに多くの次元列が含まれている場合、これはパフォーマンスに影響を与える重要な要因になり得ます。もし `time_dim` テーブルが Range Partition に基づいてデータが分割されている場合、Query Plan の `partitions` はクエリがヒットしたパーティションを示し、関係のないパーティションは自動的にフィルタリングされ、スキャンするデータ量を効果的に減少させます。現在のテーブルにマテリアライズドビューがある場合、StarRocks はクエリに基づいて自動的にマテリアライズドビューを選択します。マテリアライズドビューがない場合、クエリは自動的にベーステーブルにヒットします。これは上記の例で示されている `rollup: time_dim` です。他のフィールドについては現時点では気にする必要はありません。

`time_dim` テーブルのデータスキャンが完了すると、Fragment 4 の実行プロセスも終了し、スキャンしたデータを他の Fragment に渡します。上記の例での `EXCHANGE ID : 09` は、データが番号 `9` の受信ノードに渡されたことを示しています。

query96.sql の Query Plan において、Fragment 2、3、4 は機能が似ており、スキャンするテーブルが異なるだけです。Order/Aggregation/Join のオペレータはすべて Fragment 1 で実行されます。

Fragment 1 は 3 つの Join オペレータの実行を統合し、デフォルトの BROADCAST 方式で実行されます。これは、小さなテーブルから大きなテーブルへのブロードキャスト方式です。Join する両方のテーブルが大きい場合は、SHUFFLE 方式を使用することをお勧めします。現在 StarRocks は HASH JOIN のみをサポートしており、これはハッシュアルゴリズムを使用した Join です。上記の例での `colocate` フィールドは、2 つの Join テーブルが同じパーティション/バケット方式を採用していることを説明しています。パーティション/バケット方式が同じであれば、Join プロセスはローカルで直接実行でき、データの移動は不要です。Join の実行が完了すると、Fragment 1 は上位の Aggregation、Order by、TOP-N オペレータを実行します。

具体的な表現を抜きにして、下の図は query96.sql の Query Plan をマクロな視点から示しています。

![8-5](../assets/8-5.png)

## Profile の分析

Query Plan の分析後、BE の実行結果 Profile を分析してクラスタのパフォーマンスを理解できます。

エンタープライズ版ユーザーの場合、StarRocksManager でクエリを実行し、**クエリ履歴**をクリックすると、**実行詳細**タブで Profile の詳細なテキスト情報を、**実行時間**タブでグラフィカルな表示を見ることができます。以下は TPC-H の Q4 クエリを例に使用します。

以下の例は TPC-H の Q4 クエリを使用しています。

```sql
-- TPC-H Q4
select  o_orderpriority,  count(*) as order_count
from  orders
where
  o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
  and exists (
    select  * from  lineitem
    where  l_orderkey = o_orderkey and l_commitdate < l_receiptdate
  )
group by o_orderpriority
order by o_orderpriority;
```

上記のクエリには、group by、order by、count のサブクエリが含まれています。`order` は注文テーブル、`lineitem` は商品明細テーブルで、これらはどちらも大量のデータを持つファクトテーブルです。このクエリの意味は、注文の優先度によってグループ化し、各グループの注文数を集計することです。また、2 つのフィルタ条件があります：

* 注文作成日が `1993年7月` から `1993年10月` の間にある。
* 注文に対応する製品のコミット日 `l_commitdate` が受領日 `l_receiptdate` より前である。

クエリを実行した後、**実行時間**タブで現在のクエリを確認できます。

![8-6](../assets/8-6.png)
  
ページの左上隅で、クエリが 3.106 秒で実行されたことがわかります。各ノードをクリックすると、各部分の実行情報を確認できます。`Active` フィールドは、現在のノード（およびそのすべての子ノード）の実行時間を示します。現在のノードには 2 つの子ノード、つまり 2 つの Scan Node があり、それぞれ 5,730,776 行と 379,364,474 行のデータをスキャンし、1 回の Shuffle Join を行い、その後 5,257,429 行のデータを出力し、2 層の集約を経て、最後に Sort Node を通じて結果を出力します。Exchange Node はデータ交換ノードで、現在の例では 2 回の Shuffle が行われています。

通常、Profile を分析する際の核心は、実行時間が最も長いパフォーマンスのボトルネックがあるノードを見つけることです。上から下へと順に確認できます。

現在の例では、Hash Join Node が主な時間を占めています。

```plain text
HASH_JOIN_NODE (id=2):(Active: 3s85ms, % non-child: 34.59%)
- AvgInputProbeChunkSize: 3.653K (3653)
- AvgOutputChunkSize: 3.57K (3570)
- BuildBuckets: 67.108864M (67108864)
- BuildRows: 31.611425M (31611425)
- BuildTime: 3s56ms
    - 1-FetchRightTableTime: 2s14ms
    - 2-CopyRightTableChunkTime: 0ns
    - 3-BuildHashTableTime: 1s19ms
    - 4-BuildPushDownExprTime: 0ns
- PeakMemoryUsage: 376.59 MB
- ProbeRows: 478.599K (478599)
- ProbeTime: 28.820ms
    - 1-FetchLeftTableTimer: 369.315us
    - 2-SearchHashTableTimer: 19.252ms
    - 3-OutputBuildColumnTimer: 893.29us
    - 4-OutputProbeColumnTimer: 7.612ms
    - 5-OutputTupleColumnTimer: 34.593us
- PushDownExprNum: 0
- RowsReturned: 439.16K (439160)
- RowsReturnedRate: 142.323K /sec
```

現在の例の情報から、Hash Join の実行時間は主に `BuildTime` と `ProbeTime` の 2 部分に分かれていることがわかります。`BuildTime` は右表をスキャンしてハッシュテーブルを構築するプロセスで、`ProbeTime` は左表を取得してハッシュテーブルを検索し、マッチングして出力するプロセスです。現在のノードの `BuildTime` で、`FetchRightTableTime` と `BuildHashTableTime` が大部分を占めていることがわかります。以前のスキャン行数のデータと比較すると、現在のクエリで左表と右表の順序が理想的ではないことがわかります。左表を大表に設定することで、右表のハッシュテーブル構築の効果が向上します。さらに、現在の両方のテーブルはファクトテーブルでデータ量が多いため、データの Shuffle を避けるために Colocate Join を検討し、Join のデータ量を減らすこともできます。[Colocate Join](../using_starrocks/Colocate_join.md) を参照して Colocation 関係を確立し、SQL を以下のように改写できます：

```sql
with t1 as (
    select l_orderkey from  lineitem
    where  l_commitdate < l_receiptdate
) select o_orderpriority, count(*) as order_count from t1 right semi join orders_co on l_orderkey = o_orderkey 
    where o_orderdate >= date '1993-07-01'
  and o_orderdate < date '1993-07-01' + interval '3' month
group by o_orderpriority
order by o_orderpriority;
```

実行結果は以下の通りです：

![8-7](../assets/8-7.png)

新しいSQLの実行時間は3.106秒から1.042秒に短縮されました。二つの大きなテーブルからExchangeノードがなくなり、Colocate Joinを直接使用して結合されました。さらに、左右のテーブルの順序を変更することで、全体のパフォーマンスが大幅に向上し、新しいJoin Nodeの情報は以下の通りです：

```sql
HASH_JOIN_NODE (id=2):(Active: 996.337ms, % non-child: 52.05%)
- AvgInputProbeChunkSize: 2.588K (2588)
- AvgOutputChunkSize: 35
- BuildBuckets: 1.048576M (1048576)
- BuildRows: 478.171K (478171)
- BuildTime: 187.794ms
    - 1-FetchRightTableTime: 175.810ms
    - 2-CopyRightTableChunkTime: 5.942ms
    - 3-BuildHashTableTime: 5.811ms
    - 4-BuildPushDownExprTime: 0ns
- PeakMemoryUsage: 22.38 MB
- ProbeRows: 31.609786M (31609786)
- ProbeTime: 807.406ms
    - 1-FetchLeftTableTimer: 282.257ms
    - 2-SearchHashTableTimer: 457.235ms
    - 3-OutputBuildColumnTimer: 26.135ms
    - 4-OutputProbeColumnTimer: 16.138ms
    - 5-OutputTupleColumnTimer: 1.127ms
- PushDownExprNum: 0
- RowsReturned: 438.502K (438502)
- RowsReturnedRate: 440.114K /sec
```

## クエリヒント

StarRocksはヒント（Hint）機能をサポートしています。Hintは、クエリオプティマイザーに対して、クエリをどのように実行するかを明示的に提案する指示またはコメントです。現在、システム変数HintとJoin Hintの2種類のHintがサポートされています。Hintは単一のクエリ範囲内でのみ有効です。

### システム変数Hint

SELECT、SUBMIT TASK文で `/*+ ... */` コメント形式を使用して、一つまたは複数の[システム変数](../reference/System_variable.md)Hintを設定できます。他の文にSELECT節が含まれている場合（例：CREATE MATERIALIZED VIEW AS SELECT、CREATE VIEW AS SELECT）、そのSELECT節でシステム変数Hintを使用することもできます。

#### 構文

```SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
```

#### 例

集約クエリ文でシステム変数 `streaming_preaggregation_mode` と `new_planner_agg_stage` を使用して集約方法を設定します。

```SQL
SELECT /*+ SET_VAR(streaming_preaggregation_mode = 'force_streaming', new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
```

SUBMIT TASK文でシステム変数 `query_timeout` を使用してクエリ実行のタイムアウト時間を設定します。

```SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK 
    AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
```

物化ビューを作成する際に、SELECT節でシステム変数 `query_timeout` を使用してクエリ実行のタイムアウト時間を設定します。

```SQL
CREATE MATERIALIZED VIEW mv 
    PARTITION BY dt 
    DISTRIBUTED BY HASH(`key`) 
    BUCKETS 10 
    REFRESH ASYNC 
    AS SELECT /*+ SET_VAR(query_timeout=500) */ * FROM dual;
```

### Join Hint

複数のテーブルを結合するクエリに対して、オプティマイザーは通常、最適なJoin実行方法を自動的に選択します。特殊な状況では、Join Hintを使用してオプティマイザーにJoinの実行方法を明示的に提案したり、Join Reorderを無効にしたりすることができます。現在、Join HintでサポートされているJoinの実行方法には、Shuffle Join、Broadcast Join、Bucket Shuffle Join、Colocate Joinがあります。

Join Hintを使用してJoinの実行方法を提案した後、オプティマイザーはJoin Reorderを行わないため、右側のテーブルが小さいことを確認する必要があります。また、提案されたJoinの実行方法が[Colocate Join](../using_starrocks/Colocate_join.md)またはBucket Shuffle Joinの場合、これらのJoinの実行方法の要件を満たすようにテーブルのデータ分布を確認する必要があります。そうでない場合、提案されたJoinの実行方法は有効になりません。

#### 構文

```SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER] } ...
```

> **注記**
>
> Join Hintを使用する際は、大文字と小文字を区別しません。

#### 例

* Shuffle Join

  テーブルAとBのバケットキーが同じ値のデータ行を同じマシンにShuffleしてからJoin操作を行う場合、Shuffle JoinとしてJoin Hintを設定できます。

  ```SQL
  SELECT k1 FROM t1 JOIN [SHUFFLE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

* Broadcast Join
  
  テーブルAが大きなテーブルで、テーブルBが小さなテーブルの場合、Broadcast JoinとしてJoin Hintを設定できます。テーブルBのデータをテーブルAのデータがあるマシンに全量ブロードキャストしてからJoin操作を行います。Broadcast JoinはShuffle Joinに比べて、テーブルAのデータをShuffleするコストを節約できます。

  ```SQL
  SELECT k1 FROM t1 JOIN [BROADCAST] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

* Bucket Shuffle Join
  
  結合クエリでJoinの等価式がテーブルAのバケットキーにヒットする場合、特にテーブルAとBが大きなテーブルである場合、Bucket Shuffle JoinとしてJoin Hintを設定できます。テーブルBのデータはテーブルAのデータ分布に従ってShuffleされ、その後Join操作を行います。Bucket Shuffle JoinはBroadcast Joinをさらに最適化したもので、テーブルBのデータ量はグローバルに一つだけで、Broadcast Joinよりもはるかに少ないデータ量を転送します。

  ```SQL
  SELECT k1 FROM t1 JOIN [BUCKET] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

* Colocate Join
  
  テーブルAとBが作成時に同じColocation Groupに属している場合、テーブルAとBのバケットキーが同じ値のデータ行は必ず同じBEノードに分布しています。結合クエリでJoinの等価式がテーブルAとBのバケットキーにヒットする場合、Colocate JoinとしてJoin Hintを設定できます。同じキー値を持つデータはローカルで直接Joinされ、ノード間のデータ転送時間を削減し、クエリのパフォーマンスを向上させます。

  ```SQL
  SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
  ```

### 実際のJoin実行方法を確認する

`EXPLAIN` コマンドを使用して、Join Hintが有効かどうかを確認できます。返された結果に表示されるJoinの実行方法がJoin Hintと一致している場合、Join Hintは有効です。

```SQL
EXPLAIN SELECT k1 FROM t1 JOIN [COLOCATE] t2 ON t1.k1 = t2.k2 GROUP BY t2.k2;
```

![8-9](../assets/8-9.png)

## SQLフィンガープリントを確認する

StarRocksは、遅いクエリのSQL文を正規化し、各種類のSQL文のMD5ハッシュ値を分類して計算することをサポートしています。

**fe.audit.log.slow_query** で遅いクエリに関する情報を確認できます。MD5ハッシュ値は `Digest` フィールドに対応しています。

```plain text
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
```

SQL文の正規化は重要な構文構造のみを保持し、SQL文に対して以下の操作を行います：

* オブジェクト識別子は保持します。例えば、データベースやテーブルの名前です。
* 定数を `?` に変換します。
* コメントを削除し、スペースを調整します。

以下の2つの SQL ステートメントは、正規化後に同一のSQLタイプに属します。

```sql
SELECT * FROM orders WHERE customer_id=10 AND quantity>20

SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
```

以下は正規化後の SQL タイプです。

```sql
SELECT * FROM orders WHERE customer_id=? AND quantity>?
```

