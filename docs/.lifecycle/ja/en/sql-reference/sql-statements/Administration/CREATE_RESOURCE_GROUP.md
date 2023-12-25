---
displayed_sidebar: English
---

# リソースグループの作成

## 説明

リソースグループを作成します。

詳細については、[リソースグループ](../../../administration/resource_group.md)を参照してください。

:::tip

この操作には、SYSTEMレベルのCREATE RESOURCE GROUP権限が必要です。この権限を付与するには、[GRANT](../account-management/GRANT.md)の指示に従ってください。

:::

## 構文

```SQL
CREATE RESOURCE GROUP resource_group_name 
TO CLASSIFIER1, CLASSIFIER2, ...
WITH resource_limit
```

## パラメーター

- `resource_group_name`: 作成するリソースグループの名前。

- `CLASSIFIER`: リソース制限が適用されるクエリをフィルタリングするために使用される分類子。`"key"="value"`のペアを使用して分類子を指定する必要があります。リソースグループには複数の分類子を設定できます。

  分類子のパラメーターは以下の通りです：

    | **パラメーター** | **必須** | **説明**                                              |
    | ------------- | ------------ | ------------------------------------------------------------ |
    | user          | いいえ           | ユーザーの名前。                                            |
    | role          | いいえ           | ユーザーの役割。                                            |
    | query_type    | いいえ           | クエリの種類。`SELECT`と`INSERT`（v2.5以降）がサポートされています。`insert`として`query_type`が指定されたリソースグループにINSERTタスクがヒットした場合、BEノードはタスク用に指定されたCPUリソースを予約します。   |
    | source_ip     | いいえ           | クエリが開始されるCIDRブロック。            |
    | db            | いいえ           | クエリがアクセスするデータベース。カンマで区切られた文字列で指定できます。 |
    | plan_cpu_cost_range | いいえ     | クエリの推定CPUコスト範囲。形式は`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、制限がないことを示します。このパラメーターはv3.1.4以降でサポートされています。                  |
    | plan_mem_cost_range | いいえ     | クエリの推定メモリコスト範囲。形式は`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、制限がないことを示します。このパラメーターはv3.1.4以降でサポートされています。               |

- `resource_limit`: リソースグループに適用されるリソース制限。リソース制限は`"key"="value"`のペアを使用して指定する必要があります。リソースグループには複数のリソース制限を設定できます。

  リソース制限のパラメーターは以下の通りです：

    | **パラメーター**              | **必須** | **説明**                                              |
    | -------------------------- | ------------ | ------------------------------------------------------------ |
    | cpu_core_limit             | いいえ           | BE上でリソースグループに割り当てることができるCPUコア数のソフトリミット。実際のビジネスシナリオでは、リソースグループに割り当てられるCPUコアは、BE上のCPUコアの可用性に基づいて比例的にスケールします。有効な値：0以外の正の整数。 |
    | mem_limit                  | いいえ           | BEが提供する合計メモリの中でクエリが使用できるメモリの割合。単位：%。有効な値：(0, 1)。 |
    | concurrency_limit          | いいえ           | リソースグループ内の同時クエリの上限。多数の同時クエリによるシステムの過負荷を避けるために使用されます。 |
    | max_cpu_cores              | いいえ           | 単一BEノード上でこのリソースグループのCPUコアの上限。`0`より大きい値に設定された場合のみ有効です。範囲：[0, `avg_be_cpu_cores`]、ここで`avg_be_cpu_cores`は全BEノードの平均CPUコア数を表します。デフォルトは0です。 |
    | big_query_cpu_second_limit | いいえ           | 大規模クエリのCPU使用時間の上限。同時クエリは時間を累計します。単位は秒。 |
    | big_query_scan_rows_limit  | いいえ           | 大規模クエリでスキャンできる行数の上限。 |
    | big_query_mem_limit        | いいえ           | 大規模クエリのメモリ使用量の上限。単位はバイト。 |
    | type                       | いいえ           | リソースグループのタイプ。有効な値：<br />`short_query`：`short_query`リソースグループのクエリが実行されている場合、BEノードは`short_query.cpu_core_limit`で定義されたCPUコアを予約します。`normal`リソースグループのCPUコアは、「合計CPUコア - `short_query.cpu_core_limit`」に制限されます。<br />`normal`：`short_query`リソースグループのクエリが実行されていない場合、上記のCPUコア制限は`normal`リソースグループには適用されません。<br />注意：クラスターには`short_query`タイプのリソースグループを1つだけ作成できます。|

## 例

例1：複数の分類子に基づいてリソースグループ`rg1`を作成します。

```SQL
CREATE RESOURCE GROUP rg1
TO 
    (user='rg1_user1', role='rg1_role1', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user2', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user3', source_ip='192.168.x.x/24'),
    (user='rg1_user4'),
    (db='db1')
WITH ('cpu_core_limit' = '10',
      'mem_limit' = '20%',
      'type' = 'normal',
      'big_query_cpu_second_limit' = '100',
      'big_query_scan_rows_limit' = '100000',
      'big_query_mem_limit' = '1073741824'
);
```
