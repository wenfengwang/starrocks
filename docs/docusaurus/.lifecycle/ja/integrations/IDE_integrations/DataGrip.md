---
displayed_sidebar: "Japanese"
---

# DataGrip

DataGripはStarRocks内部データと外部データの両方をクエリサポートしています。

DataGripでデータソースを作成します。データソースとしてMySQLを選択する必要があります。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

構成する必要があるパラメータは次のように説明されています:

- **ホスト**:StarRocksクラスターのFEホストIPアドレスです。
- **ポート**:StarRocksクラスターのFEクエリポート、例えば`9030`です。
- **認証**:使用したい認証メソッドを選択します。**ユーザー名とパスワード**を選択します。
- **ユーザー**:StarRocksクラスターにログインするために使用されるユーザー名です。例えば`admin`です。
- **パスワード**:StarRocksクラスターにログインするために使用されるパスワードです。
- **データベース**:StarRocksクラスターでアクセスしたいデータソースです。このパラメータの値は`<catalog_name>.<database_name>`形式です。
  - `catalog_name`:StarRocksクラスター内のターゲットカタログの名前です。内部データと外部データの両方がサポートされています。
  - `database_name`:StarRocksクラスター内のターゲットデータベースの名前です。内部データと外部データの両方がサポートされています。