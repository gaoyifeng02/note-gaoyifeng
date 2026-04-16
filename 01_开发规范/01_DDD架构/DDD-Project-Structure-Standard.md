# DDD 项目目录结构规范

> 基于 Spring Boot + Maven 多模块的 DDD（领域驱动设计）分层架构标准目录结构。

---

## 一、整体结构总览

```
project-root/
├── pom.xml                          # 父 POM，聚合所有子模块
├── README.md                        # 项目说明
├── docs/                            # 运维文档与部署资源
├── {project}-api/                   # API 接口层
├── {project}-app/                   # 应用启动层
├── {project}-case/                  # 用例编排层（可选）
├── {project}-domain/                # 领域层（核心）
├── {project}-infrastructure/        # 基础设施层
├── {project}-trigger/               # 触发器层
└── {project}-types/                 # 通用类型层
```

> `{project}` 为项目名称前缀，如 `s-pay-mall`。

---

## 二、模块职责说明

| 模块 | 职责 | 依赖方向 |
|------|------|----------|
| **api** | 定义对外暴露的服务接口、DTO、Response | 被其他模块依赖，自身不依赖任何业务模块 |
| **types** | 通用基础类型：常量、枚举、异常、事件基类、工具类 | 被所有模块依赖，不依赖任何业务模块 |
| **domain** | 核心领域逻辑：领域模型、领域服务、端口接口 | 仅依赖 types 和 api |
| **infrastructure** | 基础设施实现：数据库、缓存、外部网关、仓储实现 | 依赖 domain（实现其端口接口） |
| **case** | 用例编排：跨领域服务编排、复杂业务流程组合 | 依赖 domain 和 api |
| **trigger** | 外部触发入口：HTTP 控制器、定时任务、消息监听器 | 依赖 case、domain 和 api |
| **app** | 应用启动、配置组装、测试入口 | 依赖所有模块 |

### 依赖关系图

```
                    ┌─────────┐
                    │   app   │  ← 启动层，依赖所有模块
                    └────┬────┘
         ┌───────────────┼──────────────────────┐
         ▼               ▼                      ▼
    ┌─────────┐   ┌───────────┐          ┌──────────────┐
    │ trigger │   │   case    │          │infrastructure│
    └────┬────┘   └─────┬─────┘          └──────┬───────┘
         │              │                       │
         │              ▼                       │
         │        ┌───────────┐                 │
         │        │  domain   │◀────────────────┘
         │        └─────┬─────┘  (实现端口接口)
         │              │    ▲
         │              ▼    │
         │        ┌─────────┐│
         │        │  api    ││
         │        └─────────┘│
         │              ▼    │
         │        ┌─────────┐│
         │        │  types  │◀
         │        └─────────┘
         └───────────────────▶ case (调用用例编排服务)

    trigger → case → domain  (推荐调用链路)
    trigger 也可直接调用 domain (简单场景)
    infrastructure 实现 domain 中定义的端口接口（依赖倒置）
```

---

## 三、各模块详细目录结构

### 3.1 API 接口层 `{project}-api`

```
{project}-api/
├── pom.xml
└── src/main/java/{base-package}/api/
    ├── I{xxx}Service.java            # 服务接口定义
    ├── dto/                          # 请求传输对象
    │   └── {xxx}RequestDTO.java
    └── response/                     # 响应对象
        └── Response.java
```

**规范要点：**
- 只定义接口，不含实现
- 只包含 DTO（入参）和 Response（出参）
- 不依赖任何业务模块

---

### 3.2 通用类型层 `{project}-types`

```
{project}-types/
├── pom.xml
└── src/main/java/{base-package}/types/
    ├── common/                       # 通用常量
    │   └── Constants.java
    ├── enums/                        # 枚举定义
    │   └── ResponseCode.java
    ├── event/                        # 事件基类
    │   └── BaseEvent.java
    ├── exception/                    # 自定义异常
    │   └── AppException.java
    └── sdk/                          # 第三方 SDK 封装
        └── {vendor}/
            └── {xxx}Util.java
```

**规范要点：**
- 不依赖任何业务模块，可被所有模块引用
- 枚举、异常、常量等与具体业务无关的通用定义统一放此处

---

### 3.3 领域层 `{project}-domain`

```
{project}-domain/
├── pom.xml
└── src/main/java/{base-package}/domain/
    └── {subdomain}/                  # 按子域划分，如 auth、order、user
        ├── adapter/                  # 适配器接口（端口）
        │   ├── port/                 # 输出端口：调用外部资源
        │   │   └── I{xxx}Port.java
        │   └── repository/           # 仓储接口：持久化
        │       └── I{xxx}Repository.java
        ├── event/                    # 领域事件
        │   └── {xxx}Event.java
        ├── model/                    # 领域模型
        │   ├── aggregate/            # 聚合根
        │   │   └── {xxx}Aggregate.java
        │   ├── entity/               # 实体
        │   │   └── {xxx}Entity.java
        │   └── valobj/               # 值对象
        │       ├── {xxx}VO.java
        │       ├── enums/            # 领域内枚举（可选）
        │       │   └── {xxx}Enum.java
        │       └── {category}/       # 按分类组织值对象（可选）
        │           └── {xxx}VO.java
        └── service/                  # 领域服务
            ├── I{xxx}Service.java            # 接口
            ├── Abstract{xxx}Service.java     # 模板方法（可选）
            ├── {subservice}/                 # 按职责分子包（可选）
            │   └── {xxx}Service.java
            └── handler/                      # 请求处理器（可选）
                ├── I{xxx}Handler.java
                └── impl/
                    └── {xxx}Handler.java
```

**规范要点：**
- 按子域（bounded context）划分顶层包，如 `auth`、`order`、`session`
- `adapter/port` 定义输出端口接口，由 infrastructure 实现
- `adapter/repository` 定义仓储接口，由 infrastructure 实现
- `model` 下严格区分聚合根(aggregate)、实体(entity)、值对象(valobj)
- 领域服务可通过模板方法模式抽取公共逻辑
- 当一个子域内服务较多时，可按职责在 `service/` 下创建子包（如 `license/`、`ratelimit/`），每个子包对应一个独立的接口实现
- `valobj/` 下可按分类创建子包（如 `gateway/`、`http/`）组织相关的值对象
- 领域内的枚举（如状态枚举）放在 `valobj/enums/` 下，区别于 types 层的通用枚举
- `handler/` 用于处理特定协议或消息类型的处理器模式（如 MCP 协议的请求处理器），配合策略模式使用

---

### 3.4 基础设施层 `{project}-infrastructure`

```
{project}-infrastructure/
├── pom.xml
└── src/main/java/{base-package}/infrastructure/
    ├── adapter/                      # 领域端口实现
    │   ├── port/                     # 输出端口实现
    │   │   └── {xxx}Port.java
    │   └── repository/               # 仓储实现
    │       └── {xxx}Repository.java
    ├── dao/                          # 数据访问对象
    │   ├── I{xxx}Dao.java            # DAO 接口（MyBatis Mapper）
    │   └── po/                       # 持久化对象（PO）
    │       └── {xxx}PO.java
    ├── gateway/                      # 外部网关/第三方调用
    │   ├── {xxx}Gateway.java         # 网关实现
    │   └── dto/                      # 外部接口 DTO
    │       └── {xxx}DTO.java
    └── redis/                        # 缓存服务
        ├── IRedisService.java
        └── {xxx}RedisService.java
```

**规范要点：**
- `adapter` 包与 domain 层的 `adapter` 一一对应，实现端口接口
- `dao/po` 存放数据库持久化对象，与数据库表对应，命名以 `PO` 后缀结尾（如 `McpGatewayPO`）
- `gateway` 封装所有外部系统调用（RPC、HTTP、MQ 等），通用 HTTP 调用可命名为 `GenericHttpGateway`
- `gateway/dto` 存放与外部系统交互的 DTO
- 不包含业务逻辑，只做技术实现

---

### 3.5 触发器层 `{project}-trigger`

```
{project}-trigger/
├── pom.xml
└── src/main/java/{base-package}/trigger/
    ├── http/                         # HTTP 接口（Controller）
    │   └── {xxx}Controller.java
    ├── job/                          # 定时任务
    │   └── {xxx}Job.java
    └── listener/                     # 消息/事件监听器
        └── {xxx}Listener.java
```

**规范要点：**
- `http`：REST API 控制器，只做参数校验和调用领域服务或用例服务
- `job`：定时任务调度
- `listener`：事件监听器（如 MQ 消费者、Spring Event 监听）
- 不包含业务逻辑，仅做触发和转发

---

### 3.6 用例编排层 `{project}-case`（可选）

```
{project}-case/
├── pom.xml
└── src/main/java/{base-package}/case/
    └── {usecase}/                    # 按业务用例划分，如 mcp、order
        ├── I{xxx}Service.java               # 用例服务接口
        ├── Abstract{xxx}ServiceSupport.java  # 抽象支撑类（模板方法）
        ├── {xxx}Service.java                # 用例服务实现
        ├── factory/                         # 工厂类（可选）
        │   └── Default{xxx}Factory.java
        └── node/                            # 责任链/流程节点（可选）
            ├── {xxx}Node.java
            └── RootNode.java
```

**规范要点：**
- **定位**：位于 trigger 和 domain 之间，用于编排跨领域服务的复杂业务流程
- **何时引入**：当 trigger 层需要调用多个领域服务组合完成一个业务流程时，提取为用例
- **与 domain/service 的区别**：domain/service 处理单一领域的业务逻辑，case 编排多个 domain/service 协同工作
- **设计模式**：常用模板方法模式（Abstract{xxx}Support）、工厂模式（factory）、责任链模式（node）
- **命名约定**：接口以 `I` 开头，抽象支撑类以 `Abstract` 开头、以 `Support` 结尾
- **不是必须层**：简单业务可直接由 trigger 调用 domain，无需引入 case 层

---

### 3.7 应用启动层 `{project}-app`

```
{project}-app/
├── pom.xml
├── Dockerfile                       # Docker 构建文件
├── build.sh                         # 构建脚本
└── src/
    ├── main/
    │   ├── java/{base-package}/
    │   │   ├── Application.java             # Spring Boot 启动类
    │   │   └── config/                      # 配置类
    │   │       ├── {xxx}Config.java         # 功能配置类（如 GuavaConfig、HTTPClientConfig）
    │   │       ├── {xxx}ConfigProperties.java # 配置属性绑定类
    │   │       ├── ThreadPoolConfig.java    # 线程池配置
    │   │       └── WebMvcConfig.java        # Web MVC 配置
    │   └── resources/
    │       ├── application.yml              # 主配置
    │       ├── application-dev.yml          # 开发环境配置
    │       ├── application-test.yml         # 测试环境配置
    │       ├── application-prod.yml         # 生产环境配置
    │       ├── logback-spring.xml           # 日志配置
    │       └── mybatis/                     # MyBatis 配置与映射
    │           ├── config/
    │           │   └── mybatis-config.xml
    │           └── mapper/
    │               └── {table}_mapper.xml
    └── test/
        └── java/{base-package}/
            └── test/                        # 测试类
                ├── ApiTest.java             # API 基础测试
                ├── domain/                  # 按领域组织测试
                │   ├── {subdomain}/
                │   │   └── {xxx}ServiceTest.java
                │   └── ...
                └── infrastructure/          # 基础设施测试
                    ├── dao/
                    │   └── {xxx}DaoTest.java
                    └── gateway/
                        └── {xxx}GatewayTest.java
```

**规范要点：**
- 启动类 `Application.java` 位于此模块
- `config` 包中放置所有 Spring 配置类及属性绑定类
- 资源文件按环境分离：dev / test / prod
- MyBatis 的 mapper XML 文件统一放 `resources/mybatis/mapper/` 下
- 测试代码按 `domain/{subdomain}` 和 `infrastructure/{dao|gateway}` 结构组织，与主代码模块结构对应

---

### 3.8 运维与部署 `docs/`

```
docs/
└── dev-ops/
    ├── docker-compose-environment.yml   # 基础环境（MySQL、Redis 等）
    ├── docker-compose-app.yml           # 应用服务部署
    ├── app/
    │   ├── start.sh                     # 应用启动脚本
    │   └── stop.sh                      # 应用停止脚本
    └── mysql/
        └── sql/
            └── init.sql                 # 数据库初始化脚本（项目特有）
```

**规范要点：**
- `docker-compose-environment.yml` 为基础环境（MySQL、Redis 等），各项目通用
- `docker-compose-app.yml` 为应用部署，按项目替换 `{project}` 和端口
- `app/start.sh`、`app/stop.sh` 为应用启停脚本，按项目替换容器名和镜像名
- `mysql/sql/init.sql` 为数据库初始化脚本，各项目自行维护

---

#### docker-compose-environment.yml

```yaml
# 命令: docker-compose -f docker-compose-environment.yml up -d
services:
  mysql:
    image: mysql:8.4.3
    container_name: mysql
    restart: always
    environment:
      TZ: Asia/Shanghai
      MYSQL_ROOT_PASSWORD: {password}
    networks:
      - my-network
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/sql:/docker-entrypoint-initdb.d
      - mysql-data:/var/lib/mysql
    healthcheck:
      test: [ "CMD", "mysqladmin" ,"ping", "-h", "localhost" ]
      interval: 5s
      timeout: 10s
      retries: 10
      start_period: 15s

  # phpmyadmin https://hub.docker.com/_/phpmyadmin
  phpmyadmin:
    image: phpmyadmin:5.2.1
    container_name: phpmyadmin
    hostname: phpmyadmin
    ports:
      - 8899:80
    environment:
      - PMA_HOST=mysql
      - PMA_PORT=3306
      - MYSQL_ROOT_PASSWORD={password}
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - my-network

  redis:
    image: redis:7.2.6
    container_name: redis
    restart: always
    hostname: redis
    ports:
      - 6379:6379
    command: redis-server --requirepass {password}
    networks:
      - my-network
    healthcheck:
      test: [ "CMD", "redis-cli", "ping" ]
      interval: 10s
      timeout: 5s
      retries: 3

networks:
  my-network:
    driver: bridge

volumes:
  mysql-data:
```

#### docker-compose-app.yml

```yaml
# 命令: docker-compose -f docker-compose-app.yml up -d
services:
  {project}:
    image: system/{project}:1.0-SNAPSHOT
    container_name: {project}
    restart: on-failure
    ports:
      - "{port}:{port}"
    environment:
      - TZ=PRC
      - SERVER_PORT={port}
    volumes:
      - ./log:/data/log
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

#### app/start.sh

```bash
CONTAINER_NAME={project}
IMAGE_NAME=system/{project}:1.0-SNAPSHOT
PORT={port}

echo "容器部署开始 ${CONTAINER_NAME}"

# 停止容器
docker stop ${CONTAINER_NAME}

# 删除容器
docker rm ${CONTAINER_NAME}

# 启动容器
docker run --name ${CONTAINER_NAME} \
-p ${PORT}:${PORT} \
-d ${IMAGE_NAME}

echo "容器部署成功 ${CONTAINER_NAME}"

docker logs -f ${CONTAINER_NAME}
```

#### app/stop.sh

```bash
docker stop {project}
```

---

## 四、命名规范

| 类别 | 命名规则 | 示例 |
|------|----------|------|
| 模块名 | `{project}-{layer}` | `ai-mcp-gateway-domain` |
| 服务接口 | `I{xxx}Service` | `IAuthLicenseService` |
| 服务实现 | `{xxx}Service` | `AuthLicenseService` |
| 模板抽象类 | `Abstract{xxx}Service` / `Abstract{xxx}Support` | `AbstractMcpMessageServiceSupport` |
| 端口接口 | `I{xxx}Port` | `ISessionPort` |
| 仓储接口 | `I{xxx}Repository` | `IAuthRepository` |
| 控制器 | `{xxx}Controller` | `McpGatewayController` |
| 定时任务 | `{xxx}Job` | `TimeoutCloseOrderJob` |
| 监听器 | `{xxx}Listener` | `OrderPaySuccessListener` |
| 聚合根 | `{xxx}Aggregate` | `CreateOrderAggregate` |
| 实体 | `{xxx}Entity` | `LicenseCommandEntity` |
| 值对象 | `{xxx}VO` | `McpGatewayAuthVO` |
| 持久化对象 | `{xxx}PO` | `McpGatewayPO` |
| 数据传输对象 | `{xxx}DTO` | `ProductDTO` |
| 请求对象 | `{xxx}RequestDTO` | `CreatePayRequestDTO` |
| 响应对象 | `{xxx}Response` 或 `Response` | `Response` |
| 配置类 | `{xxx}Config` | `HTTPClientConfig` |
| 配置属性 | `{xxx}ConfigProperties` | `ThreadPoolConfigProperties` |
| 领域事件 | `{xxx}Event` | `PaySuccessMessageEvent` |
| DAO 接口 | `I{xxx}Dao` | `IMcpGatewayDao` |
| 网关实现 | `{xxx}Gateway` | `GenericHttpGateway` |
| 请求处理器 | `{xxx}Handler` | `ToolsCallHandler` |
| 工厂类 | `{xxx}Factory` / `Default{xxx}Factory` | `DefaultMcpMessageFactory` |
| 责任链节点 | `{xxx}Node` | `MessageHandlerNode` |
| 领域内枚举 | `{xxx}Enum` | `AuthStatusEnum` |
| 错误码枚举 | `{xxx}ErrorCodes` | `McpErrorCodes` |

---

## 五、分层调用原则

1. **domain 层不依赖 infrastructure 层**：通过端口接口（port/repository）实现依赖倒置
2. **trigger 层调用 case 层或 domain 层服务**：不在控制器中直接操作 DAO 或缓存；复杂流程优先调用 case 层编排服务，简单场景可直接调用 domain 层
3. **case 层编排 domain 层服务**：不直接操作 infrastructure，通过 domain 的端口接口间接访问
4. **infrastructure 层实现 domain 定义的接口**：不定义新的业务接口
5. **api 和 types 层独立无依赖**：作为最基础的契约层
6. **app 层负责组装**：通过 Spring 的依赖注入将实现绑定到接口

---

## 六、包路径约定

```
{base-package}/
├── api/                     # api 模块
│   ├── dto/
│   └── response/
├── types/                   # types 模块
│   ├── common/
│   ├── enums/
│   ├── event/
│   ├── exception/
│   └── sdk/
├── domain/                  # domain 模块
│   └── {subdomain}/
│       ├── adapter/
│       ├── event/
│       ├── model/
│       └── service/
├── case/                    # case 模块（可选）
│   └── {usecase}/
│       ├── factory/
│       └── node/
├── infrastructure/          # infrastructure 模块
│   ├── adapter/
│   ├── dao/
│   ├── gateway/
│   └── redis/
├── trigger/                 # trigger 模块
│   ├── http/
│   ├── job/
│   └── listener/
└── config/                  # app 模块中的配置
```

> `{base-package}` 如 `cn.bugstack`，`{subdomain}` 按业务子域划分，如 `auth`、`session`、`protocol`。`{usecase}` 按业务用例划分，如 `mcp`。
