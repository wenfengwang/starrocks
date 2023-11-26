---
displayed_sidebar: "Japanese"
---

# クエリ管理

## ユーザ接続数

`Property` はユーザの粒度で設定されます。Client と FE の間の最大接続数を設定するには、次のコマンドを使用します。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

ユーザのプロパティには、ユーザに割り当てられたリソースが含まれます。ここで設定されるプロパティは、`user_identity` ではなくユーザ自体に対して設定されます。つまり、`CREATE USER` ステートメントで `jack'@'%'` と `jack'@'192.%` の2つのユーザが作成された場合、`SET PROPERTY` ステートメントは `jack` ユーザに対して動作しますが、`jack'@'%'` や `jack'@'192.%` には動作しません。

例1:

```sql
ユーザ `jack` の最大接続数を 1000 に変更する
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';

root ユーザの接続制限を確認する
SHOW PROPERTY FOR 'root'; 
```

## クエリ関連のセッション変数

セッション変数は、'key' = 'value' で設定することができ、現在のセッションでの並行性、メモリ、その他のクエリパラメータを制限することができます。例:

- parallel_fragment_exec_instance_num

  デフォルト値が 1 のクエリの並列性を示します。これは各 BE 上のフラグメントインスタンスの数を示します。BE の CPU コア数の半分に設定することで、クエリのパフォーマンスを向上させることができます。

- query_mem_limit

  クエリのメモリ制限で、クエリがメモリ不足を報告した場合に調整することができます。

- load_mem_limit

  インポートのメモリ制限で、インポートジョブがメモリ不足を報告した場合に調整することができます。

例2:

```sql
set parallel_fragment_exec_instance_num  = 8; 
set query_mem_limit  = 137438953472;
```

## データベースストレージの容量クォータ

`alter database` を使用してクォータ値を変更することができます。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

クォータの単位は: B/K/KB/M/MB/G/GB/T/TB/P/PB です。

例3:

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## クエリの終了

特定の接続上のクエリを終了するには、次のコマンドを使用します。

```sql
kill connection_id;
```

`connection_id` は `show processlist;` や `select connection_id();` で確認することができます。

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
