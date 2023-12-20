---
displayed_sidebar: English
---

# 规则

## 绝不使用 required

随着项目的发展，任何字段都可能变成可选的。但如果它被定义为 required，那么它就不能被移除。

因此，不应使用 `required`。

## 永远不要改变顺序值

为了保持向后兼容，字段的顺序值不应该被改变。

# 命名

## 文件名

消息的名称应全部小写，单词之间用下划线连接。文件应以 `.thrift` 结尾。

```
my_struct.thrift            // Good
MyStruct.thrift             // Bad
my_struct.proto             // Bad
```

## 结构体名称

结构体名称以大写字母 `T` 开头，每个新单词的首字母大写，不使用下划线：TMyStruct

```
struct TMyStruct;           // Good
struct MyStruct;            // Bad
struct TMy_Struct;          // Bad
struct TmyStruct;           // Bad
```

## 字段名称

结构体成员的名称应全部小写，单词之间用下划线连接。

```
1: optional i64 my_field;       // Good
1: optional i64 myField;        // Bad
```