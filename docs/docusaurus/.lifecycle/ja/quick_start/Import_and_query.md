---
displayed_sidebar: "Japanese"
---

# データの読み込みとクエリ

このクイックスタートチュートリアルでは、作成したテーブルにデータをロードする手順（詳細については[テーブルの作成](../quick_start/Create_table.md)を参照）と、そのデータにクエリを実行する手順を逐一説明します。

StarRocksでは、主要なクラウドサービス、ローカルファイル、またはストリーミングデータシステムなど、豊富なデータソースからデータをロードすることができます。詳細については[Ingestion Overview](../loading/Loading_intro.md)を参照してください。以下の手順では、INSERT INTOステートメントを使用してStarRocksにデータを挿入し、そのデータにクエリを実行する方法を示します。

> **注意**
>
> このチュートリアルは、既存のStarRocksインスタンス、データベース、テーブル、ユーザー、および独自のデータを使用して完了することができます。ただし、簡便のために、チュートリアルで提供されるスキーマとデータを使用することをお勧めします。

## ステップ1：INSERTでデータをロードする

INSERTを使用して追加のデータ行を挿入することができます。詳細な手順については[INSERT](../sql-reference/sql-statements/data-manipulation/INSERT.md)を参照してください。

MySQLクライアントを使用してStarRocksにログインし、次のステートメントを実行して作成した`sr_member`テーブルに以下のデータ行を挿入します。

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
クエリが正常に処理されました、影響を受ける行数：6行 (0.07秒)
{'label':'insertDemo', 'status':'VISIBLE', 'txnId':'5'}
```

> **注意**
>
> INSERT INTO VALUESを使用してデータを読み込む場合は、小規模なデータセットを使用してデモを検証する場合にのみ適用されます。大量のデータをStarRocksに読み込む場合は、シナリオに適した他のオプションについては[Ingestion Overview](../loading/Loading_intro.md)を参照してください。

## ステップ2：データにクエリを実行する

StarRocksはSQL-92に互換性があります。

- テーブル内のすべてのデータ行をリストするためのシンプルなクエリを実行します。

  ```SQL
  SELECT * FROM sr_member;
  ```

  返された結果は次の通りです。

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
  6行（0.05秒で完了）
  ```

- 指定した条件で標準クエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member
  WHERE reg_date <= "2022-03-14";
  ```

  返された結果は次の通りです。

  ```Plain
  +-------+----------+
  | sr_id | name     |
  +-------+----------+
  |     1 | tom      |
  |     3 | maruko   |
  |     2 | johndoe  |
  +-------+----------+
  3行（0.01秒で完了）
  ```

- 指定したパーティションでクエリを実行します。

  ```SQL
  SELECT sr_id, name 
  FROM sr_member 
  PARTITION (p2);
  ```

  返された結果は次の通りです。

  ```Plain
  +-------+---------+
  | sr_id | name    |
  +-------+---------+
  |     3 | maruko  |
  |     2 | johndoe |
  +-------+---------+
  2行（0.01秒で完了）
  ```

## 次に何をするか

StarRocksのデータ取り込み方法についてもっと学ぶには、[Ingestion Overview](../loading/Loading_intro.md)を参照してください。また、豊富な組み込み関数に加えて、StarRocksは[Java UDFs](../sql-reference/sql-functions/JAVA_UDF.md)もサポートしており、ビジネスシナリオに合わせた独自のデータ処理関数を作成することができます。

次のことも学ぶことができます：

- ロード時の[ETLを実行](../loading/Etl_in_loading.md)する。
- 外部データソースにアクセスするために[外部テーブル](../data_source/External_table.md)を作成する。
- クエリパフォーマンスを最適化する方法を学ぶために[クエリプランを分析](../administration/Query_planning.md)する。