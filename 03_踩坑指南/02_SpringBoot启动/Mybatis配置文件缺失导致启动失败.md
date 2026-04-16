# MyBatis 配置文件缺失导致 Spring Boot 启动失败

## 问题描述

Spring Boot 启动时报错：

```
Error creating bean with name 'sqlSessionFactory'
class path resource [mybatis/config/mybatis-config.xml] cannot be opened because it does not exist
```

## 原因

`application-dev.yml` 中配置了 MyBatis 的 config-location：

```yaml
mybatis:
  config-location: classpath:mybatis/config/mybatis-config.xml
```

但 `resources/mybatis/config/` 目录下只创建了目录和占位文件（temp.txt），没有创建实际的 `mybatis-config.xml`。MyBatis AutoConfiguration 在构建 SqlSessionFactory 时要求该文件必须存在。

## 解决方案

**新建项目时，配置文件引用的资源文件必须同步创建，不能只建目录。**

补建 `mybatis-config.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

## 注意事项

新建项目时需检查以下资源文件是否都已创建：
- `mybatis/config/mybatis-config.xml`
- `application.yml` / `application-{profile}.yml`
- `logback-spring.xml`

## 关联项目

- java-base
