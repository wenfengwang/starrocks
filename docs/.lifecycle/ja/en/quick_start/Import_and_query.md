---
displayed_sidebar: English
---

# データのロードとクエリ

このクイックスタートチュートリアルでは、作成したテーブルにデータをロードする手順（詳細は[テーブルの作成](../quick_start/Create_table.md)を参照してください）、およびデータに対するクエリの実行方法をステップバイステップで説明します。

StarRocksは、主要なクラウドサービス、ローカルファイル、ストリーミングデータシステムなど、多様なデータソースからのデータロードをサポートしています。詳細は[データ取り込みの概要](../loading/Loading_intro.md)をご覧ください。以下の手順では、INSERT INTOステートメントを使用してStarRocksにデータを挿入し、そのデータに対してクエリを実行する方法を示します。

> **注**
>
> このチュートリアルは、既存のStarRocksインスタンス、データベース、テーブル、ユーザー、およびご自身のデータを使用して完了することができます。しかし、簡単にするために、チュートリアルで提供されるスキーマとデータの使用を推奨します。

## ステップ1: INSERTを使用してデータをロードする

INSERTを使用して追加のデータ行を挿入できます。詳細な指示については[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

MySQLクライアントを介してStarRocksにログインし、以下のステートメントを実行して、作成した`sr_member`テーブルに以下のデータ行を挿入します。

```SQL
use sr_hub;
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

ロードトランザクションが成功すると、以下のメッセージが返されます。

```Plain
Query OK, 6 rows affected (0.07 sec)
{'label':'insertDemo', 'status':'VISIBLE', 'txnId':'5'}
```

> **注**
>
> INSERT INTO VALUESを使用したデータのロードは、小規模なデータセットでデモを検証する際にのみ適用されます。大規模なテストや本番環境には推奨されません。StarRocksに大量のデータをロードするには、[データ取り込みの概要](../loading/Loading_intro.md)を参照して、シナリオに合った他のオプションをご覧ください。

## ステップ2: データをクエリする

StarRocksはSQL-92と互換性があります。

- 単純なクエリを実行してテーブル内の全データ行をリストします。

  ```SQL
  SELECT * FROM sr_member;
  ```

  返された結果は以下の通りです。

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

- 指定された条件で標準クエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  返された結果は以下の通りです。

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

- 特定のパーティションでクエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member 
  PARTITION (p2);
  ```

  返された結果は以下の通りです。

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

StarRocksのデータ取り込み方法について詳しくは、[データ取り込みの概要](../loading/Loading_intro.md)をご覧ください。StarRocksは、多数の組み込み関数に加えて、[Java UDF](../sql-reference/sql-functions/JAVA_UDF.md)もサポートしており、ビジネスシナリオに合わせた独自のデータ処理関数を作成することができます。

また、以下の方法についても学ぶことができます。

- [ロード時のETLを実行する](../loading/Etl_in_loading.md)。
- [外部テーブルを作成して](../data_source/External_table.md)外部データソースにアクセスする。
- [クエリプランを分析する](../administration/Query_planning.md)ことでクエリパフォーマンスの最適化方法を学ぶ。
