~~~
️⃣密封接口（第 41 行）

  public sealed interface JSONRPCMessage permits JSONRPCRequest, JSONRPCResponse {
      String jsonrpc();
  }

  这是什么？
  - sealed 关键字（Java 17 引入）表示密封接口/类
  - permits 指定哪些类可以实现这个接口

  有什么好处？
  // 编译器强制检查：只有 JSONRPCRequest 和 JSONRPCResponse 能实现 JSONRPCMessage
  // 其他类想实现会编译错误！

  // 在 switch 表达式（Java 14+）中可以实现穷尽匹配
  String message = switch (jsonrpcMessage) {
      case JSONRPCRequest req -> "请求: " + req.method();
      case JSONRPCResponse res -> "响应";
      // 不需要 default，因为编译器知道所有可能的类型
  };

  Java 8 替代方案：
  // 没有 sealed，需要手动判断 + 强制类型转换
  if (msg instanceof JSONRPCRequest) {
      JSONRPCRequest req = (JSONRPCRequest) msg;
  }

~~~