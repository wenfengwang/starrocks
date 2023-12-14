---
displayed_sidebar: "中文"
---

# 文件管理器

通过文件管理器，您可以创建、查看和删除文件，例如用于访问外部数据源的文件：公钥文件、私钥文件和证书文件。您可以使用命令引用或访问已创建的文件。

## 基本概念

**文件**：指在StarRocks中创建并保存的文件。在文件创建并存储在StarRocks后，StarRocks会为该文件分配一个唯一的ID。您可以基于数据库名称、目录和文件名查找文件。在数据库中，只有管理员用户可以创建和删除文件，所有有权限访问数据库的用户可以使用属于数据库的文件。

## 开始之前

- 为每个FE配置以下参数。
  - `small_file_dir`：存储上传文件的路径。默认路径为`small_files/`，位于FE的运行时目录中。您需要在**fe.conf**文件中指定此参数，然后重新启动FE以使更改生效。
  - `max_small_file_size_bytes`：单个文件的最大大小。该参数的默认值为1MB。如果文件的大小超过该参数的值，则无法创建该文件。您可以使用[ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md)语句指定此参数。
  - `max_small_file_number`：集群中可以创建的最大文件数。该参数的默认值为100。如果您创建的文件数达到该参数的值，将无法继续创建文件。您可以使用ADMIN SET CONFIG语句指定此参数。

> 注意：增加这两个参数的值会导致FE的内存使用率增加。因此，除非必要，我们建议您不要增加这两个参数的值。

- 为每个BE配置以下参数。

`small_file_dir`：存储下载文件的路径。默认路径为`lib/small_files/`，位于BE的运行时目录中。您可以在**be.conf**文件中指定此参数。

## 创建文件

您可以执行CREATE FILE语句来创建文件。有关更多信息，请参见[CREATE FILE](../sql-reference/sql-statements/Administration/CREATE_FILE.md)。文件创建后，文件将上传并持久保存在StarRocks中。

## 查看文件

您可以执行SHOW FILE语句来查看存储在数据库中的文件的信息。有关更多信息，请参见[SHOW FILE](../sql-reference/sql-statements/Administration/SHOW_FILE.md)。

## 删除文件

您可以执行DROP FILE语句来删除文件。有关更多信息，请参见[DROP FILE](../sql-reference/sql-statements/Administration/DROP_FILE.md)。

## FE和BE如何使用文件

- **FE**：SmallFileMgr类将与文件相关的数据存储在FE的指定目录中。然后，SmallFileMgr类返回FE可使用的文件的本地文件路径。
- **BE**：BE调用**/api/get_small_file API**（HTTP）将文件下载到其指定目录，并记录文件的信息。当BE请求使用文件时，BE会检查文件是否已下载，然后验证文件。如果文件通过验证，将返回文件的路径。如果文件未通过验证，文件将被删除并从FE重新下载。当BE重新启动时，它将将已下载的文件预加载到其内存中。