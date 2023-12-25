---
displayed_sidebar: English
---

# クエリ管理

## ユーザー接続数

`Property` はユーザーごとに設定されます。クライアントとFE間の最大接続数を設定するには、以下のコマンドを使用します。

```sql
SET PROPERTY [FOR 'user'] 'key' = 'value' [, 'key' = 'value']
```

ユーザープロパティには、ユーザーに割り当てられたリソースが含まれます。ここで設定されるプロパティは、`user_identity`ではなくユーザー向けです。つまり、`CREATE USER` ステートメントで作成された2つのユーザー `jack'@'%'` と `jack'@'192.%'` がある場合、`SET PROPERTY` ステートメントは `jack'@'%'` や `jack'@'192.%'` ではなく、ユーザー `jack` に対して機能します。

例1:

```sql
ユーザー `jack` の最大接続数を1000に変更する
SET PROPERTY FOR 'jack' 'max_user_connections' = '1000';

root ユーザーの接続制限を確認する
SHOW PROPERTY FOR 'root'; 
```

## クエリ関連のセッション変数

セッション変数は 'key' = 'value' で設定でき、現在のセッションでの並行性、メモリ、その他のクエリパラメータを制限することができます。例えば：

- parallel_fragment_exec_instance_num

  クエリの並列実行インスタンス数で、デフォルト値は1です。これは各BE上のフラグメントインスタンスの数を示しています。BEのCPUコア数の半分に設定することで、クエリパフォーマンスを向上させることができます。

- query_mem_limit

  クエリのメモリ制限で、クエリがメモリ不足を報告した場合に調整可能です。

- load_mem_limit

  インポートのメモリ制限で、インポートジョブがメモリ不足を報告した場合に調整可能です。

例2:

```sql
set parallel_fragment_exec_instance_num = 8; 
set query_mem_limit = 137438953472;
```

## データベースストレージの容量クォータ

データベースストレージの容量クォータはデフォルトで無制限です。クォータ値は `ALTER DATABASE` を使用して変更できます。

```sql
ALTER DATABASE db_name SET DATA QUOTA quota;
```

クォータの単位は以下の通りです：B/K/KB/M/MB/G/GB/T/TB/P/PB

例3:

```sql
ALTER DATABASE example_db SET DATA QUOTA 10T;
```

## クエリのキル

以下のコマンドを使用して特定の接続のクエリを終了します：

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
