---
displayed_sidebar: Chinese
---

# Apache Superset

Apache Superset は StarRocks の内部データおよび外部データに対するクエリと可視化をサポートしています。

## 前提条件

以下のツールのインストールが完了していることを確認してください：

1. Apache Superset サーバーに StarRocks の Python クライアントをインストールします。

   ```SQL
   pip install starrocks
   ```

2. 最新バージョンの Apache Superset をインストールします。詳細は[Superset のインストール](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)を参照してください。

## 統合

Apache Superset でデータベースを作成します：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

データベースを作成する際には、以下の点に注意してください：

- **SUPPORTED DATABASES** で **StarRocks** をデータソースとして選択します。
- **SQLALCHEMY URI** に、以下の StarRocks SQLAlchemy URI の形式で URI を入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI のパラメーターは以下の通りです：

  - `User`：StarRocks クラスターにログインするためのユーザー名（例：`admin`）。
  - `Password`：StarRocks クラスターにログインするためのパスワード。
  - `Host`：StarRocks クラスターの FE ホストの IP アドレス。
  - `Port`：StarRocks クラスターの FE クエリポート（例：`9030`）。
  - `Catalog`：StarRocks クラスター内の対象カタログ。Internal Catalog および External Catalog がサポートされています。
  - `Database`：StarRocks クラスター内の対象データベース。内部データベースおよび外部データベースがサポートされています。
