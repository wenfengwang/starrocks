---
displayed_sidebar: "Japanese"
---

# Querybook（クエリブック）

Querybookは、StarRocksの内部データと外部データの両方をクエリおよび可視化することができます。

## 前提条件

以下の準備が完了していることを確認してください：

1. Querybookリポジトリをクローンしてダウンロードします。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. プロジェクトのルートディレクトリの`requirements`フォルダに`local.txt`という名前のファイルを作成します。

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

[https:///admin/query_engine/](https://localhost:10001/admin/query_engine/)にアクセスし、新しいクエリエンジンを追加します：

![Querybook](../../assets/BI_querybook_1.png)

以下のポイントに注意してください：

- **Language**には**Starrocks**を選択します。
- **Executor**には**sqlalchemy**を選択します。
- **Connection_string**には、以下の形式のStarRocks SQLAlchemy URI形式のURIを入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URIのパラメータは以下のように説明されています：

  - `User`：StarRocksクラスタにログインするためのユーザー名（例：`admin`）。
  - `Password`：StarRocksクラスタにログインするためのパスワード。
  - `Host`：StarRocksクラスタのFEホストIPアドレス。
  - `Port`：StarRocksクラスタのFEクエリポート（例：`9030`）。
  - `Catalog`：StarRocksクラスタのターゲットカタログ。内部および外部のカタログの両方がサポートされています。
  - `Database`：StarRocksクラスタのターゲットデータベース。内部および外部のデータベースの両方がサポートされています。
