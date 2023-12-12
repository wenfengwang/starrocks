---
displayed_sidebar: "Japanese" 
---

# HLL (HyperLogLog)

## 説明

HLLは[近似重複カウント]（../../../using_starrocks/Using_HLL.md）に使用されます。

HLLは、HyperLogLogアルゴリズムに基づいたプログラムの開発を可能にします。これは、HyperLogLog計算プロセスの中間結果を格納するために使用されます。テーブルの値列タイプとしてのみ使用できます。 HLLは、集計を通じてデータ量を削減し、クエリプロセスを高速化します。推定結果には1％の偏差があります。

HLL列は、インポートされたデータまたは他の列から生成されます。データがインポートされると、[hll_hash]（../../sql-functions/aggregate-functions/hll_hash.md）関数は、どの列を使用してHLL列を生成するかを指定します。 HLLは、COUNT DISTINCTを置き換え、ロールアップでユニークビュー（UV）を迅速に計算するためによく使用されます。

HLLの使用される保存スペースは、ハッシュ値の重複しない値によって決まります。保存スペースは次の3つの条件によって異なります。

- HLLが空の場合。HLLに値が挿入されず、保存コストは最も低い80バイトです。
- HLL内の異なるハッシュ値の数が160以下の場合。最高の保存コストは1360バイトです（80 + 160 * 8 = 1360）。
- HLL内の異なるハッシュ値の数が160を超える場合。保存コストは16464バイト（80 + 16 * 1024 = 16464）で固定されます。

実際のビジネスシナリオでは、データのボリュームとデータの分布がクエリのメモリ使用量と近似結果の精度に影響します。これらの2つの要因を考慮する必要があります。

- データボリューム：HLLは近似値を返します。データのボリュームが大きいほど、より正確な結果が得られます。データのボリュームが小さいほど、偏差が大きくなります。
- データの分布：データボリュームと高基数の次元列のGROUP BYの場合、データの計算にはより多くのメモリが使用されます。このような状況ではHLLは推奨されません。低基数の次元列でのノングループバイカウント重複またはGROUP BYの場合に推奨されます。
- クエリの粒度：大規模なクエリのデータを問い合わせる場合、事前にデータを集計するために集計テーブルまたはマテリアライズドビューを使用することをお勧めします。

## 関連する関数

- [HLL_UNION_AGG(hll)]（../../sql-functions/aggregate-functions/hll_union_agg.md）：この関数は、条件を満たすすべてのデータの基数を推定するための集計関数です。これはまた分析機能にも使用することができます。デフォルトウィンドウのみをサポートし、ウィンドウ句をサポートしません。

- [HLL_RAW_AGG(hll)]（../../sql-functions/aggregate-functions/hll_raw_agg.md）：この関数は、hllタイプのフィールドを集約し、hllタイプで返します。

- HLL_CARDINALITY(hll)：この関数は、単一のHLL列の基数を推定するために使用されます。

- [HLL_HASH(column_name)]（../../sql-functions/aggregate-functions/hll_hash.md）：これはHLL列タイプを生成し、挿入またはインポートに使用されます。インポートの使用方法については、指示を参照してください。

- [HLL_EMPTY]（../../sql-functions/aggregate-functions/hll_empty.md）：これは空のHLL列を生成し、挿入またはインポート時にデフォルト値を埋めるために使用されます。インポートの使用方法については、指示を参照してください。

## 例

1. HLL列`set1`と`set2`を持つテーブルを作成します。

    ```sql
    create table test(
    dt date,
    id int,
    name char(10),
    province char(10),
    os char(1),
    set1 hll hll_union,
    set2 hll hll_union)
    distributed by hash(id);
    ```

2. [ストリームロード](../../../loading/StreamLoad.md)を使用してデータをロードします。

    ```plain text
    a. テーブルの列を使用してHLL列を生成します。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
    http://host/api/test_db/test/_stream_load

    b. データの列を使用してHLL列を生成します。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
    http://host/api/test_db/test/_stream_load
    ```

3. 次の3つの方法でデータを集計します：（集計を行わないと、ベーステーブルでの直接クエリは、approx_count_distinctの使用と同じくらい遅くなる可能性があります）

    ```sql
    -- a. ロールアップを作成してHLL列を集計します。
    alter table test add rollup test_rollup(dt, set1);

    -- b. uvを計算する別のテーブルを作成し、データを挿入します

    create table test_uv(
    dt date,
    id int
    uv_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, set1 from test;

    -- c. UVを計算する別のテーブルを作成し、データを挿入し、他の列をテストしてhll_hashを介してHLL列を生成します。

    create table test_uv(
    dt date,
    id int,
    id_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, hll_hash(id) from test;
    ```

4. データをクエリします。HLL列はそれ自体の値を直接クエリをサポートしていません。マッチング関数によってクエリできます。

    ```plain text
    a. 合計UVを計算します。
    select HLL_UNION_AGG(uv_set) from test_uv;

    b. 各日のUVを計算します。
    select dt, HLL_CARDINALITY(uv_set) from test_uv;

    c. testテーブルのset1の集計値を計算します。
    select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
    select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
    ```