---
displayed_sidebar: "Japanese"
---

# クエリ管理

## ユーザー接続数

`Property` はユーザーの細かい設定です。Client と FE の間の最大接続数を設定するには、次のコマンドを使用します。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

ユーザープロパティには、ユーザーに割り当てられたリソースが含まれます。ここで設定されるプロパティは `user_identity` ではなく、ユーザー自体のものです。つまり、`CREATE USER` ステートメントによって `jack'@'%` と `jack'@'192.%` の2つのユーザーが作成された場合、`SET PROPERTY` ステートメントは `jack'@'%' や `jack'@'192.%` ではなく、ユーザー `jack` に対して機能します。

例1:

```sql
ユーザー `jack` の最大接続数を1000に変更します
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';

ルートユーザーの接続制限をチェックします
SHOW PROPERTY FOR 'root'; 
```

## クエリ関連のセッション変数

セッション変数は 'key' = 'value' で設定でき、現在のセッションでクエリの並行性、メモリおよびその他のクエリパラメータを制限できます。例:

- parallel_fragment_exec_instance_num

  デフォルト値が1のクエリの並列処理です。それは各 BE のフラグメントインスタンスの数を示します。BE のCPUコアの半分に設定することでクエリのパフォーマンスを向上させることができます。

- query_mem_limit

  クエリのメモリ制限で、クエリが十分なメモリを報告したときに調整できます。

- load_mem_limit

  インポートのためのメモリ制限で、インポートジョブが十分なメモリを報告したときに調整できます。

例2:

```sql
set parallel_fragment_exec_instance_num  = 8; 
set query_mem_limit  = 137438953472;
```

## データベースストレージの容量クォータ

データベースストレージの容量クォータはデフォルトで無制限です。そして、`alter database` を使用してクォータ値を変更できます。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

クォータの単位は: B/K/KB/M/MB/G/GB/T/TB/P/PB です。

例3:

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## クエリの削除

特定の接続上でクエリを終了するには、次のコマンドを使用してください:

```sql
kill connection_id;
```

`connection_id` は `show processlist;` または `select connection_id();` で確認できます。

```plain text
 show processlist;
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
| Id   | User       | Host                | Cluster         | Db            | Command | Time | State | Info |
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
|    1 | starrocksmgr | 172.26.34.147:56208 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  129 | root       | 172.26.92.139:54818 | default_cluster |               | Query   |    0 |       |      |
|  114 | test       | 172.26.34.147:57974 | default_cluster | ssb_100g      | Query   |    3 |       |      |
|    3 | starrocksmgr | 172.26.34.147:57268 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  100 | root       | 172.26.34.147:58472 | default_cluster | ssb_100       | Sleep   |  637 |       |      |
|  117 | starrocksmgr | 172.26.34.147:33790 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|    6 | starrocksmgr | 172.26.34.147:57632 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  119 | starrocksmgr | 172.26.34.147:33804 | default_cluster | starrocks_monitor | Sleep   |    8 |       |      |
|  111 | root       | 172.26.92.139:55472 | default_cluster |               | Sleep   | 2758 |       |      |
+------+------------+---------------------+-----------------+---------------+---------+------+-------+------+
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