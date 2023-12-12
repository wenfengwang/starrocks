---
displayed_sidebar: "Japanese"
---

# ソートキーとプレフィックスインデックス

テーブルを作成する際には、1つ以上の列を選択してソートキーを構成することができます。ソートキーは、テーブルのデータがディスクに保存される前に並べ替えられる順序を決定します。ソートキーの列はクエリのフィルタ条件として使用することができます。このため、StarRocksは関心のあるデータを迅速に見つけることができ、テーブル全体をスキャンして必要なデータを見つける必要がありません。これにより検索の複雑さが減少し、クエリが高速化されます。

さらに、メモリ消費を減らすために、StarRocksはテーブルにプレフィックスインデックスを作成することをサポートしています。プレフィックスインデックスはスパアインデックスの一種です。StarRocksはテーブルの1024行ごとにブロックにデータを保存し、そのためにプレフィックスインデックステーブルにインデックスエントリが生成されます。ブロックのプレフィックスインデックスエントリは36バイトを超えることはできず、その内容はそのブロックの最初の行にあるテーブルのソートキー列のプレフィックスです。これにより、StarRocksはプレフィックスインデックステーブルの検索を実行する際に、その行のデータを保存するブロックの開始列番号を迅速に見つけることができます。テーブルのプレフィックスインデックスは、そのサイズがテーブル自体の1024分の1です。したがって、プレフィックスインデックス全体をメモリにキャッシュすることでクエリの高速化に役立ちます。

## 原則

重複キーテーブルでは、`DUPLICATE KEY`キーワードを使用してソートキー列を定義します。

集約テーブルでは、`AGGREGATE KEY`キーワードを使用してソートキー列を定義します。

一意キーテーブルでは、`UNIQUE KEY`キーワードを使用してソートキー列を定義します。
```
A column has a high discrimination level if the number of values in the column is large and continuously grows. For example, the number of cities in the `site_access_duplicate` table is fixed, which means that the number of values in the `city_code` column of the table is fixed. However, the number of values in the `site_id` column is much greater than the number of values in the `city_code` column and continuously grows. Therefore, the `site_id` column has a higher discrimination level than the `city_code` column.

- We recommend that you do not select a large number of sort key columns. A large number of sort key columns cannot help improve query performance but increase the overheads for sorting and data loading.

In summary, take note of the following points when you select sort key columns for the `site_access_duplicate` table:

- If your queries frequently filter on both `site_id` and `city_code`, we recommend that you select `site_id` as the beginning sort key column.

- If your queries frequently filter only on `city_code` and occasionally filter on both `site_id` and `city_code`, we recommend that you select `city_code` as the beginning sort key column.

- If the number of times that your queries filter on both `site_id` and `city_code` is roughly equal to the number of times that your queries filter only on `city_code`, we recommend that you create a materialized view, for which the first column is `city_code`. As such, StarRocks creates a sort index on the `city_code` column of the materialized view.
```