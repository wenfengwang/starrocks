---
displayed_sidebar: English
---

# 文件管理器

通过文件管理器，您可以创建、查看和删除文件，例如用于访问外部数据源的文件：公钥文件、私钥文件和证书文件。您可以使用命令来引用或访问这些创建的文件。

## 基本概念

**文件**：指的是在StarRocks中创建并保存的文件。一旦文件在StarRocks中被创建并储存，系统会为其分配一个唯一的ID。您可以通过数据库名、目录和文件名来查找文件。在数据库中，只有管理员用户可以创建和删除文件，而所有有权限访问数据库的用户都可以使用该数据库中的文件。

## 开始之前

- 为每个FE配置以下参数：
  - `small_file_dir`: 上传文件的存储路径。默认路径为 `small_files/`，位于FE的运行目录下。您需要在 **fe.conf** 文件中指定此参数，然后重启FE以使更改生效。
  - `max_small_file_size_bytes`: 单个文件的最大尺寸。此参数的默认值是1MB。如果文件的大小超过这个值，将无法创建文件。您可以通过使用[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)语句来指定此参数。
  - max_small_file_number：集群中能够创建的文件的最大数量。此参数的默认值是100。如果您创建的文件数量达到这个值，您将无法继续创建文件。您可以使用ADMIN SET CONFIG语句来指定此参数。

> 注意：提高这两个参数的值会增加FE的内存使用量。因此，我们建议除非必要，否则不要增加这两个参数的值。

- 为每个BE配置以下参数：

`small_file_dir`: 下载文件存储的路径。默认路径是 `lib/small_files/`，位于BE的运行目录中。您可以在**be.conf**文件中指定此参数。

## 创建文件

您可以执行CREATE FILE语句来创建文件。有关更多信息，请参见[CREATE FILE](../sql-reference/sql-statements/Administration/CREATE_FILE.md)。文件创建后，会被上传并持久化存储在StarRocks中。

## 查看文件

您可以执行`SHOW FILE`语句来查看存储在数据库中的文件信息。有关更多信息，请参见[SHOW FILE](../sql-reference/sql-statements/Administration/SHOW_FILE.md)。

## 删除文件

您可以执行**DROP FILE**语句来删除文件。有关更多信息，请参见[DROP FILE](../sql-reference/sql-statements/Administration/DROP_FILE.md)。

## FE和BE如何使用文件

- **FE**：SmallFileMgr类负责将文件数据存储在FE指定的目录下。然后，SmallFileMgr类会返回一个本地文件路径供FE使用该文件。
- **BE**：BE通过调用**/api/get_small_file API**（HTTP）将文件下载到其指定的目录，并记录文件信息。当BE请求使用文件时，会先检查文件是否已下载，然后对文件进行验证。如果文件通过验证，就会返回文件的路径；如果文件验证失败，文件会被删除并从FE重新下载。当BE重启时，会将已下载的文件预加载到其内存中。
