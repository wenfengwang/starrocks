---
displayed_sidebar: English
---

# 引擎

`engines` 提供有关存储引擎的信息。

`engines` 中提供了以下字段：

|**字段**|**描述**|
|---|---|
|ENGINE|存储引擎的名称。|
|SUPPORT|服务器对存储引擎的支持级别。有效值：<ul><li>`YES`：引擎受支持且处于活动状态。</li><li>`DEFAULT`：与 `YES` 类似，此外它是默认引擎。</li><li>`NO`：引擎不受支持。</li><li>`DISABLED`：引擎受支持但已被禁用。</li></ul>|
|COMMENT|存储引擎的简要描述。|
|TRANSACTIONS|存储引擎是否支持事务。|
|XA|存储引擎是否支持XA事务。|
|SAVEPOINTS|存储引擎是否支持保存点。|