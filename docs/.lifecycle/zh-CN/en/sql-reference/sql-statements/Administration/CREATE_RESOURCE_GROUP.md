---
displayed_sidebar: "Chinese"
---

# 创建资源组

## 描述

创建一个资源组。

有关更多信息，请参阅[资源组](../../../administration/resource_group.md)。

## 语法

```SQL
CREATE RESOURCE GROUP resource_group_name 
TO CLASSIFIER1, CLASSIFIER2, ...
WITH resource_limit
```

## 参数

- `resource_group_name`：要创建的资源组的名称。

- `CLASSIFIER`：用于对资源限制进行过滤的分类器。您必须使用“key”=“value”对指定分类器。您可以为资源组设置多个分类器。

  分类器的参数如下所示：

    | 参数           | 是否必需 | 描述                                      |
    | -------------- | -------- | ------------------------------------------ |
    | user           | 否       | 用户的名称。                              |
    | role           | 否       | 用户的角色。                              |
    | query_type     | 否       | 查询的类型。支持`SELECT`和`INSERT`（从v2.5开始）。当`query_type`为`insert`的INSERT任务命中资源组时，BE节点会为任务保留指定的CPU资源。 |
    | source_ip      | 否       | 发起查询的CIDR块。                       |
    | db             | 否       | 查询访问的数据库。可以由逗号（,）分隔的字符串指定。 |
    | plan_cpu_cost_range | 否 | 查询的预估CPU成本范围。格式为（DOUBLE, DOUBLE]。默认值为NULL，表示没有此类限制。自v3.1.4版本开始支持此参数。 |
    | plan_mem_cost_range | 否 | 查询的预估内存成本范围。格式为（DOUBLE, DOUBLE]。默认值为NULL，表示没有此类限制。自v3.1.4版本开始支持此参数。 |

- `resource_limit`：要对资源组施加的资源限制。您必须使用“key”=“value”对指定资源限制。您可以为资源组设置多个资源限制。

  资源限制的参数如下所示：

    | 参数                      | 是否必需 | 描述                            |
    | ------------------------- | -------- | ------------------------------- |
    | cpu_core_limit            | 否       | 在BE上为资源组分配的CPU核心数量的软限制。在实际业务场景中，按比例分配给该资源组的CPU核心会基于BE上的CPU核心的可用性。有效值：任何非零正整数。 |
    | mem_limit                 | 否       | 能够用于查询的内存百分比，占BE提供的总内存的百分比。单位：%。有效值：（0，1）。 |
    | concurrency_limit         | 否       | 资源组中并发查询的上限。用于避免过多并发查询导致系统过载。 |
    | max_cpu_cores             | 否       | 单个BE节点上此资源组的CPU核心限制。仅在设置大于`0`时生效。范围：[0，`avg_be_cpu_cores`]，其中`avg_be_cpu_cores`表示所有BE节点上的CPU核心的平均数量。默认值：0。 |
    | big_query_cpu_second_limit | 否       | 大型查询的CPU占用的时间上限。并发查询会累积时间。单位为秒。 |
    | big_query_scan_rows_limit | 否       | 大型查询可以扫描的行数上限。 |
    | big_query_mem_limit       | 否       | 大型查询的内存使用上限。单位为字节。 |
    | type                      | 否       | 资源组的类型。有效值：<br/>`short_query`：当来自`short_query`资源组的查询运行时，BE节点会保留`short_query.cpu_core_limit`中定义的CPU核心。所有`normal`资源组的CPU核心都限制为“总CPU核心数-`short_query.cpu_core_limit`”。<br/>`normal`：当来自`short_query`资源组的查询未运行时，不会对`normal`资源组施加上述CPU核心限制。<br/>请注意，在集群中只能创建一个`short_query`资源组。 |

## 示例

示例1：基于多个分类器创建资源组`rg1`。

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