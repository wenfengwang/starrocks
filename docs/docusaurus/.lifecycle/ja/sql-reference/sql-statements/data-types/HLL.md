---
displayed_sidebar: "Japanese"
---

# HLL（HyperLogLog）

## 説明

HLLは[近似データ数のカウント](../../../using_starrocks/Using_HLL.md)に使用されます。

HLLは、HyperLogLogアルゴリズムに基づいたプログラムの開発を可能にします。これはHyperLogLog計算プロセスの中間結果を格納するために使用されます。これはテーブルの値列タイプとしてのみ使用できます。HLLは集計を通じてデータ量を減らし、クエリ処理を高速化します。推定結果には1%のばらつきが生じる可能性があります。

HLL列は、インポートされたデータまたは他の列から生成されます。データをインポートする際、[hll_hash](../../sql-functions/aggregate-functions/hll_hash.md) 関数はHLL列を生成するために使用される列を指定します。HLLは、COUNT DISTINCTを置換し、ロールアップでユニークビュー（UV）を迅速に計算する際によく使用されます。

HLLの使用するストレージスペースは、ハッシュ値の一意な値の数によって決まります。ストレージスペースの使用は以下の3つの条件によって異なります：

- HLLは空である場合。HLLに値が挿入されていないため、ストレージコストが最も低い80バイトです。
- HLL内の一意なハッシュ値の数が160以下である場合。最も高いストレージコストは1360バイトです（80 + 160 * 8 = 1360）。
- HLL内の一意なハッシュ値の数が160より大きい場合。ストレージコストは16,464バイトに固定されます（80 + 16 * 1024 = 16464）。

実際のビジネスシナリオにおいては、データ量とデータ分布がクエリのメモリ使用量や近似結果の精度に影響を与えます。以下の2つの要因を考慮する必要があります：

- データ量：HLLは近似値を返します。データ量が大きいほど、より正確な結果が得られます。データ量が少ないほど、ばらつきが大きくなります。
- データ分布：データ量が大きく、高基数の次元列をGROUP BYする場合、データ計算にはより多くのメモリが使用されます。このような状況ではHLLはお勧めできません。低基数の次元列でのノングループバイCOUNT DISTINCTやGROUP BY時にはお勧めします。
- クエリの粒度：大きなクエリ粒度でデータをクエリする場合は、事前にデータを集計してデータ量を減らすために、集約テーブルまたはマテリアライズドビューを使用することをお勧めします。

## 関連する関数

- [HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md): この関数は条件を満たすすべてのデータの基数を推定するために使用される集約関数です。これはまた、分析関数としても使用できます。デフォルトウィンドウのみサポートし、window句はサポートしません。

- [HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md): この関数はhll型のフィールドを集約し、hll型で返します。

- HLL_CARDINALITY(hll): この関数は単一のHLL列の基数を推定するために使用されます。

- [HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md): これはHLL列タイプを生成し、挿入またはインポートに使用されます。インポートの使用方法についての手順については、説明を参照してください。

- [HLL_EMPTY](../../sql-functions/aggregate-functions/hll_empty.md): これは空のHLL列を生成し、挿入またはインポート中にデフォルト値を埋めるために使用されます。インポートの使用方法についての手順については、説明を参照してください。

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

2. [ストリームロード](../../../loading/StreamLoad.md)を使用してデータをロードします。

    ```plain text
    a. 表の列を使用してHLL列を生成します。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
    http://host/api/test_db/test/_stream_load

    b. データ列を使用してHLL列を生成します。
    curl --location-trusted -uname:password -T data -H "label:load_1" \
        -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
    http://host/api/test_db/test/_stream_load
    ```

3. 次の3つの方法でデータを集計します：（集計なしでベーステーブルを直接クエリすると、approx_count_distinctの使用と同じくらい遅くなります）

    ```sql
    -- a. ロールアップを作成してHLL列を集計します。
    alter table test add rollup test_rollup(dt, set1);

    -- b. uvを計算する別のテーブルを作成し、データを挿入します

    create table test_uv(
    dt date,
    id int
    uv_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, set1;

    -- c. UVを計算する別のテーブルを作成し、データを挿入し、他の列をテストしてhll_hashを使用してHLL列を生成します。

    create table test_uv(
    dt date,
    id int,
    id_set hll hll_union)
    distributed by hash(id);

    insert into test_uv select dt, id, hll_hash(id) from test;
    ```

4. データをクエリします。HLL列はその元の値を直接クエリサポートしていません。一致する関数によってクエリできます。

    ```plain text
    a. 総UVを計算します。
    select HLL_UNION_AGG(uv_set) from test_uv;

    b. 各日のUVを計算します。
    select dt, HLL_CARDINALITY(uv_set) from test_uv;

    c. testテーブルのset1の集計値を計算します。
    select dt, HLL_CARDINALITY(uv) from (select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
    select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
    ```