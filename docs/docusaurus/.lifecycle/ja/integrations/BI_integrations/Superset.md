---
displayed_sidebar: "Japanese"
---

# Apache Superset

Apache Supersetは、StarRocks内部データおよび外部データのクエリおよび可視化をサポートしています。

## 前提条件

次のインストールが完了していることを確認してください。

1. Apache SupersetサーバーにStarRocksのPythonクライアントをインストールしてください。

   ```SQL
   pip install starrocks
   ```

2. Apache Supersetの最新バージョンをインストールしてください。詳細については、[ゼロからのSupersetのインストール](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)を参照してください。

## 統合

Apache Supersetでデータベースを作成します：

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

次のポイントに注意してください：

- **SUPPORTED DATABASES**では、データソースとして**StarRocks**を選択してください。
- **SQLALCHEMY URI**では、次のようにStarRocks SQLAlchemy URI形式のURIを入力してください：

  ```SQL
  starrocks://<User>:<Password>@<Host>:<Port>/<Catalog>.<Database>
  ```

  URIのパラメータは次のように記載されています：

  - `User`：StarRocksクラスタにログインするためのユーザー名。例: `admin`。
  - `Password`：StarRocksクラスタにログインするためのパスワード。
  - `Host`：StarRocksクラスタのFEホストIPアドレス。
  - `Port`：StarRocksクラスタのFEクエリポート。例: `9030`。
  - `Catalog`：StarRocksクラスタの対象カタログ。内部および外部の両方のカタログがサポートされています。
  - `Database`：StarRocksクラスタの対象データベース。内部および外部の両方のデータベースがサポートされています。
  