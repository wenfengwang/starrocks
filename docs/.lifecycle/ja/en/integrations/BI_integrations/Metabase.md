---
displayed_sidebar: English
---

# Metabase

Metabaseは、StarRocksの内部データと外部データの両方に対するクエリと視覚化をサポートしています。

Metabaseを起動し、以下の手順に従ってください：

1. Metabaseのホームページの右上隅にある**設定**アイコンをクリックし、**管理設定**を選択します。

   ![Metabase - 管理設定](../../assets/Metabase/Metabase_1.png)

2. 上部のメニューバーで**データベース**を選択します。

3. **データベース**ページで、**データベースを追加**をクリックします。

   ![Metabase - データベースを追加](../../assets/Metabase/Metabase_2.png)

4. 表示されるページで、データベースパラメータを設定し、**保存**をクリックします。

   - **データベースタイプ**: **MySQL**を選択します。
   - **ホスト**と**ポート**: ご利用のユースケースに適したホストとポート情報を入力します。
   - **データベース名**: `<catalog_name>.<database_name>`の形式でデータベース名を入力します。v3.2より前のStarRocksバージョンでは、StarRocksクラスタの内部カタログのみをMetabaseと統合できます。StarRocks v3.2以降では、StarRocksクラスタの内部カタログと外部カタログの両方をMetabaseに統合できます。
   - **ユーザー名**と**パスワード**: StarRocksクラスタのユーザー名とパスワードを入力します。

   その他のパラメータはStarRocksには関係ありません。ビジネスニーズに基づいて設定してください。

   ![Metabase - データベースを設定](../../assets/Metabase/Metabase_3.png)
