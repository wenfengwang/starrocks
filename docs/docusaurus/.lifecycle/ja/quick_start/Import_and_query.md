---
displayed_sidebar: "Japanese"
---

# データの読み込みとクエリ

このクイックスタートチュートリアルでは、以下の手順に従って、作成したテーブルにデータを読み込み（詳細は[テーブルの作成](../quick_start/Create_table.md)を参照）、データをクエリする方法を一歩一歩ご紹介します。

StarRocksは、主要なクラウドサービス、ローカルファイル、またはストリーミングデータシステムなど、豊富なデータソースからデータを読み込むことができます。詳細については、[データ取り込みの概要](../loading/Loading_intro.md)をご覧ください。以下の手順では、INSERT INTOステートメントを使用してデータをStarRocksに挿入し、そのデータをクエリする方法を示します。

> **注意**
>
> このチュートリアルを完了するには、既存のStarRocksのインスタンス、データベース、テーブル、ユーザー、およびご自分のデータを使用できます。ただし、簡単のために、チュートリアルが提供するスキーマやデータを使用することを推奨します。

## ステップ1：INSERTを使用してデータを読み込む

INSERTを使用して追加のデータ行を挿入できます。詳細は[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

MySQLクライアントを使用してStarRocksにログインし、以下のステートメントを実行して、作成した`sr_member`テーブルに次のデータ行を挿入します。

```SQL
use sr_hub
INSERT INTO sr_member
WITH LABEL insertDemo
VALUES
    (001,"tom",100000,"2022-03-13",true),
    (002,"johndoe",210000,"2022-03-14",false),
    (003,"maruko",200000,"2022-03-14",true),
    (004,"ronaldo",100000,"2022-03-15",false),
    (005,"pavlov",210000,"2022-03-16",false),
    (006,"mohammed",300000,"2022-03-17",true);
```

読み込みトランザクションが成功した場合、以下のメッセージが返されます。

```Plain
Query OK, 6 rows affected (0.07 sec)
{'label':'insertDemo', 'status':'VISIBLE', 'txnId':'5'}
```

> **注意**
>
> INSERT INTO VALUESを使用してデータを読み込むのは、小規模なデータセットでのデモを検証する必要がある場合にのみ適用されます。大規模なテストや本番環境には推奨されません。StarRocksに大量のデータを読み込むには、シナリオに合った他のオプションについては、[データ取り込みの概要](../loading/Loading_intro.md)をご覧ください。

## ステップ2：データをクエリする

StarRocksはSQL-92に対応しています。

- テーブルのすべてのデータ行をリストアップするための簡単なクエリを実行します。

  ```SQL
  SELECT * FROM sr_member;
  ```

  返された結果は次のようになります：

  ```Plain
  +-------+----------+-----------+------------+----------+
  | sr_id | name     | city_code | reg_date   | verified |
  +-------+----------+-----------+------------+----------+
  |     3 | maruko   |    200000 | 2022-03-14 |        1 |
  |     1 | tom      |    100000 | 2022-03-13 |        1 |
  |     4 | ronaldo  |    100000 | 2022-03-15 |        0 |
  |     6 | mohammed |    300000 | 2022-03-17 |        1 |
  |     5 | pavlov   |    210000 | 2022-03-16 |        0 |
  |     2 | johndoe  |    210000 | 2022-03-14 |        0 |
  +-------+----------+-----------+------------+----------+
  6 rows in set (0.05 sec)
  ```

- 指定された条件付きで標準クエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  返された結果は次のようになります：

  ```Plain
  +-------+----------+
  | sr_id | name     |
  +-------+----------+
  |     1 | tom      |
  |     3 | maruko   |
  |     2 | johndoe  |
  +-------+----------+
  3 rows in set (0.01 sec)
  ```

- 指定されたパーティションでクエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member 
  PARTITION (p2);
  ```

  返された結果は次のようになります：

  ```Plain
  +-------+---------+
  | sr_id | name    |
  +-------+---------+
  |     3 | maruko  |
  |     2 | johndoe |
  +-------+---------+
  2 rows in set (0.01 sec)
  ```

## 次に何をするか

StarRocksのデータ取り込み方法について詳しく知るには、[データ取り込みの概要](../loading/Loading_intro.md)を参照してください。多数の組み込み関数に加え、StarRocksは[Java UDFs](../sql-reference/sql-functions/JAVA_UDF.md)もサポートしており、ビジネスシナリオに適した独自のデータ処理関数を作成することができます。

また、次のような方法も学ぶことができます：

- 読み込み時の[ETLを実行](../loading/Etl_in_loading.md)する方法。
- 外部データソースにアクセスするための[外部テーブルの作成](../data_source/External_table.md)方法。
- クエリのパフォーマンスを最適化する方法を学ぶために、[クエリプランの分析](../administration/Query_planning.md)を実行する方法。