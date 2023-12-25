---
displayed_sidebar: Chinese
---

# Hex

Hex は StarRocks 内のデータと外部データのクエリと可視化処理をサポートしています。

Hex にデータ接続を追加します。MySQL を接続タイプとして選択する必要があります。

![Hex](../../assets/BI_hex_1.png)

設定が必要なパラメータは以下の通りです：

- **Name**: データ接続の名前。
- **Host &** **port**: StarRocks クラスタの FE ホストの IP アドレスと FE クエリポート。クエリポートは例えば `9030` に設定できます。
- **Database**: StarRocks クラスタでアクセスするデータソース。形式は `<catalog_name>.<database_name>` です。
  - `catalog_name`: StarRocks クラスタ内の対象 Catalog の名前。Internal Catalog と External Catalog がサポートされています。
  - `database_name`: StarRocks クラスタ内の対象データベースの名前。内部データベースと外部データベースがサポートされています。
- **Type**: 認証方式。**Password** を選択します。
- **User**: StarRocks クラスタにログインするユーザー名、例えば `admin`。
- **Password**: StarRocks クラスタにログインするユーザーのパスワード。
