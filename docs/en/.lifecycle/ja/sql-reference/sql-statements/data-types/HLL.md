---
displayed_sidebar: "Japanese"
---

# HLL (HyperLogLog)

## 説明

HLLは[近似的な重複カウント](../../../using_starrocks/Using_HLL.md)に使用されます。

HLLはHyperLogLogアルゴリズムに基づいたプログラムの開発を可能にします。HLLはHyperLogLog計算プロセスの中間結果を格納するために使用されます。テーブルの値列タイプとしてのみ使用できます。HLLはデータの集約によってデータ量を減らし、クエリプロセスを高速化します。推定結果には1%の偏差がある場合があります。

HLL列は、インポートされたデータまたは他の列のデータに基づいて生成されます。データのインポート時には、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)関数を使用して、HLL列の生成に使用する列を指定します。HLLは、COUNT DISTINCTの代わりに使用され、ロールアップを使用して一意のビュー（UV）を迅速に計算するためによく使用されます。

HLLの使用するストレージスペースは、ハッシュ値の一意の値によって決まります。ストレージスペースは次の3つの条件によって異なります：

- HLLが空の場合。HLLに値が挿入されていない場合、ストレージコストは最も低い80バイトです。
- HLL内の一意のハッシュ値の数が160以下の場合。最も高いストレージコストは1360バイトです（80 + 160 * 8 = 1360）。
- HLL内の一意のハッシュ値の数が160より大きい場合。ストレージコストは固定の16464バイトです（80 + 16 * 1024 = 16464）。

実際のビジネスシナリオでは、データのボリュームとデータの分布がクエリのメモリ使用量と近似結果の精度に影響を与えます。これらの2つの要素を考慮する必要があります：

- データのボリューム：HLLは近似値を返します。データのボリュームが大きいほど、より正確な結果が得られます。データのボリュームが小さいほど、偏差が大きくなります。
- データの分布：大きなデータボリュームと高基数の次元列を持つGROUP BYの場合、データの計算にはより多くのメモリが使用されます。このような場合にはHLLは推奨されません。低基数の次元列でのno-group-by count distinctまたはGROUP BYを実行する場合に推奨されます。
- クエリの粒度：大きなクエリの粒度でデータをクエリする場合、データのボリュームを減らすために集計テーブルまたはマテリアライズドビューを使用することをお勧めします。

## 関連する関数

- [HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md): この関数は、条件を満たすすべてのデータの基数を推定するための集約関数です。また、関数の解析にも使用できます。デフォルトのウィンドウのみをサポートし、ウィンドウ句はサポートしません。

- [HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md): この関数は、hll型のフィールドを集約し、hll型で返します。

- HLL_CARDINALITY(hll): この関数は、単一のhll列の基数を推定するために使用されます。

- [HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md): これはHLL列タイプを生成し、挿入またはインポートに使用されます。インポートの使用方法については、指示を参照してください。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md): これは空のHLL列を生成し、挿入またはインポート時のデフォルト値の埋め込みに使用されます。インポートの使用方法については、指示を参照してください。

## 例

1. `set1`と`set2`のHLL列を持つテーブルを作成します。

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

2. [Stream Load](../../../loading/StreamLoad.md)を使用してデータをロードします。

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

3. 次の3つの方法でデータを集計します：（集計なしでベーステーブルを直接クエリすると、approx_count_distinctを使用する場合と同じくらい遅くなる場合があります）

    ```sql
    -- a. ロールアップを作成してHLL列を集計します。
    alter table test add rollup test_rollup(dt, set1);

    -- b. uvを計算するための別のテーブルを作成し、データを挿入します

    create table test_uv(
    dt date,
    id int
    uv_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, set1 from test;

    -- c. UVを計算するための別のテーブルを作成し、他の列をテストしてhll_hashを介してデータを挿入し、HLL列を生成します。

    create table test_uv(
    dt date,
    id int,
    id_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, hll_hash(id) from test;
    ```

4. データをクエリします。HLL列は元の値に直接クエリをサポートしていません。関数の一致によってクエリできます。

    ```plain text
    a. 合計UVを計算します。
    select HLL_UNION_AGG(uv_set) from test_uv;

    b. 各日のUVを計算します。
    select dt, HLL_CARDINALITY(uv_set) from test_uv;

    c. testテーブルのset1の集計値を計算します。
    select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
    select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
    ```
