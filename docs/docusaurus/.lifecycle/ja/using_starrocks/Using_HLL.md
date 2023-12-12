---
displayed_sidebar: "Japanese"
---

# 近似カウントのためにHLLを使用する

## 背景

現実世界のシナリオでは、データの量が増加するにつれて、データの重複排除のプレッシャーが増大します。データのサイズが一定レベルに達すると、正確な重複排除のコストは比較的高くなります。この場合、ユーザーは通常、計算のプレッシャーを軽減するために近似アルゴリズムを使用します。このセクションで紹介されるHyperLogLog（HLL）は、優れた空間計算量O(mloglogn)と時間計算量O(n)を持つ近似重複排除アルゴリズムです。さらに、データセットのサイズや使用されるハッシュ関数によって、計算結果の誤差率を約1%〜10%にコントロールできます。

## HyperLogLogとは

HyperLogLogは、非常に少ないストレージスペースを消費する近似重複排除アルゴリズムです。**HLL**タイプは、HyperLogLogアルゴリズムの実装に使用されます。HyperLogLogの計算の中間結果を保持し、データテーブルのインジケーターカラムタイプとしてのみ使用できます。

HLLアルゴリズムには多くの数学的知識が関わるため、実際の例を使用して説明します。ランダム化実験Aを設計し、最初の表が出るまでコインの裏表を独立して繰り返します。最初の表が出るまでのコインの回数をランダム変数Xとして記録すると、以下のようになります：

* X=1, P(X=1)=1/2
* X=2, P(X=2)=1/4
* ...
* X=n, P(X=n)=(1/2)<sup>n</sup>

実験Aを使用してランダムテストBを構築し、テストAのN回の独立した繰り返しを行い、N個の独立同一分布のランダム変数X<sub>1</sub>, X<sub>2</sub>, X<sub>3</sub>, ..., X<sub>N</sub>を生成します。これらのランダム変数の最大値をX<sub>max</sub>とします。大きな確率推定を利用して、Nの推定値は2<sup>X<sub>max</sub></sup>となります。
<br/>

次に、与えられたデータセットにハッシュ関数を使用して上記の実験をシミュレーションします：

* テストA: データセット要素のハッシュ値を計算し、ハッシュ値をバイナリ表現に変換します。バイナリの最下位ビットから1の出現を記録します。
* テストB: テストBのデータセット要素について、テストAのプロセスを繰り返します。各テストの最初の1の出現の最大位置"m"を更新します。
* データセットの非重複要素の数をm<sup>2</sup>と推定します。

実際には、HLLアルゴリズムは、要素を下位kビットの要素ハッシュに基づいてK=2<sup>k</sup>のバケツに分割します。k+1番目のビットから最初の1の出現の最大値として数え上げ、k+1番目のビットの最初の1の出現の最大値をm<sub>1</sub>, m<sub>2</sub>,..., m<sub>k</sub>とし、バケツ内の非重複要素の数を2<sup>m<sub>1</sub></sup>, 2<sup>m<sub>2</sub></sup>,..., 2<sup>m<sub>k</sub></sup>と推定します。データセット内の非重複要素の数は、バケツの数とバケツ内の非重複要素の数の平均の積の総和として推定されます：N = K(K/(2<sup>\-m<sub>1</sub></sup>+2<sup>\-m<sub>2</sub></sup>,..., 2<sup>\-m<sub>K</sub></sup>)）。
<br/>

HLLは推定結果に補正係数を乗じて、より正確な結果を得ます。

StarRocks SQLステートメントを使用してHLL重複排除アルゴリズムを実装するための記事[https://gist.github.com/avibryant/8275649](https://gist.github.com/avibryant/8275649)を参照してください。

~~~sql
SELECT floor((0.721 * 1024 * 1024) / (sum(pow(2, m * -1)) + 1024 - count(*))) AS estimate
FROM(select(murmur_hash3_32(c2) & 1023) AS bucket,
     max((31 - CAST(log2(murmur_hash3_32(c2) & 2147483647) AS INT))) AS m
     FROM db0.table0
     GROUP BY bucket) bucket_values
~~~

このアルゴリズムは、db0.table0のcol2を重複排除します。

* ハッシュ関数`murmur_hash3_32`を使用してcol2のハッシュ値を32ビットの符号付き整数として計算します。
* 1024のバケツを使用し、補正係数は0.721であり、ハッシュ値の下位10ビットをバケットの添え字とします。
* ハッシュ値の符号ビットを無視して、次の最上位ビットから下位ビットまでの位置が最初の1の出現位置を決定します。
* バケットで計算されたハッシュ値をバケットごとにグループ化し、`MAX`集約を使用してバケット内の最初の1の出現位置の最大値を見つけます。
* 集約結果はサブクエリとして使用し、すべてのバケットの推定値の平均をバケツの数と補正係数で乗じます。
* 空のバケット数は1であることに注意してください。

上記のアルゴリズムは、データのボリュームが大きい場合に非常に低い誤差率を持ちます。

これがHLLアルゴリズムの中核的な考え方です。興味がある場合は、[HyperLogLog論文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)を参照してください。

### HyperLogLogの使用方法

1. HyperLogLog重複排除を使用するには、対象のインジケーターカラムタイプを`HLL`に設定し、テーブル作成ステートメントで集約関数を`HLL_UNION`に設定する必要があります。
2. 現在、集約モデルのみがインジケーターカラムタイプとしてHLLをサポートしています。
3. `HLL`タイプのカラムに`count distinct`を使用する場合、StarRocksは自動的に`HLL_UNION_AGG`計算に変換します。

#### 例

まず、**HLL**カラムを持つテーブルを作成します。ここで、uvは集計カラムであり、カラムタイプは`HLL`であり、集約関数は[HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)です。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> * 注意：データのボリュームが大きい場合は、高頻度のHLLクエリに対応するために対応するロールアップテーブルを作成した方が良いです。

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

ブローカーのロードモード：

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

* HLLカラムには、その元の値を直接クエリできないため、関数[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)を使用します。
* 合計uvを見つけるには、

`SELECT HLL_UNION_AGG(uv) FROM test;`

このステートメントは、次のステートメントと同等です。

`SELECT COUNT(DISTINCT uv) FROM test;`

* 毎日のuvをクエリする

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;`

### 注意事項

BitmapとHLLのどちらを選択すべきか？データセットのベースが数百万または数千万であり、数十台のマシンがある場合は、`count distinct`を使用します。ベースが数億であり、正確な重複排除が必要な場合は、`Bitmap`を使用します。おおよその重複排除が許容される場合は、`HLL`タイプを使用します。
```
Bitmap only supports TINYINT, SMALLINT, INT, and BIGINT. Note that LARGEINT is not supported. For other types of data sets to be de-duplicated, a dictionary needs to be built to map the original type to an integer type. Building a dictionary is complex, and requires a trade-off between data volume, update frequency, query efficiency, storage, and other issues. HLL does not need a dictionary, but it needs the corresponding data type to support the hash function. Even in an analytical system that does not support HLL internally, it is still possible to use the hash function and SQL to implement HLL de-duplication.

For common columns, users can use the NDV function for approximate de-duplication. This function returns an approximate aggregation of COUNT(DISTINCT col) results, and the underlying implementation converts the data storage type to the HyperLogLog type for calculation. The NDV function consumes a lot of resources when calculating and is therefore not well suited for high concurrency scenarios.

If you want to perform user behavior analysis, you may consider IntersectCount or custom UDAF.
```  
ビットマップはTINYINT、SMALLINT、INT、およびBIGINTのみをサポートしています。LARGEINTはサポートされていません。他のデータセットの場合、元のタイプを整数型にマッピングするための辞書が構築される必要があります。辞書の構築は複雑であり、データボリューム、更新頻度、クエリ効率、ストレージなどの問題とのトレードオフが必要です。 HLLには辞書は必要ありませんが、ハッシュ関数をサポートする対応するデータ型が必要です。HLLを内部でサポートしていない解析システムでも、ハッシュ関数とSQLを使用してHLLの重複除去を実装することができます。

一般の列の場合、ユーザーはNDV関数を使用しておおよその重複除去を行うことができます。この関数はおおよそのCOUNT(DISTINCT col)の結果の集計を返し、基礎となる実装ではデータストレージタイプをHyperLogLogタイプに変換して計算します。 NDV関数は計算時に多くのリソースを消費するため、高同時実行シナリオには適していません。

ユーザー行動分析を行いたい場合は、IntersectCountまたはカスタムUDAFを考慮することができます。