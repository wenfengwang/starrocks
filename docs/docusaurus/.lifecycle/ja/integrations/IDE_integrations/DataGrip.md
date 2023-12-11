---
displayed_sidebar: "日本語"
---

# DataGrip

DataGripはStarRocks内部データと外部データの両方をクエリすることができます。

DataGripでデータソースを作成します。データソースとしてMySQLを選択する必要があることに注意してください。

![DataGrip - 1](../../assets/BI_datagrip_1.png)

![DataGrip - 2](../../assets/BI_datagrip_2.png)

設定する必要があるパラメータは次のとおりです：

- **ホスト**: StarRocksクラスターのFEホストIPアドレス。
- **ポート**: StarRocksクラスターのFEクエリポート。たとえば、`9030`。
- **認証**: 使用したい認証方法。**ユーザー名とパスワード**を選択します。
- **ユーザー**: StarRocksクラスターにログインするために使用するユーザー名。たとえば、`admin`。
- **パスワード**: StarRocksクラスターにログインするために使用するパスワード。
- **データベース**: StarRocksクラスターでアクセスしたいデータソース。このパラメータの値は`<catalog_name>.<database_name>`形式です。
  - `catalog_name`: StarRocksクラスター内のターゲットカタログの名前。内部および外部の両方のカタログがサポートされています。
  - `database_name`: StarRocksクラスター内のターゲットデータベースの名前。内部および外部の両方のデータベースがサポートされています。