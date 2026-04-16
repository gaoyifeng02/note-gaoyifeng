# Maven 多模块项目 `spring-boot:run -pl` 找不到兄弟模块依赖

## 问题描述

在 Maven 多模块项目中，使用 `mvn spring-boot:run -pl java-base-app` 单独启动 app 模块时，报错找不到兄弟模块（trigger、infrastructure）的依赖：

```
Could not resolve dependencies for project com.gaoyifeng:java-base-app:jar:1.0.0-SNAPSHOT
Could not find artifact com.gaoyifeng:java-base-trigger:jar:1.0.0-SNAPSHOT in aliyun-public
Could not find artifact com.gaoyifeng:java-base-infrastructure:jar:1.0.0-SNAPSHOT in aliyun-public
```

## 原因

`-pl` 指定子模块后，Maven 只从本地仓库（`~/.m2/repository`）和远程仓库解析依赖，不会自动编译关联的兄弟模块。如果兄弟模块没有 install 到本地仓库，就会找不到。

## 解决方案

**先从根目录执行全量 install，再启动单模块：**

```bash
# 第一步：安装所有模块到本地仓库
mvn install -DskipTests

# 第二步：启动指定模块
mvn spring-boot:run -pl java-base-app
```

## 适用场景

所有 Maven 多模块项目（DDD 分层架构项目均适用）。

## 关联项目

- java-base
