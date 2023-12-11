---
displayed_sidebar: "Japanese"
---

# 近似重複数のカウントにHLLを使用する

## 背景

現実世界のシナリオでは、データの量が増えるにつれてデータの重複を排除する圧力が増大します。データのサイズがある程度のレベルに達すると、正確な重複排除のコストは比較的高くなります。このようなケースでは、通常、ユーザーは計算上の圧力を軽減するために近似アルゴリズムを使用します。このセクションでは紹介されるHyperLogLog（HLL）は、優れた空間計算量O(mloglogn)と時間計算量O(n)を持つ近似重複排除アルゴリズムです。さらに、データセットのサイズや使用されるハッシュ関数に応じて、計算結果の誤差率を約1%〜10%に制御できます。

## HyperLogLogとは

HyperLogLogは非常に少ないストレージスペースを消費する近似重複排除アルゴリズムです。**HLLタイプ**はHyperLogLogアルゴリズムの実装に使用されます。HyperLogLogの計算中間結果を保持し、データテーブルのインジケーターカラムタイプとしてのみ使用できます。

HLLアルゴリズムは多くの数学的知識を必要としますが、具体的な例を用いて説明します。ランダム化実験Aを設計し、最初の表が出るまでのコインフリップを独立して繰り返します。最初の表が出るためのコインフリップの回数を確立変数Xとして記録し、次のようになります:

* X=1, P(X=1)=1/2
* X=2, P(X=2)=1/4
* ...
* X=n, P(X=n)=(1/2)<sup>n</sup>

テストAを使用して、データセット上で上記の実験をシミュレーションします。

* テストA: データセット要素のハッシュ値を計算し、ハッシュ値をバイナリ表現に変換します。バイナリの最下位ビットからビット=1の発生を記録します。
* テストB: テストBのデータセット要素に対してテストAのプロセスを繰り返します。各テストの最初のビット1の発生の最大位置 "m" を更新します。
* データセット内の非重複要素の数を m<sup>2</sup> と推定します。

HLLアルゴリズムでは、要素を下位kビットに基づいてK=2<sup>k</sup>のバケツに分割します。要素のハッシュによって最初のビット1の発生の最大値を、k+1番目のビットからの最大数としてm<sub>1</sub>, m<sub>2</sub>,..., m<sub>k</sub>を数え上げ、バケツ内の非重複要素の数を2<sup>m<sub>1</sub></sup>, 2<sup>m<sub>2</sub></sup>,..., 2<sup>m<sub>k</sub></sup>と推定します。データセット内の非重複要素数は、バケツ内の非重複要素数の平均数をバケツの数で乗算したものの合計となります: N = K(K/(2<sup>-m<sub>1</sub></sup>+2<sup>-m<sub>2</sub></sup>,..., 2<sup>-m<sub>K</sub></sup>)).
<br/>

HLLは推定結果に補正係数を乗じて、より正確な結果とします。

HLL重複排除アルゴリズムをStarRocks SQLステートメントで実装した記事[https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649)を参照してください。

~~~sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM(select(murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
~~~

このアルゴリズムは、db0.table0のcol2の重複を除去します。

* ハッシュ関数 `murmur_hash3_32` を使用して、col2のハッシュ値を32ビット符号付き整数として計算します。
* 1024のバケツを使用し、補正係数は0.721であり、ハッシュ値の最下位10ビットをバケットの添字とします。
* ハッシュ値の符号ビットを無視し、最上位ビットから最下位ビットまでの位置を決定し、最初のビット1の発生地点を決定します。
* 計算されたハッシュ値をバケツごとにグループ化し、`MAX` 集計関数を使用してバケット内の最初のビット1の発生地点を見つけます。
* 集計結果はサブクエリとして使用し、すべてのバケツの推定値の平均をバケツの数と補正係数で乗算します。
* 空のバケット数は1であることに注意してください。

データ量が大きい場合、上記のアルゴリズムは非常に低い誤差率を持ちます。

これがHLLアルゴリズムの核心アイデアです。興味がある場合は[HyperLogLog論文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)を参照してください。

### HyperLogLogの使用方法

1. HyperLogLog重複排除を使用するには、対象のインジケーターカラムタイプを`HLL` に設定し、テーブル作成ステートメントで集約関数を `HLL_UNION` に設定する必要があります。
2. 現在、集約モデルのみがインジケーターカラムタイプとしてHLLをサポートしています。
3. `HLL` タイプの列に `count distinct` を使用する場合、StarRocksは自動的に `HLL_UNION_AGG` 計算に変換します。

#### 例

まず、uvが集約されたカラムで、カラムタイプが `HLL` であり、集約関数が [HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md) であるテーブルを作成します。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> * 注: データ量が大きい場合は、高頻度のHLLクエリ用に対応するロールアップテーブルを作成することが望ましいです。

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用してデータをロードします:

~~~bash
curl --location-trusted -u <username>:<password> -H "label:label_1600997542287" \
    -H "column_separator:," \
    -H "columns:dt,id,user_id, uv=hll_hash(user_id)" -T /root/test.csv http://starrocks_be0:8040/api/db0/test/_stream_load
{
    "TxnId": 2504748,
    "Label": "label_1600997542287",
    "Status": "Success",
    "Message": "OK",
    "NumberTotalRows": 5,
    "NumberLoadedRows": 5,
    "NumberFilteredRows": 0,
    "NumberUnselectedRows": 0,
    "LoadBytes": 120,
    "LoadTimeMs": 46,
    "BeginTxnTimeMs": 0,
    "StreamLoadPutTimeMs": 1,
    "ReadDataTimeMs": 0,
    "WriteDataTimeMs": 29,
    "CommitAndPublishTimeMs": 14
}
~~~

Broker Load モード:

~~~sql
LOAD LABEL test_db.label
 (
    DATA INFILE("hdfs://<hdfs_host>:<hdfs_port>/user/starrocks/data/input/file")
    INTO TABLE `test`
    COLUMNS TERMINATED BY ","
    (dt, id, user_id)
    SET (
      uv = HLL_HASH(user_id)
    )
 );
~~~

データのクエリ

* HLL列の元の値を直接クエリすることはできません。[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md) 関数を使用してください。
* 総uvを見つけるためには

`SELECT HLL_UNION_AGG(uv) FROM test;`

このステートメントは等価です

`SELECT COUNT(DISTINCT uv) FROM test;`

* 日ごとのuvをクエリする

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;`

### 注意事項

BitmapとHLLのどちらを選択すべきですか？データセットの基盤が数百万や数千万で、数十台のマシンがある場合は `count distinct`を使用します。基盤が数億であり、正確な重複排除が必要な場合は `Bitmap`を使用します。近似重複を許容する場合は `HLL` タイプを使用してください。
```
Bitmap only supports TINYINT, SMALLINT, INT, and BIGINT. Note that LARGEINT is not supported. For other types of data sets to be de-duplicated, a dictionary needs to be built to map the original type to an integer type.  Building a dictionary  is complex, and requires a trade-off between data volume, update frequency, query efficiency, storage, and other issues. HLL does not need a dictionary, but it needs the corresponding data type to support the hash function. Even in an analytical system that does not support HLL internally, it is still possible to use the hash function and SQL to implement HLL de-duplication.

For common columns, users can use the NDV function for approximate de-duplication. This function returns an approximate aggregation of COUNT(DISTINCT col) results, and the underlying implementation converts the data storage type to the HyperLogLog type for calculation. The NDV function consumes a lot of resources  when calculating and is therefore  not well suited for high concurrency scenarios.

If you want to perform user behavior analysis, you may consider IntersectCount or custom UDAF.
```