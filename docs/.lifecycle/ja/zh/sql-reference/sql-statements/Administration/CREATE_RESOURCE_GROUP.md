---
displayed_sidebar: Chinese
---

# リソースグループの作成

## 機能

リソースグループを作成します。リソースグループについての詳細は、[リソース隔離](../../../administration/resource_group.md)を参照してください。

:::tip

この操作にはSYSTEMレベルのCREATE RESOURCE GROUP権限が必要です。[GRANT](../account-management/GRANT.md)を参照してユーザーに権限を付与してください。

:::

## 文法

```SQL
CREATE RESOURCE GROUP resource_group_name
TO CLASSIFIER1, CLASSIFIER2, ...
WITH resource_limit
```

## パラメータ説明

- `resource_group_name`：作成するリソースグループの名前。

- `CLASSIFIER`：リソース制限を適用するクエリを区別するための分類器で、`"key"="value"`の形式を取ります。同一リソースグループに複数の分類器を設定できます。

  - 分類器には以下のパラメータを含めることができます：

    | **パラメータ**   | **必須** | **説明**                                            |
    | ---------- | -------- | --------------------------------------------------- |
    | user       | いいえ       | ユーザー名。                                            |
    | role       | いいえ       | ユーザーが属するロール名。                                    |
    | query_type | いいえ       | リソース制限を適用するクエリのタイプ。現在は `SELECT` と `INSERT` (2.5以降)をサポートしています。`query_type`が`insert`のリソースグループにインポートタスクが実行中の場合、現在のBEノードはそれに対応する計算リソースを予約します。 |
    | source_ip  | いいえ       | クエリを発行するIPアドレス。CIDR形式。                   |
    | db         | いいえ       | クエリがアクセスするデータベース。カンマ（,）で区切られた文字列です。   |
    | plan_cpu_cost_range | いいえ     | 推定されるクエリのCPUコスト範囲。形式は`(DOUBLE, DOUBLE]`です。デフォルトはNULLで、制限がないことを意味します。v3.1.4からサポート。                   |
    | plan_mem_cost_range | いいえ     | 推定されるクエリのメモリコスト範囲。形式は`(DOUBLE, DOUBLE]`です。デフォルトはNULLで、制限がないことを意味します。v3.1.4からサポート。                |

- `resource_limit`：現在のリソースグループに設定されたリソース制限で、`"key"="value"`の形式を取ります。同一リソースグループに複数のリソース制限を設定できます。

  - リソース制限には以下のパラメータを含めることができます：

    | **パラメータ**                   | **必須** | **説明**                                                     |
    | -------------------------- | -------- | ------------------------------------------------------------ |
    | cpu_core_limit             | いいえ       | リソースグループが現在のBEノードで使用できるCPUコア数のソフトリミット。実際に使用されるCPUコア数は、ノードのリソースの空き状況に応じて比例してスケールします。正の整数を取ります。 |
    | mem_limit                  | いいえ       | リソースグループが現在のBEノードでクエリに使用できるメモリ(query_pool)が全体のメモリに占める割合(%)。値の範囲は(0,1)です。 |
    | concurrency_limit          | いいえ       | リソースグループ内の並行クエリの上限。過多な並行クエリの送信による過負荷を防ぐためです。 |
    | max_cpu_cores              | いいえ       | リソースグループが単一のBEノードで使用できるCPUコア数の上限。`0`より大きい値を設定した場合にのみ有効です。値の範囲は[0, `avg_be_cpu_cores`]で、`avg_be_cpu_cores`はすべてのBEのCPUコア数の平均値を表します。デフォルト値は0です。v3.1.4からサポート。 |
    | big_query_cpu_second_limit | いいえ       | 大規模クエリが使用できるCPUの時間上限。並列タスクはCPU使用時間を累積します。単位は秒。 |
    | big_query_scan_rows_limit  | いいえ       | 大規模クエリがスキャンできる行数の上限。                               |
    | big_query_mem_limit        | いいえ       | 大規模クエリが使用できるメモリの上限。単位はバイト。                  |
    | type                       | いいえ       | リソースグループのタイプ。有効な値は以下の通りです：<br />`short_query`：`short_query`リソースグループにクエリが実行中の場合、現在のBEノードは`short_query.cpu_core_limit`のCPUリソースを予約します。すべての`normal`リソースグループのCPUコア数の合計の使用上限は、BEノードのコア数から`short_query.cpu_core_limit`を引いた数にハードリミットされます。<br />`normal`：`short_query`リソースグループにクエリが実行中でない場合、すべての`normal`リソースグループのCPUコア数にハードリミットはありません。<br />注意：同一クラスタ内で、`short_query`リソースグループを1つだけ作成できます。|

## 例

例1：複数の分類器を基にリソースグループ`rg1`を作成します。

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
