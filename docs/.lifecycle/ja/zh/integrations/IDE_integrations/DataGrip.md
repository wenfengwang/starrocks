---
displayed_sidebar: Chinese
---

# DataGrip

DataGrip は StarRocks の内部データと外部データのクエリをサポートしています。

DataGrip でデータソースを作成します。作成する際に **MySQL** をデータソース (**Data** **Source**) として選択する必要があります。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

設定が必要なパラメータは以下の通りです：

- **Host**：StarRocks クラスタの FE ホストの IP アドレス。
- **Port**：StarRocks クラスタの FE クエリポート、例えば `9030`。
- **Authentication**：認証方式。**Username & Password** を選択します。
- **User**：StarRocks クラスタにログインするためのユーザー名、例えば `admin`。
- **Password**：StarRocks クラスタにログインするためのパスワード。
- **Database**：StarRocks クラスタでアクセスするデータソース。形式は `<catalog_name>.<database_name>` です。
  - `catalog_name`：StarRocks クラスタ内の対象カタログの名前。Internal Catalog と External Catalog の両方がサポートされています。
  - `database_name`：StarRocks クラスタ内の対象データベースの名前。内部データベースと外部データベースの両方がサポートされています。
