---
displayed_sidebar: Chinese
---

# AWS CloudFormation を使用して AWS 上に StarRocks クラスタをデプロイする

StarRocks は AWS CloudFormation との統合をサポートしており、AWS CloudFormation を使用して AWS 上に迅速に StarRocks クラスタをデプロイして使用することができます。

## AWS CloudFormation

AWS CloudFormation は AWS が提供するサービスで、AWS リソースおよびサードパーティリソース（例えば StarRocks クラスタ）のモデリングと設定を簡単かつ迅速に行うことができ、リソース管理の時間コストを削減し、これらのリソースの使用により多くの時間を費やすことができます。必要なリソースを記述したテンプレートを作成し、AWS CloudFormation がこれらのリソースの設定を行います。詳細は[What is AWS CloudFormation](https://docs.aws.amazon.com/zh_tw/AWSCloudFormation/latest/UserGuide/Welcome.html)を参照してください。

## 基本概念

### テンプレート

テンプレート（Template）は JSON または YAML 形式のテキストファイルで、AWS リソースとサードパーティリソース、およびそれらの属性を記述しています。詳細は[テンプレート](https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b7)を参照してください。

### スタック

スタック（Stack）は、テンプレートで記述されたリソースを作成および管理するために使用されます。スタックの作成、更新、削除を行うことで、一連のリソースを作成、更新、削除することができます。スタック内のすべてのリソースはスタックのテンプレートによって定義されています。様々なリソースを記述したテンプレートを作成した場合、そのテンプレートを提出してスタックを作成することで、AWS CloudFormation がこれらのリソースを設定します。詳細は[スタック](https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html#w2aab5c15b9)を参照してください。

## 操作手順

1. [AWS CloudFormation コンソール](https://console.aws.amazon.com/cloudformation/)にログインします。

2. **スタックの作成** > **新しいリソースを使用（標準）** を選択します。<br />
   ![新しいリソースを使用](../assets/8.1.3-1.png)
3. 次の手順に従ってテンプレートを指定します。
   ![テンプレートを指定](../assets/8.1.3-2.png)
   1. **事前条件 - テンプレートの準備**セクションで、**テンプレートが準備できている**を選択します。
   2. **テンプレートの指定**セクションで、**テンプレートのソース**を**Amazon S3 URL**に設定し、**Amazon S3 URL**に以下のURLを入力します：
      `https://cf-templates-1euv6e68138u2-us-east-1.s3.amazonaws.com/templates/starrocks.template.yaml`
      > 説明：**テンプレートのソース**を**テンプレートファイルをアップロード**に設定し、**ファイルを選択**をクリックして **starrocks.template.yaml** ファイルをアップロードすることもできます。ファイルのダウンロードアドレスは、StarRocks プロジェクトの [aws-cloudformation リポジトリ](https://github.com/StarRocks/aws-cloudformation)を参照してください。![starrocks.template.yaml ファイル](../assets/8.1.3-3.png)

   3. **次へ**をクリックします。

4. スタックの詳細を指定し、**スタック名**と**パラメータ**を入力して、**次へ**をクリックします。
   1. **スタック名**ボックスにスタック名を入力します。
      スタック名は、スタックリストから特定のスタックを見つけるための識別子です。スタック名は、英字（大文字と小文字を区別）、数字、ハイフンのみを含めることができ、128文字を超えることはできず、英字で始まる必要があります。

   2. 以下の情報を参考にしてパラメータを入力します：

      | カテゴリ           | パラメータ                                                     | 説明                                                         |
      | ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
      | ネットワーク設定   | Availability Zones                                           | StarRocks クラスタをデプロイするための使用可能なゾーンを選択します。詳細は[利用可能なゾーン](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)を参照してください。 |
      | EC2 設定           | Key pair name                                                | キーペアは、公開キーと秘密キーのセットで構成されるセキュリティクレデンシャルで、Amazon EC2 インスタンスへの接続時にあなたの身元を証明するために使用されます。詳細は[キーペア](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ec2-key-pairs.html)を参照してください。注意：キーペアが作成されていない場合は、[キーペアの作成](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/create-key-pairs.html)を参照して作成してください。 |
      | 環境設定           | CloudFormation テンプレートで最新の Amazon Linux AMI を参照する | 最新バージョンの Amazon Machine Images (AMI) ID、アーキテクチャは64ビット (`x86_64`) で、Amazon EC2 インスタンスの起動に使用されます。デフォルトは StarRocks の共有 AMI ID です。注意：AMI は AWS が提供およびメンテナンスするイメージで、インスタンスの起動に必要な情報を提供します。詳細は[Amazon Machine Images](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AMIs.html)を参照してください。 |
      |                    | JDK 1.8のダウンロードURL                                      | JDK 1.8 のダウンロードアドレス。                             |
      |                    | StarRocksのURL                                             | StarRocks のバイナリパッケージのダウンロードアドレス。       |
      | StarRocks クラスタ設定 | StarRocks Fe の数                                       | FE ノードの数、デフォルトは **1**、範囲は **1** または **3**。 |
      |                    | Fe インスタンスタイプ                                             | FE ノードに対応する Amazon EC2 のインスタンスタイプ、デフォルトは **t2.micro**。インスタンスタイプの詳細は [Amazon EC2 インスタンスタイプ](https://aws.amazon.com/cn/ec2/instance-types/)を参照してください。 |
      |                    | StarRocks Be の数                                       | BE ノードの数、デフォルトは **3**、範囲は **3**～**6**。     |
      |                    | Be インスタンスタイプ                                             | BE ノードに対応する Amazon EC2 のインスタンスタイプ、デフォルトは **t2.micro**。インスタンスタイプの詳細は [Amazon EC2 インスタンスタイプ](https://aws.amazon.com/cn/ec2/instance-types/)を参照してください。 |
      | FE 設定項目        | FE ログを保存するディレクトリ                                           | FE ログの保存パス、絶対パスである必要があります。                            |
      |                    | システムログレベル                                                | FE ログレベル、デフォルトは **INFO**、選択肢は **INFO**、**WARN**、 **ERROR**、**FATAL**。 |
      |                    | メタデータディレクトリ                                                | FE メタデータの保存パス、絶対パスである必要があります。デフォルトは **feDefaultMetaPath**、デフォルトパス **/home/starrocks/StarRocks/fe/meta** を使用することを意味します。 |
      | BE 設定項目        | BE システムログを保存するディレクトリ                                       | BE ログの保存パス、絶対パスである必要があります。                        |
      |                    | システムログレベル                                                | BE ログレベル、デフォルトは **INFO**、選択肢は **INFO**、**WARN**、 **ERROR**、**FATAL**。 |
      |                    | Be ノードのボリュームタイプ                                      | Amazon EBS ボリュームタイプ。Amazon EBS ボリューム（EBS ボリュームとも呼ばれる）はブロックストレージボリュームで、Amazon EC2 インスタンスにマウントされます。詳細とタイプの説明は[Amazon EBS ボリューム](https://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/ebs-volumes.html)を参照してください。 |
      |                    | Be ノードのボリュームサイズ                                      | BE ノードのデータストレージに使用可能な EBS ボリュームの容量、単位は GB。            |

5. スタックの追加オプションを設定します。詳細は[AWS CloudFormation スタックオプションの設定](https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cfn-console-add-tags.html)を参照してください。

    設定が完了したら、**次へ**をクリックします。

6. 以前に設定したスタック情報、テンプレート、詳細情報、追加オプションをレビューし、スタックのコストを評価します。詳細は[スタック情報のレビューとスタックコストの評価](https://docs.aws.amazon.com/zh_cn/AWSCloudFormation/latest/UserGuide/cfn-using-console-create-stack-review.html)を参照してください。

   > 説明：スタック情報を変更する必要がある場合は、該当するセクションの右上隅にある**編集**をクリックして関連ページに戻ります。

7. 以下の二つのチェックボックスを選択し、**スタックの作成**をクリックします。

![スタックの作成](../assets/8.1.3-4.png)
