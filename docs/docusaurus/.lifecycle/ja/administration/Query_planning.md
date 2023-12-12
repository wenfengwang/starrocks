---
displayed_sidebar: "Japanese"
---

# クエリ解析

クエリのパフォーマンスの最適化方法はよくある質問です。遅いクエリはユーザーエクスペリエンスだけでなく、クラスタのパフォーマンスも損ないます。クエリのパフォーマンスを分析して最適化することが重要です。

`fe/log/fe.audit.log` でクエリ情報を表示できます。それぞれのクエリには`QueryID`が対応しており、その`QueryID`を使用してクエリの`QueryPlan`と`Profile`を検索することができます。 `QueryPlan`はFEによってSQLステートメントを解析して生成された実行プランです。`Profile`はBEの実行結果であり、各ステップで消費された時間や各ステップで処理されるデータ量などの情報を含んでいます。

## プラン解析

StarRocksでは、SQLステートメントのライフサイクルをクエリ解析、クエリプランニング、クエリ実行の3つのフェーズに分けることができます。 クエリ解析は、通常、分析ワークロードの必要なQPSが高くないため、ボトルネックにはなりません。

StarRocksにおけるクエリのパフォーマンスは、クエリプランニングとクエリ実行によって決まります。クエリプランニングは演算子（Join / Order / Aggregate）を調整する責任があり、クエリ実行は特定の操作を実行する責任があります。

クエリプランはDBAにとってマクロな視点を提供し、クエリの情報へのアクセスの鍵であり、DBAが参照できる良いリソースです。次のコードスニペットは、`TPCDSクエリ96`を例にとって、クエリプランを表示する方法を示しています。

~~~SQL
-- query96.sql
select  count(*)
from store_sales
    ,household_demographics
    ,time_dim
    , store
where ss_sold_time_sk = time_dim.t_time_sk
    and ss_hdemo_sk = household_demographics.hd_demo_sk
    and ss_store_sk = s_store_sk
    and time_dim.t_hour = 8
    and time_dim.t_minute >= 30
    and household_demographics.hd_dep_count = 5
    and store.s_store_name = 'ese'
order by count(*) limit 100;
~~~

クエリプランには、論理クエリプランと物理クエリプランの2つのタイプがあります。ここで説明するクエリプランは、論理クエリプランを指します。 `TPCDS query96.sq`に対応するクエリプランは以下の通りです。

~~~sql
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
                                                                              |
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
                                                                              |
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
                                                                              |
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
~~~

クエリ96は、いくつかのStarRocksの概念を含むクエリプランを示しています。

|Name|Explanation|
|--|--|
|avgRowSize|スキャンされたデータ行の平均サイズ|
|cardinality|スキャンされたテーブルのデータ行の総数|
|colocate|テーブルがコロケートモードにあるかどうか|
|numNodes|スキャンされるノードの数|
|rollup|マテリアライズドビュー|
|preaggregation|事前集計|
|predicates|述語、クエリのフィルタ|

クエリ96のクエリプランは0から4までの5つのフラグメントに分かれており、クエリプランを下から上に1つずつ読むことができます。
フラグメント4は、`time_dim`テーブルをスキャンし、関連するクエリ条件（すなわち、`time_dim.t_hour = 8 and time_dim.t_minute >= 30`）を事前に実行する責務を担っています。このステップは述部の押し付けとしても知られています。StarRocksは、集約テーブルの`PREAGGREGATION`を有効にするかどうかを決定します。前の図では、`time_dim`の前集約は無効になっています。この場合、`time_dim`のすべての次元列が読み取られますが、テーブルに多くの次元列がある場合、パフォーマンスに悪影響を与える可能性があります。`time_dim`テーブルがデータの分割に`range partition`を選択すると、クエリプランでいくつかのパーティションがヒットし、関連のないパーティションが自動的にフィルタリングされます。マテリアライズド・ビューがある場合、StarRocksは自動的にクエリに基づいてマテリアライズド・ビューを選択します。マテリアライズド・ビューがない場合、クエリは自動的にベース・テーブルにヒットします（たとえば、前の図の`rollup: time_dim`）。

スキャンが完了したら、フラグメント4は終了します。データは他のフラグメントに渡され、前の図のEXCHANGE ID：09で示されているように、受信ノード9に渡されます。

Query 96のクエリプランでは、フラグメント2、3、および4は類似の機能を持っていますが、異なるテーブルをスキャンする責務があります。具体的には、クエリ内の`Order/Aggregation/Join`操作は、フラグメント1で実行されます。

フラグメント1では、`BROADCAST`メソッドを使用して`Order/Aggregation/Join`操作を行います。つまり、小さなテーブルを大きなテーブルにブロードキャストします。両方のテーブルが大きい場合は、`SHUFFLE`メソッドを使用することをお勧めします。現在、StarRocksは`HASH JOIN`のみをサポートしています。`colocate`フィールドは、2つの結合されたテーブルが同じ方法でパーティション分割およびバケット化されていることを示し、データを移行せずにローカルで結合操作を行うことができることを示しています。結合操作が完了すると、上位レベルでの`aggregation`、`order by`、および`top-n`操作が行われます。

具体的な表現を削除して（演算子のみ保持）、クエリプランをより大局的な視点で示すことができます。次の図に示すように。

![8-5](../assets/8-5.png)

## クエリヒント

クエリヒントは、クエリの実行方法についてクエリオプティマイザへ明示的に示唆する指令またはコメントです。現在、StarRocksは変数設定ヒントと結合ヒントの2種類のヒントをサポートしています。ヒントは単一のクエリ内でのみ効果があります。

### 変数設定ヒント

`SET_VAR`ヒントを使用して1つまたは複数の[システム変数](../reference/System_variable.md)を設定できます。このヒントの形式は、SELECTおよびSUBMIT TASKステートメント、あるいは他のステートメントに含まれるSELECT句での形式、例えばCREATE MATERIALIZED VIEW AS SELECTおよびCREATE VIEW AS SELECTなどで使用します。

#### 構文

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 例

集約クエリ用にシステム変数`streaming_preaggregation_mode`および`new_planner_agg_stage`の値を設定して集約方法をヒントする。

~~~SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

クエリのタスク実行タイムアウト期間をシステム変数`query_timeout`で設定する。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

マテリアライズド・ビューを作成する際に、SELECT句内でシステム変数`query_timeout`を設定して、クエリの実行タイムアウト期間をヒントする。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
~~~

### 結合ヒント

複数テーブルの結合クエリの場合、オプティマイザは通常、最適な結合実行方法を選択します。特定のケースでは、結合ヒントを使用してオプティマイザに結合実行方法を明示的に指示したり、Join Reorderを無効にしたりすることが可能です。現在、結合ヒントは、結合実行方法としてShuffle Join、Broadcast Join、Bucket Shuffle Join、またはColocate Joinを明示的に示唆することをサポートしています。結合ヒントを使用すると、オプティマイザはJoin Reorderを実行しません。そのため、小さいテーブルを右側のテーブルとして選択する必要があります。また、[Colocate Join](../using_starrocks/Colocate_join.md)またはBucket Shuffle Joinを結合実行方法として示唆する場合、結合されるテーブルのデータ分布がこれらの結合実行方法の要件を満たしていることを確認する必要があります。そうでない場合、示唆された結合実行方法は有効になりません。

#### 構文

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
~~~

> **注記**
>
> 結合ヒントは大文字と小文字を区別しません。

#### 例

- Shuffle Join

  テーブルAとテーブルBから同じバケットキーのデータ行をJoin操作前に、シャッフルを行う必要がある場合、結合実行方法としてShuffle Joinを示唆できます。

  ~~~SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Broadcast Join
  
  テーブルAが大きなテーブルであり、テーブルBが小さなテーブルである場合、結合実行方法をBroadcast Joinとして示唆できます。テーブルBのデータは、テーブルAのデータが存在するマシン全体に完全にブロードキャストされ、その後、Join操作が行われます。Shuffle Joinと比較して、Broadcast JoinはテーブルAのデータをシャッフルするコストを節約します。

  ~~~SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Bucket Shuffle Join
  
  結合クエリ内のJoin等結合式にテーブルAのバケットキーが含まれる場合、特にテーブルAとテーブルBの両方が大きなテーブルである場合、結合実行方法をBucket Shuffle Joinとして示唆できます。テーブルBのデータは、テーブルAのデータのデータ分布に従って、テーブルAのデータが存在するマシンにシャッフルされ、その後、Join操作が行われます。Broadcast Joinと比較して、Bucket Shuffle JoinはテーブルBのデータをグローバルで一度だけシャッフルするため、データ転送を大幅に削減します。

  ~~~SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Colocate Join
  
  テーブルAとテーブルBが同じ配置グループに属しており、結合クエリ内のJoin等結合式にテーブルAとBのバケットキーが含まれる場合、結合実行方法をColocate Joinとして示唆できます。同じキー値を持つデータは直接ローカルで結合されるため、ノード間のデータ転送に費やす時間が短縮され、クエリのパフォーマンスが向上します。

  ~~~SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

### ビューの結合実行方法

`EXPLAIN`コマンドを使用して実際の結合実行方法を表示します。返される結果が結合ヒントと一致している場合、結合ヒントが有効であることを示します。

~~~SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQLフィンガープリント

SQLフィンガープリントは、遅いクエリを最適化し、システムリソースの利用を向上させるために使用されます。StarRocksは、SQLフィンガープリント機能を使用して、遅いクエリログ（`fe.audit.log.slow_query`）のSQLステートメントを正規化し、SQLステートメントを異なるタイプに分類し、それぞれのSQLタイプのMD5ハッシュ値を計算して遅いクエリを識別します。MD5ハッシュ値は`Digest`フィールドで指定されます。

~~~SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
~~~

SQLステートメントの正規化は、ステートメントテキストをより正規化された形式に変換し、重要なステートメント構造だけを保持します。

- データベースおよびテーブル名などのオブジェクト識別子を保持します。

- 定数を疑問符（？）に変換します。

- コメントを削除し、スペースを整形します。

例えば、次の2つのSQLステートメントは正規化後、同じタイプに属します。

- 正規化前のSQLステートメント
```SQL
SELECT * FROM 注文 WHERE 顧客id=10 そして 数量>20



SELECT * FROM 注文 WHERE 顧客id = 20 そして 数量 > 100
```