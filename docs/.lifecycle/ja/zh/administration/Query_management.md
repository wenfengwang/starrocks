---
displayed_sidebar: Chinese
---

# クエリ管理

この文書では、クエリの管理方法について説明します。

## ユーザー接続数の管理

Property はユーザー粒度での属性設定項目を指します。ユーザーの属性を設定することで、ユーザーに割り当てられるリソースなどを含みます。ここでのユーザー属性は、user_identity ではなく user に対する属性を指します。

以下のコマンドを使用して、特定のユーザーのクライアントから FE への最大接続数を管理できます。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value'[, ...];
```

以下の例では、ユーザー jack の最大接続数を 1000 に変更しています。

```sql
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';
```

以下のコマンドで特定のユーザーの接続数制限を確認できます。

```sql
SHOW PROPERTY FOR 'user';
```

## セッション変数に関連するクエリの設定

現在のセッション内のクエリの並行性やメモリなどを調整するために、セッションレベルのクエリ関連変数を設定できます。

### クエリの並行性の調整

クエリの並行性を調整するには、Pipeline 実行エンジンに関連する変数を変更することをお勧めします。

> **説明**
>
> - バージョン 2.2 から、Pipeline エンジンが正式にリリースされました。
> - バージョン 3.0 から、クエリの並行性に基づいて `pipeline_dop` を自動調整する機能がサポートされています。

```sql
SET enable_pipeline_engine = true;
SET pipeline_dop = 0;
```

| パラメータ                             | 説明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| enable_pipeline_engine              | Pipeline 実行エンジンを有効にするかどうか。true: 有効（デフォルト）、false: 無効。 |
| pipeline_dop                        | Pipeline インスタンスの並列数。デフォルト値の 0 を推奨します。これは、システムが各 pipeline の並列度を自動調整することを意味します。0 より大きい値に設定することもできますが、通常は BE ノードの CPU 物理コア数の半分です。 |

インスタンスの並列数を設定することで、クエリの並行性を調整することもできます。

```sql
SET GLOBAL parallel_fragment_exec_instance_num = INT;
```

`parallel_fragment_exec_instance_num`：Fragment インスタンスの並列数。1つの Fragment インスタンスは BE ノードの1つの CPU を使用するため、クエリの並行性は Fragment インスタンスの並列数に等しくなります。クエリの並行性を向上させたい場合は、このパラメータを BE の CPU コア数の半分に設定できます。

> 注意：
>
> - 実際のシナリオでは、Fragment インスタンスの並列数には上限があり、それは BE 内のテーブルの Tablet 数によって決まります。例えば、3つのパーティションと32のバケットを持つテーブルが4つの BE ノードに分散している場合、1つの BE ノードの Tablet 数は 32 * 3 / 4 = 24 となり、したがってその BE ノード上の Fragment インスタンスの並列数の上限は 24 です。この場合、パラメータを `32` に設定しても、実際に使用される並列数は 24 になります。
> - 高並行性のシナリオでは、CPU リソースはすでに十分に利用されていることが多いため、Fragment インスタンスの並列数を `1` に設定することをお勧めします。これにより、異なるクエリ間のリソース競合を減らし、全体的なクエリ効率を向上させることができます。

### クエリのメモリ上限の調整

以下のコマンドを使用して、クエリのメモリ上限を調整できます。

```sql
SET query_mem_limit = INT;
```

`query_mem_limit`：単一クエリのメモリ制限で、単位は Byte です。17179869184（16GB）以上に設定することを推奨します。

## データベースのストレージ容量 Quota の調整

デフォルト設定では、各データベースのストレージ容量に制限はありません。以下のコマンドを使用して調整できます。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

> 説明：`quota` の単位は `B`、`K`、`KB`、`M`、`MB`、`G`、`GB`、`T`、`TB`、`P`、または `PB` です。

例：

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

詳細は [ALTER DATABASE](../sql-reference/sql-statements/data-definition/ALTER_DATABASE.md) を参照してください。

## クエリの停止

以下のコマンドを使用して、特定の接続上のクエリを停止できます。

```sql
KILL connection_id;
```

`connection_id`：特定の接続の ID。`SHOW processlist;` または `select connection_id();` を使用して確認できます。

例：

```plain text
mysql> show processlist;
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
| Id   | User         | Host                | Cluster         | Db                | Command | Time | State | Info |
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
|    1 | starrocksmgr | 172.26.34.147:56208 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  129 | root         | 172.26.92.139:54818 | default_cluster |                   | Query   |    0 |       |      |
|  114 | test         | 172.26.34.147:57974 | default_cluster | ssb_100g          | Query   |    3 |       |      |
|    3 | starrocksmgr | 172.26.34.147:57268 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  100 | root         | 172.26.34.147:58472 | default_cluster | ssb_100           | Sleep   |  637 |       |      |
|  117 | starrocksmgr | 172.26.34.147:33790 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|    6 | starrocksmgr | 172.26.34.147:57632 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  119 | starrocksmgr | 172.26.34.147:33804 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  111 | root         | 172.26.92.139:55472 | default_cluster |                   | Sleep   | 2758 |       |      |
+------+--------------+---------------------+-----------------+-------------------+---------+------+-------+------+
9 rows in set (0.00 sec)

mysql> select connection_id();
+-----------------+
| CONNECTION_ID() |
+-----------------+
|              98 |
+-----------------+


mysql> kill 114;
Query OK, 0 rows affected (0.02 sec)
```

> 説明
>
> `Info` 列には対応する SQL ステートメントが表示されます。SQL ステートメントが長すぎて切り捨てられた場合は、`SHOW FULL processlist;` を使用して完全な SQL ステートメントを表示できます。
