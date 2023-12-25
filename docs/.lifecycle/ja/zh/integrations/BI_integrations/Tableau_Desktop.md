---
displayed_sidebar: Chinese
---

# Tableau Desktop

Tableau Desktop は StarRocks の内部データおよび外部データに対するクエリと可視化処理をサポートしています。

Tableau Desktop でデータベースを作成するには：

![Tableau Desktop](../../assets/BI_tableau_1.png)

以下の点に注意してください：

- データソースとして **Other Databases(****JDBC****)** を選択します。
- **Dialect** で **MySQL** を選択します。
- **URL** には、以下の MySQL URI 形式で URL を入力します：

  ```SQL
  jdbc:mysql://<Host>:<Port>/<Catalog>.<Database>
  ```

  URL のパラメーター説明は以下の通りです：

  - `Host`：StarRocks クラスタの FE ホストの IP アドレス。
  - `Port`：StarRocks クラスタの FE クエリポート、例えば `9030`。
  - `Catalog`：StarRocks クラスタ内の対象 Catalog。Internal Catalog と External Catalog の両方がサポートされています。
  - `Database`：StarRocks クラスタ内の対象データベース。内部データベースと外部データベースの両方がサポートされています。

- **Username** と **Password** にユーザー名とパスワードを入力します。
  - **Username**：StarRocks クラスタにログインするためのユーザー名、例えば `admin`。
  - **Password**：StarRocks クラスタにログインするためのユーザーパスワード。
