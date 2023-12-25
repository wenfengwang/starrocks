---
displayed_sidebar: English
---

# DataGrip

DataGripはStarRocksの内部データと外部データの両方に対するクエリをサポートしています。

DataGripでデータソースを作成します。データソースとしてMySQLを選択する必要があります。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

設定が必要なパラメータは以下の通りです：

- **Host**: StarRocksクラスターのFEホストIPアドレス。
- **Port**: StarRocksクラスターのFEクエリポート。例えば、`9030`。
- **Authentication**: 使用する認証方法。**Username & Password**を選択してください。
- **User**: StarRocksクラスターにログインするためのユーザー名。例：`admin`。
- **Password**: StarRocksクラスターにログインするためのパスワード。
- **Database**: StarRocksクラスターでアクセスしたいデータソース。このパラメータの値は`<catalog_name>.<database_name>`の形式です。
  - `catalog_name`: StarRocksクラスター内の目的のカタログ名。内部カタログと外部カタログがサポートされています。
  - `database_name`: StarRocksクラスター内の目的のデータベース名。内部データベースと外部データベースがサポートされています。
