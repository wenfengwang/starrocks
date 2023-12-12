---
displayed_sidebar: "Japanese"
---

# DBeaver（ディービーバー）

DBeaver（ディービーバー）は、SQLクライアントソフトウェアアプリケーションおよびデータベース管理ツールです。このツールには、データベースへの接続手順を案内する便利なアシスタントが備わっています。

## 前提条件

DBeaver（ディービーバー）がインストールされていることを確認してください。

DBeaver Communityエディションは[https://dbeaver.io](https://dbeaver.io/)からダウンロードできます。DBeaver PROエディションは[https://dbeaver.com](https://dbeaver.com/)から入手可能です。

## 統合

以下の手順に従ってデータベースに接続してください。

1. DBeaver（ディービーバー）を起動します。

2. DBeaverウィンドウの左上隅にあるプラス記号（**+**）アイコンをクリックするか、メニューバーで**Database** > **New Database Connection**を選択してアシスタントにアクセスします。

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - アシスタントにアクセス](../../assets/IDE_dbeaver_2.png)

3. MySQLドライバを選択します。

   **Select your database**ステップでは、利用可能なドライバのリストが表示されます。MySQLドライバを素早く見つけるには、左側のペインで**Analytical**をクリックし、次に**MySQL**アイコンをダブルクリックします。

   ![DBeaver - データベースを選択](../../assets/IDE_dbeaver_3.png)

4. データベースへの接続を構成します。

   **Connection Settings**ステップでは、**Main**タブに移動し、次の重要な接続設定を構成します。

   - **サーバーホスト**: StarRocksクラスタのFEホストIPアドレス。
   - **ポート**: StarRocksクラスタのFEクエリポート（たとえば`9030`）。
   - **データベース**: StarRocksクラスタのターゲットデータベース。内部および外部データベースの両方がサポートされていますが、外部データベースの機能は不完全である場合があります。
   - **ユーザー名**: StarRocksクラスタにログインするためのユーザー名（たとえば`admin`）。
   - **パスワード**: StarRocksクラスタにログインするためのパスワード。

   ![DBeaver - Connection Settings - Main tab](../../assets/IDE_dbeaver_4.png)

   必要に応じて、**Driver properties**タブでMySQLドライバのプロパティを表示および編集することもできます。特定のプロパティを編集するには、そのプロパティの**Value**列をクリックします。

   ![DBeaver - Connection Settings - Driver properties tab](../../assets/IDE_dbeaver_5.png)

5. データベースへの接続をテストします。

   接続設定の正確性を確認するために**Test Connection**をクリックします。MySQLドライバの情報が表示されるダイアログボックスが表示されます。情報を確認するには、ダイアログボックスで**OK**をクリックします。接続設定を正常に構成した後は、**Finish**をクリックしてプロセスを完了します。

   ![DBeaver - 接続をテスト](../../assets/IDE_dbeaver_6.png)

6. データベースに接続します。

   接続が確立された後は、左側のデータベース接続ツリーやDBeaver内でデータベースに効果的に接続できます。

   ![DBeaver - データベースに接続](../../assets/IDE_dbeaver_7.png)