---
displayed_sidebar: English
---

# AWSにStarRocksをデプロイする

StarRocksとAWSは、[AWSパートナーソリューション](https://aws.amazon.com/solutions/partners)を通じて、AWS上にStarRocksを迅速にデプロイするためのソリューションを提供しています。このトピックでは、StarRocksをデプロイし、アクセスするための手順をステップバイステップで説明します。

## 基本概念

[AWSパートナーソリューション](https://aws-ia.github.io/content/qs_info.html)

AWSパートナーソリューションは、AWSソリューションアーキテクトとAWSパートナーが構築した自動化されたリファレンスデプロイメントです。AWSパートナーソリューションは、AWSリソースとサードパーティリソース（StarRocksクラスターなど）をAWSクラウド上に自動的にデプロイする[AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)テンプレートを使用します。

[テンプレート](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)

テンプレートは、AWSリソースとサードパーティリソース、およびそれらのプロパティを記述するJSONまたはYAML形式のテキストファイルです。

[スタック](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2ab1b5c15b9)

スタックは、テンプレートに記述されたリソースを作成および管理するために使用されます。テンプレートを作成して、そのテンプレートを提出しスタックを作成することにより、AWS CloudFormationがそれらのリソースを設定します。

## StarRocksクラスターをデプロイする

1. [AWSアカウントにサインイン](https://console.aws.amazon.com/console/home)します。アカウントをお持ちでない場合は、[AWSでサインアップ](https://aws.amazon.com/)してください。

2. 上部のツールバーからAWSリージョンを選択します。

3. この[パートナーソリューション](https://aws.amazon.com/quickstart/architecture/starrocks-starrocks/)を起動するためのデプロイオプションを選択します。AWS CloudFormationコンソールが開き、1つのFEと3つのBEを持つStarRocksクラスターをデプロイするために使用される事前に入力されたテンプレートが表示されます。デプロイには約30分かかります。

   1. [新しいVPCにStarRocksをデプロイする](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-new-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=yo-6I1O2W0f0VcoqYOVvSwMmhRkC7Vod1M9vWbiMWUM&code_challenge_method=SHA-256)。このオプションは、VPC、サブネット、NATゲートウェイ、セキュリティグループ、バスチオンホスト、その他のインフラコンポーネントからなる新しいAWS環境を構築し、その新しいVPCにStarRocksをデプロイします。
   2. [既存のVPCにStarRocksをデプロイする](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fus-east-1.console.aws.amazon.com%2Fcloudformation%2Fhome%3Fregion%3Dus-east-1%26state%3DhashArgs%2523%252Fstacks%252Fnew%253FstackName%253Dstarrocks-starrocks%2526templateURL%253Dhttps%253A%252F%252Faws-quickstart.s3.us-east-1.amazonaws.com%252Fquickstart-starrocks-starrocks%252Ftemplates%252Fstarrocks-entrypoint-existing-vpc.template.yaml%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fcloudformation&forceMobileApp=0&code_challenge=dDa178BxB6UkFfrpADw5CIoZ4yDUNRTG7sNM1EO__eo&code_challenge_method=SHA-256)。このオプションは、既存のAWSインフラストラクチャにStarRocksをプロビジョニングします。

4. 正しいAWSリージョンを選択します。

5. **スタックの作成**ページで、テンプレートURLのデフォルト設定をそのまま使用し、**次へ**を選択します。

6. **スタックの詳細を指定**ページで、

   1. 必要に応じてスタック名をカスタマイズします。

   2. テンプレートのパラメータを構成し、確認します。

      1. 必要なパラメータを構成します。

         - 新しいVPCにStarRocksをデプロイする場合、以下のパラメータに注意してください：

             | タイプ                            | パラメータ                 | 必須                                             | 説明                                                  |
             | ------------------------------- | ------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ |
             | ネットワーク構成           | アベイラビリティゾーン        | はい                                                  | StarRocksクラスタをデプロイするための2つのアベイラビリティゾーンを選択します。詳細は[リージョンとゾーン](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-local-zones)を参照してください。 |

             | EC2 設定               | キーペア名             | はい                                                  | パブリックキーとプライベートキーから成るキーペアは、EC2 インスタンスに接続する際に身元を証明するために使用するセキュリティ認証情報です。詳細は[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 > 注意 > > キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
             | StarRocks クラスタ設定 | StarRocksのルートパスワード | はい                                                  | StarRocksのルートアカウントのパスワードを入力してください。ルートアカウントを使用してStarRocksクラスタに接続する際には、このパスワードが必要です。 |
             |  |ルートパスワードの確認           | はい                       | StarRocksのルートアカウントのパスワードを確認してください。 |                                                              |

         - 既存のVPCにStarRocksをデプロイする際には、以下のパラメータに注意してください：

           | 種別                            | パラメータ                 | 必須                                                     | 説明                                                  |
           | ------------------------------- | ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
           | ネットワーク設定           | VPC ID                    | はい                                                          | 既存のVPCのIDを入力してください。[AWS S3のVPCエンドポイントを設定する](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)ことを確認してください。 |
           | |プライベートサブネット1 ID             | はい                       | 既存のVPCのアベイラビリティゾーン1にあるプライベートサブネットのIDを入力してください（例：subnet-fe9a8b32）。 |                                                              |
           | |パブリックサブネット1 ID              | はい                       | 既存のVPCのアベイラビリティゾーン1にあるパブリックサブネットのIDを入力してください。 |                                                              |
           | |パブリックサブネット2 ID              | はい                       | 既存のVPCのアベイラビリティゾーン2にあるパブリックサブネットのIDを入力してください。 |                                                              |
           | EC2 設定               | キーペア名             | はい                                                          | パブリックキーとプライベートキーから成るキーペアは、EC2 インスタンスに接続する際に身元を証明するために使用するセキュリティ認証情報です。詳細は[キーペア](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。 <br /> **注記** <br /> キーペアを作成する必要がある場合は、[キーペアの作成](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照してください。 |
           | StarRocks クラスタ設定 | StarRocksのルートパスワード | はい                                                          | StarRocksのルートアカウントのパスワードを入力してください。ルートアカウントを使用してStarRocksクラスタに接続する際には、このパスワードが必要です。 |
           | |ルートパスワードの確認           | はい                       | StarRocksのルートアカウントのパスワードを確認してください。         |                                                              |

      2. その他のパラメータは、デフォルト設定をレビューし、必要に応じてカスタマイズしてください。

   3. パラメータの設定とレビューが完了したら、**次へ**を選択してください。

7. **スタックオプションの設定**ページで、デフォルト設定をそのまま使用し、**次へ**をクリックしてください。

8. **starrocks-starrocksのレビュー**ページで、上記で設定したスタック情報を確認してください。これにはテンプレート、詳細、その他のオプションが含まれます。詳細は[AWS CloudFormationコンソールでのスタックのレビューとスタックコストの見積もり](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)を参照してください。

    > **注記**
    >
    > パラメータを変更する必要がある場合は、関連セクションの右上隅にある**編集**をクリックして、該当するページに戻ってください。

9. 次の2つのチェックボックスを選択し、**スタックの作成**をクリックしてください。

    ![StarRocks_on_AWS_1](../assets/StarRocks_on_AWS_1.png)

    **このパートナーソリューションを実行する際に発生するAWSサービスおよびサードパーティライセンスのコストはお客様の責任です**。コスト見積もりについては、ご利用のAWSサービスごとの料金ページを参照してください。

## StarRocksクラスタへのアクセス

StarRocksクラスタはプライベートサブネットにデプロイされているため、まずEC2バスチオンホストに接続し、そこからStarRocksクラスタにアクセスする必要があります。

1. StarRocksクラスタへのアクセスに使用するEC2バスチオンホストに接続します。

   1. AWS CloudFormationコンソールで、`BastionStack`の**出力**タブから`EIP1`の値をメモしてください。
   ![StarRocks_on_AWS_2](../assets/StarRocks_on_AWS_2.png)

   2. EC2コンソールで、EC2バスチオンホストを選択します。
   ![StarRocks_on_AWS_3](../assets/StarRocks_on_AWS_3.png)

   3. EC2バスチオンホストに関連付けられているセキュリティグループのインバウンドルールを編集し、自分のマシンからEC2バスチオンホストへのトラフィックを許可するようにします。

   4. EC2バスチオンホストに接続します。

2. StarRocksクラスタにアクセスします。

   1. EC2バスチオンホストにMySQLをインストールします。

   2. 以下のコマンドを使用してStarRocksクラスタに接続します：

      ```Bash
      mysql -u root -h 10.0.xx.xx -P 9030 -p
      ```

      - ホスト：
        FEのプライベートIPアドレスは、以下の手順に従って見つけることができます：

        1. AWS CloudFormation コンソールで、`StarRocksClusterStack`の**Outputs**タブにある`FeLeaderInstance`の値をクリックします。
        ![StarRocks_on_AWS_4](../assets/StarRocks_on_AWS_4.png)

        2. インスタンスのサマリーページで、FEのプライベートIPアドレスを探します。
        ![StarRocks_on_AWS_5](../assets/StarRocks_on_AWS_5.png)

      - password: ステップ5で設定したパスワードを入力します。
