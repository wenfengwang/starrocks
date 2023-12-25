---
displayed_sidebar: Chinese
---

# Querybook

Querybook は StarRocks の内部データと外部データのクエリと可視化処理をサポートしています。

## 前提条件

以下の準備作業が完了していることを確認してください：

1. Querybook リポジトリをクローンしてダウンロードします。

   ```SQL
   git clone git@github.com:pinterest/querybook.git
   cd querybook
   ```

2. プロジェクトのルートディレクトリにある `requirements` フォルダ内に `local.txt` ファイルを作成します。

   ```SQL
   touch requirements/local.txt
   ```

3. 必要なパッケージをファイルに追加します。

   ```SQL
   echo -e "starrocks\nmysqlclient" > requirements/local.txt 
   ```

4. コンテナを起動します。

   ```SQL
   make
   ```

## 統合

[https://localhost:10001/admin/query_engine/](https://localhost:10001/admin/query_engine/) にアクセスして、クエリエンジンを追加します。

![Querybook](../../assets/BI_querybook_1.png)

- **Language** で **StarRocks** を選択します。
- **Executor** で **sqlalchemy** を選択します。
- **Connection_string** に以下の StarRocks SQLAlchemy URI 形式に従って URI を入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI のパラメーター説明は以下の通りです：

  - `User`：StarRocks クラスタにログインするためのユーザー名、例えば `admin`。
  - `Password`：StarRocks クラスタにログインするためのパスワード。
  - `Host`：StarRocks クラスタの FE ホストの IP アドレス。
  - `Port`：StarRocks クラスタの FE クエリポート、例えば `9030`。
  - `Catalog`：StarRocks クラスタ内の対象カタログ。Internal Catalog と External Catalog がサポートされています。
  - `Database`：StarRocks クラスタ内の対象データベース。内部データベースと外部データベースがサポートされています。
