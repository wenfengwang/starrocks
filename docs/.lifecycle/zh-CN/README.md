# 如何贡献文档

非常感谢您为 StarRocks 文档做出贡献！您的帮助对于改进文档至关重要！

在贡献之前，请仔细阅读本文，以便快速了解技巧、编写流程和文档模板。

## 技巧

1. 语言：请至少使用一种语言，中文或英文。强烈推荐使用双语版本。
2. 目录：当您添加一个主题时，还需要在目录文件中为该主题添加一个条目，例如 `[介绍](../docs/zh-cn/introduction/what_is_starrocks.md)`。**您的主题路径必须是相对于 `docs` 目录的相对路径。** 此目录文件最终将呈现为我们官方网站上文档的侧边导航栏。
3. 图像：图像必须首先放入 **assets** 文件夹中。在文档中插入图片时，请使用相对路径，例如 `![测试图片](../../assets/test.png)`。
4. 链接：对于内部链接（链接到我们官方网站上的文档），请使用文档的相对路径，例如 `[测试 md](./data_source/catalog/hive_catalog.md)`。对于外部链接，格式必须是 `[链接文本](链接URL)`。
5. 代码块：您必须为代码块添加一个语言标识符，例如 `sql`。
6. 目前，不支持特殊符号。

## 编写流程

1. **编写阶段**：按照以下模板编写主题（使用 Markdown），并在主题的目录文件中将其索引添加到目录文件中，如果主题是新增的。

    > - *因为文档是用 Markdown 编写的，我们建议您使用 markdown-lint 检查文档是否符合 Markdown 语法。*
    > - *在添加主题索引时，请注意* *在目录文件中的分类。* *例如，***流式加载*** *主题* *属于***加载*** *章节。*

2. **提交阶段**：创建一个拉取请求，将文档更改提交到我们 GitHub 上的文档存储库，英文文档位于 [StarRocks 存储库](https://github.com/StarRocks/starrocks) 的 `docs/` 文件夹中，中文文档存储库位于 [中文文档存储库](https://github.com/StarRocks/docs.zh-cn)。

   > **注意**
   >
   > 您 PR 中的所有提交都应该是签名的。要签名提交，您可以添加 `-s` 参数。例如：
   >
   > `commit -s -m "更新 MV 文档"`

3. 设置列表

   以下这样的设置长列表不易在搜索中索引，并且读者在输入设置的确切名称时也无法找到相应信息：

   ```markdown
   - setting_name_foo

     有关 foo 的详细信息

   - setting_name_bar
     有关 bar 的详细信息
   ...
   ```

   取而代之的是，使用一个章节标题（例如，`###`）用于设置名称，并删除文本的缩进：

   ```markdown
   ### setting_name_foo

   有关 foo 的详细信息

   ### setting_name_bar
   有关 bar 的详细信息
   ...
   ```

   |带有长列表的搜索结果：|带有 H3 标题的搜索结果：|
   |------------------------|-------------------------|
   |![image](https://github.com/StarRocks/starrocks/assets/25182304/681580e6-820a-4a5a-8d68-65852687a0df)|![image](https://github.com/StarRocks/starrocks/assets/25182304/8623e005-d6e1-4b73-9270-8bc86a2aa680)|


  
4. **审核阶段**：

    审核阶段包括自动检查和手动审核。

    - 自动检查：检查提交者是否已签署贡献者许可协议（CLA），以及文档是否符合 Markdown 语法。
    - 手动审核：贡献者将阅读并与您就文档进行沟通。它将被合并到 StarRocks 文档存储库中，并更新在官方网站上。

## 文档模板

- [函数](https://github.com/StarRocks/docs/blob/main/sql-reference/sql-functions/How_to_Write_Functions_Documentation.md)
- [SQL 命令模板](https://github.com/StarRocks/docs/blob/main/sql-reference/sql-statements/SQL_command_template.md)
- [加载数据模板](https://github.com/StarRocks/starrocks/blob/main/docs/loading/Loading_data_template.md)