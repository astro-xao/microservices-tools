---
sidebar_position: 2
---
# Proto API 定义最佳实践

## 1. 基本设计原则
- **保持简单**：API 的设计应尽量简洁，避免过度复杂的嵌套结构。
- **向后兼容**：确保 API 的更新不会破坏现有客户端。
- **明确语义**：字段和消息的命名应清晰，表达其用途。

---

## 2. 文件结构
- **模块化设计**：将 `.proto` 文件按功能模块拆分，避免单个文件过大。
- **统一命名空间**：使用 `package` 定义命名空间，避免命名冲突。
  ```proto
  syntax = "proto3";

  package myapp.user.v1; // 包名应包含版本号
  ```
- **版本管理**：在包名或服务名中包含版本号，方便未来的版本升级。
  ```proto
  package myapp.user.v1; // 包含版本号
  ```
## 3. 字段设计
- **字段编号**：字段编号应从 1 开始，且不能重复或更改已使用的编号。
  ```proto
  message User {
    int32 id = 1;        // 字段编号从 1 开始
    string name = 2;     // 不要更改已使用的编号
  }
  ```
- **字段类型**：选择合适的字段类型，避免使用 optional 和 required（Proto3 中已移除）。
  ```proto
  message User {
      string email = 3;    // 使用 string 表示文本
      bool is_active = 4;  // 使用 bool 表示布尔值
  }
  ```  
- **字段命名**：使用 snake_case 命名字段，保持一致性。
  ```proto
  message User {
      string first_name = 1;
      string last_name = 2;
  }
  ```
- **避免删除字段**：如果字段不再使用，可以标记为废弃，但不要删除，以保持向后兼容。
  ```proto
  message User {
      string deprecated_field = 5 [deprecated = true];
  }
  ```
## 4. 服务定义
- **明确服务职责**：每个服务应专注于单一职责，避免过多的 RPC 方法。
  ```proto
  service UserService {
      rpc GetUser(GetUserRequest) returns (GetUserResponse);
      rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  }
  ```
- **方法命名**：使用动词短语命名方法，清晰表达操作意图。
  ```proto
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  ```
- **请求和响应消息**：每个 RPC 方法应有独立的请求和响应消息，便于扩展。
  ```proto
  message GetUserRequest {
      int32 user_id = 1;
  }

  message GetUserResponse {
      User user = 1;
  }
  ```
## 5. 错误处理
- **使用 gRPC 状态码**：通过 gRPC 的状态码（如 NOT_FOUND、INVALID_ARGUMENT）传递错误信息。
- **扩展错误信息：**：在响应中添加 error_details 字段，提供详细的错误描述。
  ```proto
  message ErrorResponse {
      string error_message = 1;
      int32 error_code = 2;
  }
  ```
## 6. 扩展性
- **使用 oneof**：对于互斥字段，使用 oneof 提高可读性和扩展性。
  ```proto
  message SearchRequest {
      oneof query {
          string user_id = 1;
          string email = 2;
      }
  }
  ```
- **嵌套消息**：将相关的消息嵌套在一起，避免命名冲突。
  ```proto
  message CreateUserRequest {
      message Address {
          string street = 1;
          string city = 2;
      }
      Address address = 1;
  }
  ```
## 7. 注释和文档
- **添加注释**：为每个字段、消息和服务添加注释，方便生成文档。
  ```proto
  // 用户信息
  message User {
      // 用户 ID
      int32 id = 1;
      // 用户姓名
      string name = 2;
  }
  ```
- **使用工具生成文档**：使用工具（如 protoc-gen-doc<sup><a href="#1-protoc-gen-doc">[1]</a></sup>）生成 API 文档。

## 8. 性能优化
- **避免嵌套过深**：深层嵌套会增加解析复杂度，影响性能。
- **使用流式传输**：对于大数据量或实时数据，使用流式 RPC。
  ```proto
  rpc StreamUsers(StreamUsersRequest) returns (stream User);
  ```
## 9. 示例
```proto
syntax = "proto3";

package myapp.user.v1;

// 用户服务
service UserService {
    // 获取用户信息
    rpc GetUser(GetUserRequest) returns (GetUserResponse);
    // 创建用户
    rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
}

// 用户信息
message User {
    int32 id = 1;        // 用户 ID
    string name = 2;     // 用户姓名
    string email = 3;    // 用户邮箱
}

// 请求和响应消息
message GetUserRequest {
    int32 user_id = 1;   // 用户 ID
}

message GetUserResponse {
    User user = 1;       // 用户信息
}

message CreateUserRequest {
    string name = 1;     // 用户姓名
    string email = 2;    // 用户邮箱
}

message CreateUserResponse {
    User user = 1;       // 创建的用户信息
}
```

## 总结
通过遵循这些最佳实践，可以设计出高效、可扩展且易于维护的 gRPC API，同时确保向后兼容性和良好的开发体验。
主要内容整理自：
- [Proto Best Practices
](https://protobuf.dev/best-practices/dos-donts/)
- [API Best Practices
](https://protobuf.dev/best-practices/api/)
- [1-1-1 Best Practice
](https://protobuf.dev/best-practices/1-1-1/)
## 引用
### [1] protoc-gen-doc
  - https://github.com/pseudomuto/protoc-gen-doc/releases