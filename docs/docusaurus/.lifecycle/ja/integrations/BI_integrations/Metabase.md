---
displayed_sidebar: "Japanese"
---

# Metabase

Metabaseは、StarRocksの内部データと外部データの両方のクエリと可視化をサポートしています。

Metabaseを起動し、以下のようにします：

1. Metabaseホームページの右上隅で、**設定**アイコンをクリックし、**管理者設定**を選択します。

   ![Metabase - 管理者設定](../../assets/Metabase/Metabase_1.png)

2. 上部メニューバーで**データベース**を選択します。

3. **データベース**ページで、**データベースを追加**をクリックします。

   ![Metabase - データベースを追加](../../assets/Metabase/Metabase_2.png)

4. 表示されるページで、データベースパラメータを構成し、**保存**をクリックします。

   - **データベースの種類**: **MySQL**を選択します。
   - **ホスト**と**ポート**: 使用状況に適したホストおよびポート情報を入力します。
   - **データベース名**: `<catalog_name>.<database_name>`形式でデータベース名を入力します。StarRocks v3.2より前のバージョンでは、StarRocksクラスタの内部カタログのみをMetabaseと統合できます。StarRocks v3.2以降では、StarRocksクラスタの内部カタログと外部カタログの両方をMetabaseと統合できます。
   - **ユーザー名**と**パスワード**: StarRocksクラスタユーザーのユーザー名とパスワードを入力します。

   その他のパラメータはStarRocksに関与しません。ビジネスの必要に応じて構成してください。

   ![Metabase - データベースの構成](../../assets/Metabase/Metabase_3.png)