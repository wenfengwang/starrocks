---
displayed_sidebar: "Japanese"
---

# 近似カウントのためのHLLの使用

## 背景

実際のシナリオでは、データのボリュームが増加するにつれて、データの重複排除の圧力も増加します。データのサイズが一定のレベルに達すると、正確な重複排除のコストは比較的高くなります。この場合、ユーザーは通常、計算負荷を軽減するために近似アルゴリズムを使用します。このセクションでは、このような場合に導入される近似重複排除アルゴリズムであるHyperLogLog（HLL）について説明します。HLLは、優れた空間複雑性O(mloglogn)と時間複雑性O(n)を持つ近似重複排除アルゴリズムであり、データセットのサイズとハッシュ関数の使用に応じて、計算結果の誤差率を約1％〜10％に制御することができます。

## HyperLogLogとは

HyperLogLogは、非常に少ないストレージスペースを消費する近似重複排除アルゴリズムです。HyperLogLogアルゴリズムの実装には、**HLLタイプ**が使用されます。これは、HyperLogLog計算の中間結果を保持し、データテーブルの指標列タイプとしてのみ使用できます。

HLLアルゴリズムは、多くの数学的な知識を必要とするため、実際の例を使用して説明します。まず、ランダム化された実験Aを設計します。これは、最初のヘッドが出るまでコインを独立して反転し続けるものです。最初のヘッドのコイン反転回数をランダム変数Xとして記録します。すると、次のようになります。

* X=1, P(X=1)=1/2
* X=2, P(X=2)=1/4
* ...
* X=n, P(X=n)=(1/2)<sup>n</sup>

次に、テストAを使用して、テストAのデータセット要素をN回独立して繰り返し、N個の独立した同一分布のランダム変数X<sub>1</sub>, X<sub>2</sub>, X<sub>3</sub>, ..., X<sub>N</sub>を生成します。ランダム変数の最大値をX<sub>max</sub>とします。最尤推定を利用して、Nの推定値は2<sup>X<sub>max</sub></sup>です。
<br/>

次に、与えられたデータセット上で上記の実験をハッシュ関数を使用してシミュレートします。

* テストA: データセット要素のハッシュ値を計算し、ハッシュ値をバイナリ表現に変換します。バイナリの最下位ビットから1の出現回数を記録します。
* テストB: テストBのデータセット要素に対してテストAのプロセスを繰り返します。各テストの最初のビット1の出現の最大位置"m"を更新します。
* データセット内の非重複要素の数をm<sup>2</sup>として推定します。

実際には、HLLアルゴリズムは、要素を下位kビットに基づいてK=2<sup>k</sup>のバケットに分割します。k+1番目のビットから最初のビット1の出現の最大値をm<sub>1</sub>, m<sub>2</sub>,..., m<sub>k</sub>としてカウントし、バケット内の非重複要素の数を2<sup>m<sub>1</sub></sup>, 2<sup>m<sub>2</sub></sup>,..., 2<sup>m<sub>k</sub></sup>として推定します。データセット内の非重複要素の数は、バケットの数とバケット内の非重複要素の数をかけたものの合計平均です: N = K(K/(2<sup>\-m<sub>1</sub></sup>+2<sup>\-m<sub>2</sub></sup>,..., 2<sup>\-m<sub>K</sub></sup>)）。
<br/>

HLLは、推定結果に補正係数を乗算して、結果をより正確にします。

HLLの重要なアルゴリズムのアイデアです。興味がある場合は、[HyperLogLog論文](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)を参照してください。

### HyperLogLogの使用方法

1. HyperLogLogの重複排除を使用するには、テーブル作成文で対象の指標列タイプを`HLL`に設定し、集計関数を`HLL_UNION`に設定する必要があります。
2. 現在、集計モデルのみがHLLを指標列タイプとしてサポートしています。
3. HLLタイプの列に対して`count distinct`を使用する場合、StarRocksは自動的に`HLL_UNION_AGG`の計算に変換します。

#### 例

まず、**HLL**列を持つテーブルを作成します。ここで、uvは集計列であり、列タイプは`HLL`で、集計関数は[HLL_UNION](../sql-reference/sql-functions/aggregate-functions/hll_union.md)です。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> * 注意: データのボリュームが大きい場合は、高頻度のHLLクエリに対して対応するロールアップテーブルを作成することをお勧めします。

[Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md)を使用してデータをロードします。

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

ブローカーロードモード:

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

* HLL列は、その元の値を直接クエリすることはできません。代わりに、関数[HLL_UNION_AGG](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)を使用してクエリします。
* 総uvを見つけるには、

`SELECT HLL_UNION_AGG(uv) FROM test;`

この文は次と同等です。

`SELECT COUNT(DISTINCT uv) FROM test;`

* 各日のuvをクエリする

`SELECT COUNT(DISTINCT uv) FROM test GROUP BY ID;`

### 注意事項

BitmapとHLLのどちらを選択すべきですか？データセットのベースが数百万または数千万で、数十台のマシンがある場合は、`count distinct`を使用します。ベースが数億で正確な重複排除が必要な場合は、`Bitmap`を使用します。近似重複排除が許容される場合は、`HLL`タイプを使用します。

BitmapはTINYINT、SMALLINT、INT、BIGINTのみをサポートしています。LARGEINTはサポートされていません。他のタイプのデータセットを重複排除する場合は、元のタイプを整数型にマッピングするための辞書を構築する必要があります。辞書の構築は複雑であり、データのボリューム、更新頻度、クエリの効率、ストレージなどの問題の間でトレードオフをする必要があります。HLLには辞書が必要ありませんが、対応するデータ型がハッシュ関数をサポートしている必要があります。HLLを内部的にサポートしていない解析システムでも、ハッシュ関数とSQLを使用してHLLの重複排除を実装することができます。

一般的な列に対しては、近似重複排除のためにNDV関数を使用できます。この関数は、COUNT(DISTINCT col)の結果の近似集計を返し、基礎となる実装ではデータのストレージタイプをHyperLogLogタイプに変換して計算します。NDV関数は計算時に多くのリソースを消費するため、高並行性のシナリオには適していません。

ユーザーの行動分析を行う場合は、IntersectCountまたはカスタムUDAFを検討することがあります。
