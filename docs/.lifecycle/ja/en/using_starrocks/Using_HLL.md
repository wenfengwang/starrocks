---
displayed_sidebar: English
---

# HLL を使用したおおよそのカウントディスティンクト

## 背景

実際のシナリオでは、データ量が増加するにつれて、データの重複排除のプレッシャーが高まります。データのサイズが一定レベルに達すると、正確な重複排除のコストは比較的高くなります。この場合、ユーザーは通常、計算負荷を減らすために近似アルゴリズムを使用します。このセクションで紹介する HyperLogLog（HLL）は、優れた空間計算量 O(mloglogn) と時間計算量 O(n) を持つ近似重複排除アルゴリズムです。さらに、計算結果のエラー率はデータセットのサイズと使用されるハッシュ関数に応じて約1%-10%に制御可能です。

## HyperLogLog とは何か

HyperLogLog は、非常に少ないストレージスペースを消費する近似重複排除アルゴリズムです。**HLL_type** は HyperLogLog アルゴリズムを実装するために使用されます。これは HyperLogLog 計算の中間結果を保持し、データテーブルのインジケーター列タイプとしてのみ使用されます。

HLL アルゴリズムは多くの数学的知識を含むため、実用的な例を用いて説明します。ランダムな実験 A を設計し、それは最初の表が出るまでコインを独立に投げ続けることです。最初の表のコイン投げの回数を確率変数 X として記録し、次のようになります：

* X=1、P(X=1)=1/2
* X=2、P(X=2)=1/4
* ...
* X=n、P(X=n)=(1/2)<sup>n</sup>

実験 A を用いてランダムな実験 B を構築します。これは実験 A を N 回独立して繰り返し、N 個の独立同分布の確率変数 X<sub>1</sub>、X<sub>2</sub>、X<sub>3</sub>、...、X<sub>N</sub> を生成します。確率変数の最大値を X<sub>max</sub> とし、最尤推定を利用して N の推定値は 2<sup>X<sub>max</sub></sup> になります。
<br/>

次に、与えられたデータセットに対してハッシュ関数を使用して上記の実験をシミュレートします：

* テスト A：データセット要素のハッシュ値を計算し、ハッシュ値をバイナリ表現に変換します。バイナリの最下位ビットから始まる bit=1 の出現を記録します。
* テスト B：テスト B のデータセット要素に対してテスト A のプロセスを繰り返し、各テストで最初のビット 1 の出現の最大位置 "m" を更新します。
* データセット内の非重複要素の数を m<sup>2</sup> として推定します。

実際には、HLL アルゴリズムは要素ハッシュの下位 k ビットに基づいて要素を K=2<sup>k</sup> のバケットに分割します。k+1 番目のビットから最初のビット 1 の出現の最大値を m<sub>1</sub>、m<sub>2</sub>、...、m<sub>K</sub> としてカウントし、バケット内の非重複要素の数を 2<sup>m<sub>1</sub></sup>、2<sup>m<sub>2</sub></sup>、...、2<sup>m<sub>K</sub></sup> として推定します。データセット内の非重複要素の数は、バケットの数にバケット内の非重複要素の数を掛けた平均の合計です：N = K * (K / (2<sup>-m<sub>1</sub></sup> + 2<sup>-m<sub>2</sub></sup> + ... + 2<sup>-m<sub>K</sub></sup>))。
<br/>

HLL は補正係数を推定結果に乗じて、より正確な結果を得るために使用します。

StarRocks SQL ステートメントを使用した HLL 重複排除アルゴリズムの実装に関する記事 [https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649) を参照してください。

~~~sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM (SELECT (murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
~~~

このアルゴリズムは db0.table0 の col2 を重複排除するために使用されます。

* ハッシュ関数 `murmur_hash3_32` を使用して col2 のハッシュ値を 32 ビット符号付き整数として計算します。
* 1024 バケットを使用し、補正係数は 0.721 で、ハッシュ値の下位 10 ビットをバケットのインデックスとして使用します。
* ハッシュ値の符号ビットを無視し、次に高いビットから下位ビットにかけて、最初のビット 1 の出現位置を決定します。

* 計算されたハッシュ値をバケットごとにグループ化し、`MAX` 集計関数を使用してバケット内で最初のビット1が出現する最大位置を見つけます。
* 集計結果はサブクエリとして使用され、すべてのバケット推定値の平均の合計にバケット数と補正係数を乗じます。
* 空のバケット数が1であることに注意してください。

上記のアルゴリズムは、データ量が多い場合に非常に低いエラー率を持ちます。

これがHLLアルゴリズムの核心的なアイデアです。興味があれば、[HyperLogLogの論文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)を参照してください。

### HyperLogLogの使い方

1. HyperLogLogによる重複排除を使用するには、テーブル作成ステートメントで対象の指標列のタイプを`HLL`に設定し、集計関数を`HLL_UNION`に設定する必要があります。
2. 現在、集計モデルのみが指標列タイプとしてHLLをサポートしています。
3. HLLタイプの列に`count distinct`を使用する場合、StarRocksは自動的に`HLL_UNION_AGG`計算に変換します。

#### 例

まず、uvが集計列で、列タイプが`HLL`で集計関数が[HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)であるテーブルを作成します。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(id);
~~~

> * 注: データ量が多い場合は、高頻度のHLLクエリ用に対応するロールアップテーブルを作成することが望ましいです

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用してデータをロードします：

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

Broker Loadモード：

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

データのクエリ方法

* HLL列では、その原始値を直接クエリすることはできません。関数[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)を使用してクエリします。
* 総UVを求めるには、

`SELECT HLL_UNION_AGG(uv) FROM test;`

このステートメントは以下と同等です。

`SELECT COUNT(DISTINCT uv) FROM test;`

* 毎日のUVをクエリするには、

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY dt;`

### 注意点

BitmapとHLLのどちらを選ぶべきか？データセットの基盤が数百万または数千万で、数十台のマシンを持っている場合は`count distinct`を使用します。基盤が数億に達し、正確な重複排除が必要な場合は`Bitmap`を使用し、おおよその重複排除で良い場合は`HLL`タイプを使用します。

BitmapはTINYINT、SMALLINT、INT、BIGINTのみをサポートしており、LARGEINTはサポートされていません。他のタイプのデータセットを重複排除するには、元のタイプを整数タイプにマッピングするための辞書を作成する必要があります。辞書の構築は複雑であり、データ量、更新頻度、クエリ効率、ストレージなどの問題とのトレードオフが必要です。HLLには辞書は不要ですが、ハッシュ関数をサポートする対応するデータ型が必要です。HLLを内部的にサポートしていない分析システムであっても、ハッシュ関数とSQLを使用してHLLによる重複排除を実装することが可能です。
一般的なカラムについては、ユーザーはNDV関数を使用しておおよその重複排除を行うことができます。この関数はCOUNT(DISTINCT col)の結果の近似値を返し、その基盤となる実装ではデータストレージタイプをHyperLogLogタイプに変換して計算を行います。NDV関数は計算時に多くのリソースを消費するため、高い並行性が求められるシナリオにはあまり適していません。

ユーザー行動分析を行いたい場合は、IntersectCountやカスタムUDAFを検討することをお勧めします。
