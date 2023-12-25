---
displayed_sidebar: Chinese
---

# HLL (HyperLogLog)

## 説明

HyperLogLog型は、近似的な重複排除に使用されます。詳しい使用方法は [HyperLogLogを使用した近似重複排除](../../../using_starrocks/Using_HLL.md) を参照してください。

HLLはHyperLogLogアルゴリズムに基づいた実装で、HyperLogLog計算の中間結果を保存するために使用されます。HLL型の列はテーブルのvalue列としてのみ使用でき、集約を通じてデータ量を減らし、クエリの速度を向上させる目的で使用されます。HLLに基づく結果は推定値であり、誤差は約1%です。

HLL列は他の列やインポートされたデータから生成され、インポート時には`HLL_HASH`関数を使用してどの列をHLL列の生成に使用するかを指定します。これは`count distinct`の代わりとしてよく使用され、rollupを組み合わせてビジネス上でUVの迅速な計算に使用されます。

HLL型の使用するストレージ容量は、HLLに挿入されたハッシュ値の重複なしの数に依存します。3つのシナリオが考慮されます:

- HLLが空: どの値も挿入されていない場合、HLLのストレージコストは最小で、80バイトを使用します。
- HLL内の重複なしハッシュ値の数が160以下の場合
  最大コストは: 80 + 160 * 8 = 1360バイト。
- HLL内の重複なしハッシュ値の数が160を超える場合
  ストレージコストは固定で: 80 + 16 * 1024 = 16464バイト。

実際の使用では、データ量とデータの分布がクエリのメモリ使用量と近似結果の精度に影響を与えるため、以下を考慮する必要があります:

- データ量: HLLは統計推定アルゴリズムであるため、データ量が多いほど誤差は小さくなり、データ量が少ないと誤差が大きくなります。
- データ分布: データ量が多い場合、group byの次元列の基数が高いと、より多くのメモリを使用し、HLLによる重複排除は推奨されません。no-group-byでの重複排除や、group byの次元列の基数が低い場合は、HLLによる重複排除が推奨されます。
- ユーザーが粗い粒度でクエリを実行する場合: できるだけ集約モデルやマテリアライズドビューを使用してデータを事前に集約し、rollupを行ってデータの規模を減らすことが最善です。

## 関連関数

**[HLL_UNION_AGG(hll)](../../sql-functions/aggregate-functions/hll_union_agg.md)**：この関数は集約関数で、条件に合致するすべてのデータの基数推定を計算するために使用されます。この関数は分析関数としても使用でき、デフォルトウィンドウのみをサポートし、window句はサポートしていません。

**[HLL_RAW_AGG(hll)](../../sql-functions/aggregate-functions/hll_raw_agg.md)**：この関数は集約関数で、hll型のフィールドを集約し、戻り値もhll型です。

**[HLL_CARDINALITY(hll)](../../sql-functions/scalar-functions/hll_cardinality.md)**：この関数は、単一のhll列の基数推定を計算するために使用されます。

**[HLL_HASH(column_name)](../../sql-functions/aggregate-functions/hll_hash.md)**：HLL列型を生成するために使用され、insertやインポート時に使用されます。インポート時の使用については関連説明を参照してください。

**[HLL_EMPTY()](../../sql-functions/aggregate-functions/hll_empty.md)**：空のHLL列を生成するために使用され、insertやインポート時にデフォルト値を補うために使用されます。インポート時の使用については関連説明を参照してください。

## 例

### HLL列を含むテーブルの作成

```sql
create table test(
    dt date,
    id int,
    name char(10),
    province char(10),
    os char(1),
    set1 hll hll_union,
    set2 hll hll_union
)
distributed by hash(id);
```

### [Stream load](../data-manipulation/STREAM_LOAD.md)を通じてデータをインポート

```bash
a. テーブルの列を使用してHLL列を生成します。
curl --location-trusted -uname:password -T data -H "label:load_1" \
    -H "columns:dt, id, name, province, os, set1=hll_hash(id), set2=hll_hash(name)"
http://host/api/test_db/test/_stream_load

b. データの特定の列を使用してHLL列を生成します
curl --location-trusted -uname:password -T data -H "label:load_1" \
    -H "columns:dt, id, name, province, sex, cuid, os, set1=hll_hash(cuid), set2=hll_hash(os)"
http://host/api/test_db/test/_stream_load
```

### HLLを使用してデータを集約

データを集約するには3つの一般的な方法があります：

```sql
-- 最初の例で作成されたHLLテーブルにrollupを追加し、UV列を集約します。
alter table test add rollup test_rollup(dt, set1);


-- UVを計算するための別のテーブルを作成し、データをINSERTします。
create table test_uv(
    dt date,
    id int,
    uv_set hll hll_union
)
distributed by hash(id);

insert into test_uv select dt, id, set1 from test;


-- UVを計算するための別のテーブルを作成し、testの他の非HLL列を使用してhll_hashによりHLLを生成し、データをINSERTします。
create table test_uv(
    dt date,
    id int,
    id_set hll hll_union
)
distributed by hash(id);

insert into test_uv select dt, id, hll_hash(id) from test;
```

### HLL列のクエリ

HLL列の原始値を直接クエリすることはできませんが、対応する関数を使用してクエリすることができます。

```sql
-- 総UVを求めます。
select HLL_UNION_AGG(uv_set) from test_uv;

-- 日ごとのUVを求めます。
select dt, HLL_CARDINALITY(uv_set) from test_uv;

-- testテーブルのset1の集約値を求めます。
select dt, HLL_CARDINALITY(uv) from (
    select dt, HLL_RAW_AGG(set1) as uv from test group by dt) tmp;
select dt, HLL_UNION_AGG(set1) as uv from test group by dt;
```
