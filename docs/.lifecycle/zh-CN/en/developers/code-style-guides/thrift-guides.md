---
displayed_sidebar: "Chinese"
---

# 规则

## 永远不要使用required

随着项目的发展，任何字段都可能变成可选的。但如果它被定义为必需的，就不能被移除。

因此不应该使用 `required`。

## 永远不要改变序数

为了向后兼容，字段的序数不应该被改变。

# 命名

## 文件名

消息的名称都应采用小写，单词之间以下划线分隔。
文件应以 `.thrift` 结尾。

```
my_struct.thrift            // 好
MyStruct.thrift             // 差
my_struct.proto             // 差
```

## 结构体名称

结构体名称以大写字母 `T` 开头，每个新单词的首字母大写，无下划线：TMyStruct

```
struct TMyStruct;           // 好
struct MyStruct;            // 差
struct TMy_Struct;          // 差
struct TmyStruct;           // 差
```

## 字段名称

结构体成员的名称都应采用小写，单词之间以下划线分隔。

```
1: optional i64 my_field;       // 好
1: optional i64 myField;        // 差