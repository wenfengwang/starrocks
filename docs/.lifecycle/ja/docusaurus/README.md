# ドキュメント

## `docusaurus` ディレクトリ

テクニカルライターがローカルでドキュメントを構築できるようにします。

ドキュメントは `docs/en` と `docs/zh` にあります。Docusaurusでバージョニングと国際化が有効になっている場合、
マークダウンファイルは `docs/` と `i18n/zh/some long path` に配置する必要があります。最善の対処法は、
DocusaurusをDockerコンテナで実行し、`docs/en` と `docs/zh` ディレクトリをDocusaurusが期待する場所にマウントすることです。

### Docusaurus用のDockerイメージをビルドする

DocusaurusをDockerで実行するためには、適切なDocusaurus設定ファイル、Node.jsバージョンなどを含むDockerイメージが必要です。
`docusaurus.config.js` や `sidebars.json` の設定ファイルを変更した場合は、ビルドが必要です。
ビルドは非常に迅速で、少なくとも一度は行う必要があります。

- ターミナルを開き、`docusaurus` ディレクトリにcdします
- `./scripts/docker-image.sh` スクリプトを実行します

  > 注意：
  >
  > スクリプトは `docusaurus` ディレクトリから実行してください

### 開発モードでDocusaurusを実行する

開発モードでDocusaurusを実行すると、ドキュメントファイルに加えた変更がブラウザに表示されます。
Dockerコンテナがページを保存するたびにページをリビルドするため、変更が即時に反映されます。

- `./scripts/docker-run.sh` スクリプトを実行します

  > 注意：
  >
  > スクリプトは `docusaurus` ディレクトリから実行してください

### 英語と中国語の両方で最適化されたビルドを作成する

言語を切り替えた際の効果を確認し、ナビゲーションが英語と中国語の両方に適しているかを確認するために、完全なビルドを行うことが重要です。
`./scripts/docker-build.sh` を実行してフルビルドを行ってください。

  > 注意：
  >
  > このビルドはHTMLを生成し、それを提供します。ページは編集後には更新されません。コンテナを停止して再起動する必要があります。`sidebars.json` ファイルを編集する場合は、`./scripts/docker-image.sh` コマンドを実行することを忘れないでください。

### ナビゲーション

ナビゲーションはサイドバーファイルと `_category_.json` ファイルで管理されます。現在、これらは
docs-siteリポジトリにありますが、Gatsbyをまだ使用している間にstarrocksリポジトリに追加すると問題が発生したでしょう。

できるだけ早くサイドバーをstarrocksリポジトリに移動し、ドキュメントとカテゴリファイルを一緒に管理できるようにします。

現在、ナビゲーションはディレクトリ名に基づいて自動生成されています。

## リリースノート

> 注意：
>
> このREADMEのセクションはまだ実装されていません。以下に説明するようにリリースノートをビルドしましたが、英語から中国語へのリリースノートの切り替えが信頼できなかったため、取りやめました。時間があるときに、Docusaurus RDと協力して機能するようにします。

DocusaurusとGatsbyではリリースノートのレンダリング方法が異なります。Gatsbyでは、リンク `../quick_start/abc.md` は、読者がどのバージョンのドキュメントを見ていても、メインブランチ（または3.1かもしれません）を参照します。Docusaurusでは、特定のバージョンにリリースノートファイルを追加すると、それらのリンクはそのバージョンのドキュメントを探します。これは、3.1のリリースノートから1.19バージョンにコピーしたほとんどのリンクが失敗することを意味します。

Docusaurusサイトがバージョン管理されないものを扱う方法は、それらを別のナビに追加することです。ページの上部には `Documentation`、`Release Notes`、バージョンリスト、言語リストがあります。リリースノートは常にメインブランチからのものです。

ビルドプロセス中、英語のリリースノートとエコシステムのリリースノートのマークダウンファイルは `docs-site/releasenotes` ディレクトリに配置する必要があります。

ビルドプロセス中、中国語のリリースノートとエコシステムのリリースノートのマークダウンファイルは `docs-site/i18n/zh/docusaurus-plugin-content-docs-releasenotes` ディレクトリに配置する必要があります。

## ナビゲーションの編集

いずれ、ナビゲーションの管理に使用されるファイルをドキュメントリポジトリに移動します。まず、ライターがドキュメントを迅速に構築し、PRのプレビューを確認できるようにするための構成を記述する必要があります。これには、編集中のバージョンのみをビルドし、ナビゲーションとコンテンツが両方の言語で確認できるように、中国語と英語の両方をビルドすることが含まれます。ナビゲーションを編集する基本的な手順は以下の通りですが、詳細な手順は後ほど説明します。

1. `StarRocks/docs-site` をチェックアウトします。
1. `docusaurus` ブランチに切り替えます。
1. `docusaurus` ブランチから新しい作業ブランチを作成します。
1. 作業中のバージョンのナビゲーションを編集します。
1. PRを提出します。
1. PRをレビューし、マージします。
1. ステージングにデプロイするためのワークフローを実行します。

### 単純なケース、ドキュメントの削除または追加

この例ではドキュメントを削除しますが、ドキュメントを追加するには次の2つの方法があります：

- ナビゲーションが自動生成されるディレクトリにマークダウンファイルを追加する
- 項目リストにエントリを追加する

この例では、項目リストからドキュメントを削除します：

#### `StarRocks/docs-site` をチェックアウトします

ハッ！私はVS Codeでこれらすべての作業をしようとしましたが、なんと悪夢でした。私の指はコマンドラインに慣れており、マウスとメニューではうまくいきません。いずれにせよ、この作業をする方法はすでにご存知でしょう。

#### `docusaurus` ブランチに切り替えます

現在、`docusaurus` という名前のブランチで作業しているため、最初にそこに切り替えます。

#### `docusaurus` ブランチから新しい作業ブランチを作成します

PRで作業するブランチを作成する際は、`docusaurus` ブランチをベースにしてください。masterブランチではありません。

#### 作業中のバージョンのナビゲーションを編集します

ナビゲーションファイルは [`versioned_sidebars/`](https://github.com/StarRocks/docs-site/tree/docusaurus/versioned_sidebars) にあります（Docusaurusではナビゲーションは**Sidebar**と呼ばれます）。3.1バージョンで作業している場合は `versioned_sidebars/version-3.1` を使用します。このファイルには英語と中国語のサイドバーが含まれています。

> ファイル構造に関する注意：
>
> 英語と中国語のファイル構造は同じである必要があります。英語のファイルが中国語に存在しない場合は、その英語のドキュメントが両方の言語で使用されます。逆に、中国語のドキュメントが英語に存在しない場合、Docusaurusはビルドに失敗します。Dataphinのドキュメントが英語でまだ利用できなかった時期があり、私はダミーファイルを作成しなければなりませんでした。
>
> ナビゲーションに違いがある場合もあります。例えば、英語のDataphinドキュメントがなかった時、私はダミーファイルを作成し、ナビゲーションからは除外しました。これは、エントリが少ないカテゴリでは全てをリストできるため簡単ですが、ファイルが多数含まれる大規模なディレクトリでは、Docusaurusにディレクトリ内の全ファイルを含めるよう指示するため、ファイルを無視することはできません。GatsbyでのSQL関数のTOC.mdとDocusaurusのサイドバーを比較すると、SQL関数の全ファイルをリストしていないことがわかります。異なるカテゴリが1つのディレクトリ内に混在している場合のみリストしています。将来的には、ナビゲーションに合わせてファイルをディレクトリに移動するために、より多くのディレクトリを作成し、労力を節約したいと考えています。

#### PRを提出する

このPRは[ナビゲーションに表示されるべきでないファイルを削除します](https://github.com/StarRocks/docs-site/pull/140/files)。これは、ファイルを個別にリストする場合に簡単ですが、このケースでも同様です。

#### PRをレビューし、マージする

いつも通りです

#### ステージング環境へデプロイするためのワークフローを実行する

ワークフローの実行はGatsbyの場合と同様です。Actionsを開き、ワークフローを選択し、ボタンを押します。現在のワークフローの名前は`__Stage__Deploy_docusaurus`と`__Prod__Deploy_docusaurus`です。

`__Stage`を実行し、`https://docs-stage.docusaurus.io`でドキュメントを確認し、内容に満足したら本番環境にデプロイします。

### ドキュメントの名前を変更する

時にはドキュメントのタイトルが非常に長く、ナビゲーションでその長い名前を使用したくない場合があります。または、`# Rules`というタイトルのドキュメントがある場合もあります（開発者 > スタイルガイドに2つの例があります）。選択肢は2つありますが、もう1つの問題を修正するまで2番目の選択肢はまだ使用できないため、現時点では1つだけを提供します。

サイドバーに表示される名前を変更するには、ドキュメントのタイトルを編集するだけです。

#### タイトルを変更して、それによりナビゲーションラベルを変更する

TOC.mdでは、各カテゴリとファイルに関連付けられるラベルを指定しました。Docusaurusでもそれを行うことができますが、サイドバーラベルとしてドキュメントのタイトルを使用することをお勧めします。docs-siteリポジトリの課題の1つは、ナビゲーション内のファイル名が誤っていることに関するものです。簡単な修正は、ファイル内のタイトルを変更することです。この[PR in StarRocks/starrocks](https://github.com/StarRocks/starrocks/pull/34243/files#diff-70c336ebca1518c87e270411fc53419ffb44cd95792a85afd1592fafd6c57e9f)はその問題を解決します。

### カテゴリのレベルを変更する

ナビゲーションにはいくつかのエラーがあります。[issue 105](https://github.com/StarRocks/docs-site/issues/105)はその一つを指摘しています。管理セクションのJSONを書いていた時、私はすべてが「管理 > 管理」の下にあると考えていました。JSONは以下のようになります：

```json
    {
      "type": "category",
      "label": "Administration",
      "link": {"type": "doc", "id": "administration/administration"},
      "items": [
        {
          "type": "category",
          "label": "Management",
          "link": {"type": "doc", "id": "cover_pages/management"},
          "items": [
            "administration/Scale_up_down",
            "administration/Backup_and_restore",
            "administration/Configuration",
            "administration/Monitor_and_Alert",
            "administration/audit_loader",
            "administration/enable_fqdn",
            "administration/timezone",
            "administration/information_schema",
            "administration/monitor_manage_big_queries",
            {
              "type": "category",
              "label": "Resource management",
              "link": {"type": "doc", "id": "cover_pages/resource_management"},
              "items": [
                "administration/resource_group",
                "administration/query_queues",
                "administration/Query_management",
                "administration/Memory_management",
                "administration/spill_to_disk",
                "administration/Load_balance",
                "administration/Replica",
                "administration/Blacklist"
              ]

            },
            "administration/Data_recovery",
            {
              "type": "category",
              "label": "User Privileges and Authentication",
              "link": {"type": "doc", "id": "administration/privilege_overview"},
              "items": [
                "administration/privilege_item",
                "administration/User_privilege",
                "administration/privilege_faq",
                "administration/Authentication"
              ]
            },
            {
              "type": "category",
              "label": "Performance Tuning",
              "link": {"type": "doc", "id": "cover_pages/performance_tuning"},
              "items": [
                "administration/Query_planning",
                "administration/query_profile",
                "administration/Profiling"
              ]
            }
          ]
        }
      ]
    },
```

User PrivilegesとPerformance Tuningは、ManagementとData Recoveryと同じレベルに移動する必要があると思います。

## 過去の情報

**ここより下は無視してください**

## GitHub Actionsを使用したビルド

`.github/workflows/`には、テストビルドとGitHub Pagesへのデプロイジョブがあります。
これらは英語と中国語のドキュメントをプルし、バージョンをチェックアウトし、
Docusaurus用のMarkdownファイルを配置します。

HTMLを生成する前にMarkdownファイルにいくつかの変更が加えられます：

- TOC.mdとREADME.mdファイルを削除
- StarRocks_introページをDocusaurusスタイルを使用するものに置き換える
- すべてのMarkdownにfrontmatterを追加して、使用するサイドバー（英語または中国語）を指定する
- `docs/assets/`ディレクトリは`_assets`に名前が変更されます。これはDocusaurusが自動的に
アンダースコアで始まるディレクトリ内のマークダウンファイルを無視するためです。これが私が`_IGNORE`
ディレクトリを持つ理由です。これは、ドキュメントに直接含めたくないマークダウンファイルを保管する場所です。

本番環境に移行すると、上記の3つの変更は不要になります：

- TOC.mdファイルは使用されないため削除し、READMEはナビゲーションから除外する
- 現在のイントロページをDocusaurusで動作する新しいものに置き換える
- リポジトリのMarkdownファイルにfrontmatterを追加する
- `assets`ディレクトリの名前を`_assets`に変更して、ビルド時にこれらの変更を行う必要がなくなるようにする

## ローカルでのビルド

### Nodeバージョン

Docusaurus v3はNode 18を必要とします

私はNodeに8GBを割り当てており、Netlifyでは`netlify.toml`ファイルでビルドコマンドを設定し、
ローカルでは以下を使用します：

```shell
NODE_OPTIONS=--max_old_space_size=8192
```

### Docusaurusのインストール

```shell
yarn install
```

### ビルドスクリプト

`_IGNORE/testbuild`スクリプトは、

- バージョン3.1、3.0、2.5の中国語と英語のドキュメントをプルします
- 組み込みのナビゲーションコンポーネントを使用するためにイントロページを削除します
- TOCをJSON形式に移行中のため、TOCを削除します
- MDXチェッカーを実行します
- サイトをビルドします

```shell
./_IGNORE/testbuild`
```

注: ディレクトリは`_IGNORE`と名付けられています。これは、Docusaurusがナビゲーションに追加しないようにアンダースコアで始まるディレクトリに移動する必要があったマークダウンファイルがいくつかあったためです。

## ページをローカルでサーブする

```shell
yarn serve
```

