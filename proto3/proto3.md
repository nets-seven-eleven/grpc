# Protocol Buffers v3

## 一、定义消息
```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

// a、该文件的第一行指定您正在使用 `proto3` 语法，如果您不这样做，协议缓冲区编译器将假定您正在使用 `proto2`。这必须是文件的第一个非空、非注释行。

// b、消息中每个字段都有名称和类型（名称/值对）
```



### 1、指定字段类型

&emsp;&emsp; 1）字段类型可以是 `标量类型`，也可以是 `复合类型`，包括 `枚举` 和 其他复合类型

### 2、分配字段编号

&emsp;&emsp; 1）消息定义中的每个字段都有一个唯一编号。</br>

&emsp;&emsp;&emsp;&emsp; a）这些字段编号用于在 `二进制格式消息` 中标识字段，一旦消息类型被使用，就不应该更改。</br>

&emsp;&emsp;&emsp;&emsp; b）请注意，`1~15` 范围内的字段编号需要一个字节进行编码，包括字段编号和字段类型。`16~2047` 范围内的字段编号占用两个字节。因此，我们应该为非常频繁出现的消息元素保留数字1~15。也可以为将来可能添加的频繁出现的元素流出一些空间。</br>

&emsp;&emsp;&emsp;&emsp; c）可以指定的 `最小字段编号为1`，`最大字段编号为2^29 - 1，即 536870911`。但是，也不能使用数字 `19000～19999（FieldDescriptor::kFirstReservedNumber到 FieldDescriptor::kLastReservedNumber）`，因为它们是为 `Protocol Buffers` 实现保留的。</br>







