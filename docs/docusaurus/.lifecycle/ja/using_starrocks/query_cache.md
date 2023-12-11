```yaml
---
displayed_sidebar: "Japanese"
---

# クエリキャッシュ

クエリキャッシュは、StarRocksの強力な機能であり、集計クエリのパフォーマンスを大幅に向上させることができます。ローカル集計の中間結果をメモリに格納することで、クエリキャッシュは以前のクエリと同一または類似の新しいクエリに対して不要なディスクアクセスと計算を避けることができます。クエリキャッシュにより、StarRocksは高並行のシナリオで、多くのユーザーが大規模で複雑なデータセットで類似のクエリを実行する場合に特に有用です。

この機能はv2.5以降でサポートされています。

v2.5では、クエリキャッシュは単一のフラットテーブルでの集計クエリのみをサポートしています。v3.0以降、クエリキャッシュはスタースキーマで結合された複数のテーブルに対する集計クエリもサポートしています。

## アプリケーションシナリオ

次のシナリオでクエリキャッシュを使用することを推奨します。

- 個々のフラットテーブルまたはスタースキーマで接続された複数の結合テーブルで集計クエリを頻繁に実行します。
- ほとんどの集計クエリが非GROUP BY集計クエリおよび低カーディナリティGROUP BY集計クエリです。
- データは時間パーティションで追加モードでロードされ、アクセス頻度に基づいてホットデータとコールドデータに分類できます。

クエリキャッシュは、次の条件を満たすクエリをサポートしています。

- クエリエンジンがPipelineであること。Pipelineエンジンを有効にするには、セッション変数`enable_pipeline_engine`を`true`に設定します。

  > **注意**
  >
  > 他のクエリエンジンはクエリキャッシュをサポートしていません。

- クエリがネイティブOLAPテーブル（v2.5以降）またはクラウドネイティブテーブル（v3.0以降）であること。クエリキャッシュは外部テーブルのクエリをサポートしていません。また、同期マテリアライズドビューへのアクセスを必要とする計画のクエリをサポートしていますが、非同期マテリアライズドビューへのアクセスを必要とする計画のクエリをサポートしていません。

- クエリが個々のテーブルまたは複数の結合テーブルの集計クエリであること。

  **注意**
  >
  > - クエリキャッシュはBroadcast JoinおよびBucket Shuffle Joinをサポートしています。
  > - クエリキャッシュはJoin演算子を含む2つのツリー構造をサポートしています：Aggregation-JoinおよびJoin-Aggregation。Aggregation-Joinツリー構造ではShuffle Joinはサポートされておらず、Join-Aggregationツリー構造ではHash Joinはサポートされていません。

- `rand`、 `random`、 `uuid`、 `sleep`などの不確定な関数を含まないクエリであること。

クエリキャッシュは、以下のパーティションポリシーを使用するテーブルのクエリをサポートしています：Unpartitioned、Multi-Column Partitioned、Single-Column Partitioned。

## 機能の境界

- クエリキャッシュは、Pipelineエンジンのテーブレット計算に基づいています。テーブレット計算は、各個々のBEがクエリのために処理する必要があるテーブレットの数が、このクエリを実行するために呼び出されるパイプラインドライバの数と同じかそれ以上の場合に機能します。呼び出されるパイプラインドライバの数は実際の並列度（DOP）を表します。テーブレットの数がパイプラインドライバの数よりも少ない場合、各パイプラインドライバは特定のテーブレットの一部のみを処理します。この状況では、テーブレット計算の結果は生成されず、そのためクエリキャッシュは機能しません。
- StarRocksでは、集計クエリは少なくとも4つのステージで構成されます。第1ステージのAggregateNodeによって生成されたテーブレット単位の計算結果は、OlapScanNodeおよびAggregateNodeが同じフラグメントからデータを計算する場合にのみキャッシュできます。他のステージでAggregateNodeによって生成されたテーブレット単位の計算結果はキャッシュできません。一部のDISTINCT集計クエリでは、セッション変数`cbo_cte_reuse`が`true`に設定されている場合、OlapScanNodeがデータを生成し、stage-1 AggregateNodeが生成されたデータを処理し、その間にExchangeNodeが介在する場合、クエリキャッシュは機能しません。次の2つの例は、CTEの最適化が行われる場合であり、そのためクエリキャッシュは機能しません：
  - 出力列が集計関数`avg(distinct)`を使用して計算されます。
  - 出力列が複数のDISTINCT集計関数で計算されます。
- データが集計前にシャッフルされている場合、クエリキャッシュはそのデータのクエリを加速することはできません。
- テーブルのグループ化に使用される列または重複排除列が高カーディナリティ列である場合、そのテーブルの集計クエリに対して大量の結果が生成されます。これらの場合、クエリは実行時にクエリキャッシュをバイパスします。
- クエリキャッシュは、BEによって提供されるわずかなメモリ量を占有して計算結果を保存します。クエリキャッシュのサイズはデフォルトで512 MBに設定されています。そのため、大きなサイズのデータアイテムを保存するにはクエリキャッシュは適していません。また、クエリキャッシュを有効にした後、キャッシュヒット率が低い場合、クエリのパフォーマンスが低下します。したがって、特定のテーブレットで生成される計算結果のサイズが`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定された閾値を超える場合、クエリキャッシュはクエリに対して機能せず、クエリはPassthroughモードに切り替わります。

## 仕組み

クエリキャッシュが有効になると、各BEはクエリのローカル集計を次の2つのステージに分割します。

1. テーブレット単位の集計

   BEは各テーブレットを個別に処理します。BEがテーブレットの処理を開始すると、まずクエリキャッシュをプローブしてそのテーブレットの集計の中間結果がクエリキャッシュにあるかどうかを確認します。クエリキャッシュにある場合（キャッシュヒット）、BEはクエリキャッシュから中間結果を直接取得します。そうでない場合（キャッシュミス）、BEはディスク上のデータにアクセスし、ローカル集計を行って中間結果を計算します。BEがテーブレットの処理を完了すると、そのテーブレットの集計の中間結果をクエリキャッシュに保存します。

2. テーブレット間の集計

   BEはクエリに関与するすべてのテーブレットから中間結果を収集し、それらを最終結果にマージします。

   ![Query cache - 仕組み - 1](../assets/query_cache_principle-1.png)

将来類似したクエリが実行されると、前のクエリのキャッシュされた結果を再利用できます。たとえば、次の図に示すクエリは、3つのテーブレット（テーブレット0から2）を関連付けています。最初のテーブレット（テーブレット0）の中間結果は既にクエリキャッシュにあります。この例では、BEはディスク上のデータにアクセスする代わりに、クエリキャッシュからテーブレット0の結果を直接取得できます。クエリキャッシュが完全にウォームアップされている場合、すべての3つのテーブレットの中間結果を含むことができるため、BEはディスク上のデータにアクセスする必要がありません。

![Query cache - 仕組み - 2](../assets/query_cache_principle-2.png)

余分なメモリを解放するために、クエリキャッシュはLeast Recently Used（LRU）ベースのエビクションポリシーを採用して、クエリキャッシュ内のキャッシュエントリを管理します。このエビクションポリシーによると、クエリキャッシュが占有するメモリ量が事前に定義されたサイズ（`query_cache_capacity`）を超える場合、最も最近使用されたキャッシュエントリがクエリキャッシュから排除されます。

> **注意**
>
> 将来的には、StarRocksはTime to Live（TTL）ベースのエビクションポリシーもサポートされ、このポリシーに基づいてクエリキャッシュからキャッシュエントリを排除できるようになります。

FEは、クエリがクエリキャッシュによって加速される必要があるかどうかを決定し、クエリの意味に影響を与えない取るに足らないリテラルの詳細を除去してクエリを正規化します。

クエリキャッシュの悪いケースによるパフォーマンスペナルティを防ぐために、BEは実行時にクエリキャッシュをバイパスする適応ポリシーを採用します。

## クエリキャッシュの有効化

このセクションでは、クエリキャッシュを有効にし、構成するために使用されるパラメータとセッション変数について説明します。

### FEセッション変数

| **変数**               | **デフォルト値** | **動的に構成可能** | **説明**                             |
| ---------------------- | ---------------- | ------------------- | ------------------------------------ |
| enable_query_cache     | false            | はい                | クエリキャッシュを有効にするかどうかを指定します。有効な値：`true`および`false`。`true`はこの機能を有効にし、`false`はこの機能を無効にします。クエリキャッシュが有効な場合、このトピックの"[アプリケーションシナリオ](../using_starrocks/query_cache.md#application-scenarios)"で指定された条件を満たすクエリにのみ機能します。 |
| query_cache_entry_max_bytes | 4194304   | はい                | Passthroughモードをトリガするしきい値を指定します。有効な値：`0`から`9223372036854775807`。クエリによってアクセスされる特定のテーブレットの計算結果からのバイト数または行数が、`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定されたしきい値を超える場合、クエリはPassthroughモードに切り替わります。<br />`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータが`0`に設定されている場合、関係するテーブレットからの計算結果が生成されない場合でも、Passthroughモードが使用されます。 |
| query_cache_entry_max_rows  | 409600     | はい                | 上記と同様。                         |

### BEパラメータ

BE設定ファイル**be.conf**で次のパラメータを構成する必要があります。BEのためにこのパラメータを再構成した後は、新しいパラメータ設定を有効にするためにBEを再起動する必要があります。

| **パラメータ** | **必須** | **説明** |
| -------------- | -------- | ------- |
```
| query_cache_capacity | No | クエリキャッシュのサイズを指定します。単位：バイト。デフォルトサイズは512 MBです。<br />各BEには独自のローカルクエリキャッシュがあり、自分自身のクエリキャッシュのみを使って構築および調査されます。<br />クエリキャッシュのサイズは4 MB未満にすることはできません。BEのメモリ容量が期待されるクエリキャッシュサイズのプロビジョニングに十分でない場合は、BEのメモリ容量を増やすことができます。|

## あらゆるシナリオで最大のキャッシュヒット率を実現するよう設計されています

クエリが文字通りに完全に一致しなくても、クエリキャッシュが効果的である三つのシナリオを考えてみましょう。これらの三つのシナリオは次のとおりです：

- 意味的に同等なクエリ
- スキャンされたパーティションが重なるクエリ
- 追加のみのデータ変更（UPDATEまたはDELETEの操作なし）を対象とするクエリ

### 意味的に同等なクエリ

二つのクエリが似ていると、それが文字通りに同等である必要はなく、実行計画の中で意味的に等価なスニペットを含むことを意味します。それらは意味的に同等であり、互いの計算結果を再利用することができます。広義には、二つのクエリがデータを同じソースから問い合わせ、同じ計算方法を使用し、同じ実行計画を持つ場合、それらは意味的に同等です。StarRocksは、二つのクエリが意味的に同等であるかどうかを評価するために以下のルールを適用します:

- 二つのクエリが複数の集計を含む場合、最初の集計が意味的に同等であるかぎり、それらは意味的に同等と評価されます。例えば、次の二つのクエリ、Q1とQ2はどちらも複数の集計を含みますが、最初の集計が意味的に同等であるため、Q1とQ2は意味的に同等と評価されます。

  - Q1

    ```SQL
    SELECT
        (
            ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
    FROM
        (
            SELECT
                date_trunc('hour', ts) AS hour,
                k0,
                sum(v1) AS __c_0
            FROM
                t0
            WHERE
                ts between '2022-01-03 00:00:00'
                and '2022-01-03 23:59:59'
            GROUP BY
                date_trunc('hour', ts),
                k0
        ) AS t;
    ```

  - Q2

    ```SQL
    SELECT
        date_trunc('hour', ts) AS hour,
        k0,
        sum(v1) AS __c_0
    FROM
        t0
    WHERE
        ts between '2022-01-03 00:00:00'
        and '2022-01-03 23:59:59'
    GROUP BY
        date_trunc('hour', ts),
        k0
    ```

- 二つのクエリが以下のクエリタイプのいずれかに属する場合、それらは意味的に同等と評価されます。HAVING句を含むクエリは、HAVING句を含まないクエリと意味的に同等と評価されることはありません。ただし、ORDER BY句やLIMIT句の付加は、二つのクエリが意味的に同等かどうかの評価に影響しません。

  - GROUP BY集計

    ```SQL
    SELECT <GroupByItems>, <AggFunctionItems> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    [HAVING <HavingPredicate>] 
    ```

    > **注記**
    >
    > 上記の例では、HAVING句はオプションです。

  - GROUP BY DISTINCT集計

    ```SQL
    SELECT DISTINCT <GroupByItems>, <Items> 
    FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    GROUP BY <GroupByItems>
    HAVING <HavingPredicate>
    ```

    > **注記**
    >
    > 上記の例では、HAVING句はオプションです。

  - GROUP BY未集計

    ```SQL
    SELECT <AggFunctionItems> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

  - GROUP BY DISTINCT未集計

    ```SQL
    SELECT DISTINCT <Items> FROM <Table> 
    WHERE <Predicates> [and <PartitionColumnRangePredicate>]
    ```

- どちらかのクエリが`PartitionColumnRangePredicate`を含む場合、`PartitionColumnRangePredicate`は、二つのクエリが意味的に同等かどうかを評価する前に削除されます。`PartitionColumnRangePredicate`は、パーティション化された列を参照する述語の以下のタイプを指定します:

  - `col between v1 and v2`：パーティション化された列の値が[v1, v2]の範囲にある。ここで、`v1`と`v2`は定数式です。
  - `v1 < col and col < v2`：パーティション化された列の値が(v1, v2)の範囲にある。ここで、`v1`と`v2`は定数式です。
  - `v1 < col and col <= v2`：パーティション化された列の値が(v1, v2]の範囲にある。ここで、`v1`と`v2`は定数式です。
  - `v1 <= col and col < v2`：パーティション化された列の値が[v1, v2)の範囲にある。ここで、`v1`と`v2`は定数式です。
  - `v1 <= col and col <= v2`：パーティション化された列の値が[v1, v2]の範囲にある。ここで、`v1`と`v2`は定数式です。

- 二つのクエリのSELECT句の出力列が並べ替えられた後も同じであれば、その二つのクエリは意味的に同等と評価されます。

- 二つのクエリのGROUP BY句の出力列が並べ替えられた後も同じであれば、その二つのクエリは意味的に同等と評価されます。

- 二つのクエリのWHERE句の残りの述語が、`PartitionColumnRangePredicate`が削除されて意味的に同等であれば、その二つのクエリは意味的に同等と評価されます。

- 二つのクエリのHAVING句内の述語が意味的に同等であれば、その二つのクエリは意味的に同等と評価されます。

次の表 `lineorder_flat` を例として使用します：

```SQL
CREATE TABLE `lineorder_flat`
(
    `lo_orderdate` date NOT NULL COMMENT "",
    `lo_orderkey` int(11) NOT NULL COMMENT "",
    `lo_linenumber` tinyint(4) NOT NULL COMMENT "",
    `lo_custkey` int(11) NOT NULL COMMENT "",
    `lo_partkey` int(11) NOT NULL COMMENT "",
    `lo_suppkey` int(11) NOT NULL COMMENT "",
    `lo_orderpriority` varchar(100) NOT NULL COMMENT "",
    `lo_shippriority` tinyint(4) NOT NULL COMMENT "",
    `lo_quantity` tinyint(4) NOT NULL COMMENT "",
    `lo_extendedprice` int(11) NOT NULL COMMENT "",
    `lo_ordtotalprice` int(11) NOT NULL COMMENT "",
    `lo_discount` tinyint(4) NOT NULL COMMENT "",
    `lo_revenue` int(11) NOT NULL COMMENT "",
    `lo_supplycost` int(11) NOT NULL COMMENT "",
    `lo_tax` tinyint(4) NOT NULL COMMENT "",
    `lo_commitdate` date NOT NULL COMMENT "",
    `lo_shipmode` varchar(100) NOT NULL COMMENT "",
    `c_name` varchar(100) NOT NULL COMMENT "",
    `c_address` varchar(100) NOT NULL COMMENT "",
    `c_city` varchar(100) NOT NULL COMMENT "",
    `c_nation` varchar(100) NOT NULL COMMENT "",
    `c_region` varchar(100) NOT NULL COMMENT "",
    `c_phone` varchar(100) NOT NULL COMMENT "",
    `c_mktsegment` varchar(100) NOT NULL COMMENT "",
    `s_name` varchar(100) NOT NULL COMMENT "",
    `s_address` varchar(100) NOT NULL COMMENT "",
    `s_city` varchar(100) NOT NULL COMMENT "",
    `s_nation` varchar(100) NOT NULL COMMENT "",
    `s_region` varchar(100) NOT NULL COMMENT "",
    `s_phone` varchar(100) NOT NULL COMMENT "",
    `p_name` varchar(100) NOT NULL COMMENT "",
    `p_mfgr` varchar(100) NOT NULL COMMENT "",
    `p_category` varchar(100) NOT NULL COMMENT "",
    `p_brand` varchar(100) NOT NULL COMMENT "",
    `p_color` varchar(100) NOT NULL COMMENT "",
    `p_type` varchar(100) NOT NULL COMMENT "",
    `p_size` tinyint(4) NOT NULL COMMENT "",
    `p_container` varchar(100) NOT NULL COMMENT ""
)
ENGINE=OLAP 
DUPLICATE KEY(`lo_orderdate`, `lo_orderkey`)
COMMENT "olap"
PARTITION BY RANGE(`lo_orderdate`)
(PARTITION p1 VALUES [('0000-01-01'), ('1993-01-01')),
PARTITION p2 VALUES [('1993-01-01'), ('1994-01-01')),
PARTITION p3 VALUES [('1994-01-01'), ('1995-01-01')),
PARTITION p4 VALUES [('1995-01-01'), ('1996-01-01')),
```SQL
PARTITION p5 VALUES [('1996-01-01'), ('1997-01-01')),
PARTITION p6 VALUES [('1997-01-01'), ('1998-01-01')),
PARTITION p7 VALUES [('1998-01-01'), ('1999-01-01')))
DISTRIBUTED BY HASH(`lo_orderkey`)
PROPERTIES 
(
    "replication_num" = "1",
    "colocate_with" = "groupxx1",
    "storage_format" = "DEFAULT",
    "enable_persistent_index" = "false",
    "compression" = "LZ4"
);
```

以下の2つのクエリ、Q1とQ2は、以下のように処理された後、テーブル`lineorder_flat`の意味的な等価性を持ちます：

1. SELECTステートメントの出力列を並べ替えます。
2. GROUP BY句の出力列を並べ替えます。
3. ORDER BY句の出力列を削除します。
4. WHERE句の述語を並べ替えます。
5. `PartitionColumnRangePredicate`を追加します。

- Q1

  ```SQL
  SELECT sum(lo_revenue)), year(lo_orderdate) AS year,p_brand
  FROM lineorder_flat
  WHERE p_category = 'MFGR#12' AND s_region = 'AMERICA'
  GROUP BY year,p_brand
  ORDER BY year,p_brand;
  ```

- Q2

  ```SQL
  SELECT year(lo_orderdate) AS year, p_brand, sum(lo_revenue))
  FROM lineorder_flat
  WHERE s_region = 'AMERICA' AND p_category = 'MFGR#12' AND 
     lo_orderdate >= '1993-01-01' AND lo_orderdate <= '1993-12-31'
  GROUP BY p_brand, year(lo_orderdate)
  ```

意味的な等価性は、クエリの物理的な計画に基づいて評価されます。したがって、クエリの文法上の違いは、意味的な等価性の評価に影響しません。加えて、クエリからは定数式が削除され、クエリの最適化中に`cast`式が削除されます。したがって、これらの式は意味的な等価性の評価に影響しません。第三に、列やリレーションのエイリアスも意味的な等価性の評価に影響しません。

### オーバーラップするスキャンされたパーティションを持つクエリ

クエリキャッシュは述語ベースのクエリ分割をサポートしています。

述語に基づくクエリの分割は、部分的な計算結果の再利用を実装するのに役立ちます。クエリにテーブルの分割列を参照する述語が含まれ、述語が値範囲を指定する場合、StarRocksは、表の分割に基づいて複数の区間に範囲を分割できます。各個々の区間からの計算結果は、他のクエリによって別々に再利用されることができます。

次の表`t0`を例に挙げます：

```SQL
CREATE TABLE if not exists t0
(
    ts DATETIME NOT NULL,
    k0 VARCHAR(10) NOT NULL,
    k1 BIGINT NOT NULL,
    v1 DECIMAL64(7, 2) NOT NULL 
)
ENGINE=OLAP
DUPLICATE KEY(`ts`, `k0`, `k1`)
COMMENT "OLAP"
PARTITION BY RANGE(ts)
(
  START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 day) 
)
DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
PROPERTIES
(
    "replication_num" = "1", 
    "storage_format" = "default"
);
```

テーブル`t0`は、日によってパーティション化されており、列`ts`がテーブルのパーティショニング列です。以下の4つのクエリのうち、Q2、Q3、Q4は、Q1に対してキャッシュされた部分的な計算結果を再利用することができます：

- Q1

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' AND '2022-01-14 23:59:59'
  GROUP BY day;
  ```

  Q1の述語 `ts between '2022-01-02 12:30:00' and '2022-01-14 23:59:59'` で指定された値範囲は、以下の区間に分割できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  12. [2022-01-13 00:00:00, 2022-01-14 00:00:00),
  13. [2022-01-14 00:00:00, 2022-01-15 00:00:00),
  ```

- Q2

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-02 12:30:00' AND  ts < '2022-01-05 00:00:00'
  GROUP BY day;
  ```

  Q2は、Q1の以下の区間内の計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ```

- Q3

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts >= '2022-01-01 12:30:00' AND  ts <= '2022-01-10 12:00:00'
  GROUP BY day;
  ```

  Q3は、Q1の以下の区間内の計算結果を再利用できます：

  ```SQL
  2. [2022-01-03 00:00:00, 2022-01-04 00:00:00),
  3. [2022-01-04 00:00:00, 2022-01-05 00:00:00),
  ...
  8. [2022-01-09 00:00:00, 2022-01-10 00:00:00),
  ```

- Q4

  ```SQL
  SELECT date_trunc('day', ts) as day, sum(v0)
  FROM t0
  WHERE ts BETWEEN '2022-01-02 12:30:00' and '2022-01-02 23:59:59'
  GROUP BY day;
  ```

  Q4は、Q1の以下の区間内の計算結果を再利用できます：

  ```SQL
  1. [2022-01-02 12:30:00, 2022-01-03 00:00:00),
  ```

部分的な計算結果再利用のサポートは、使用されるパーティション化ポリシーによって異なります。

| **パーティショニングポリシー** | **部分的な計算結果の再利用のサポート**                    |
| ------------------------- | ------------------------------------------------------------ |
| 非パーティション化              | サポートされていません                                       |
| 複数列のパーティショニング       | サポートされていません<br />**注意**<br />この機能は将来サポートされる可能性があります。 |
| 単一列のパーティショニング       | サポートされています                                       |

### 追加のみのデータ変更を持つデータに対するクエリ

クエリキャッシュはマルチバージョンのキャッシングをサポートしています。

データロードが行われるたびに、新しいバージョンのタブレットが生成されます。その結果、クエリキャッシュに保存された古いタブレットの計算結果は古くなり、最新のタブレットのバージョンに遅れることになります。このような状況では、マルチバージョンキャッシングメカニズムは、クエリキャッシュに保存された古い結果とディスク上に保存されたタブレットの増分バージョンを最終結果にマージしようとします。このように、新しいクエリが最新のタブレットバージョンを持つことができるようにします。マルチバージョンのキャッシングは、テーブルの種類、クエリのタイプ、およびデータの更新タイプによって制約されます。

マルチバージョンのキャッシングのサポートは、テーブルの種類とクエリのタイプによって異なります。

| **テーブルタイプ**       | **クエリ** **タイプ**                                         | **マルチバージョンのキャッシングのサポート**             |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 重複キー テーブル    | 基底テーブルに対するクエリ または同期マテリアライズドビューに対するクエリ | <ul><li>基本テーブルに対するクエリ：増分タブレットのバージョンにデータ削除レコードが含まれていない場合を除き、すべての状況でサポートされます。</li><li>同期マテリアライズドビューに対するクエリ：クエリのGROUP BY、HAVING、またはWHERE句で集計列が参照されている場合を除き、すべての状況でサポートされます。</li></ul> |
| 集約テーブル           | 基底テーブルに対するクエリまたは同期マテリアライズドビューに対するクエリ | 次の場合を除き、すべての状況でサポートされます：基本テーブルのスキーマに`replace`集約関数が含まれている。クエリのGROUP BY、HAVING、またはWHERE句で集計列が参照されている。増分タブレットのバージョンにデータ削除レコードが含まれている。 |
| Unique Key table    | N/A                                                          | Not supported. However, the query cache is supported.        |
| Primary Key table   | N/A                                                          | Not supported. However, the query cache is supported.        |

データ更新タイプがマルチバージョンキャッシュに与える影響は次の通りです。

- データ削除

  増分バージョンのタブレットに削除操作が含まれている場合、マルチバージョンキャッシングを使用することはできません。

- データ挿入

  - タブレットの空のバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効のまま取得できます。
  - タブレットの空でないバージョンが生成された場合、クエリキャッシュ内のタブレットの既存データは有効のままですが、そのバージョンはタブレットの最新バージョンから遅れています。 この状況では、StarRocksは既存データから最新のタブレットの増分データを読み込み、既存データを増分データとマージし、マージされたデータをクエリキャッシュに格納します。

- スキーマ変更とタブレットの切り捨て

  テーブルのスキーマが変更された場合やテーブルの特定のタブレットが切り捨てられた場合、テーブルの新しいタブレットが生成されます。この結果、クエリキャッシュ内のテーブルの既存データは無効になります。

## メトリック

クエリキャッシュが機能するクエリのプロファイルには`CacheOperator`の統計情報が含まれます。

クエリのソースプランにおいて、パイプラインに`OlapScanOperator`が含まれる場合、`OlapScanOperator`および集計演算子の名前は`MultilaneOperator`を使用してタブレットごとの演算を行うことを表すために、`ML_`が接頭辞として付けられます。 `CacheOperator`は、パススルー、ポピュレート、およびプローブモードでクエリキャッシュの実行を制御するロジックを処理するために、`ML_CONJUGATE_AGGREGATE`の直前に挿入されます。クエリのプロファイルには、クエリキャッシュの使用状況を理解するのに役立つ以下の`CacheOperator`メトリックが含まれます。

| **メトリック**                | **説明**                                              |
| ------------------------- | ------------------------------------------------------------ |
| CachePassthroughBytes     | パススルーモードで生成されたバイト数。           |
| CachePassthroughChunkNum  | パススルーモードで生成されたチャンク数。          |
| CachePassthroughRowNum    | パススルーモードで生成された行数。            |
| CachePassthroughTabletNum | パススルーモードで生成されたタブレット数。     |
| CachePassthroughTime:     | パススルーモードで実行された計算時間。    |
| CachePopulateBytes        | ポピュレートモードで生成されたバイト数。 |
| CachePopulateChunkNum     | ポピュレートモードで生成されたチャンク数。   |
| CachePopulateRowNum       | ポピュレートモードで生成された行数。   |
| CachePopulateTabletNum    | ポピュレートモードで生成されたタブレット数。   |
| CachePopulateTime         | ポピュレートモードで実行された計算時間。   |
| CacheProbeBytes           | プローブモードでキャッシュヒットしたバイト数。   |
| CacheProbeChunkNum        | プローブモードでキャッシュヒットしたチャンク数。   |
| CacheProbeRowNum          | プローブモードでキャッシュヒットした行数。   |
| CacheProbeTabletNum       | プローブモードでキャッシュヒットしたタブレット数。   |
| CacheProbeTime            | プローブモードで実行された計算時間。   |

`CachePopulate`*`XXX`*メトリックは、クエリキャッシュが更新されるクエリキャッシュミスに関する統計情報を提供します。

`CachePassthrough`*`XXX`*メトリックは、パータブレットの計算結果のサイズが大きいため、クエリキャッシュが更新されないクエリキャッシュミスに関する統計情報を提供します。

`CacheProbe`*`XXX`*メトリックは、クエリキャッシュヒットに関する統計情報を提供します。

マルチバージョンキャッシングメカニズムでは、`CachePopulate`メトリックと`CacheProbe`メトリックは同じタブレットの統計情報を含む場合があり、`CachePassthrough`メトリックと`CacheProbe`メトリックも同じタブレットの統計情報を含む場合があります。例えば、StarRocksが各タブレットのデータを計算する際、タブレットの過去のバージョンで生成された計算結果を使用します。この状況では、StarRocksはタブレットの最新バージョンに増分データを読み込み、データを計算し、増分データをキャッシュされたデータとマージします。マージ後の計算結果のサイズが`query_cache_entry_max_bytes`または`query_cache_entry_max_rows`パラメータで指定された閾値を超えない場合、タブレットの統計情報は`CachePopulate`メトリックに収集されます。それ以外の場合、タブレットの統計情報は`CachePassthrough`メトリックに収集されます。

## RESTful API操作

- `metrics |grep query_cache`

  このAPI操作は、クエリキャッシュに関連するメトリックをクエリするために使用されます。

  ```shell
  curl -s  http://<be_host>:<be_http_port>/metrics |grep query_cache
  
  # TYPE starrocks_be_query_cache_capacity gauge
  starrocks_be_query_cache_capacity 536870912
  # TYPE starrocks_be_query_cache_hit_count gauge
  starrocks_be_query_cache_hit_count 5084393
  # TYPE starrocks_be_query_cache_hit_ratio gauge
  starrocks_be_query_cache_hit_ratio 0.984098
  # TYPE starrocks_be_query_cache_lookup_count gauge
  starrocks_be_query_cache_lookup_count 5166553
  # TYPE starrocks_be_query_cache_usage gauge
  starrocks_be_query_cache_usage 0
  # TYPE starrocks_be_query_cache_usage_ratio gauge
  starrocks_be_query_cache_usage_ratio 0.000000
  ```

- `api/query_cache/stat`

  このAPI操作は、クエリキャッシュの使用状況をクエリするために使用されます。

  ```shell
  curl  http://<be_host>:<be_http_port>/api/query_cache/stat
  {
      "capacity": 536870912,
      "usage": 0,
      "usage_ratio": 0.0,
      "lookup_count": 5025124,
      "hit_count": 4943720,
      "hit_ratio": 0.983800598751394
  }
  ```

- `api/query_cache/invalidate_all`

  このAPI操作は、クエリキャッシュをクリアするために使用されます。

  ```shell
  curl  -XPUT http://<be_host>:<be_http_port>/api/query_cache/invalidate_all
  
  {
      "status": "OK"
  }
  ```

前述のAPI操作のパラメータは次のとおりです。

- `be_host`：BEが存在するノードのIPアドレス。
- `be_http_port`：BEが存在するノードのHTTPポート番号。

## 注意事項

- StarRocksは、初めて開始されるクエリの計算結果でクエリキャッシュをポピュレートする必要があります。そのため、クエリのパフォーマンスが想定よりも若干低下し、クエリの待機時間が増加する可能性があります。
- 大きなクエリキャッシュサイズを構成すると、BEでクエリ評価に割り当てられるメモリ量が減少します。クエリキャッシュサイズが、クエリ評価に割り当てられたメモリ容量の1/6を超えないように推奨します。
- 処理する必要があるタブレットの数が`pipeline_dop`の値よりも小さい場合、クエリキャッシュは機能しません。クエリキャッシュを機能させるには、`pipeline_dop`を`1`のような小さい値に設定できます。v3.0以降では、StarRocksはこのパラメータをクエリ並列処理に基づいて自動的に調整します。

## 例

### データセット

1. StarRocksクラスターにログインし、対象のデータベースに移動し、次のコマンドを実行して` t0`というテーブルを作成します。

   ```SQL
   CREATE TABLE t0
   (
         `ts` datetime NOT NULL COMMENT "",
         `k0` varchar(10) NOT NULL COMMENT "",
         `k1` char(6) NOT NULL COMMENT "",
         `v0` bigint(20) NOT NULL COMMENT "",
         `v1` decimal64(7, 2) NOT NULL COMMENT ""
   )
   ENGINE=OLAP 
   DUPLICATE KEY(`ts`, `k0`, `k1`)
   COMMENT "OLAP"
   PARTITION BY RANGE(`ts`)
   (
       START ("2022-01-01 00:00:00") END ("2022-02-01 00:00:00") EVERY (INTERVAL 1 DAY)
   )
   DISTRIBUTED BY HASH(`ts`, `k0`, `k1`)
   PROPERTIES
   (
       "replication_num" = "1",
       "storage_format" = "DEFAULT",
       "enable_persistent_index" = "false"
   );
   ```

2. 次のデータレコードを` t0`に挿入します。

   ```SQL
   INSERT INTO t0
   VALUES
       ('2022-01-11 20:42:26', 'n4AGcEqYp', 'hhbawx', '799393174109549', '8029.42'),
       ('2022-01-27 18:17:59', 'i66lt', 'mtrtzf', '100400167', '10000.88'),
       ('2022-01-28 20:10:44', 'z6', 'oqkeun', '-58681382337', '59881.87'),
       ('2022-01-29 14:54:31', 'qQ', 'dzytua', '-19682006834', '43807.02'),
```
      + ('2022-01-31 08:08:11', 'qQ', 'dzytua', '7970665929984223925', '-8947.74'),
      + ('2022-01-15 00:40:58', '65', 'hhbawx', '4054945', '156.56'),
    + ('2022-01-24 16:17:51', 'onqR3JsK1', 'udtmfp', '-12962', '-72127.53'),
  + ('2022-01-01 22:36:24', 'n4AGcEqYp', 'fabnct', '-50999821', '17349.85'),
```
```
       ('2022-01-19 15:20:20', '7Bh', 'lbsvqu', '347317095885', '-76393.49'),
       ('2022-01-08 16:18:22', 'z6', 'fghmcd', '2', '90315.60'),
       ('2022-01-02 00:23:06', 'Pc', 'piszhr', '-3651517384168400', '58220.34'),
       ('2022-01-12 08:23:31', 'onqR3JsK1', 'udtmfp', '5636394870355729225', '33224.25'),
       ('2022-01-28 10:46:44', 'onqR3JsK1', 'oqkeun', '-28102078612755', '6469.53'),
       ('2022-01-23 23:16:11', 'onqR3JsK1', 'ghftus', '-707475035515433949', '63422.66'),
       ('2022-01-03 05:32:31', 'z6', 'hhbawx', '-45', '-49680.52'),
       ('2022-01-27 03:24:33', 'qQ', 'qmwhvr', '375943906057539870', '-66092.96'),
       ('2022-01-25 20:07:22', '7Bh', 'slfght', '1', '72440.21'),
       ('2022-01-04 16:07:24', 'qQ', 'uqkghp', '751213107482249', '16417.31'),
       ('2022-01-23 19:22:00', 'Pc', 'hhbawx', '-740731249600493', '88439.40'),
       ('2022-01-05 09:04:20', '7Bh', 'cawanm', '23602', '302.44');
   ```

### クエリの例

このセクションのクエリキャッシュ関連のメトリクスについての統計情報は例であり、参考情報としてのみ提供されています。

#### クエリキャッシュはステージ1のローカル集約に対して機能します

これには3つの状況が含まれます。

- クエリが単一のタブレットにのみアクセスする場合
- クエリがテーブルの共有グループを構成する複数のパーティションから複数のタブレットにアクセスし、集約のためにデータをシャッフルする必要がない場合
- クエリがテーブルの同じパーティションから複数のタブレットにアクセスし、集約のためにデータをシャッフルする必要がない場合

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    k0,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    k0
```

以下の図は、クエリプロファイルでのクエリキャッシュ関連のメトリクスを示しています。

![Query Cache - Stage 1 - Metrics](../assets/query_cache_stage1_agg_with_cache_en.png)

#### クエリキャッシュはステージ1のリモート集約に対して機能しません

ステージ1で複数のタブレットでの集約が強制的に実行される場合、データはまずシャッフルされ、その後集約されます。

クエリの例：

```SQL
SET new_planner_agg_stage = 1;

SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

#### クエリキャッシュはステージ2のローカル集約に対して機能します

これには3つの状況が含まれます。

- クエリのステージ2の集約は同じタイプのデータを比較するようにコンパイルされます。最初の集約はローカル集約です。最初の集約が完了すると、最初の集約から生成された結果を使用して、2番目の集約であるグローバル集約が実行されます。
- クエリはSELECT DISTINCTクエリである
- クエリが以下のいずれかのDISTINCT集約関数を含む場合： `sum(distinct)`、 `count(distinct)`、および `avg(distinct)`。ほとんどの場合、このようなクエリではステージ3または4で集約が実行されます。ただし、このクエリの場合は、`set new_planner_agg_stage = 1` を実行してクエリのステージ2で強制的に集約を実行する必要があります。また、`avg(distinct)` を含む場合は、`set cbo_cte_reuse = false` を実行してCTEの最適化を無効にする必要があります。

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd
```

以下の図は、クエリプロファイルでのクエリキャッシュ関連のメトリクスを示しています。

![Query Cache - Stage 2 - Metrics](../assets/query_cache_stage2_agg_with_cache_en.png)

#### クエリキャッシュはステージ3のローカル集約に対して機能します

クエリは、単一のDISTINCT集約関数を含むGROUP BY集約クエリです。

サポートされるDISTINCT集約関数は、`sum(distinct)`、`count(distinct)`、`avg(distinct)` です。

> **注意**
>
> クエリに `avg(distinct)` が含まれる場合、CTEの最適化を無効にするために `set cbo_cte_reuse = false` を実行する必要があります。

クエリの例：

```SQL
SELECT
    date_trunc('hour', ts) AS hour,
    v0 % 2 AS is_odd,
    sum(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
GROUP BY
    date_trunc('hour', ts),
    is_odd;
```

以下の図は、クエリプロファイルでのクエリキャッシュ関連のメトリクスを示しています。

![Query Cache - Stage 3 - Metrics](../assets/query_cache_stage3_agg_with_cache_en.png)

#### クエリキャッシュはステージ4のローカル集約に対して機能します

単一のDISTINCT集約関数を含むGROUP BY集約クエリです。

サポートされるDISTINCT集約関数は、`sum(distinct)`、`count(distinct)`、`avg(distinct)` です。

クエリの例：

```SQL
SELECT
    count(distinct v1) AS __c_0
FROM
    t0
WHERE
    ts between '2022-01-03 00:00:00'
    and '2022-01-03 23:59:59'
```

以下の図は、クエリプロファイルでのクエリキャッシュ関連のメトリクスを示しています。

![Query Cache - Stage 4 - Metrics](../assets/query_cache_stage4_agg_with_cache_en.png)

#### 2つのクエリの最初の集約が意味論的に等価であるため、キャッシュされた結果が再利用される2つのクエリ

次の2つのクエリ、Q1とQ2、を例に使用します。Q1とQ2はともに複数の集約を含みますが、最初の集約が意味論的に等価です。そのため、Q1とQ2は意味論的に同等と見なされ、クエリキャッシュに保存された相互の計算結果を再利用できます。

- Q1

  ```SQL
  SELECT
      (
          ifnull(sum(murmur_hash3_32(hour)), 0) + ifnull(sum(murmur_hash3_32(k0)), 0) + ifnull(sum(murmur_hash3_32(__c_0)), 0)
        ) AS fingerprint
  FROM
      (
          SELECT
              date_trunc('hour', ts) AS hour,
              k0,
              sum(v1) AS __c_0
          FROM
              t0
          WHERE
              ts between '2022-01-03 00:00:00'
              and '2022-01-03 23:59:59'
          GROUP BY
              date_trunc('hour', ts),
              k0
      ) AS t;
  ```

- Q2

  ```SQL
  SELECT
      date_trunc('hour', ts) AS hour,
      k0,
      sum(v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59'
  GROUP BY
      date_trunc('hour', ts),
      k0
  ```

以下の図は、Q1の `CachePopulate` メトリクスを示しています。

![Query Cache - Q1 - Metrics](../assets/query_cache_reuse_Q1_en.png)

以下の図は、Q2の `CacheProbe` メトリクスを示しています。

![Query Cache - Q2 - Metrics](../assets/query_cache_reuse_Q2_en.png)

#### CTEの最適化が有効になっているDISTINCTクエリにはクエリキャッシュは機能しません
```
```SQL
'set cbo_cte_reuse = true' を実行してCTEの最適化を有効にした後は、DISTINCT集約関数を含む特定のクエリの計算結果はキャッシュされません。いくつかの例を以下に示します。

- クエリに単一のDISTINCT集約関数 `avg(distinct)` が含まれる場合：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 1](../assets/query_cache_distinct_with_cte_Q1_en.png)

- 同じ列を参照する複数のDISTINCT集約関数が含まれるクエリの場合：

  ```SQL
  SELECT
      avg(distinct v1) AS __c_0,
      sum(distinct v1) AS __c_1,
      count(distinct v1) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 2](../assets/query_cache_distinct_with_cte_Q2_en.png)

- 異なる列を参照する複数のDISTINCT集約関数が含まれるクエリの場合：

  ```SQL
  SELECT
      sum(distinct v1) AS __c_1,
      count(distinct v0) AS __c_2
  FROM
      t0
  WHERE
      ts between '2022-01-03 00:00:00'
      and '2022-01-03 23:59:59';
  ```

![Query Cache - CTE - 3](../assets/query_cache_distinct_with_cte_Q3_en.png)

## ベストプラクティス

テーブルを作成する際には、適切なパーティション記述および適切な分散方法を指定してください。具体的には以下のことが含まれます：

- パーティション列として単一のDATE型列を選択してください。テーブルに複数のDATE型列が含まれる場合は、データが増分的に取り込まれる際に値が進む列を選択し、クエリの興味深い時間範囲を定義するために使用されます。
- 適切なパーティション幅を選択してください。最新に取り込まれたデータが最新のパーティションを修正する可能性があります。したがって、最新のパーティションを含むキャッシュエントリは安定しておらず、無効になりやすいです。
- テーブル作成文の分散記述には、数十個のバケット番号を指定してください。バケット番号が極端に小さい場合、BEが処理する必要があるタブレットの数が `pipeline_dop` の値よりも少ない場合、クエリキャッシュが効果を発揮しません。
```