---
displayed_sidebar: English
---

# Tableau Desktop

Tableau Desktop は、StarRocks の内部データと外部データの両方をクエリして視覚化することをサポートしています。

Tableau Desktop でデータベースを作成する方法：

![Tableau Desktop](../../assets/BI_tableau_1.png)

次の点に注意してください：

- データソースとして **Other Databases(****JDBC****)** を選択します。
- **Dialect** で **MySQL** を選択します。
- **URL** には、以下のような MySQL URI 形式の URL を入力します：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Database>
  ```

  URL のパラメータは以下の通りです：

  - `Host`: StarRocks クラスタの FE ホスト IP アドレス。
  - `Port`: StarRocks クラスタの FE クエリポート、例えば `9030`。
  - `Catalog`: StarRocks クラスタ内の対象カタログ。内部カタログと外部カタログの両方がサポートされています。
  - `Database`: StarRocks クラスタ内の対象データベース。内部データベースと外部データベースの両方がサポートされています。
- **Username** と **Password** を設定します。
  - **Username**: StarRocks クラスタにログインするためのユーザー名、例えば `admin`。
  - **Password**: StarRocks クラスタにログインするためのパスワード。
