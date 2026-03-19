● 🗺️Map.of 详解（第 18-28 行）

  Map.of(
      "protocolVersion", "2024-11-05",
      "capabilities", Map.of(...),
      "serverInfo", Map.of(...)
  )

  ---
  什么是 Map.of？

  Map.of() 是 Java 9 引入的创建不可变 Map 的便捷方法

  ---
  基本用法

  // ✅ Map.of - 创建不可变 Map
  Map<String, String> map1 = Map.of("key1", "value1", "key2", "value2");

  // ❌ 尝试修改会抛出异常
  map1.put("key3", "value3");  // UnsupportedOperationException

  ---
  支持的重载方法（最多 10 对键值）

  // 空Map
  Map<String, String> empty = Map.of();

  // 1 对
  Map.of("key1", "value1");

  // 2 对
  Map.of("key1", "value1", "key2", "value2");

  // ...最多支持 10 对
  Map.of(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5,
        k6, v6, k7, v7, k8, v8, k9, v9, k10, v10);

  // 超过 10 对，使用 Map.ofEntries()
  Map.ofEntries(
      Map.entry("key1", "value1"),
      Map.entry("key2", "value2"),
      Map.entry("key3", "value3"),
      // ... 无限制
  );

  ---
  嵌套使用（你的代码示例）

  Map.of(
      "protocolVersion", "2024-11-05",           // String → String
      "capabilities", Map.of(                     // String → Map（嵌套）
          "tools", Map.of(),                      // 空 Map
          "resources", Map.of()
      ),
      "serverInfo", Map.of(                       // String → Map（嵌套）
          "name", "MCP Weather Proxy Server",
          "version", "1.0.0"
      )
  )

  等价的 JSON 结构：
  {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {},
      "resources": {}
    },
    "serverInfo": {
      "name": "MCP Weather Proxy Server",
      "version": "1.0.0"
    }
  }

  ---
  Java 8 vs Java 9+ 对比

  ❌ Java 8 写法（繁琐）

  Map<String, Object> serverInfo = new HashMap<>();
  serverInfo.put("name", "MCP Weather Proxy Server");
  serverInfo.put("version", "1.0.0");

  Map<String, Object> capabilities = new HashMap<>();
  capabilities.put("tools", new HashMap<>());
  capabilities.put("resources", new HashMap<>());

  Map<String, Object> result = new HashMap<>();
  result.put("protocolVersion", "2024-11-05");
  result.put("capabilities", capabilities);
  result.put("serverInfo", serverInfo);

  ✅ Java 9+ 写法（简洁）

  Map.of(
      "protocolVersion", "2024-11-05",
      "capabilities", Map.of("tools", Map.of(), "resources", Map.of()),
      "serverInfo", Map.of("name", "MCP Weather Proxy Server", "version", "1.0.0")
  );

  ---
  ⚠️ 重要限制

  // 1. 不能有 null 键或值
  Map.of("key", null);  // ❌ NullPointerException

  // 2. 不能有重复键
  Map.of("key", "value1", "key", "value2");  // ❌ IllegalArgumentException

  // 3. 不可变（不能修改）
  Map<String, String> map = Map.of("key", "value");
  map.put("key2", "value2");  // ❌ UnsupportedOperationException
  map.remove("key");           // ❌ UnsupportedOperationException

  // 4. 最多 10 个键值对（使用 Map.of() 时）
  Map.of(k1, v1, ..., k10, v10, k11, v11);  // ❌ 编译错误

  ---
  相关方法（Java 9+）

  ┌────────────────────────────────────┬──────────────────┬────────┐
  │                方法                │       说明       │  版本  │
  ├────────────────────────────────────┼──────────────────┼────────┤
  │ Map.of()                           │ 创建不可变空 Map │ Java 9 │
  ├────────────────────────────────────┼──────────────────┼────────┤
  │ Map.of(k, v)                       │ 1 对键值         │ Java 9 │
  ├────────────────────────────────────┼──────────────────┼────────┤
  │ Map.ofEntries(entry1, entry2, ...) │ 无限制键值       │ Java 9 │
  ├────────────────────────────────────┼──────────────────┼────────┤
  │ List.of()                          │ 创建不可变 List  │ Java 9 │
  ├────────────────────────────────────┼──────────────────┼────────┤
  │ Set.of()                           │ 创建不可变 Set   │ Java 9 │
  └────────────────────────────────────┴──────────────────┴────────┘

  ---
  你的代码场景分析

  你的代码中 Map.of 很适合这个场景，因为：

  1. ✅ 数据结构固定 - 响应格式不会改变
  2. ✅ 不需要修改 - 只是构造返回数据
  3. ✅ 代码简洁 - 一行搞定嵌套结构
  4. ✅ 线程安全 - 不可变对象天然线程安全

  ---
  如果需要可变 Map，还是用传统方式：
  Map<String, Object> map = new HashMap<>();
  map.put("key", "value");
  map.put("key2", "value2");  // ✅ 可以修改