# Protocol Buffers v3

## 定义服务

```proto3
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}

// 可以在 .proto 文件中定义一个 RPC 服务接口
// protocolbuffer 编译器将以你选择的编译器生成服务接口代码和存根
```

</br>

### 1、gRPC

&emsp;&emsp; 一种由 `Google` 开发的语言和平台中立的开源 `RPC` 系统，`gRPC` 特别适用于协议缓冲区，并允许你的 `.proto` 使用指定的协议缓冲区编译器插件直接从文件生成相关的 `RPC` 代码。

</br>

### 2、JSON 映射

&emsp;&emsp; 1）`Proto3` 支持 `JSON` 中的编码规范，从而更容易在系统之间共享数据。

&emsp;&emsp;&emsp;&emsp; a）当将 `JSON` 编码的数据解析到协议缓冲区时，如果缺少一个值或者它的值为 `null`，他将被解释为相应的默认值。

&emsp;&emsp;&emsp;&emsp; b）从协议缓冲区生成 `JSON` 编码的输出时，如果 `protobuf` 字段具有默认值并且该字段不支持字段存在，则默认情况下它将从输出中省略。实现可以提供选项以在输出中包含具有默认值的字段。

&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; i）使用 `optional` 关键字定义的 `proto3` 字段支持字段存在。设置了值切支持字段存在的字段始终在 `JSON` 编码输出中包含字段值，即使它是默认值。




</br>

