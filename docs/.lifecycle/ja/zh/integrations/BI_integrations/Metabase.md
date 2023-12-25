---
displayed_sidebar: Chinese
---

# Metabase

Metabase は StarRocks 内の内部データと外部データのクエリと可視化をサポートしています。

Metabase にアクセスし、以下の手順に従って操作してください：

1. Metabase ホームページの右上隅にある **Settings** アイコンをクリックし、**Admin settings** を選択します。

   ![Metabase - Admin settings](../../assets/Metabase/Metabase_1.png)

2. ページ上部のメニューバーで **Databases** を選択します。

3. **Databases** ページで、**Add database** をクリックします。

   ![Metabase - Add database](../../assets/Metabase/Metabase_2.png)

4. ポップアップされたページで、データベースのパラメータを設定し、**Save** をクリックします。

   パラメータ設定には以下の点に注意が必要です：

   - **Database type**：データベースタイプとして **MySQL** を選択します。
   - **Host** と **Port**：使用シナリオに応じてホストとポート情報を入力します。
   - **Database name**：データベース名を `<catalog_name>.<database_name>` の形式で入力します。3.2 バージョン以前では、StarRocks は Internal Catalog のみを Metabase と統合をサポートしていました。3.2 バージョンからは、StarRocks は Internal Catalog と External Catalog の両方を Metabase と統合をサポートしています。
   - **Username** と **Password**：StarRocks クラスタのユーザー名とパスワードを入力します。

   その他のパラメータは StarRocks とは関係ありませんので、実際のニーズに応じて記入してください。

   ![Metabase - Configure database](../../assets/Metabase/Metabase_3.png)
