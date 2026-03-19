~~~
  public record JSONRPCRequest(
      @JsonProperty("jsonrpc") String jsonrpc,
      @JsonProperty("method") String method,
      @JsonProperty("id") Object id,
      @JsonProperty("params") Object params
  ) implements JSONRPCMessage {}

  这是什么？
  - record 是不可变数据类（Java 14 预览，Java 16 正式）
  - 自动生成：构造器、getter、equals()、hashCode()、toString()

  自动生成的代码等价于：
  public class JSONRPCRequest implements JSONRPCMessage {
      private final String jsonrpc;
      private final String method;
      private final Object id;
      private final Object params;

      // 自动生成全参构造器
      public JSONRPCRequest(String jsonrpc, String method, Object id, Object params) {
          this.jsonrpc = jsonrpc;
          this.method = method;
          this.id = id;
          this.params = params;
      }

      // 自动生成 getter（注意：没有 get 前缀！）
      public String jsonrpc() { return jsonrpc; }
      public String method() { return method(); }

      // 自动生成 equals、hashCode、toString
  }

  使用示例：
  // 创建实例（简洁！）
  JSONRPCRequest request = new JSONRPCRequest("2.0", "tools/list", 1, null);

  // 访问字段（直接调用方法名）
  String method = request.method();  // 不是 request.getMethod()
~~~