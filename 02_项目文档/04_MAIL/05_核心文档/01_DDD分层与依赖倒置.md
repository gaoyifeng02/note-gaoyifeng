# 核心文档 01 - DDD 分层架构与依赖倒置实现

---

## 1. 六层架构设计

s-pay-mall 严格遵循 DDD 六层架构，每层有明确的职责边界：

```
app（应用启动层）
  │
  ├── trigger（触发器层）──── 调用 ────▶ domain（领域层）
  │                                      │
  │                                      ├── model（领域模型）
  │                                      │   ├── aggregate（聚合根）
  │                                      │   ├── entity（实体）
  │                                      │   └── valobj（值对象）
  │                                      │
  │                                      ├── service（领域服务）
  │                                      │   ├── 接口
  │                                      │   ├── 抽象类（模板方法）
  │                                      │   └── 实现
  │                                      │
  │                                      └── adapter（端口接口）◀── 依赖倒置
  │                                           ├── port（外部调用端口）
  │                                           └── repository（仓储端口）
  │                                                  ▲
  └──────────────────────────────────────────────────┘
                                     infrastructure（基础设施层）实现端口接口
```

## 2. 依赖倒置的实现方式

**核心原则**：domain 层定义接口（端口），infrastructure 层实现接口。

### 2.1 仓储端口（Repository）

```java
// domain 层定义接口
public interface IOrderRepository {
    OrderEntity queryUnPayOrder(ShopCartEntity shopCartEntity);
    void doSaveOrder(CreateOrderAggregate orderAggregate);
    void updateOrderPayInfo(PayOrderEntity payOrderEntity);
    void changeOrderPaySuccess(String orderId);
}

// infrastructure 层实现
@Repository
public class OrderRepository implements IOrderRepository {
    @Resource
    private IOrderDao orderDao;       // MyBatis Mapper
    @Resource
    private IRedisService redisService; // Redis
    @Resource
    private EventBus eventBus;         // 事件总线

    @Override
    public void doSaveOrder(CreateOrderAggregate orderAggregate) {
        // 1. 转换为 PO 对象
        PayOrder order = new PayOrder();
        // ... 属性映射
        // 2. 写入 MySQL
        orderDao.insert(order);
        // 3. 写入 Redis
        redisService.setValue(PayOrder.cacheKey(userId, orderId), order);
    }
}
```

### 2.2 外部服务端口（Port）

```java
// domain 层定义接口
public interface IProductPort {
    ProductEntity queryProductByProductId(String productId);
}

public interface ILoginPort {
    String createQrCodeTicket();
    void sendLoginTempleteMessage(String openid);
}

// infrastructure 层实现
@Component
public class ProductRPC implements IProductPort {
    @Override
    public ProductEntity queryProductByProductId(String productId) {
        // Mock 实现，未来可替换为真实 RPC
        return ProductEntity.builder()
                .productId(productId)
                .productName("DDD 架设计与开发")
                .price(new BigDecimal("1.68"))
                .build();
    }
}
```

## 3. 为什么 domain 不依赖 infrastructure

这是 DDD 的核心原则，本项目的实现方式：

| 做法 | domain 层 | infrastructure 层 |
|------|-----------|-------------------|
| 定义接口 | `IOrderRepository` | - |
| 实现接口 | - | `OrderRepository implements IOrderRepository` |
| 依赖方向 | domain 不知道 MySQL/Redis 的存在 | infrastructure 知道 domain 的接口 |
| Spring 注入 | domain 层 `@Resource IOrderRepository` | infrastructure 的实现被自动注入 |

这样 domain 层只关注业务逻辑，持久化的具体技术（MySQL、Redis、MongoDB）对 domain 完全透明。
