# ドキュメントへの貢献方法

StarRocksのドキュメントに貢献していただき、誠にありがとうございます！あなたの支援はドキュメントの改善にとって重要です！

貢献する前に、この記事を注意深く読んで、ヒント、執筆プロセス、およびドキュメントテンプレートをすばやく理解してください。

## ヒント

1. 言語: 少なくとも一つの言語、中国語または英語を使用してください。バイリンガルバージョンが強く推奨されます。
2. インデックス: トピックを追加する際には、目次(TOC)ファイルにもトピックのエントリを追加する必要があります。例: `[introduction](../docs/en/introduction/what_is_starrocks.md)`。**トピックへのパスは、`docs`ディレクトリからの相対パスでなければなりません。**このTOCファイルは、最終的に公式ウェブサイトのドキュメントのサイドナビゲーションバーとしてレンダリングされます。
3. 画像: 画像は最初に**assets**フォルダに配置する必要があります。ドキュメントに画像を挿入する際は、相対パスを使用してください。例: `![test image](../../assets/test.png)`。
4. リンク: 内部リンク（公式ウェブサイトのドキュメントへのリンク）の場合は、ドキュメントの相対パスを使用してください。例: `[test md](./data_source/catalog/hive_catalog.md)`。外部リンクの場合、フォーマットは`[link text](link URL)`でなければなりません。
5. コードブロック: コードブロックには言語識別子を追加する必要があります。例: `sql`。
6. 現在、特殊記号はサポートされていません。

## 執筆プロセス

1. **執筆フェーズ**: 次のテンプレートに従ってトピックを(Markdownで)記述し、トピックが新しく追加された場合はTOCファイルにトピックのインデックスを追加します。

    > - *ドキュメントはMarkdownで記述されているため、markdown-lintを使用してドキュメントがMarkdown構文に準拠しているかどうかをチェックすることをお勧めします。*
    > - *トピックインデックスを追加する際は、TOCファイル内の*カテゴリに注意してください。例えば、*Stream Load*トピックは*Loading*章に属します。*

2. **提出フェーズ**: GitHubのドキュメントリポジトリにドキュメントの変更を提出するプルリクエストを作成します。英語のドキュメントは[StarRocksリポジトリ](https://github.com/StarRocks/starrocks)の`docs/`フォルダ（英語版の場合）、中国語のドキュメントは[中国語ドキュメントリポジトリ](https://github.com/StarRocks/docs.zh-cn)（中国語版の場合）にあります。

   > **注記**
   >
   > PR内のすべてのコミットは署名されている必要があります。コミットに署名するには、`-s`引数を追加できます。例:
   >
   > `git commit -s -m "Update the MV doc"`

3. 設定のリスト

   このような長い設定リストは検索でうまくインデックスされず、読者が設定の正確な名前を入力しても情報を見つけられないことがあります。

   ```markdown
   - setting_name_foo

     Details for foo

   - setting_name_bar
     Details for bar
   ...
   ```
 
   代わりに、設定名にはセクション見出し（例: `###`）を使用し、テキストのインデントを削除してください:

   ```markdown
   ### setting_name_foo

   Details for foo

   ### setting_name_bar
   Details for bar
   ...
   ```

   |長いリストを含む検索結果:|H3見出しを含む検索結果|
   |--------------------------------|-------------------------------|
   |![画像](https://github.com/StarRocks/starrocks/assets/25182304/681580e6-820a-4a5a-8d68-65852687a0df)|![画像](https://github.com/StarRocks/starrocks/assets/25182304/8623e005-d6e1-4b73-9270-8bc86a2aa680)|


  
5. **レビューフェーズ**:

    レビューフェーズには自動チェックと手動レビューが含まれます。

    - 自動チェック: 提出者がContributor License Agreement（CLA）に署名しているかどうか、およびドキュメントがMarkdown構文に準拠しているかどうかを確認します。
    - 手動レビュー: コミッターがドキュメントを読み、あなたとコミュニケーションを取ります。それがStarRocksのドキュメントリポジトリにマージされ、公式ウェブサイトで更新されます。

## ドキュメントテンプレート

- [Functions](https://github.com/StarRocks/docs/blob/main/sql-reference/sql-functions/How_to_Write_Functions_Documentation.md)
- [SQLコマンドテンプレート](https://github.com/StarRocks/docs/blob/main/sql-reference/sql-statements/SQL_command_template.md)
- [データロードテンプレート](https://github.com/StarRocks/starrocks/blob/main/docs/loading/Loading_data_template.md)
