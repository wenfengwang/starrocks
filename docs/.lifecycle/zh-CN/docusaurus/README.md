# 文档

## `docusaurus` 目录

允许技术作者在本地构建文档。

我们的文档位于 `docs/en` 和 `docs/zh`。当启用版本控制和国际化时，在 Docusaurus 中 markdown 文件需要位于 `docs/` 和 `i18n/zh/some long path`。处理这个的最佳方式是在 Docker 容器中运行 Docusaurus，并挂载 `docs/en` 和 `docs/zh` 目录到 Docusaurus 期望找到它们的位置。

### 为 Docusaurus 构建 Docker 镜像


为了在 Docker 中运行 Docusaurus，我们需要一个含有正确的 Docusausaurus 配置文件、Nodejs 版本等内容的 Docker 镜像。如果更改了配置文件 `docusaurus.config.js` 或 `sidebars.json`，那么需要进行构建。构建非常快，至少需要进行一次。

- 在终端中切换到 `docusaurus` 目录
- 运行脚本 `./scripts/docker-image.sh`

  > 注:
  >

  > 从 `docusaurus` 目录运行脚本


### 以开发模式运行 Docusaurus

当以开发模式运行 Docusaurus 时，对文档文件的任何更改都将在浏览器中可见，因为 Docker 容器将随着将它们保存到磁盘上而重新构建页面。

- 运行脚本 `./scripts/docker-run.sh`

  > 注:
  >
  > 从 `docusaurus` 目录运行脚本

### 构建既包含英语又包含中文语言的优化版本

进行完整构建非常重要，以便您可以看到切换语言的效果，并确保导航对英语和中文都是良好的。使用 `./scripts/docker-build.sh` 进行完整构建。

  > 注:

  >

  > 此构建将生成 HTML 然后提供服务。页面不会在编辑它们时更新，您需要停止容器然后重新启动它。如果您正在编辑 `sidebars.json` 文件，请记得运行 `./scripts/docker-image.sh` 命令。

### 导航

导航由侧边栏文件和 `_category_.json` 文件管理。目前，这些文件位于 docs-site 仓库中，因为在我们还在运行 Gatsby 时将它们添加到 starrocks 仓库会导致问题。

尽快我会将侧边栏移动到 starrocks 仓库中，然后您就可以管理它们以及与文档一起的类别文件。

现在，导航是基于目录名自动生成的。

## 发布说明

> 注:

>
> 此 README 部分尚未实现。我尝试按照下面描述的方式构建发布说明，并且取得了一些进展，但是切换从英语到中文的发布说明不够可靠，因此我取消了这一操作。我有时间时会与 Docusaurus RD 合作解决这个问题。

在 Docusaurus 和 Gatsby 中呈现发布说明的方式不同。在 Gatsby 中，诸如 `../quick_start/abc.md` 这样的链接无论读者查看的是哪个版本的文档，都会指向主分支（或者可能是 3.1 版本？）。在 Docusaurus 中，当我们将发布说明文件添加到特定版本时，这些链接会定位到该版本的文档。这意味着我们将从 3.1 的发布说明复制到 1.19 版本时，几乎每个链接都会失败。

Docusaurus 站点处理不应该进行版本控制的内容时会将其添加到单独的导航中。在我们的页面顶部，我们将拥有 `文档`、`发布说明`、版本列表、语言列表。发布说明总是来自主分支。

在构建过程中，英语发布说明和生态系统发布说明的 markdown 文件需要位于 `docs-site/releasenotes` 目录中

在构建过程中，中文发布说明和生态系统发布说明的 markdown 文件需要位于 `docs-site/i18n/zh/docusaurus-plugin-content-docs-releasenotes` 目录中

## 编辑导航

在某个时候，我会将用于管理导航的文件移动到文档仓库中。首先，我需要编写一个配置，允许作者快速构建文档，并在创建请求的预览中查看。这将涉及仅构建正在编辑的版本，并为中文和英文都进行构建，以便可以在两个地方验证导航和内容。以下是编辑导航的基本步骤，请关注完整细节:

1. 检出 `StarRocks/docs-site`

1. 切换到 `docusaurus` 分支

1. 从 `docusaurus` 分支创建一个新的工作分支

1. 编辑您正在操作的版本的导航

1. 提交请求
1. 对请求进行审查和合并
1. 运行工作流进行部署到临时环境

### 简单情况，删除或增加一个文档

此示例从列表中删除一个文档，而增加一个文档可以通过以下两种方式之一完成：

- 将一个 markdown 文件添加到自动生成导航的目录中

- 在项目列表中添加一个条目

以下示例从列表中删除一个文档:


#### 检出 `StarRocks/docs-site`

啊！我尝试在 VS Code 中完成所有这些操作，但简直是一场噩梦。我的手指熟悉命令行，而且我无法用鼠标和菜单做这件事。不过您已经知道该怎么做了。

#### 切换到 `docusaurus` 分支

现在我们在一个名为 `docusaurus` 的分支上工作，所以首先切换到那里。

#### 从 `docusaurus` 分支创建一个新的工作分支

当您创建要处理的请求的分支时，要基于 `docusaurus` 分支创建，而不是基于主分支。

#### 编辑您正在操作的版本的导航

导航文件位于 [`versioned_sidebars/`](https://github.com/StarRocks/docs-site/tree/docusaurus/versioned_sidebars)（Docusaurus 中的导航称为**侧边栏**）。如果您正在编辑 3.1 版本，则在 `versioned_sidebars/version-3.1` 中找到它。此文件包含英语和中文的侧边栏。

> 关于文件结构的说明:

>
> 英语和中文的文件结构应该是相同的，如果有一个文件是英文而中文没有，则英文文件将同时用于英文和中文。如果有一份中文文档没有对应的英文文档，Docusaurus 将不进行构建。以前 Dataphin 文档尚未有英文版本，因此我不得不创建一个虚拟文件。
>
> 可能存在导航的差异，例如当英文中不存在 Dataphin 文档时，我创建了一个虚拟文件，然后直接从导航中删除了它。如果是只有一些条目的类别，则很容易只列出它们中的所有条目，但是对于充满文件的大目录，我只告诉 Docusaurus 包含目录中的所有文件，因此如果这样做，我们不能忽略文件。如果您比较 Gatsby 中 SQL 函数的 TOC.md 和 Docusaurus 中的侧边栏，您会发现我没有列出所有 SQL 函数的文件，如果将不同类别混合在一个目录中，则会进行列出。未来，我希望创建更多目录，并将文件移入目录以匹配导航，那时我们就可以省点功夫了。


#### 提交请求

这个请求 [删除了一个不应该在导航中显示的文件](https://github.com/StarRocks/docs-site/pull/140/files)。这在我们单独列出文件的情况下很容易实现，这在这个案例中就是这样。

#### 审查并合并请求

和往常一样

#### 运行工作流进行部署到临时环境

运行工作流与过去在 Gatsby 中的操作相同，打开 Actions，选择工作流，然后点击按钮。现在工作流的名称是 `__Stage__Deploy_docusaurus` 和 `__Prod__Deploy_docusaurus`

运行 `__Stage` 的工作流，并在 `https://docs-stage.docusaurus.io` 查看文档，如果喜欢看到的内容，则部署到 Production。

### 更改文档的名称

有时我们的文档标题非常长，不希望在导航中显示整个长标题。或者有时我们的文档标题是 `# 规则`（参见 Developers > 样式指南，有两个例子！）。有两种选择，但目前我只给出一种选择，因为在我修复另一个问题之前还不能使用第二种选择。

要更改侧边栏中显示的标题，只需编辑文档的标题:

#### 更改标题，因此更改导航标签

在 TOC.md 中，我们指定了与每个类别和文件关联的标签。在 Docusaurus 中，我们也可以这样做，但我建议我们使用文档的标题作为侧边栏的标签。文档库中的一个问题是导航中的文件名称不准确。简单的解决方案是更改文件中的标题。这个[在 StarRocks/starrocks 中的 PR](https://github.com/StarRocks/starrocks/pull/34243/files#diff-70c336ebca1518c87e270411fc53419ffb44cd95792a85afd1592fafd6c57e9f) 解决了这个问题。

### 更改类别的级别

导航中有一些错误，其中[105号问题](https://github.com/StarRocks/docs-site/issues/105)提出了其中之一。当我为管理部分编写JSON时，我认为所有内容都属于Administration > Management。以下是JSON的样子：

```json
    {
      "type": "category",
      "label": "Administration",
      "link": {"type": "doc", "id": "administration/administration"},
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
        }
      ]
    }
```

我认为用户权限和性能调优需要移到和管理以及数据恢复同级的位置。

## 旧的历史信息

**忽略下面的内容**

## 使用GitHub actions构建

在`.github/workflows/`中有测试构建和部署到GitHub Pages的作业。这些作业提取英文文档和中文文档，检查版本，并将Markdown文件放到Docusaurus的指定位置。

在生成HTML之前对Markdown文件做了一些修改：

- 删除TOC.md和README.md文件
- 用使用Docusaurus样式的页面替换StarRocks_intro页面
- 在所有Markdown文件中添加Frontmatter以指定使用哪个侧边栏（英文或中文）
- 将`docs/assets/`目录重命名为`_assets`。这是因为Docusaurus会自动忽略下划线开头的目录中的Markdown文件。这也是为什么我有一个`_IGNORE`目录。我把我不想包含在文档中的Markdown文件放在那里。

一旦我们进入生产阶段，可以移除上述三处修改，因为我们将：

- 删除TOC.md文件，因为它们不会被使用，并将README从导航栏中去掉
- 用新的使用Docusaurus的页面替换当前的intro页面
- 向这些仓库的Markdown文件中添加Frontmatter
- 将`assets`目录重命名为`_assets`，这样我们就不必在构建中做这些修改。

## 本地构建

### Node版本

Docusaurus v3需要Node 18

我为Node使用了8GB，在Netlify中我在文件`netlify.toml`中设置了构建命令，本地我使用：

```shell
NODE_OPTIONS=--max_old_space_size=8192
```

### 安装Docusaurus

```shell
yarn install
```

### 构建脚本

脚本`_IGNORE/testbuild`

- 提取版本3.1、3.0和2.5的中文和英文文档
- 移除intro页面，同时我们更新它以使用内置导航组件
- 移除TOC，同时我们将其迁移到JSON格式
- 运行MDX检查器
- 构建站点

```shell
./_IGNORE/testbuild`
```

注意：目录名为`_IGNORE`，因为我有一些必须移动到Docusaurus不会将其包含在导航栏中的Markdown文件；它不会从下划线开头的目录中向导航栏新增文件。

## 本地浏览页面

```shell
yarn serve
```