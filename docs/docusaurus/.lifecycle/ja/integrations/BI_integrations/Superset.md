---
displayed_sidebar: "Japanese"
---

# Apache Superset

Apache Supersetは、StarRocksの内部データと外部データの両方をクエリおよび可視化することをサポートしています。

## 前提条件

以下のインストールが完了していることを確認してください。

1. Apache SupersetサーバーにStarRocksのPythonクライアントをインストールします。

   ```SQL
   pip install starrocks
   ```

2. Apache Supersetの最新バージョンをインストールします。詳細については、[Installing Superset from Scratch](https://superset.apache.org/docs/installation/installing-superset-from-scratch/)を参照してください。

## 統合

Apache Supersetでデータベースを作成します。

![Apache Superset - 1](../../assets/BI_superset_1.png)

![Apache Superset - 2](../../assets/BI_superset_2.png)

次の点に注意してください：

- **SUPPORTED DATABASES** で **StarRocks** を選択し、これをデータソースとして使用します。
- **SQLALCHEMY URI** では、以下の形式のStarRocks SQLAlchemy URIを入力します：

  ```SQL
  starrocks://<ユーザー>:<パスワード>@<ホスト>:<ポート>/<カタログ>.<データベース>
  ```

  URIのパラメータは以下のように説明されています：

  - `ユーザー`：StarRocksクラスタにログインするためのユーザー名、例：`admin`。
  - `パスワード`：StarRocksクラスタにログインするためのパスワード。
  - `ホスト`：StarRocksクラスタのFEホストIPアドレス。
  - `ポート`：StarRocksクラスタのFEクエリポート、例：`9030`。
  - `カタログ`：StarRocksクラスタ内のターゲットカタログ。内部および外部のカタログの両方がサポートされています。
  - `データベース`：StarRocksクラスタ内のターゲットデータベース。内部および外部のデータベースの両方がサポートされています。
