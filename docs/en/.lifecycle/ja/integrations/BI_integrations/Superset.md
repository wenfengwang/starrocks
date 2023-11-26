---
displayed_sidebar: "Japanese"
---

# Apache Superset

Apache Supersetは、StarRocksの内部データと外部データの両方をクエリおよび可視化することができます。

## 前提条件

以下のインストールが完了していることを確認してください。

1. Apache SupersetサーバーにStarRocksのPythonクライアントをインストールします。

   ```SQL
   pip install starrocks
   ```

2. 最新バージョンのApache Supersetをインストールします。詳細については、[Installing Superset from Scratch](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)を参照してください。

## 統合

Apache Supersetでデータベースを作成します：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

次のポイントに注意してください：

- **SUPPORTED DATABASES**で、データソースとして**StarRocks**を選択します。
- **SQLALCHEMY** **URI**で、以下のStarRocks SQLAlchemy URI形式のURIを入力します：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URIのパラメータは次のように説明されています：

  - `User`：StarRocksクラスターにログインするためのユーザー名（例：`admin`）。
  - `Password`：StarRocksクラスターにログインするためのパスワード。
  - `Host`：StarRocksクラスターのFEホストIPアドレス。
  - `Port`：StarRocksクラスターのFEクエリポート（例：`9030`）。
  - `Catalog`：StarRocksクラスターのターゲットカタログ。内部および外部のカタログの両方がサポートされています。
  - `Database`：StarRocksクラスターのターゲットデータベース。内部および外部のデータベースの両方がサポートされています。
