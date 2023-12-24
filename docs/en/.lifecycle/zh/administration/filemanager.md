---
displayed_sidebar: English
---

# 文件管理器

使用文件管理器，您可以创建、查看和删除文件，例如用于访问外部数据源的文件：公钥文件、私钥文件和证书文件。您可以使用命令引用或访问创建的文件。

## 基本概念

**文件**：指在 StarRocks 中创建并保存的文件。文件创建并存储在 StarRocks 中后，StarRocks 会为该文件分配一个唯一的 ID。您可以根据数据库名称、目录和文件名查找文件。在数据库中，只有管理员用户才能创建和删除文件，并且所有有权访问数据库的用户都可以使用属于该数据库的文件。

## 准备工作

- 针对每个 FE 配置以下参数。
  - `small_file_dir`：存储上传文件的路径。默认路径为 `small_files/`，位于 FE 的运行时目录下。您需要在 **fe.conf** 文件中指定该参数，然后重新启动 FE 以使更改生效。
  - `max_small_file_size_bytes`：单个文件的最大大小。此参数的默认值为 1 MB。如果文件大小超过该参数的值，则无法创建该文件。您可以使用 [ADMIN SET CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md) 语句指定此参数。
  - `max_small_file_number`：群集中可以创建的最大文件数。此参数的默认值为 100。如果已创建的文件数达到此参数的值，则无法继续创建文件。您可以使用 ADMIN SET CONFIG 语句指定此参数。

> 注意：增加这两个参数的值会导致 FE 的内存使用量增加。因此，除非必要，否则建议不要增加这两个参数的值。

- 针对每个 BE 配置以下参数。

`small_file_dir`：存储下载文件的路径。默认路径是 `lib/small_files/`，它位于 BE 的运行时目录中。您可以在 **be.conf** 文件中指定此参数。

## 创建文件

您可以执行 CREATE FILE 语句来创建文件。有关详细信息，请参阅 [CREATE FILE](../sql-reference/sql-statements/Administration/CREATE_FILE.md)。文件创建完成后，文件会上传并持久化到 StarRocks 中。

## 查看文件

您可以执行 SHOW FILE 语句查看数据库中存储的文件信息。有关详细信息，请参阅 [SHOW FILE](../sql-reference/sql-statements/Administration/SHOW_FILE.md)。

## 删除文件

您可以执行 DROP FILE 语句删除文件。有关详细信息，请参阅 [DROP FILE](../sql-reference/sql-statements/Administration/DROP_FILE.md)。

## FE 和 BE 如何使用文件

- **FE：** SmallFileMgr 类将与文件相关的数据存储在 FE 的指定目录下。然后 SmallFileMgr 类返回一个本地文件路径，供 FE 使用该文件。
- **BE：** BE 调用 **/api/get_small_file API** (HTTP) 将文件下载到指定目录，并记录文件信息。当 BE 请求使用该文件时，BE 会检查文件是否已下载，然后验证该文件。如果文件通过验证，则返回文件的路径。如果文件未通过校验，则会删除该文件，然后从 FE 重新下载。当 BE 重新启动时，它会将下载的文件预加载到其内存中。
