# 风格指南

</br>

### 1、标准文件格式

- 将行长度保持在 80 个字符以内
- 使用 2 个空格的缩进
- 更喜欢对字符串使用双引号


</br>

### 2、文件结构

- 文件应命名为 `lower_snake_case.proto`

- 所有文件应按以下方式排序：
  - License header （许可证，如果适用）
  - File overview （文件概览）
  - Syntax （语法版本）
  - Package （包名）
  - Imports （导入包）
  - File options （文件选项）
  - Everything else （其他一切）


</br>

### 3、Package

- 包名应该是小写的

- 包名称应该具有基于项目名称的唯一名称，并且可能基于包含协议缓冲区类型定义的文件的路径

</br>

### 4、消息和字段名称

- 对消息名称使用 `PascalCase` （首字母大写）

- 将 `lower_snake_case` 用于字段名称

```proto3
    message SongServerRequest {
      optional string song_name = 1;
    }
```

- 如果你的字段名称包含数字，则该数字应出现在字母之后而不是下划线之后。例如：使用 `song_name1` 而不是 `song_name_1`

</br>

### 5、重复字段

- 对重复字段使用复数名称

```proto3
    repeated string keys = 1;
        ...
        repeated MyMessage accounts = 17;
```

</br>

### 6、枚举

- 枚举类型名称使用 `PascalCase` （首字母大写）

- 值名称使用 `CAPITALS_WITH_UNDERSCORES`

```proto3
    enum FooBar {
      FOO_BAR_UNSPECIFIED = 0;
      FOO_BAR_FIRST_VALUE = 1;
      FOO_BAR_SECOND_VALUE = 2;
    }
```

- 每个枚举值都应以分号而不是逗号结尾

- 更喜欢在枚举值前面加上前缀，而不是将它们包围在封闭的消息中

- 零值枚举应具有后缀 `UNSPECIFIED`，因为获得意外枚举值的服务器或应用程序在原型实例中将该字段标记为未设置。然后，字段访问器将返回默认值，对于枚举字段，默认值是第一个枚举值。

</br>

### 7、服务

- 服务名称和任何 `RPC` 方法名称使用 `PascalCase` （首字母大写）

```proto3
    service FooService {
      rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
      rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
    }
```

</br>

### 8、要避免的事情

- Required fields （必填字段，仅适用于 `proto2`）

- Groups （组，仅适用于 `proto2`）






