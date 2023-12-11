---
displayed_sidebar: "Japanese"
---

# Tableau Desktop

Tableau Desktopは、StarRocks内部データと外部データの両方のクエリと視覚化をサポートしています。

Tableau Desktopでデータベースを作成するには:

![Tableau Desktop](../../assets/BI_tableau_1.png)

以下の点に注意してください:

- **Other Databases(JDBC)**をデータソースとして選択します。
- **方言**には、**MySQL**を選択します。
- **URL**には、以下のようなMySQL URI形式のURLを入力します:

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Databases>
  ```

  URLのパラメータは以下のように説明されています:

  - `Host`：StarRocksクラスターのFEホストIPアドレスです。
  - `Port`：StarRocksクラスターのFEクエリポートです。たとえば、`9030`です。
  - `Catalog`：StarRocksクラスター内のターゲットカタログです。内部カタログと外部カタログの両方がサポートされています。
  - `Database`：StarRocksクラスター内のターゲットデータベースです。内部データベースと外部データベースの両方がサポートされています。
- **ユーザー名**と**パスワード**を設定します。
  - **ユーザー名**：StarRocksクラスターにログインする際に使用するユーザー名です。たとえば、`admin`です。
  - **パスワード**：StarRocksクラスターにログインする際に使用するパスワードです。