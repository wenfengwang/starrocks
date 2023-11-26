---
displayed_sidebar: "Japanese"
---

# DataGrip

DataGripは、StarRocks内部データと外部データの両方をクエリすることができます。

DataGripでデータソースを作成します。データソースとしてMySQLを選択する必要があることに注意してください。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

設定する必要があるパラメータは次のように説明されています：

- **ホスト**：StarRocksクラスタのFEホストIPアドレスです。
- **ポート**：StarRocksクラスタのFEクエリポートです。例：`9030`。
- **認証**：使用する認証方法を選択します。**ユーザー名とパスワード**を選択します。
- **ユーザー**：StarRocksクラスタにログインするために使用されるユーザー名です。例：`admin`。
- **パスワード**：StarRocksクラスタにログインするために使用されるパスワードです。
- **データベース**：StarRocksクラスタでアクセスしたいデータソースです。このパラメータの値は`<catalog_name>.<database_name>`形式です。
  - `catalog_name`：StarRocksクラスタのターゲットカタログの名前です。内部および外部のカタログの両方がサポートされています。
  - `database_name`：StarRocksクラスタのターゲットデータベースの名前です。内部および外部のデータベースの両方がサポートされています。
