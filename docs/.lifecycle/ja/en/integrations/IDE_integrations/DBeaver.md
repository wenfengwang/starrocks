---
displayed_sidebar: English
---

# DBeaver

DBeaverはSQLクライアントソフトウェアアプリケーションおよびデータベース管理ツールであり、データベースへの接続プロセスを順を追って案内する便利なアシスタントを提供します。

## 前提条件

DBeaverがインストールされていることを確認してください。

DBeaver Community Editionは[https://dbeaver.io](https://dbeaver.io/)から、DBeaver PRO Editionは[https://dbeaver.com](https://dbeaver.com/)からダウンロードできます。

## 統合

データベースに接続するには、以下の手順に従ってください。

1. DBeaverを起動します。

2. DBeaverウィンドウの左上隅にあるプラス記号(**+**)アイコンをクリックするか、メニューバーで**Database** > **New Database Connection**を選択してアシスタントにアクセスします。

   ![DBeaver - アシスタントへのアクセス](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - アシスタントへのアクセス](../../assets/IDE_dbeaver_2.png)

3. MySQLドライバを選択します。

   **Select your database**ステップでは、利用可能なドライバーのリストが表示されます。左側のペインで**Analytical**をクリックしてMySQLドライバをすばやく見つけます。その後、**MySQL**アイコンをダブルクリックします。

   ![DBeaver - データベースの選択](../../assets/IDE_dbeaver_3.png)

4. データベースへの接続を設定します。

   **Connection Settings**ステップで、**Main**タブに移動し、以下の重要な接続設定を行います：

   - **Server Host**: StarRocksクラスタのFEホストIPアドレス。
   - **Port**: StarRocksクラスタのFEクエリポート、例えば`9030`。
   - **Database**: StarRocksクラスタ内の対象データベース。内部データベースと外部データベースがサポートされていますが、外部データベースの機能は不完全かもしれません。
   - **Username**: StarRocksクラスタにログインするためのユーザー名、例えば`admin`。
   - **Password**: StarRocksクラスタにログインするためのパスワード。

   ![DBeaver - 接続設定 - メインタブ](../../assets/IDE_dbeaver_4.png)

   必要に応じて、**Driver properties**タブでMySQLドライバのプロパティを表示および編集することもできます。特定のプロパティを編集するには、そのプロパティの**Value**列の行をクリックします。

   ![DBeaver - 接続設定 - ドライバプロパティタブ](../../assets/IDE_dbeaver_5.png)

5. データベースへの接続をテストします。

   **Test Connection**をクリックして接続設定の正確性を確認します。MySQLドライバの情報が表示されるダイアログボックスが現れます。ダイアログボックスで**OK**をクリックして情報を確認します。接続設定が正しく構成された後、**Finish**をクリックしてプロセスを完了します。

   ![DBeaver - 接続テスト](../../assets/IDE_dbeaver_6.png)

6. データベースに接続します。

   接続が確立された後、左側のデータベース接続ツリーで表示され、DBeaverはデータベースに効果的に接続できるようになります。

   ![DBeaver - データベース接続](../../assets/IDE_dbeaver_7.png)
