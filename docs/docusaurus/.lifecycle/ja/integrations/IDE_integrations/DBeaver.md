---
displayed_sidebar: "Japanese"
---

# DBeaver（デービーバー）

DBeaver（デービーバー）は、SQLクライアントソフトウェアアプリケーションおよびデータベース管理ツールであり、データベースに接続するプロセスを手助けする便利なアシスタント機能を提供しています。

## 前提条件

DBeaverをインストールしていることを確認してください。

DBeaver Communityエディションは[https://dbeaver.io](https://dbeaver.io/)から、DBeaver PROエディションは[https://dbeaver.com](https://dbeaver.com/)からダウンロードできます。

## 統合

データベースに接続するために、以下の手順に従ってください:

1. DBeaverを起動します。

2. DBeaverウィンドウの左上隅にあるプラス記号（**+**）アイコンをクリックするか、メニューバーで **Database** > **New Database Connection** を選択してアシスタントにアクセスします。

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_2.png)

3. MySQLドライバを選択します。

   **データベースを選択** のステップで、利用可能なドライバのリストが表示されます。MySQLドライバを素早く見つけるために、左側のペインで **Analytical** をクリックしてから **MySQL** アイコンをダブルクリックします。

   ![DBeaver - データベースを選択](../../assets/IDE_dbeaver_3.png)

4. データベースへの接続を構成します。

   **接続設定** のステップで、**Main** タブに移動して以下の必須接続設定を構成します:

   - **Server Host**: StarRocksクラスターのFEホストIPアドレス。
   - **Port**: StarRocksクラスターのFEクエリポート（例: `9030`）。
   - **Database**: StarRocksクラスターのターゲットデータベース。内部および外部の両方のデータベースがサポートされていますが、外部データベースの機能は不完全かもしれません。
   - **Username**: StarRocksクラスターにログインするために使用されるユーザー名（例: `admin`）。
   - **Password**: StarRocksクラスターにログインするために使用されるパスワード。

   ![DBeaver - 接続設定 - Mainタブ](../../assets/IDE_dbeaver_4.png)

   必要に応じて、**Driver properties** タブでMySQLドライバのプロパティを表示および編集できます。特定のプロパティを編集するには、そのプロパティの **Value** 列の行をクリックします。

   ![DBeaver - 接続設定 - Driver propertiesタブ](../../assets/IDE_dbeaver_5.png)

5. データベースへの接続をテストします。

   **Test Connection** をクリックして接続設定の正確さを検証します。MySQLドライバの情報が表示されるダイアログボックスが表示されます。情報を確認するために、ダイアログボックス内の **OK** をクリックします。接続設定の構成が完了したら、 **Finish** をクリックしてプロセスを完了します。

   ![DBeaver - テスト接続](../../assets/IDE_dbeaver_6.png)

6. データベースに接続します。

   接続が確立されたら、左側のデータベース接続ツリーで確認でき、DBeaverは効果的にデータベースに接続できます。

   ![DBeaver - データベースに接続](../../assets/IDE_dbeaver_7.png)
