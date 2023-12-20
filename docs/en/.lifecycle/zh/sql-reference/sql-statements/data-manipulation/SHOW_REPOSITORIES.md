---
displayed_sidebar: English
---

# 显示存储库

## 描述

查看在 StarRocks 中创建的存储库。

## 语法

```SQL
SHOW REPOSITORIES
```

## 返回值

|**返回**|**描述**|
|---|---|
|RepoId|存储库 ID。|
|RepoName|存储库名称。|
|CreateTime|存储库创建时间。|
|IsReadOnly|存储库是否为只读。|
|Location|存储库在远程存储系统中的位置。|
|Broker|创建存储库时使用的 Broker。|
|ErrMsg|连接存储库时的错误信息。|

## 示例

示例 1：查看在 StarRocks 中创建的存储库。

```SQL
SHOW REPOSITORIES;
```