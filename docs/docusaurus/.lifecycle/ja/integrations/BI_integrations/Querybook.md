---
displayed_sidebar: "Japanese"
---

# クエリブック

Querybookは、StarRocksで内部データと外部データの両方を問い合わせて可視化することをサポートしています。

## 前提条件

次の準備が完了していることを確認してください。

1. Querybookリポジトリをクローンしてダウンロードします。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. プロジェクトのルートディレクトリの`requirements`フォルダに、`local.txt`という名前のファイルを作成します。

   ```SQL
   touch requirements/local.txt
   ```

3. 必要なパッケージを追加します。

   ```SQL
   echo -e "starrocks\nmysqlclient" > requirements/local.txt 
   ```

4. コンテナを起動します。

   ```SQL
   make
   ```

## 統合

[https:///admin/query_engine/](https://localhost:10001/admin/query_engine/)を訪れ、新しいクエリエンジンを追加します:

![Querybook](../../assets/BI_querybook_1.png)

次の点に注意してください:

- **言語**として、**Starrocks**を選択します。
- **実行エンジン**として、**sqlalchemy**を選択します。
- **Connection_string**には、次のようにStarRocks SQLAlchemy URI形式のURIを入力します:

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URIのパラメータは以下のように説明されています:

  - `User`: StarRocksクラスタにログインするために使用されるユーザー名。たとえば、`admin`です。
  - `Password`: StarRocksクラスタにログインするために使用されるパスワードです。
  - `Host`: StarRocksクラスタのFEホストIPアドレスです。
  - `Port`: StarRocksクラスタのFEクエリポート。たとえば、`9030`です。
  - `Catalog`: StarRocksクラスタのターゲットカタログです。内部および外部のカタログの両方がサポートされています。
  - `Database`: StarRocksクラスタのターゲットデータベースです。内部および外部のデータベースの両方がサポートされています。