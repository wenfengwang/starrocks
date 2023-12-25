---
displayed_sidebar: English
---

# 大規模クエリの監視と管理

このトピックでは、StarRocks クラスターで大規模クエリを監視および管理する方法について説明します。

大規模クエリには、多数の行をスキャンするクエリや、多くの CPU とメモリリソースを占有するクエリが含まれます。これらはクラスタリソースを容易に枯渇させ、制限がなければシステムの過負荷を引き起こす可能性があります。この問題に対処するために、StarRocksは大規模クエリを監視および管理する一連の対策を提供し、クエリがクラスタリソースを独占するのを防ぎます。

StarRocksで大規模クエリを扱う全体的なアイデアは以下の通りです：

1. リソースグループとクエリキューを使用して大規模クエリに対する自動的な予防措置を設定します。
2. 大規模クエリをリアルタイムで監視し、予防措置を迂回するクエリを終了させます。
3. 監査ログとビッグクエリログを分析して大規模クエリのパターンを研究し、以前に設定した予防メカニズムを微調整します。

この機能は v3.0 からサポートされています。

## 大規模クエリに対する予防措置を設定する

StarRocksは、大規模クエリを処理するための2つの予防策、リソースグループとクエリキューを提供しています。リソースグループを使用して、大規模クエリが実行されないように防ぐことができます。一方、クエリキューは、同時実行のしきい値やリソース制限に達したときに入ってくるクエリをキューに入れ、システムの過負荷を防ぐのに役立ちます。

### リソースグループを通じて大規模クエリをフィルタリングする

リソースグループは、大規模クエリを自動的に識別し終了させることができます。リソースグループを作成する際に、クエリが使用できる CPU 時間、メモリ使用量、またはスキャン行数の上限を指定できます。リソースグループに該当するすべてのクエリの中で、より多くのリソースを要求するクエリは拒否され、エラーとともに返されます。詳細は[リソース分離](../administration/resource_group.md)を参照してください。

リソースグループを作成する前に、リソースグループ機能が依存するパイプラインエンジンを有効にするために、以下のステートメントを実行する必要があります：

```SQL
SET GLOBAL enable_pipeline_engine = true;
```

次の例では、`bigQuery` というリソースグループを作成し、CPU 時間の上限を `100` 秒、スキャン行数の上限を `100000`、メモリ使用量の上限を `1073741824` バイト（1 GB）に制限しています：

```SQL
CREATE RESOURCE GROUP bigQuery
TO 
    (db='sr_hub')
WITH (
    'cpu_core_limit' = '10',
    'mem_limit' = '20%',
    'big_query_cpu_second_limit' = '100',
    'big_query_scan_rows_limit' = '100000',
    'big_query_mem_limit' = '1073741824'
);
```

クエリがこれらの制限のいずれかを超える必要がある場合、クエリは実行されず、エラーで返されます。次の例は、クエリが許可されたスキャン行数を超えた場合に返されるエラーメッセージを示しています：

```Plain
ERROR 1064 (HY000): exceed big query scan_rows limit: current is 4 but limit is 1
```

リソースグループを初めて設定する場合は、通常のクエリに影響を与えないように、比較的高めの制限を設定することを推奨します。大規模クエリのパターンに関する理解が深まった後、これらの制限を微調整することができます。

### クエリキューを利用してシステム過負荷を緩和する

クエリキューは、クラスタリソースの占有が事前に設定されたしきい値を超えた際にシステム過負荷の悪化を緩和するために設計されています。最大同時実行数、メモリ使用量、CPU 使用率のしきい値を設定できます。StarRocksは、これらのしきい値のいずれかに達した場合に、自動的に入ってくるクエリをキューに入れます。保留中のクエリは、キューで実行を待つか、指定されたリソースしきい値に達した場合にキャンセルされます。詳細は[クエリキュー](../administration/query_queues.md)を参照してください。

SELECT クエリのクエリキューを有効にするには、次のステートメントを実行します：

```SQL
SET GLOBAL enable_query_queue_select = true;
```

クエリキュー機能を有効にした後、クエリキューをトリガするルールを定義できます。

- クエリキューをトリガする同時実行のしきい値を指定します。

  次の例では、同時実行のしきい値を `100` に設定しています：

  ```SQL
  SET GLOBAL query_queue_concurrency_limit = 100;
  ```

- クエリキューをトリガするメモリ使用率のしきい値を指定します。

  次の例では、メモリ使用率のしきい値を `0.9` に設定しています：

  ```SQL
  SET GLOBAL query_queue_mem_used_pct_limit = 0.9;
  ```

- クエリキューをトリガするCPU使用率のしきい値を指定します。

  次の例では、CPU 使用率パーミル（CPU 使用率 × 1000）のしきい値を `800` に設定しています：

  ```SQL
  SET GLOBAL query_queue_cpu_used_permille_limit = 800;
  ```

また、キュー内の保留中のクエリに対して最大キュー長とタイムアウトを設定することで、これらのキューに入れられたクエリの扱い方を決めることができます。

- クエリキューの最大長を指定します。このしきい値に達した場合、新たなクエリは拒否されます。

  次の例では、クエリキューの最大長を `100` に設定しています：

  ```SQL
  SET GLOBAL query_queue_max_queued_queries = 100;
  ```

- キュー内の保留中のクエリの最大タイムアウトを指定します。このしきい値に達した場合、該当するクエリは拒否されます。

  次の例では、最大タイムアウトを `480` 秒に設定しています：

  ```SQL
  SET GLOBAL query_queue_pending_timeout_second = 480;
  ```

クエリが保留中であるかどうかは、[SHOW PROCESSLIST](../sql-reference/sql-statements/Administration/SHOW_PROCESSLIST.md)を使用して確認できます。

```Plain
mysql> SHOW PROCESSLIST;
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
| Id   | User | Host                | Db    | Command | ConnectionStartTime | Time | State | Info              | IsPending |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+

|    2 | root | xxx.xx.xxx.xx:xxxxx |       | Query   | 2022-11-24 18:08:29 |    0 | OK    | SHOW PROCESSLIST  | false     |
+------+------+---------------------+-------+---------+---------------------+------+-------+-------------------+-----------+
```

`IsPending`が`true`の場合、該当するクエリはクエリキューで保留中です。

## リアルタイムでの大規模クエリの監視

v3.0以降、StarRocksはクラスターで現在処理されているクエリとそれらが消費するリソースを表示する機能をサポートしています。これにより、大規模クエリが予防策を回避して予期せぬシステム過負荷を引き起こす場合に備えて、クラスターを監視できます。

### MySQLクライアントを通じた監視

1. [SHOW PROC](../sql-reference/sql-statements/Administration/SHOW_PROC.md)を使用して、現在処理中のクエリ(`current_queries`)を表示できます。

   ```SQL
   SHOW PROC '/current_queries';
   ```

   StarRocksはクエリID(`QueryId`)、接続ID(`ConnectionId`)、および各クエリのリソース消費量を返します。これにはスキャンしたデータサイズ(`ScanBytes`)、処理した行数(`ProcessRows`)、CPU時間(`CPUCostSeconds`)、メモリ使用量(`MemoryUsageBytes`)、実行時間(`ExecTime`)が含まれます。

   ```Plain
   mysql> SHOW PROC '/current_queries';
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   | QueryId                              | ConnectionId | Database   | User | ScanBytes | ProcessRows    | CPUCostSeconds | MemoryUsageBytes | ExecTime |
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   | 7c56495f-ae8b-11ed-8ebf-00163e00accc | 4            | tpcds_100g | root | 37.88 MB  | 1075769 Rows   | 11.13 Seconds  | 146.70 MB        | 3804     |
   | 7d543160-ae8b-11ed-8ebf-00163e00accc | 6            | tpcds_100g | root | 13.02 GB  | 487873176 Rows | 81.23 Seconds  | 6.37 GB          | 2090     |
   +--------------------------------------+--------------+------------+------+-----------+----------------+----------------+------------------+----------+
   2 rows in set (0.01 sec)
   ```

2. クエリIDを指定して、各BEノードでのクエリのリソース消費をさらに詳しく調べることができます。

   ```SQL
   SHOW PROC '/current_queries/<QueryId>/hosts';
   ```

   StarRocksは、各BEノードでのクエリのスキャンデータサイズ(`ScanBytes`)、スキャン行数(`ScanRows`)、CPU時間(`CPUCostSeconds`)、メモリ使用量(`MemUsageBytes`)を返します。

   ```Plain
   mysql> SHOW PROC '/current_queries/7c56495f-ae8b-11ed-8ebf-00163e00accc/hosts';
   +--------------------+-----------+-------------+----------------+---------------+
   | Host               | ScanBytes | ScanRows    | CPUCostSeconds | MemUsageBytes |
   +--------------------+-----------+-------------+----------------+---------------+
   | 172.26.34.185:8060 | 11.61 MB  | 356252 Rows | 52.93 Seconds  | 51.14 MB      |
   | 172.26.34.186:8060 | 14.66 MB  | 362646 Rows | 52.89 Seconds  | 50.44 MB      |
   | 172.26.34.187:8060 | 11.60 MB  | 356871 Rows | 52.91 Seconds  | 48.95 MB      |
   +--------------------+-----------+-------------+----------------+---------------+
   3 rows in set (0.00 sec)
   ```

### FEコンソールを通じた監視

MySQLクライアントに加えて、FEコンソールを使用して視覚化されたインタラクティブな監視が可能です。

1. 以下のURLを使用してブラウザでFEコンソールにアクセスします。

   ```Bash
   http://<fe_IP>:<fe_http_port>/system?path=//current_queries
   ```

   ![FEコンソール1](../assets/console_1.png)

   **システム情報**ページで、現在処理中のクエリとそのリソース消費量を確認できます。

2. クエリの**QueryID**をクリックします。

   ![FEコンソール2](../assets/console_2.png)

   表示されるページで、ノード固有のリソース消費の詳細情報を確認できます。

### 大規模クエリを手動で終了する

設定した予防措置を大規模クエリが回避し、システムの可用性を脅かす場合、[KILL](../sql-reference/sql-statements/Administration/KILL.md)ステートメントで対応する接続IDを使用して手動で終了できます。

```SQL
KILL QUERY <ConnectionId>;
```

## 大規模クエリログの分析

v3.0以降、StarRocksは**fe/log/fe.big_query.log**ファイルに保存されるBig Query Logsをサポートしています。StarRocksの監査ログと比較して、Big Query Logsは以下の3つのフィールドを追加で出力します。

- `bigQueryLogCPUSecondThreshold`
- `bigQueryLogScanBytesThreshold`
- `bigQueryLogScanRowsThreshold`

これら3つのフィールドは、クエリが大規模クエリかどうかを判断するために定義したリソース消費の閾値に対応しています。

Big Query Logsを有効にするには、次のステートメントを実行します。

```SQL
SET GLOBAL enable_big_query_log = true;
```

Big Query Logsが有効になったら、Big Query Logsをトリガするルールを定義できます。

- Big Query LogsをトリガするCPU時間の閾値を指定します。

  以下の例では、CPU時間の閾値を`600`秒に設定します。

  ```SQL
  SET GLOBAL big_query_log_cpu_second_threshold = 600;
  ```

- Big Query Logsをトリガするスキャンデータサイズの閾値を指定します。

  以下の例では、スキャンデータサイズの閾値を`10737418240`バイト（10GB）に設定します。

  ```SQL
  SET GLOBAL big_query_log_scan_bytes_threshold = 10737418240;
  ```
  
- Big Query Logsをトリガするスキャン行数の閾値を指定します。

  以下の例では、スキャン行数の閾値を`1500000000`に設定します。

  ```SQL
  SET GLOBAL big_query_log_scan_rows_threshold = 1500000000;
  ```

## 予防措置の微調整

リアルタイムモニタリングとBig Query Logsから得られた統計を基に、クラスタ内で見落とされた大規模クエリ（または誤って大規模クエリと診断された通常のクエリ）のパターンを研究し、リソースグループとクエリキューの設定を最適化できます。

大規模クエリの大部分が特定のSQLパターンに一致しており、そのSQLパターンを永続的に禁止したい場合、そのパターンをSQLブラックリストに追加できます。StarRocksはSQLブラックリストに指定されたパターンに一致するすべてのクエリを拒否し、エラーを返します。詳細は[SQLブラックリストの管理](../administration/Blacklist.md)を参照してください。

SQLブラックリストを有効にするには、以下のステートメントを実行します。

```SQL
ADMIN SET FRONTEND CONFIG ("enable_sql_blacklist" = "true");
```

次に、[ADD SQLBLACKLIST](../sql-reference/sql-statements/Administration/ADD_SQLBLACKLIST.md)を使用して、SQL パターンを表す正規表現を SQL ブラックリストに追加できます。

以下の例では、`COUNT(DISTINCT)` を SQL ブラックリストに追加しています。

```SQL
ADD SQLBLACKLIST "SELECT COUNT(DISTINCT .+) FROM .+";
```
