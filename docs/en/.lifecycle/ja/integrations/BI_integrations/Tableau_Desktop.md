---
displayed_sidebar: "Japanese"
---

# Tableau Desktop

Tableau Desktopは、内部データと外部データの両方をStarRocksでクエリと可視化することができます。

Tableau Desktopでデータベースを作成する方法:

![Tableau Desktop](../../assets/BI_tableau_1.png)

以下のポイントに注意してください:

- データソースとして**その他のデータベース(JDBC)**を選択します。
- **Dialect**には**MySQL**を選択します。
- **URL**には、以下のMySQL URI形式のURLを入力します:

  ```SQL
  jdbc:mysql://<ホスト>:<ポート>/<カタログ>.<データベース>
  ```

  URLのパラメータは以下のように説明されています:

  - `ホスト`: StarRocksクラスタのFEホストIPアドレスです。
  - `ポート`: StarRocksクラスタのFEクエリポートです。例: `9030`。
  - `カタログ`: StarRocksクラスタのターゲットカタログです。内部および外部のカタログの両方がサポートされています。
  - `データベース`: StarRocksクラスタのターゲットデータベースです。内部および外部のデータベースの両方がサポートされています。
- **ユーザー名**と**パスワード**を設定します。
  - **ユーザー名**: StarRocksクラスタにログインするために使用されるユーザー名です。例: `admin`。
  - **パスワード**: StarRocksクラスタにログインするために使用されるパスワードです。
