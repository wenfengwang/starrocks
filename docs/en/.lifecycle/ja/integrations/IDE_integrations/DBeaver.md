---
displayed_sidebar: "Japanese"
---

# DBeaver

DBeaverはSQLクライアントソフトウェアアプリケーションおよびデータベース管理ツールであり、データベースへの接続プロセスを案内する便利なアシスタントを提供しています。

## 前提条件

DBeaverがインストールされていることを確認してください。

DBeaver Communityエディションは[https://dbeaver.io](https://dbeaver.io/)からダウンロードできます。また、DBeaver PROエディションは[https://dbeaver.com](https://dbeaver.com/)からダウンロードできます。

## 統合

データベースに接続するための手順は以下の通りです：

1. DBeaverを起動します。

2. DBeaverウィンドウの左上隅にあるプラス記号（**+**）アイコンをクリックするか、メニューバーで**Database** > **New Database Connection**を選択してアシスタントにアクセスします。

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_2.png)

3. MySQLドライバを選択します。

   **データベースの選択**ステップでは、利用可能なドライバのリストが表示されます。左側のペインで**Analytical**をクリックしてMySQLドライバを素早く見つけることができます。その後、**MySQL**アイコンをダブルクリックします。

   ![DBeaver - データベースの選択](../../assets/IDE_dbeaver_3.png)

4. データベースへの接続を設定します。

   **接続設定**ステップでは、**Main**タブに移動し、以下の必須の接続設定を構成します：

   - **Server Host**: StarRocksクラスタのFEホストのIPアドレス。
   - **Port**: StarRocksクラスタのFEクエリポート（例：`9030`）。
   - **Database**: StarRocksクラスタのターゲットデータベース。内部および外部の両方のデータベースがサポートされていますが、外部データベースの機能は不完全な場合があります。
   - **Username**: StarRocksクラスタにログインするためのユーザー名（例：`admin`）。
   - **Password**: StarRocksクラスタにログインするためのパスワード。

   ![DBeaver - 接続設定 - Mainタブ](../../assets/IDE_dbeaver_4.png)

   必要に応じて、**Driver properties**タブでMySQLドライバのプロパティを表示および編集することもできます。特定のプロパティを編集するには、そのプロパティの**Value**列の行をクリックします。

   ![DBeaver - 接続設定 - Driver propertiesタブ](../../assets/IDE_dbeaver_5.png)

5. データベースへの接続をテストします。

   **Test Connection**をクリックして接続設定の正確性を確認します。MySQLドライバの情報が表示されるダイアログボックスが表示されます。ダイアログボックスで**OK**をクリックして情報を確認します。接続設定を正常に構成した後、**Finish**をクリックしてプロセスを完了します。

   ![DBeaver - 接続のテスト](../../assets/IDE_dbeaver_6.png)

6. データベースに接続します。

   接続が確立されると、左側のデータベース接続ツリーに表示され、DBeaverは効果的にデータベースに接続できます。

   ![DBeaver - データベースに接続](../../assets/IDE_dbeaver_7.png)
