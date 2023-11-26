---
displayed_sidebar: "Japanese"
---

# リソースグループの作成

## 説明

リソースグループを作成します。

詳細については、[リソースグループ](../../../administration/resource_group.md)を参照してください。

## 構文

```SQL
CREATE RESOURCE GROUP resource_group_name 
TO CLASSIFIER1, CLASSIFIER2, ...
WITH resource_limit
```

## パラメータ

- `resource_group_name`: 作成するリソースグループの名前です。

- `CLASSIFIER`: リソース制限が適用されるクエリをフィルタリングするために使用される分類子です。`"key"="value"`のペアを使用して分類子を指定する必要があります。リソースグループに複数の分類子を設定することができます。

  分類子のパラメータは以下の通りです：

    | **パラメータ** | **必須** | **説明**                                              |
    | ------------- | ------------ | ------------------------------------------------------------ |
    | user          | いいえ           | ユーザーの名前です。                                            |
    | role          | いいえ           | ユーザーの役割です。                                            |
    | query_type    | いいえ           | クエリのタイプです。`SELECT`および`INSERT`（v2.5以降）がサポートされています。`query_type`が`insert`であるリソースグループにINSERTタスクがヒットすると、BEノードは指定されたCPUリソースをタスクに対して予約します。   |
    | source_ip     | いいえ           | クエリが開始されるCIDRブロックです。            |
    | db            | いいえ           | クエリがアクセスするデータベースです。カンマ（,）で区切られた文字列で指定することができます。 |
    | plan_cpu_cost_range | いいえ     | クエリの推定CPUコストの範囲です。フォーマットは`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、この制限はありません。このパラメータはv3.1.4以降でサポートされています。                  |
    | plan_mem_cost_range | いいえ     | クエリの推定メモリコストの範囲です。フォーマットは`(DOUBLE, DOUBLE]`です。デフォルト値はNULLで、この制限はありません。このパラメータはv3.1.4以降でサポートされています。               |

- `resource_limit`: リソースグループに課せられるリソース制限です。`"key"="value"`のペアを使用してリソース制限を指定する必要があります。リソースグループに複数のリソース制限を設定することができます。

  リソース制限のパラメータは以下の通りです：

    | **パラメータ**              | **必須** | **説明**                                              |
    | -------------------------- | ------------ | ------------------------------------------------------------ |
    | cpu_core_limit             | いいえ           | BE上のリソースグループに割り当てることができるCPUコアの数のソフト制限です。実際のビジネスシナリオでは、リソースグループに割り当てられるCPUコアは、BE上のCPUコアの利用可能性に比例してスケーリングされます。有効な値：ゼロ以外の正の整数。 |
    | mem_limit                  | いいえ           | BEが提供する総メモリのうち、クエリに使用できるメモリの割合です。単位：％。有効な値：（0、1）。 |
    | concurrency_limit          | いいえ           | リソースグループ内の同時クエリの上限です。システムの過負荷を避けるために使用されます。 |
    | max_cpu_cores              | いいえ           | 単一のBEノード上のこのリソースグループのCPUコア制限です。`0`より大きい場合にのみ効果があります。範囲：[0、`avg_be_cpu_cores`]、`avg_be_cpu_cores`はすべてのBEノードの平均CPUコア数を表します。デフォルト：0。 |
    | big_query_cpu_second_limit | いいえ           | ビッグクエリのCPU占有時間の上限です。同時クエリは時間を加算します。単位は秒です。 |
    | big_query_scan_rows_limit  | いいえ           | ビッグクエリによってスキャンできる行数の上限です。 |
    | big_query_mem_limit        | いいえ           | ビッグクエリのメモリ使用量の上限です。単位はバイトです。 |
    | type                       | いいえ           | リソースグループのタイプです。有効な値：<br />`short_query`：`short_query`リソースグループからのクエリが実行されている場合、BEノードは`short_query.cpu_core_limit`で定義されたCPUコアを予約します。すべての`normal`リソースグループのCPUコアは、「総CPUコア数 - `short_query.cpu_core_limit`」に制限されます。<br />`normal`：`short_query`リソースグループからのクエリが実行されていない場合、上記のCPUコア制限は`normal`リソースグループには適用されません。<br />なお、クラスター内には`short_query`リソースグループを1つだけ作成できます。 |

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
