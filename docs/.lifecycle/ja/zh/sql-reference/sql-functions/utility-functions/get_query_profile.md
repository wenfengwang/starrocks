---
displayed_sidebar: Chinese
---

# get_query_profile

## 機能

`query_id` を使用して特定のクエリの Profile を取得します。`query_id` が存在しないか正確でない場合は、空を返します。

Query Profile を取得するには、Profile 機能を有効にする必要があります。つまり、セッション変数 `set enable_profile = true;` を設定します。有効にしていない場合、この関数は空を返します。

この関数はバージョン 3.0 からサポートされています。

## 文法

```Haskell
get_query_profile(x)
```

## パラメータ説明

`x`: `query_id` の文字列。サポートされるデータ型は VARCHAR です。

## 戻り値の説明

Query Profile には通常、以下のフィールドが含まれます。詳細は [Query Profile の分析を見る](../../../administration/query_profile.md) を参照してください。

```SQL
Query:
  Summary:
  Planner:
  Execution Profile 7de16a85-761c-11ed-917d-00163e14d435:
    Fragment 0:
      Pipeline (id=2):
        EXCHANGE_SINK (plan_node_id=18):
        LOCAL_MERGE_SOURCE (plan_node_id=17):
      Pipeline (id=1):
        LOCAL_SORT_SINK (plan_node_id=17):
        AGGREGATE_BLOCKING_SOURCE (plan_node_id=16):
      Pipeline (id=0):
        AGGREGATE_BLOCKING_SINK (plan_node_id=16):
        EXCHANGE_SOURCE (plan_node_id=15):
    Fragment 1:
       ...
    Fragment 2:
       ...
```

## 例

```sql
-- profile 機能を有効にする。
set enable_profile = true;

-- 単純なクエリを実行します。
select 1;

-- 最近のクエリの query_id を取得します。
select last_query_id();
+--------------------------------------+
| last_query_id()                      |
+--------------------------------------+
| 502f3c04-8f5c-11ee-a41f-b22a2c00f66b |
+--------------------------------------+

-- そのクエリの Profile を取得します。内容が長いため、ここには表示しません。
select get_query_profile('502f3c04-8f5c-11ee-a41f-b22a2c00f66b');

-- regexp_extract 関数を使用して、Query Profile から特定のパターンに一致する QueryPeakMemoryUsage を取得します。
select regexp_extract(get_query_profile('502f3c04-8f5c-11ee-a41f-b22a2c00f66b'), 'QueryPeakMemoryUsage: [0-9\.]* [KMGB]*', 0);
+-----------------------------------------------------------------------------------------------------------------------+
| regexp_extract(get_query_profile('bd3335ce-8dde-11ee-92e4-3269eb8da7d1'), 'QueryPeakMemoryUsage: [0-9.]* [KMGB]*', 0) |
+-----------------------------------------------------------------------------------------------------------------------+
| QueryPeakMemoryUsage: 3.828 KB                                                                                        |
+-----------------------------------------------------------------------------------------------------------------------+
```

## 関連関数

- [last_query_id](./last_query_id.md)
- [regexp_extract](../like-predicate-functions/regexp_extract.md)
