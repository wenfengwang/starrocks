---
displayed_sidebar: "Japanese"
---

# データの読み込みとクエリ

このクイックスタートチュートリアルでは、作成したテーブルにデータを読み込む手順（詳細は[テーブルの作成](../quick_start/Create_table.md)を参照）と、データに対するクエリの実行方法をステップバイステップで説明します。

StarRocksは、主要なクラウドサービス、ローカルファイル、ストリーミングデータシステムなど、さまざまなデータソースからデータを読み込むことができます。詳細については、[データの読み込み概要](../loading/Loading_intro.md)を参照してください。以下の手順では、INSERT INTOステートメントを使用してデータをStarRocksに挿入し、データに対してクエリを実行する方法を示します。

> **注意**
>
> このチュートリアルは、既存のStarRocksインスタンス、データベース、テーブル、ユーザー、および独自のデータを使用して完了することができます。ただし、簡単のために、チュートリアルが提供するスキーマとデータを使用することをお勧めします。

## ステップ1：INSERTを使用してデータを読み込む

INSERTを使用して追加のデータ行を挿入することができます。詳細な手順については、[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

MySQLクライアントを使用してStarRocksにログインし、次のステートメントを実行して、作成した`sr_member`テーブルに以下のデータ行を挿入します。

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
> INSERT INTO VALUESを使用してデータを読み込むのは、小規模なデータセットでのデモを検証する場合にのみ適用されます。大量のテストや本番環境には推奨されません。StarRocksに大量のデータを読み込むには、シナリオに合わせた他のオプションについては、[データの読み込み概要](../loading/Loading_intro.md)を参照してください。

## ステップ2：データのクエリ

StarRocksはSQL-92と互換性があります。

- テーブル内のすべてのデータ行をリストするためのシンプルなクエリを実行します。

  ```SQL
  SELECT * FROM sr_member;
  ```

  返される結果は次のようになります。

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

- 指定された条件で標準的なクエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  返される結果は次のようになります。

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

  返される結果は次のようになります。

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

StarRocksのデータの読み込み方法について詳しくは、[データの読み込み概要](../loading/Loading_intro.md)を参照してください。StarRocksには多数の組み込み関数が用意されているだけでなく、[Java UDF](../sql-reference/sql-functions/JAVA_UDF.md)もサポートされており、ビジネスシナリオに合わせた独自のデータ処理関数を作成することができます。

次の方法も学ぶことができます：

- [読み込み時のETL](../loading/Etl_in_loading.md)の実行方法。
- 外部データソースにアクセスするための[外部テーブル](../data_source/External_table.md)の作成方法。
- クエリのパフォーマンスを最適化するための[クエリプランの分析](../administration/Query_planning.md)の方法。
