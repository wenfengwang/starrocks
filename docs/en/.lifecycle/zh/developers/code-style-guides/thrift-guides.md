---
displayed_sidebar: English
---

# 规则

## 永远不使用 required

随着项目的发展，任何字段都可能变成可选的。但如果将其定义为必需的，则不能将其删除。

因此不应使用 `required`。

## 永远不更改序数

为了保持向后兼容，字段的序数不应更改。

# 命名

## 文件名

消息的名称都是小写的，单词之间使用下划线。
文件应以 `.thrift` 结尾。

```
my_struct.thrift            // 正确
MyStruct.thrift             // 错误
my_struct.proto             // 错误
```

## 结构名称

结构名称以大写字母 `T` 开头，每个新单词都有一个大写字母，没有下划线：TMyStruct

```
struct TMyStruct;           // 正确
struct MyStruct;            // 错误
struct TMy_Struct;          // 错误
struct TmyStruct;           // 错误
```

## 字段名称

结构成员的名称都是小写的，单词之间使用下划线。

```
1: optional i64 my_field;       // 正确
1: optional i64 myField;        // 错误