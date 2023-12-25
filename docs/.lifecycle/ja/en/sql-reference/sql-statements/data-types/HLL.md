---
displayed_sidebar: English
---

# HLL (HyperLogLog)

## 説明

HLLは、[近似カウントディスティンクト](../../../using_starrocks/Using_HLL.md)に使用されます。

HLLはHyperLogLogアルゴリズムに基づいたプログラムの開発を可能にします。これはHyperLogLog計算プロセスの中間結果を格納するために使用されます。テーブルの値の列タイプとしてのみ使用できます。HLLは集約を通じてデータ量を減らし、クエリプロセスを加速します。推定結果には1%の偏差が生じる可能性があります。

HLL列はインポートされたデータまたは他の列のデータに基づいて生成されます。データをインポートする際には、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md)関数がHLL列を生成するために使用する列を指定します。HLLはCOUNT DISTINCTの代わりとしてよく使用され、ロールアップでユニークビュー(UV)を迅速に計算するために使用されます。

HLLが使用するストレージスペースは、ハッシュ値の異なる値によって決まります。ストレージスペースは次の3つの条件に応じて変わります：

- HLLが空の場合。HLLに値が挿入されておらず、ストレージコストは最低で80バイトです。
- HLLの異なるハッシュ値の数が160以下の場合。最大ストレージコストは1360バイトです（80 + 160 * 8 = 1360）。
- HLLの異なるハッシュ値の数が160を超える場合。ストレージコストは一定で16,464バイトです（80 + 16 * 1024 = 16464）。

実際のビジネスシナリオでは、データ量とデータ分布がクエリのメモリ使用量と近似結果の精度に影響を与えます。これら2つの要因を考慮する必要があります：

- データ量：HLLは近似値を返します。データ量が多ければ多いほど、結果は正確になります。データ量が少なければ、偏差は大きくなります。
- データ分布：データ量が多く、カーディナリティが高い次元列でGROUP BYを行う場合、データ計算にはより多くのメモリが使用されます。このような状況ではHLLは推奨されません。no-group-byカウントディスティンクトやカーディナリティが低い次元列でのGROUP BYを行う場合に推奨されます。
- クエリの粒度：大きなクエリ粒度でデータをクエリする場合は、集約テーブルやマテリアライズドビューを使用してデータを事前に集約し、データ量を減らすことを推奨します。

## 関連関数

- [HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md): この関数は集約関数で、条件を満たす全データのカーディナリティを推定するために使用されます。関数分析にも使用できます。デフォルトウィンドウのみをサポートし、ウィンドウ句はサポートしません。

- [HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md): この関数はhll型のフィールドを集約するために使用される集約関数で、hll型で返します。

- HLL_CARDINALITY(hll): この関数は単一のhll列のカーディナリティを推定するために使用されます。

- [HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md): これはHLL列タイプを生成し、挿入やインポートに使用されます。インポートの使用方法については、こちらをご覧ください。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md): これは空のHLL列を生成し、挿入やインポート時にデフォルト値を埋めるために使用されます。インポートの使用方法については、こちらをご覧ください。

## 例

1. HLL列 `set1` と `set2` を持つテーブルを作成します。

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

3. 次の3つの方法でデータを集約します（集約なしでベーステーブルに直接クエリすると、approx_count_distinctを使用するのと同じくらい遅くなる可能性があります）：

    ```sql
    -- a. HLL列を集約するためのロールアップを作成します。
    alter table test add rollup test_rollup(dt, set1);

    -- b. UVを計算し、データを挿入するための別のテーブルを作成します。

    create table test_uv(
    dt date,
    id int,
    uv_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, set1 from test;

    -- c. UVを計算するための別のテーブルを作成します。データを挿入し、hll_hashを通じて他の列をテストしてHLL列を生成します。

    create table test_uv(
    dt date,
    id int,
    id_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, hll_hash(id) from test;
    ```

4. データをクエリします。HLL列は元の値に直接クエリすることはできませんが、適切な関数を使ってクエリすることができます。

    ```plain text
    a. 合計UVを計算します。
    select HLL_UNION_AGG(uv_set) from test_uv;

    b. 日ごとのUVを計算します。
    select dt, HLL_CARDINALITY(uv_set) from test_uv;

    c. testテーブルのset1の集約値を計算します。
    select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
    select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
    ```
