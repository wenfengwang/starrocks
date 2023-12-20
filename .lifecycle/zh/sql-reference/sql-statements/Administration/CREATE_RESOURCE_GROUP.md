---
displayed_sidebar: English
---

# 创建资源组

## 说明

创建一个资源组。

更多信息，请参见[资源组](../../../administration/resource_group.md)部分。

:::提示

执行此操作需要系统级别的 CREATE RESOURCE GROUP 权限。您可以遵循 [GRANT](../account-management/GRANT.md) 中的指南来授予这项权限。

:::

## 语法

```SQL
CREATE RESOURCE GROUP resource_group_name 
TO CLASSIFIER1, CLASSIFIER2, ...
WITH resource_limit
```

## 参数

- resource_group_name：将要创建的资源组的名称。

- CLASSIFIER：用来过滤对查询施加资源限制的分类器。您必须使用“键”=“值”对来指定分类器。一个资源组可以设置多个分类器。

  以下列出了分类器的参数：

  |参数|必填|说明|
|---|---|---|
  |用户|否|用户名。|
  |角色|否|用户的角色。|
  |query_type|否|查询的类型。支持 SELECT 和 INSERT（从 v2.5 开始）。当 INSERT 任务命中 query_type 为 insert 的资源组时，BE 节点为任务保留指定的 CPU 资源。|
  |source_ip|No|发起查询的 CIDR 块。|
  |db|No|查询访问的数据库。可以用逗号(,)分隔的字符串来指定。|
  |plan_cpu_cost_range|否|查询的估计 CPU 成本范围。格式为(DOUBLE,DOUBLE]。默认值为NULL，表示无此限制。从v3.1.4开始支持该参数。|
  |plan_mem_cost_range|否|查询的估计内存成本范围。格式为(DOUBLE,DOUBLE]。默认值为NULL，表示无此限制。从v3.1.4开始支持该参数。|

- resource_limit：要施加在资源组上的资源限制。您必须使用“键”=“值”对来指定资源限制。一个资源组可以设置多个资源限制。

  以下列出了资源限制的参数：

  |参数|必填|说明|
|---|---|---|
  |cpu_core_limit|No|BE 上可分配给资源组的 CPU 核心数的软限制。在实际业务场景中，分配给资源组的CPU核数会根据BE上CPU核数的可用性进行比例伸缩。有效值：任何非零正整数。|
  |mem_limit|No|可用于查询的内存占 BE 提供的总内存的百分比。单元： ％。有效值：(0, 1)。|
  |concurrency_limit|No|资源组中并发查询的上限。用于避免并发查询过多而导致系统过载。|
  |max_cpu_cores|No|单个 BE 节点上此资源组的 CPU 核心限制。仅当设置大于0时才生效。范围：[0, avg_be_cpu_cores]，其中avg_be_cpu_cores表示所有BE节点的平均CPU核数。默认值：0。|
  |big_query_cpu_second_limit|否|大查询的CPU占用时间上限。并发查询会增加时间。单位是第二。|
  |big_query_scan_rows_limit|否|大查询可以扫描的行数上限。|
  |big_query_mem_limit|否|大查询的内存使用上限。单位为字节。|
  |type|No|资源组的类型。有效值：short_query：当运行来自short_query 资源组的查询时，BE 节点会保留short_query.cpu_core_limit 中定义的CPU 核心。所有普通资源组的 CPU 核心限制为“CPU 核心总数 -short_query.cpu_core_limit”。正常：当short_query资源组没有查询正在运行时，上述CPU核心限制不会施加在正常资源组上。请注意，您只能在集群中创建一个 Short_query 资源组。|

## 示例

示例1：根据多个分类器创建名为rg1的资源组。

```SQL
CREATE RESOURCE GROUP rg1
TO 
    (user='rg1_user1', role='rg1_role1', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user2', query_type in ('select'), source_ip='192.168.x.x/24'),
    (user='rg1_user3', source_ip='192.168.x.x/24'),
    (user='rg1_user4'),
    (db='db1')
WITH ('cpu_core_limit' = '10',
      'mem_limit' = '20%',
      'type' = 'normal',
      'big_query_cpu_second_limit' = '100',
      'big_query_scan_rows_limit' = '100000',
      'big_query_mem_limit' = '1073741824'
);
```
