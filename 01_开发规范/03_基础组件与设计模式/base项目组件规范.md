# Base 项目基础组件与设计模式规范

> 所有新建项目必须基于 java-base 项目，使用其中定义的统一响应封装和设计模式。
> 代码仓库：`D:\workspace\gaoyifeng02\java-base`

---

## 一、统一响应封装（Result 体系）

> 所在模块：`java-base-types` → `com.gaoyifeng.java.base.types.common`

### 1.1 核心类关系

```
ResultConstant (接口)          ← 响应码契约
    ├── ResponseCode (枚举)    ← 标准响应码
    └── 业务自定义实现          ← 业务可扩展自己的响应码

Result<T>                      ← 统一响应包装
PageResult<T>                  ← 分页数据封装（作为 Result.data 使用）
```

### 1.2 ResultConstant 接口

所有响应码的契约接口，业务如需自定义响应码，必须实现此接口。

```java
public interface ResultConstant {
    String getCode();  // 响应码
    String getInfo();  // 响应信息
}
```

**规范要点：**
- 业务模块新增自定义响应码时，实现此接口即可，无需修改 ResponseCode
- 示例：`public enum OrderResponseCode implements ResultConstant { ... }`

### 1.3 ResponseCode 标准响应码

标准响应码枚举，覆盖通用场景，所有项目共用：

| 响应码 | 枚举值 | 含义 |
|--------|--------|------|
| `0000` | `SUCCESS` | 成功 |
| `E0001` | `FAIL` | 失败 |
| `E0002` | `UNAUTHORIZED` | 未授权 |
| `E0003` | `PARAM_ERROR` | 参数错误 |
| `E0004` | `NOT_FOUND` | 数据不存在 |
| `E0005` | `DUPLICATE` | 数据已存在 |
| `E0006` | `TOO_MANY_REQUESTS` | 请求过于频繁 |
| `E9999` | `INTERNAL_ERROR` | 系统异常 |

**规范要点：**
- 业务特定的错误码（如 `E1001` 订单不存在）应新建枚举实现 `ResultConstant`，不修改此枚举
- 编码范围建议：`E1xxx` 订单、`E2xxx` 支付、`E3xxx` 用户，按业务域划分

### 1.4 Result 统一响应类

所有 Controller 接口必须返回 `Result<T>` 类型。

```java
@Data
@Builder
public class Result<T> {
    private String code;    // 响应码
    private String info;    // 响应信息
    private T data;         // 业务数据
}
```

**使用方式：**

```java
// 成功（带数据）
return Result.success(userEntity);

// 成功（无数据）
return Result.success();

// 失败（标准响应码）
return Result.fail(ResponseCode.PARAM_ERROR);

// 失败（自定义响应码）
return Result.fail(OrderResponseCode.ORDER_NOT_FOUND);

// 失败（自定义响应码 + 覆盖信息）
return Result.fail(OrderResponseCode.ORDER_NOT_FOUND, "订单已取消");
```

### 1.5 PageResult 分页封装

分页查询接口统一使用 `Result<PageResult<T>>` 作为返回类型。

```java
@Data
@Builder
public class PageResult<T> {
    private Long total;              // 总条数
    private Integer pageNo;          // 当前页码
    private Integer pageSize;        // 每页大小
    private Collection<T> result;    // 当前页数据
}
```

**使用方式：**

```java
PageResult<OrderDTO> pageResult = PageResult.<OrderDTO>builder()
        .total(totalCount)
        .pageNo(pageNo)
        .pageSize(pageSize)
        .result(orderList)
        .build();
return Result.success(pageResult);
```

---

## 二、设计模式（design 包）

> 所在模块：`java-base-types` → `com.gaoyifeng.java.base.types.design`
> 包含两个子包：`tree`（策略模式）、`link`（责任链模式）

### 2.1 策略模式（tree 包）

**适用场景：** 根据不同的请求参数，选择不同的处理策略（如不同支付方式、不同消息类型处理）。

#### 核心类

| 类 | 职责 |
|-----|------|
| `StrategyHandler<T, D, R>` | 策略处理接口，定义 `apply()` 方法 |
| `StrategyMapper<T, D, R>` | 策略映射接口，定义 `get()` 方法选择策略 |
| `AbstractStrategyRouter<T, D, R>` | 策略路由抽象类，串联映射和执行 |
| `AbstractMultiThreadStrategyRouter<T, D, R>` | 多线程策略路由，支持异步数据加载 |
| `DynamicContext` | 上下文对象，在策略间共享数据 |

#### 使用步骤

**第一步：定义上下文（如需要共享数据）**

```java
public class OrderContext extends DynamicContext {
    // 添加业务特有字段
}
```

**第二步：实现策略处理器**

```java
@Component
public class AlipayHandler implements StrategyHandler<OrderRequest, DynamicContext, Result> {
    @Override
    public Result apply(OrderRequest request, DynamicContext context) throws Exception {
        // 支付宝支付逻辑
        return Result.success();
    }
}
```

**第三步：实现策略路由器**

```java
@Component
public class PaymentRouter extends AbstractStrategyRouter<OrderRequest, DynamicContext, Result> {

    @Override
    public StrategyHandler<OrderRequest, DynamicContext, Result> get(OrderRequest request, DynamicContext context) {
        switch (request.getPayType()) {
            case ALIPAY: return alipayHandler;
            case WECHAT: return wechatHandler;
            default: return null; // 走默认策略
        }
    }
}
```

**第四步：调用**

```java
Result result = paymentRouter.router(orderRequest, new DynamicContext());
```

#### 多线程策略路由

当策略执行前需要并行加载多个数据源时，继承 `AbstractMultiThreadStrategyRouter`：

```java
public class OrderRouter extends AbstractMultiThreadStrategyRouter<OrderRequest, DynamicContext, Result> {

    @Override
    protected void multiThread(OrderRequest request, DynamicContext context)
            throws ExecutionException, InterruptedException, TimeoutException {
        // 并行加载数据，结果存入 context
    }

    @Override
    protected Result doApply(OrderRequest request, DynamicContext context) throws Exception {
        // 使用 context 中的数据执行业务逻辑
        return Result.success();
    }
}
```

生命周期方法：
- `applyBefore()` → `multiThread()` → `doApply()` → `applyAfter()`
- 异常时回调 `applyAfterException()`

---

### 2.2 责任链模式（link 包）

**适用场景：** 多个处理器按顺序依次处理请求，每个处理器可决定是否继续传递（如参数校验 → 风控检查 → 业务处理 → 日志记录）。

#### 核心类

| 类 | 职责 |
|-----|------|
| `ILogicHandler<T, D, R>` | 处理器接口，定义生命周期方法 |
| `BusinessLinkedList<T, D, R>` | 链路容器，继承 LinkedList，驱动处理器依次执行 |
| `LinkArmory<T, D, R>` | 链路组装工厂，简化链路构建 |
| `DynamicContext` | 上下文对象，含 `proceed` 和 `jump` 控制标识 |

#### 处理器生命周期

每个处理器按以下顺序执行：

```
applyBefore() → apply() → applyAfter()
                         → applyAfterException() (异常时)
```

#### 链路控制

| 方法 | 效果 | 场景 |
|------|------|------|
| `next()` | `proceed=true, jump=false` | 继续执行下一个节点（正常流转） |
| `stop()` | `proceed=false` | 中断整个链路（校验失败、业务终止） |
| `jump()` | `proceed=true, jump=true` | 跳过当前节点，继续下一个（条件不满足时跳过） |

#### 使用步骤

**第一步：定义处理器**

```java
@Component
public class ParamValidateHandler implements ILogicHandler<OrderRequest, DynamicContext, Result> {

    @Override
    public Result apply(OrderRequest request, DynamicContext context) throws Exception {
        if (request.getAmount() == null) {
            return stop(request, context, Result.fail(ResponseCode.PARAM_ERROR));
        }
        return next(request, context); // 继续下一个
    }
}

@Component
public class RiskCheckHandler implements ILogicHandler<OrderRequest, DynamicContext, Result> {
    @Override
    public Result apply(OrderRequest request, DynamicContext context) throws Exception {
        // 风控检查逻辑
        boolean safe = riskService.check(request);
        if (!safe) {
            return stop(request, context, Result.fail(ResponseCode.FAIL));
        }
        return next(request, context);
    }
}

@Component
public class OrderCreateHandler implements ILogicHandler<OrderRequest, DynamicContext, Result> {
    @Override
    public Result apply(OrderRequest request, DynamicContext context) throws Exception {
        // 创建订单
        return stop(request, context, Result.success(orderId)); // 最终结果
    }
}
```

**第二步：组装链路**

```java
@Configuration
public class OrderLinkConfig {

    @Bean
    public LinkArmory<OrderRequest, DynamicContext, Result> orderLink(
            ParamValidateHandler paramValidate,
            RiskCheckHandler riskCheck,
            OrderCreateHandler orderCreate) {
        return new LinkArmory<>("orderLink", paramValidate, riskCheck, orderCreate);
    }
}
```

**第三步：调用**

```java
BusinessLinkedList<OrderRequest, DynamicContext, Result> link = orderLink.getLogicLink();
Result result = link.apply(orderRequest, new DynamicContext());
```

---

## 三、通用规范

### 3.1 DynamicContext 使用

两个设计模式的 `DynamicContext` 都支持通过 key-value 存取共享数据：

```java
// 存
context.setValue("userId", 12345L);

// 取
Long userId = context.getValue("userId");
```

如需扩展上下文字段，继承 `DynamicContext`：

```java
public class OrderContext extends DynamicContext {
    public Long getUserId() { return getValue("userId"); }
    public void setUserId(Long id) { setValue("userId", id); }
}
```

### 3.2 泛型参数约定

| 泛型 | 含义 | 常见类型 |
|------|------|---------|
| `T` | 请求参数 | `XxxRequest`、`XxxDTO` |
| `D` | 动态上下文 | `DynamicContext` 或其子类 |
| `R` | 返回结果 | 通常是 `Result<XxxResponse>` |

### 3.3 模块归属

- 所有设计模式的基础类放在 `java-base-types` 模块的 `design` 包下
- 具体业务的策略/链路处理器放在各自业务模块中（domain/case）
- 新增通用设计模式组件时，应更新 java-base 项目
