---
displayed_sidebar: English
---

# Apache Superset

Apache Superset は StarRocks の内部データと外部データの両方に対するクエリと可視化をサポートしています。

## 前提条件

以下のインストールが完了していることを確認してください：

1. StarRocks の Python クライアントを Apache Superset サーバーにインストールします。

   ```SQL
   pip install starrocks
   ```

2. Apache Superset の最新バージョンをインストールします。詳細は[Superset をゼロからインストールする](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)をご覧ください。

## 統合

Apache Superset でデータベースを作成します：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

以下の点に注意してください：

- **SUPPORTED DATABASES** で、データソースとして使用される **StarRocks** を選択します。
- **SQLALCHEMY URI** には、以下の StarRocks SQLAlchemy URI 形式で URI を入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URI のパラメータは以下の通りです：

  - `User`: StarRocks クラスタへのログインに使用するユーザー名です（例：`admin`）。
  - `Password`: StarRocks クラスタへのログインに使用するパスワード。
  - `Host`: StarRocks クラスタの FE ホスト IP アドレス。
  - `Port`: StarRocks クラスタの FE クエリポート（例：`9030`）。
  - `Catalog`: StarRocks クラスタ内の対象カタログ。内部カタログと外部カタログの両方がサポートされています。
  - `Database`: StarRocks クラスタ内の対象データベース。内部データベースと外部データベースの両方がサポートされています。
  