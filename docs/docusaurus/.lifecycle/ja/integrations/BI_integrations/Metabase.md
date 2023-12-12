---
displayed_sidebar: "Japanese"
---

# Metabase

Metabaseは、StarRocks内部データと外部データの両方をクエリおよび視覚化することをサポートしています。

Metabaseを起動して、以下の手順を実行します。

1. Metabaseのホームページの右上隅で、**設定**アイコンをクリックして**管理設定**を選択します。

   ![Metabase - 管理設定](../../assets/Metabase/Metabase_1.png)

2. 上部メニューバーで**データベース**を選択します。

3. **データベース**ページで、**データベースを追加**をクリックします。

   ![Metabase - データベースを追加](../../assets/Metabase/Metabase_2.png)

4. 表示されるページで、データベースのパラメータを構成し、**保存**をクリックします。

   - **データベースの種類**：**MySQL**を選択します。
   - **ホスト**と**ポート**：使用するケースに適したホストとポート情報を入力します。
   - **データベース名**：`<catalog_name>.<database_name>`形式でデータベース名を入力します。StarRocks v3.2より前のバージョンでは、StarRocksクラスターの内部カタログのみをMetabaseと統合できます。StarRocks v3.2以降、StarRocksクラスターの内部カタログと外部カタログの両方をMetabaseと統合できます。
   - **ユーザー名**と**パスワード**：StarRocksクラスターのユーザー名とパスワードを入力します。

   他のパラメータにはStarRocksは関与しません。ビジネスの要件に基づいて構成してください。

   ![Metabase - データベースの構成](../../assets/Metabase/Metabase_3.png)