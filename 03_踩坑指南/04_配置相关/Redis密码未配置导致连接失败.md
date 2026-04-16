# Redis 密码未配置导致连接失败

## 问题描述

Spring Boot 启动时连接 Redis 失败（或运行时操作 Redis 报认证错误）。

## 原因

`docker-compose-environment.yml` 中 Redis 启用了密码认证：

```yaml
redis:
  command: redis-server --requirepass gaoyifeng
```

但 `application-dev.yml` 中 Redis 配置缺少 `password` 字段：

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      # 缺少 password
```

## 解决方案

**在应用配置中补上与 docker-compose 一致的密码：**

```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password: gaoyifeng
```

## 注意事项

新建项目时，`application-dev.yml` 中的数据库密码、Redis 密码必须与 `docker-compose-environment.yml` 中定义的密码保持一致。建议在创建项目时同步检查两处配置。

## 关联项目

- java-base
