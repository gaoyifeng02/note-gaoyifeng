# Java Record 使用指南

## 1. 概述

Record 是 Java 14 预览、**Java 16 正式发布**的语法特性，用于快速创建**不可变的数据载体类**。

传统写法（一个纯数据类需要大量样板代码）：

```java
public final class User {
    private final String name;
    private final int age;

    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }

    @Override
    public boolean equals(Object o) { /* ... */ }
    @Override
    public int hashCode() { /* ... */ }
    @Override
    public String toString() { /* ... */ }
}
```

Record 写法（一行搞定）：

```java
public record User(String name, int age) {}
```

编译器自动生成：`final` 类、私有 `final` 字段、全参构造器、访问器方法（`name()`、`age()`）、`equals()`、`hashCode()`、`toString()`。

> **本工程要求 Java 21**（`pom.xml` 中 `<java.version>21</java.version>`），完全支持 Record 所有特性。

---

## 2. 基本语法

### 2.1 定义与使用

```java
// 定义
public record Point(double x, double y) {}

// 使用
Point p = new Point(3.0, 4.0);
System.out.println(p.x());    // 3.0（注意：不是 getX()）
System.out.println(p.y());    // 4.0
System.out.println(p);        // Point[x=3.0, y=4.0]
```

### 2.2 访问器方法命名

Record 的访问器方法名与字段名一致，**不带 `get` 前缀**：

```java
record User(String name, int age) {}

User u = new User("Tom", 20);
u.name();  // "Tom"  ✓
u.getName(); // 编译错误 ✗
```

### 2.3 Record 的隐式规则

| 特性 | 说明 |
|------|------|
| `final` 类 | Record 不能被继承 |
| 字段 `private final` | 自动私有、不可变 |
| 不可声明实例字段 | 只能有头部声明的状态参数 |
| 可以声明 `static` 字段和方法 | 允许 |
| 可以实现接口 | 允许 |
| 不能继承类 | Record 隐式继承 `java.lang.Record` |
| 不能是 `abstract` | Record 就是具体的 |

---

## 3. 自定义构造器

### 3.1 紧凑构造器（Compact Constructor）

用于参数校验或规范化，编译器自动在之后赋值给字段：

```java
public record Range(int min, int max) {

    // 紧凑构造器 —— 没有参数列表，自动获取头部参数
    public Range {
        if (min > max) {
            throw new IllegalArgumentException("min > max");
        }
    }
}

new Range(1, 10);   // 正常
new Range(10, 1);   // 抛出 IllegalArgumentException
```

也可以在紧凑构造器中修改参数值：

```java
public record NormalizedText(String value) {
    public NormalizedText {
        value = value.trim().toUpperCase(); // 修改后再赋给字段
    }
}
```

### 3.2 自定义全参构造器

完全控制构造过程：

```java
public record User(String name, int age) {

    public User(String name, int age) {
        if (age < 0) throw new IllegalArgumentException("age < 0");
        this.name = name;
        this.age = age;
    }
}
```

### 3.3 附加构造器（ Convenience Constructor ）

提供简化调用方式，内部委托给规范构造器：

```java
public record User(String name, int age) {

    // 附加构造器必须委托给其他构造器
    public User(String name) {
        this(name, 0);
    }
}
```

---

## 4. 实现接口

### 4.1 普通接口

```java
public record Point(double x, double y) implements Comparable<Point> {
    @Override
    public int compareTo(Point o) {
        return Double.compare(this.x * this.x + this.y * this.y,
                              o.x * o.x + o.y * o.y);
    }
}
```

### 4.2 Sealed Interface（密封接口）

Record 与 `sealed interface` 配合是天作之合——Record 隐式为 `final`，天然满足 sealed 的约束。

```java
// 定义密封接口，只允许三个 record 实现
public sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
}

public record Circle(double radius) implements Shape {
    @Override
    public double area() { return Math.PI * radius * radius; }
}

public record Rectangle(double width, double height) implements Shape {
    @Override
    public double area() { return width * height; }
}

public record Triangle(double base, double height) implements Shape {
    @Override
    public double area() { return 0.5 * base * height; }
}
```

配合 `switch` 模式匹配（Java 21）：

```java
String describe(Shape shape) {
    return switch (shape) {
        case Circle(var r)      -> "圆形，半径=" + r;
        case Rectangle(var w, var h) -> "矩形，宽=" + w + "，高=" + h;
        case Triangle(var b, var h)  -> "三角形，底=" + b + "，高=" + h;
    };
}
```

---

## 5. 嵌套 Record

Record 可以定义在其他类或 Record 内部，天然适合建模层级数据结构：

```java
public record ServerCapabilities(
        CompletionCapabilities completions,
        LoggingCapabilities logging,
        ToolCapabilities tools) {

    // 嵌套 record
    public record CompletionCapabilities() {}
    public record LoggingCapabilities() {}
    public record ToolCapabilities(Boolean listChanged) {}
}

// 使用
ServerCapabilities caps = new ServerCapabilities(
    new ServerCapabilities.CompletionCapabilities(),
    new ServerCapabilities.LoggingCapabilities(),
    new ServerCapabilities.ToolCapabilities(true)
);
```

---

## 6. Record + Jackson

Record 与 Jackson 配合良好（Jackson 2.12+ 完整支持），是 JSON 数据建模的理想选择。

### 6.1 基本序列化/反序列化

```java
public record User(String name, int age) {}

ObjectMapper mapper = new ObjectMapper();

// 序列化
String json = mapper.writeValueAsString(new User("Tom", 20));
// {"name":"Tom","age":20}

// 反序列化
User user = mapper.readValue(json, User.class);
```

### 6.2 字段名映射

Record 访问器不带 `get` 前缀，Jackson 默认按字段名序列化。如果 JSON 中的 key 与 Java 字段名不同，使用 `@JsonProperty`：

```java
public record User(
    @JsonProperty("user_name") String userName,
    @JsonProperty("user_age")  int age
) {}

// JSON: {"user_name":"Tom","user_age":20}
```

### 6.3 忽略未知字段 + 省略空值

这是 Record + Jackson 最常见的组合注解：

```java
@JsonInclude(JsonInclude.Include.NON_ABSENT)
@JsonIgnoreProperties(ignoreUnknown = true)
public record InitializeRequest(
    @JsonProperty("protocolVersion") String protocolVersion,
    @JsonProperty("capabilities") ClientCapabilities capabilities,
    @JsonProperty("clientInfo") Implementation clientInfo
) {}
```

| 注解 | 作用 |
|------|------|
| `@JsonIgnoreProperties(ignoreUnknown = true)` | JSON 中有 Java 类没有的字段时不报错 |
| `@JsonInclude(Include.NON_ABSENT)` | null / Optional.empty 都不序列化 |

---

## 7. Record + Builder 模式

Record 本身不支持 setter（不可变），当构造参数较多时，可以搭配 Builder：

```java
public record ServerCapabilities(
        CompletionCapabilities completions,
        Map<String, Object> experimental,
        LoggingCapabilities logging,
        PromptCapabilities prompts,
        ResourceCapabilities resources,
        ToolCapabilities tools) {

    // 嵌套 record
    public record CompletionCapabilities() {}
    public record LoggingCapabilities() {}
    public record PromptCapabilities(Boolean listChanged) {}
    public record ResourceCapabilities(Boolean subscribe, Boolean listChanged) {}
    public record ToolCapabilities(Boolean listChanged) {}

    // Builder
    public static Builder builder() {
        return new Builder();
    }

    public static class Builder {
        private CompletionCapabilities completions;
        private Map<String, Object> experimental;
        private LoggingCapabilities logging = new LoggingCapabilities();
        private PromptCapabilities prompts;
        private ResourceCapabilities resources;
        private ToolCapabilities tools;

        public Builder completions() {
            this.completions = new CompletionCapabilities();
            return this;
        }

        public Builder tools(Boolean listChanged) {
            this.tools = new ToolCapabilities(listChanged);
            return this;
        }

        // ... 其他字段的 setter

        public ServerCapabilities build() {
            return new ServerCapabilities(completions, experimental, logging, prompts, resources, tools);
        }
    }
}
```

使用：

```java
ServerCapabilities caps = ServerCapabilities.builder()
        .logging()
        .tools(true)
        .build();
```

---

## 8. Record 的自定义方法

Record 可以声明实例方法（但只能读取字段，不能修改）和静态方法：

```java
public record Tool(String name, String description, JsonSchema inputSchema) {

    // 自定义构造器 —— 便捷创建方式
    public Tool(String name, String description, String schema) {
        this(name, description, parseSchema(schema));
    }

    // 静态工厂方法
    private static JsonSchema parseSchema(String schema) {
        try {
            return objectMapper.readValue(schema, JsonSchema.class);
        } catch (IOException e) {
            throw new IllegalArgumentException("Invalid schema: " + schema, e);
        }
    }
}

// 两种创建方式
new Tool("search", "搜索", jsonSchemaObj);   // 标准构造器
new Tool("search", "搜索", "{ ... }");        // 自定义构造器（传入 JSON 字符串）
```

---

## 9. Record vs Lombok @Value / Class

| 特性 | `record` | Lombok `@Value` | Lombok `@Data` |
|------|----------|-----------------|----------------|
| 不可变 | 字段 `final`，无 setter | 字段 `final`，无 setter | 有 setter，可变 |
| 语言级支持 | Java 原生语法 | 编译期字节码增强 | 编译期字节码增强 |
| `equals/hashCode` | 自动生成 | 自动生成 | 自动生成 |
| 可继承 | 不可（隐式 final） | 可配置 | 可配置 |
| 实现 sealed 接口 | 天然支持 | 不支持 | 不支持 |
| 额外依赖 | 无 | 需要 Lombok | 需要 Lombok |
| 自定义实例字段 | 不允许 | 允许 | 允许 |

**选择建议**：

- **纯数据载体 / DTO / 协议消息** → 优先用 `record`
- **需要可变性或继承** → 用 Lombok `@Data` / `@Value`
- **密封类型层次（sealed interface + record）** → 必须 record

---

## 10. 项目中的实际应用

本项目在 `McpSchemaVO.java` 中大量使用 Record + Sealed Interface 建模 MCP 协议（JSON-RPC 2.0），是 Record 用法的完整范例。

### 10.1 Sealed Interface 限定消息类型

```java
// 只允许三种消息类型，编译器强制检查 switch 完备性
public sealed interface JSONRPCMessage
        permits JSONRPCRequest, JSONRPCNotification, JSONRPCResponse {
    String jsonrpc();
}
```

### 10.2 Record 建模各消息类型

```java
// 请求 —— 有 method + id
public record JSONRPCRequest(
        @JsonProperty("jsonrpc") String jsonrpc,
        @JsonProperty("method") String method,
        @JsonProperty("id") Object id,
        @JsonProperty("params") Object params
) implements JSONRPCMessage {}

// 通知 —— 有 method，无 id
public record JSONRPCNotification(
        @JsonProperty("jsonrpc") String jsonrpc,
        @JsonProperty("method") String method,
        @JsonProperty("params") Object params
) implements JSONRPCMessage {}

// 响应 —— 有 result 或 error
public record JSONRPCResponse(
        @JsonProperty("jsonrpc") String jsonrpc,
        @JsonProperty("id") Object id,
        @JsonProperty("result") Object result,
        @JsonProperty("error") JSONRPCError error
) implements JSONRPCMessage {}
```

### 10.3 嵌套 Record 建模复合结构

```java
public record ClientCapabilities(
        @JsonProperty("experimental") Map<String, Object> experimental,
        @JsonProperty("roots") RootCapabilities roots,
        @JsonProperty("sampling") Sampling sampling) {

    // 嵌套子结构
    public record RootCapabilities(@JsonProperty("listChanged") Boolean listChanged) {}
    public record Sampling() {}

    // 搭配 Builder 方便构造
    public static Builder builder() { return new Builder(); }
    public static class Builder { /* ... */ }
}
```

### 10.4 自定义构造器提供便捷创建

```java
public record Tool(String name, String description, JsonSchema inputSchema) {

    // 便捷构造器：直接传 JSON 字符串
    public Tool(String name, String description, String schema) {
        this(name, description, parseSchema(schema));
    }
}

public record CallToolRequest(String name, Map<String, Object> arguments) implements Request {

    // 便捷构造器：直接传 JSON 字符串
    public CallToolRequest(String name, String jsonArguments) {
        this(name, parseJsonArguments(jsonArguments));
    }
}
```

---

## 11. 常见问题速查

| 问题 | 解决方案 |
|------|---------|
| 能不能 set 字段值？ | 不能。Record 是不可变的，如需修改可创建新实例 |
| 能不能继承 Record？ | 不能。Record 隐式 `final` |
| 能不能声明额外的实例字段？ | 不能。只能声明 `static` 字段 |
| Record 可以用在 JPA Entity 吗？ | 不推荐。JPA Entity 需要无参构造器和 setter，与 Record 设计理念冲突 |
| Record 可以做 Map 的 key 吗？ | 可以。自动生成的 `equals/hashCode` 基于所有字段 |
| Record 的 `toString()` 能自定义吗？ | 可以，手动覆写 `toString()` 方法 |
| Jackson 反序列化 Record 报错？ | 确保Jackson 2.12+，Record 需要规范构造器（自动生成） |
| `switch` 中如何使用 Record？ | Java 21+ 支持 `case Point(var x, var y) -> ...` 模式匹配 |
