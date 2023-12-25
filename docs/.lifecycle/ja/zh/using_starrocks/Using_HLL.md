---
displayed_sidebar: Chinese
---

# HyperLogLog を使用した近似的な重複排除

この記事では、StarRocks で HLL（HyperLogLog）を使用して近似的な重複排除を実装する方法について説明します。

HLL は近似的な重複排除アルゴリズムであり、重複排除の精度要求がそれほど高くないシナリオでは、HLL アルゴリズムを使用してデータの重複排除分析の計算負荷を軽減することができます。データセットのサイズと使用されるハッシュ関数のタイプに応じて、HLL アルゴリズムの誤差は約 1% から 10% の範囲で制御できます。

## HLL 列を含むテーブルの作成

HLL を使用した重複排除では、テーブル作成ステートメントで目標指標列のタイプを **HLL** に設定し、集約関数を **HLL_UNION** に設定する必要があります。集約モデルテーブル（Aggregate Key）のみが HLL タイプの列をサポートしています。

> 説明
>
> HLL 列にデータをインポートする必要はありません。HLL 列のデータは、インポートされたデータに基づいて指定された `HLL_HASH` 関数によって自動的に生成されます。データをインポートする際、この関数は指定された列に基づいて自動的に HLL 列を生成します。HLL アルゴリズムは `count distinct` の代替としてよく使用され、物化ビューと組み合わせてビジネス上での UV の迅速な計算に使用されます。

以下の例では、DATE データタイプの列 `dt`、INT データタイプの列 `id`、および HLL タイプの列 `uv`（使用する `HLL_HASH` 関数は `HLL_UNION`）を含む `test` テーブルを作成します。

~~~sql
CREATE TABLE test(
        dt DATE,
        id INT,
        uv HLL HLL_UNION
)
DISTRIBUTED BY HASH(ID);
~~~

> 注意
>
> * 現在のバージョンでは、集約テーブルのみが HLL タイプの指標列をサポートしています。
> * データ量が多い場合は、頻繁に HLL クエリを行うための物化ビューを作成することをお勧めします。

## データのインポート

データファイル **test.csv** を作成し、以前に作成した `test` テーブルにインポートします。

現在の例では、以下の原始データを使用しており、10 行のデータのうち 3 行が重複しています。

~~~plain text
2022-03-10,0
2022-03-11,1
2022-03-12,2
2022-03-13,3
2022-03-14,4
2022-03-15,5
2022-03-16,6
2022-03-14,4
2022-03-15,5
2022-03-16,6
~~~

**test.csv** をインポートするには、Stream Load または Broker Load モードを使用できます。

* [Stream Load](../sql-reference/sql-statements/data-manipulation/STREAM_LOAD.md) モード:

~~~bash
curl --location-trusted -u <username>:<password> -H "label:987654321" -H "column_separator:," -H "columns:dt,id,uv=hll_hash(id)" -T test.csv http://fe_host:http_port/api/db_name/test/_stream_load
~~~

* [Broker Load](../sql-reference/sql-statements/data-manipulation/BROKER_LOAD.md) モード:

~~~sql
LOAD LABEL test_db.label
     (
     DATA INFILE("hdfs://hdfs_host:hdfs_port/user/starRocks/data/input/file")
     INTO TABLE `test`
     COLUMNS TERMINATED BY ","
     (dt, id, uv)
     SET (
          uv = HLL_HASH(id)
     )
     );
~~~

## HLL を使用したデータの集約

以下の三つの方法でデータを集約し、クエリを高速化できます。

* 物化ビューを作成して HLL 列を集約します。

~~~sql
ALTER TABLE test ADD ROLLUP test_rollup(dt, uv);
~~~

* HLL 列の計算用の新しいテーブルを作成し、元のサンプルテーブルから関連データを挿入します。

~~~sql
create table test_uv1(
id int,
uv_set hll hll_union)
distributed by hash(id);

insert into test_uv1 select id, uv from test;
~~~

* `HLL_HASH` を使用して元のサンプルテーブルから関連データに基づいて HLL 列を生成し、その HLL 列の計算用の新しいテーブルを作成します。

~~~sql
create table test_uv2(
id int,
uv_set hll hll_union)
distributed by hash(id);

insert into test_uv2 select id, hll_hash(id) from test;
~~~

## データのクエリ

HLL 列は原始値を直接クエリすることはできません。関数 `HLL_UNION_AGG` を使用してクエリを実行できます。

~~~sql
SELECT HLL_UNION_AGG(uv) FROM test;
~~~

以下のように返されます：

~~~plain text
+---------------------+
| hll_union_agg(`uv`) |
+---------------------+
|                   7 |
+---------------------+
~~~

HLL タイプの列で `count distinct` を使用する場合、StarRocks は自動的に [HLL_UNION_AGG(hll)](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md) に変換して計算します。したがって、上記のクエリは以下のクエリと同等です。

~~~sql
SELECT COUNT(DISTINCT uv) FROM test;
~~~

以下のように返されます：

~~~plain text
+----------------------+
| count(DISTINCT `uv`) |
+----------------------+
|                    7 |
+----------------------+
~~~

## 重複排除ソリューションの選択

データセットの基数が百万、千万のオーダーであり、数十台のマシンを持っている場合は、`count distinct` 方法を直接使用できます。データセットの基数が億を超えるレベルで、正確な重複排除が必要な場合は、[Bitmap 重複排除](./Using_bitmap.md#基于-trie-树构建全局字典)を使用する必要があります。近似的な重複排除を選択する場合は、HLL タイプの重複排除を使用できます。

Bitmap タイプは TINYINT、SMALLINT、INT、BIGINT の重複排除のみをサポートしています（LARGEINT はサポートされていないことに注意してください）。他のタイプのデータセットの重複排除には、[辞書を構築](./Using_bitmap.md#基于-trie-树构建全局字典)して元のタイプを整数タイプにマッピングする必要があります。HLL 重複排除方法では辞書の構築は不要で、対応するデータタイプがハッシュ関数をサポートしていることだけが要求されます。

通常の列については、`NDV` 関数を使用して近似的な重複排除計算を行うこともできます。`NDV` 関数の戻り値は `COUNT(DISTINCT col)` の結果の近似値の集約関数で、内部実装ではデータのストレージタイプを HLL タイプに変換して計算します。しかし、`NDV` 関数は計算時にリソースを多く消費するため、高い並行性のシナリオには適していません。

アプリケーションシナリオがユーザー行動分析である場合は、`INTERSECT_COUNT` またはカスタム UDAF による重複排除を使用することをお勧めします。

## 関連関数

* **[HLL_UNION_AGG(hll)](../sql-reference/sql-functions/aggregate-functions/hll_union_agg.md)**：この関数は集約関数で、条件を満たすすべてのデータの基数推定を計算するために使用されます。この関数は分析関数としても使用でき、デフォルトウィンドウのみをサポートし、ウィンドウ句はサポートしていません。
* **[HLL_RAW_AGG(hll)](../sql-reference/sql-functions/aggregate-functions/hll_raw_agg.md)**：この関数は集約関数で、HLL タイプのフィールドを集約し、HLL タイプを返します。
* **[HLL_CARDINALITY(hll)](../sql-reference/sql-functions/scalar-functions/hll_cardinality.md)**：この関数は、単一の HLL 列の基数を推定するために使用されます。
* **[HLL_HASH(column_name)](../sql-reference/sql-functions/aggregate-functions/hll_hash.md)**：HLL 列タイプを生成し、`insert` または HLL タイプのインポートに使用されます。
* **[HLL_EMPTY()](../sql-reference/sql-functions/aggregate-functions/hll_empty.md)**：空の HLL 列を生成し、`insert` またはデータのインポート時にデフォルト値を補うために使用されます。
