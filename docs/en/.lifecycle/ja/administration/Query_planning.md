---
displayed_sidebar: "Japanese"
---

# クエリの解析

クエリのパフォーマンスを最適化する方法は、よくある質問です。遅いクエリはユーザーエクスペリエンスだけでなく、クラスタのパフォーマンスにも影響を与えます。クエリのパフォーマンスを分析して最適化することは重要です。

`fe/log/fe.audit.log`でクエリ情報を表示できます。各クエリには`QueryID`が対応しており、クエリの`QueryPlan`と`Profile`を検索するために使用できます。`QueryPlan`はFEによってSQLステートメントを解析して生成される実行計画です。`Profile`はBEの実行結果であり、各ステップの所要時間や各ステップで処理されるデータの量などの情報を含んでいます。

## プランの分析

StarRocksでは、SQLステートメントのライフサイクルをクエリの解析、クエリの計画、クエリの実行の3つのフェーズに分けることができます。クエリの解析は一般的にボトルネックにはなりません。なぜなら、分析ワークロードの必要なQPSは高くないからです。

StarRocksのクエリパフォーマンスは、クエリの計画とクエリの実行によって決まります。クエリの計画はオペレータ（Join/Order/Aggregate）の調整を担当し、クエリの実行は具体的な操作を実行します。

クエリプランは、DBAにクエリ情報へのマクロな視点を提供します。クエリプランはクエリのパフォーマンスの鍵であり、DBAが参照するための良いリソースです。次のコードスニペットは、`TPCDS query96`を例にとって、クエリプランを表示する方法を示しています。

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

クエリプランには、論理クエリプランと物理クエリプランの2つのタイプがあります。ここで説明するクエリプランは、論理クエリプランを指します。`TPCDS query96.sql`に対応するクエリプランは以下の通りです。

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
~~~

クエリ96は、いくつかのStarRocksのコンセプトを含むクエリプランを示しています。

|名前|説明|
|--|--|
|avgRowSize|スキャンされたデータ行の平均サイズ|
|cardinality|スキャンされたテーブルのデータ行の総数|
|colocate|テーブルがコロケートモードにあるかどうか|
|numNodes|スキャンされるノードの数|
|rollup|マテリアライズドビュー|
|preaggregation|プリアグリゲーション|
|predicates|述語、クエリのフィルター|

クエリ96のクエリプランは、0から4までの5つのフラグメントに分割されています。クエリプランは、下から上に順番に読むことができます。

フラグメント4は、`time_dim`テーブルをスキャンし、関連するクエリ条件（つまり、`time_dim.t_hour = 8`および`time_dim.t_minute >= 30`）を事前に実行します。このステップはプレディケートプッシュダウンとも呼ばれます。StarRocksは、集計テーブルに対して`PREAGGREGATION`を有効にするかどうかを決定します。前の図では、`time_dim`のプリアグリゲーションは無効になっています。この場合、`time_dim`のすべての次元列が読み取られますが、テーブルに多くの次元列がある場合はパフォーマンスに悪影響を与える可能性があります。`time_dim`テーブルがデータの分割に`range partition`を選択している場合、クエリプランでは複数のパーティションがヒットし、関係のないパーティションが自動的にフィルタリングされます。マテリアライズドビューがある場合、StarRocksはクエリに基づいて自動的にマテリアライズドビューを選択します。マテリアライズドビューがない場合、クエリは自動的にベーステーブルにヒットします（たとえば、前の図の`rollup: time_dim`）。

スキャンが完了すると、フラグメント4は終了します。データは他のフラグメントに渡され、前の図のEXCHANGE ID : 09で示されるように、受信ノード9に渡されます。

クエリ96のクエリプランには、フラグメント2、3、および4が似たような機能を持っていますが、異なるテーブルをスキャンする責任があります。具体的には、クエリの`Order/Aggregation/Join`操作はフラグメント1で実行されます。

フラグメント1では、`BROADCAST`メソッドを使用して`Order/Aggregation/Join`操作を実行します。つまり、小さなテーブルを大きなテーブルにブロードキャストします。両方のテーブルが大きい場合は、`SHUFFLE`メソッドを使用することをお勧めします。現在、StarRocksは`HASH JOIN`のみをサポートしています。`colocate`フィールドは、2つの結合されたテーブルが同じようにパーティション分割およびバケット分割されていることを示すために使用されます。これにより、データを移行せずにローカルで結合操作を実行できます。Join操作が完了すると、上位の`aggregation`、`order by`、`top-n`操作が実行されます。

具体的な式を削除して（オペレータのみを保持して）、クエリプランをよりマクロな視点で表示することができます。以下の図に示すように。

![8-5](../assets/8-5.png)

## クエリヒント

クエリヒントは、クエリオプティマイザにクエリの実行方法を明示的に指示するための指令またはコメントです。現在、StarRocksは2種類のヒントをサポートしています：変数設定ヒントと結合ヒント。ヒントは単一のクエリ内でのみ有効です。

### 変数設定ヒント

`SET_VAR`ヒントを使用して、SELECTおよびSUBMIT TASKステートメント、または他のステートメントに含まれるSELECT句（CREATE MATERIALIZED VIEW AS SELECTやCREATE VIEW AS SELECTなど）の形式で、1つ以上の[システム変数](../reference/System_variable.md)を設定できます。

#### 構文

~~~SQL
[...] SELECT [/*+ SET_VAR(key=value [, key = value]*) */] ...
SUBMIT [/*+ SET_VAR(key=value [, key = value]*) */] TASK ...
~~~

#### 例

集計クエリに対して集計方法をヒントするために、システム変数`streaming_preaggregation_mode`と`new_planner_agg_stage`を設定します。

~~~SQL
SELECT /*+ SET_VAR (streaming_preaggregation_mode = 'force_streaming',new_planner_agg_stage = '2') */ SUM(sales_amount) AS total_sales_amount FROM sales_orders;
~~~

SUBMIT TASKステートメントでシステム変数`query_timeout`を設定して、クエリのタスク実行のタイムアウト期間をヒントします。

~~~SQL
SUBMIT /*+ SET_VAR(query_timeout=3) */ TASK AS CREATE TABLE temp AS SELECT count(*) AS cnt FROM tbl1;
~~~

マテリアライズドビューを作成する際に、SELECT句でシステム変数`query_timeout`を設定して、クエリの実行タイムアウト期間をヒントします。

~~~SQL
CREATE MATERIALIZED VIEW mv 
PARTITION BY dt 
DISTRIBUTED BY HASH(`key`) 
BUCKETS 10 
REFRESH ASYNC 
AS SELECT /*+ SET_VAR(query_timeout=500) */ * from dual;
~~~

### 結合ヒント

複数のテーブルを結合するクエリの場合、オプティマイザは通常、最適な結合実行方法を選択します。特殊な場合では、結合ヒントを使用してオプティマイザに結合実行方法を明示的に指示するか、Join Reorderを無効にすることができます。現在、結合ヒントはShuffle Join、Broadcast Join、Bucket Shuffle Join、またはColocate Joinを結合実行方法としてサポートしています。結合ヒントを使用する場合、オプティマイザはJoin Reorderを実行しません。そのため、右側のテーブルを小さいテーブルとして選択する必要があります。また、[Colocate Join](../using_starrocks/Colocate_join.md)またはBucket Shuffle Joinを結合実行方法として指定する場合は、結合されるテーブルのデータ分布がこれらの結合実行方法の要件を満たしていることを確認してください。そうでない場合、指定された結合実行方法は有効になりません。

#### 構文

~~~SQL
... JOIN { [BROADCAST] | [SHUFFLE] | [BUCKET] | [COLOCATE] | [UNREORDER]} ...
~~~

> **注意**
>
> 結合ヒントは大文字と小文字を区別しません。

#### 例

- Shuffle Join

  テーブルAとテーブルBから、結合操作を実行する前に同じバケットキー値を持つデータ行をシャッフルする必要がある場合、結合実行方法をShuffle Joinとしてヒントできます。

  ~~~SQL
  select k1 from t1 join [SHUFFLE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Broadcast Join
  
  テーブルAが大きなテーブルであり、テーブルBが小さなテーブルである場合、結合実行方法をBroadcast Joinとしてヒントすることができます。テーブルBのデータは、テーブルAのデータが存在するマシンに完全にブロードキャストされ、その後結合操作が実行されます。Shuffle Joinと比較して、Broadcast JoinはテーブルAのデータをシャッフルするコストを節約します。

  ~~~SQL
  select k1 from t1 join [BROADCAST] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Bucket Shuffle Join
  
  結合クエリの結合等価条件にテーブルAのバケットキーが含まれる場合、特にテーブルAとテーブルBの両方が大きなテーブルの場合、結合実行方法をBucket Shuffle Joinとしてヒントすることができます。テーブルBのデータは、テーブルAのデータのデータ分布に従って、テーブルAのデータが存在するマシンにシャッフルされ、その後結合操作が実行されます。Broadcast Joinと比較して、Bucket Shuffle Joinはデータの転送コストを大幅に削減します。なぜなら、テーブルBのデータはグローバルに一度だけシャッフルされるからです。

  ~~~SQL
  select k1 from t1 join [BUCKET] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

- Colocate Join
  
  テーブルAとテーブルBが同じ配置グループに属している場合（テーブル作成時に指定される）、テーブルAとテーブルBの同じキー値を持つデータ行は同じBEノードに分散されます。結合クエリの結合等価条件にテーブルAとテーブルBのバケットキーが含まれる場合、結合実行方法をColocate Joinとしてヒントすることができます。同じキー値を持つデータは直接ローカルで結合され、ノード間のデータ転送にかかる時間が短縮され、クエリのパフォーマンスが向上します。

  ~~~SQL
  select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
  ~~~

### ビューの結合実行方法

`EXPLAIN`コマンドを使用して実際の結合実行方法を表示できます。返された結果が結合ヒントと一致する場合、結合ヒントが有効であることを意味します。

~~~SQL
EXPLAIN select k1 from t1 join [COLOCATE] t2 on t1.k1 = t2.k2 group by t2.k2;
~~~

![8-9](../assets/8-9.png)

## SQLフィンガープリント

SQLフィンガープリントは、遅いクエリを最適化し、システムリソースの利用効率を向上させるために使用されます。StarRocksは、SQLステートメントをスローリクエストログ（`fe.audit.log.slow_query`）で正規化し、異なるタイプのSQLステートメントをカテゴリ別に分類し、各SQLタイプのMD5ハッシュ値を計算して遅いクエリを識別するためにSQLフィンガープリント機能を使用します。MD5ハッシュ値は`Digest`フィールドで指定されます。

~~~SQL
2021-12-27 15:13:39,108 [slow_query] |Client=172.26.xx.xxx:54956|User=root|Db=default_cluster:test|State=EOF|Time=2469|ScanBytes=0|ScanRows=0|ReturnRows=6|StmtId=3|QueryId=824d8dc0-66e4-11ec-9fdc-00163e04d4c2|IsQuery=true|feIp=172.26.92.195|Stmt=select count(*) from test_basic group by id_bigint|Digest=51390da6b57461f571f0712d527320f4
~~~

SQLステートメントの正規化は、ステートメントテキストをより正規化された形式に変換し、重要なステートメント構造のみを保持します。

- オブジェクト識別子（データベース名やテーブル名など）を保持します。

- 定数を疑問符（?）に変換します。

- コメントを削除し、スペースを整形します。

たとえば、次の2つのSQLステートメントは、正規化後に同じタイプに属します。

- 正規化前のSQLステートメント

~~~SQL
SELECT * FROM orders WHERE customer_id=10 AND quantity>20



SELECT * FROM orders WHERE customer_id = 20 AND quantity > 100
~~~

- 正規化後のSQLステートメント

~~~SQL
SELECT * FROM orders WHERE customer_id=? AND quantity>?
~~~
