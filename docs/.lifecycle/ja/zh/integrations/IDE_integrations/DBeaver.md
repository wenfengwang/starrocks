---
displayed_sidebar: Chinese
---

# DBeaver

SQLクライアントアプリケーションとして、DBeaverは強力なデータベース管理機能を提供し、アシスタントを通じてデータベースに迅速に接続する手助けをします。

## 前提条件

DBeaverがインストールされていることを確認してください。

[https://dbeaver.io](https://dbeaver.io/) からDBeaver Community Editionをダウンロードしてインストールするか、[https://dbeaver.com](https://dbeaver.com/) からDBeaver PRO版をダウンロードしてインストールしてください。

## 統合

以下の手順でデータベースに接続します：

1. DBeaverを起動します。
2. DBeaverウィンドウの左上隅にあるプラス記号（**+**）のアイコンをクリックするか、メニューバーで **Database** > **New Database Connection** を選択して、アシスタントを開きます。

   ![DBeaver - アシスタントへのアクセス](../../assets/IDE_dbeaver_1.png)

   ![DBeaver - アシスタントへのアクセス](../../assets/IDE_dbeaver_2.png)

3. MySQLドライバーを選択します。

   **Select your database** ウィンドウで、サポートされているすべてのドライバーを見ることができます。ウィンドウの左側で **Analytical** をクリックすると、MySQLドライバーをすばやく見つけることができます。その後、**MySQL** アイコンをダブルクリックします。

   ![DBeaver - データベースの選択](../../assets/IDE_dbeaver_3.png)

4. データベース接続を設定します。

   **Connection Settings** ウィンドウで、**Main** タブに進み、以下の接続情報を設定します（以下の情報はすべて必須です）：

   - **Server Host**：StarRocksクラスタのFEホストIPアドレス。
   - **Port**：StarRocksクラスタのFEクエリポート、例えば `9030`。
   - **Database**：StarRocksクラスタ内の対象データベース。内部データベースと外部データベースの両方がサポートされていますが、外部データベースは機能が完全ではない場合があります。
   - **Username**：StarRocksクラスタにログインするためのユーザー名、例えば `admin`。
   - **Password**：StarRocksクラスタにログインするためのパスワード。

   ![DBeaver - 接続設定 - メインタブ](../../assets/IDE_dbeaver_4.png)

   **Driver properties** タブでは、MySQLドライバーの各プロパティを確認し、プロパティがある行の **Value** 列をクリックして、そのプロパティを編集することができます。

   ![DBeaver - 接続設定 - ドライバのプロパティタブ](../../assets/IDE_dbeaver_5.png)

5. データベース接続をテストします。

   **Test Connection** をクリックして、データベース接続情報の正確さを検証します。システムは以下のようなダイアログボックスを返し、設定情報を確認するように促します。**OK** をクリックして設定情報が正確であることを確認します。その後、**Finish** をクリックして接続設定を完了します。

   ![DBeaver - 接続テスト](../../assets/IDE_dbeaver_6.png)

6. データベースに接続します。

   データベース接続が確立された後、左側のデータベース接続ナビゲーションツリーでその接続を見ることができ、DBeaverを通じて迅速にデータベースに接続することができます。

   ![DBeaver - データベースの接続](../../assets/IDE_dbeaver_7.png)
