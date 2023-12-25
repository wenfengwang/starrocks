---
displayed_sidebar: English
---

# Querybook

Querybookは、StarRocksの内部データと外部データの両方に対するクエリ実行と可視化をサポートしています。

## 前提条件

以下の準備が完了していることを確認してください：

1. Querybookリポジトリをクローンし、ダウンロードします。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. プロジェクトのルートディレクトリにある`requirements`フォルダ内に`local.txt`という名前のファイルを作成します。

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

[https://localhost:10001/admin/query_engine/](https://localhost:10001/admin/query_engine/)にアクセスし、新しいクエリエンジンを追加します：

![Querybook](../../assets/BI_querybook_1.png)

以下の点に注意してください：

- **Language**で**StarRocks**を選択します。
- **Executor**で**sqlalchemy**を選択します。
- **Connection_string**には、以下のStarRocks SQLAlchemy URI形式でURIを入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URIのパラメータは以下の通りです：

  - `User`: StarRocksクラスターにログインするためのユーザー名（例：`admin`）。
  - `Password`: StarRocksクラスターにログインするためのパスワード。
  - `Host`: StarRocksクラスターのFEホストIPアドレス。
  - `Port`: StarRocksクラスターのFEクエリポート（例：`9030`）。
  - `Catalog`: StarRocksクラスター内の対象カタログ。内部カタログと外部カタログの両方がサポートされています。
  - `Database`: StarRocksクラスター内の対象データベース。内部データベースと外部データベースの両方がサポートされています。
