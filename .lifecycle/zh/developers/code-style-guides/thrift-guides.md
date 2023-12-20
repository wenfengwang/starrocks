---
displayed_sidebar: English
---

# 规则

## 永远不要使用“必需”

随着项目的发展，任何字段都有可能变为可选。但如果一个字段一旦被定义为“必需”，那么它就不能被移除。

因此，“必需”这一术语不应该被使用。

## 绝不改变顺序值

为了保持向后兼容性，字段的顺序值不应该被更改。

# 命名规则

## 文件名

消息的名称应全部使用小写字母，并且单词之间以下划线分隔。文件名应以 .thrift 为后缀。

```
my_struct.thrift            // Good
MyStruct.thrift             // Bad
my_struct.proto             // Bad
```

## 结构体名

结构体的名称应以大写字母T开头，并且每个新单词的首字母都应大写，名称中不包含下划线：TMyStruct

```
struct TMyStruct;           // Good
struct MyStruct;            // Bad
struct TMy_Struct;          // Bad
struct TmyStruct;           // Bad
```

## 字段名

结构体成员的名称应全部使用小写字母，并且单词之间以下划线分隔。

```
1: optional i64 my_field;       // Good
1: optional i64 myField;        // Bad
```
