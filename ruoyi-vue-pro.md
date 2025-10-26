# ruoyi-vue-pro master-jdk17 分支核心原理说明

**核心：**
1. `yudao-common`
2. `yudao-spring-boot-starter-web`
3. `yudao-spring-boot-starter-mybatis`
4. `yudao-spring-boot-starter-redis`
5. `yudao-spring-boot-starter-security`
6. `yudao-spring-boot-starter-biz-tenant`
7. `yudao-spring-boot-starter-biz-data-permission`
8. `yudao-spring-boot-starter-protection`
9. `yudao-spring-boot-starter-monitor`
10. `yudao-spring-boot-starter-job`
11. `yudao-module-infra`
12. `yudao-module-system`

**非核心**：
13. `yudao-spring-boot-starter-biz-ip`
14. `yudao-spring-boot-starter-job`
15. `yudao-spring-boot-starter-mq`
16. `yudao-spring-boot-starter-excel`
17. `yudao-spring-boot-starter-websocket`
18. `yudao-spring-boot-starter-test`

---

## 1. `yudao-common`
在 `yudao-common` 最重要的包即 `pojo`、`exception`、`enums`。\

**`pojo`**:
1. 定义了 `CommonResult<T>` 用于封装接口返回数据，`CommonResult` 封装了 `code`、`msg`、`data` 字段。
2. 定义了分页参数和分页结果，分页参数 `PageParam`、分页结果 `PageResult`。若需要对分页的结果进行排序，则也提供了 `SortablePageParam`、`SortingField`。

**`exception`**:
1. 定义了表示异常的 `RuntimeException`：`ServerException` 服务器异常、`ServiceException` 业务逻辑异常。
2. 定义了全局错误码 `GlobalErrorCodeConstants` 枚举，用于定义全局错误的各种类型。
3. 提供 `ServiceExceptionUtil` 工具类，方便创建 `ServiceException`。

**`enums`**:
1. 通用状态枚举 `CommonStatusEnum`：ENABLE 启用、DISABLE 禁用
2. 终端的枚举 `TerminalEnum`：UNKNOWN、WECHAT_MINI_PROGRAM、WECHAT_WAP、H5、APP
3. 用户类型枚举 `UserTypeEnum`：ADMIN 管理员、MEMBER 会员

---
## 2. `yudao-spring-boot-starter-web`

**`jackson`**：涉及到 `json` 的操作，均使用 `JsonUtils` 工具类

**`apilog`**：
1. `@ApiAccessLog` 注解，用于记录 API 请求的日志。（直接为 HTTP 接口添加注解即可记录，原理是通过过滤器设计的日志记录，在非 `prod` 环境中也会打印日志到控制台）
2. `ApiAccessLogFilter` 日志记录过滤器（依赖 `@ApiAccessLog`）；`ApiAccessLogInterceptor` 非 `prod` 环境中打印日志到控制台拦截器

**`desensitize`**：
1. @DesensitizeBy(handler = ? extends DesensitizationHandler.class)
2. 基于 `StringDesensitizeSerializer` 实现
3. 用法：给对应字段添加响应的脱敏处理注解即可

**`encrypt`**：
1. 通过 `@ApiEncrypt` 注解，标记 HTTP 接口，设置配置项决定是否对**请求解密**或是**响应加密**
2. 加密方式：AES、RSA；通过 `api-encrypt` 配置算法和密钥

**`swagger`**：
1. 基于 OpenAPI + Springdoc + Knife4j 实现 API 文档功能
2. 正常写 HTTP 接口即可，自动生成文档

**`web`**：
1. `WebProperties` 定义了用户和管理员的 `controller` 的路径；以及请求url的前缀
2. `WebFrameworkUtils` 从 HTTP 请求中获取租户 ID、用户 ID、用户类型、终端类型等信息的方法，并支持设置和获取登录用户相关信息及通用结果对象。
3. `YudaoWebAutoConfiguration` 中定义了为管理员和用户端不同的接口添加对应前缀；以及定义了全局异常处理、全局响应处理（将CommonResult记录到请求头中）、跨域处理、请求缓存过滤器、RestTemplate

**`xss`**：
1. `XssCleaner` 接口 定义了 `clean(String html)` 方法，用于清理 XSS 漏洞。被用于 json 反序列化、HTTP 请求

---

## 3. `yudao-spring-boot-starter-mybatis`
模块基于 MyBatis-Plus 的数据访问层增强组件，提供了一整套完整的数据库访问解决方案。

**`mybatis`(重要)**：
1. `YudaoMybatisAutoConfiguration` 自动配置了
    - 动态sql解析加速（缓存机制）
    - `PaginationInnerInterceptor` 分页支持
    - 常用数据库的自增主键机制（mysql不需要配置）
    - `jacksonTypeHandler` 配置数据库存储json的方式
2. mapper 的基础接口 `BaseMapperX<T>`、DO 基础接口 `BaseDO`
3. `LambdaQueryWrapperX<T>`、`MPJLambdaWrapperX<T>`、`QueryWrapperX<T>` 作为查询条件的封装
4. `MybatisUtils`(分页、字段排序场景常用)、`JdbcUtils`
5. 通过 `@TableField` 注解，指定字段的 `typeHandler`，例如：`@TableField(typeHandler = EncryptTypeHandler.class)` 为密码字段加密
6. `IdTypeEnvironmentPostProcessor` 根据配置的 `spring.datasource.dynamic.primary` 数据源，动态设置 id 自增类型

**`datasource`**：
`DataSourceEnum` 中配置了对应于多数据源中不同数据源配置，master、salve（业务开发中搭配 **@DS** 注解使用，或 **@MASTER**、**@SALVE** 使用）

**`translate`**：
1. `TranslateUtils` 用于将 List<T> data 转换成 List<VO> dataVOList（需要搭配 `@Trans` 注解使用）

---

## 4. `yudao-spring-boot-starter-redis`
1. 配置 `TimeoutRedisCacheManager` 缓存管理器，更方便的配置超时时间。key 的设置可以满足 `<key1>#<entryTtl>:<key2>`，一般情况下使用 `<key>#<entryTtl>` 即可；entryTtl 的单位是 d/h/m/s(默认)
2. 注入 `RedisTemplate<String, Object>` Bean
说明：可直接使用 Spring Cache 相关注解。如：`@Cacheable(cacheNames = "cacheName", key = "#id")`、`@CachePut(cacheNames = "cacheName", key = "#id")`、`@CacheEvict(cacheNames = "cacheName", key = "#id")`

---

## 6. `yudao-spring-boot-starter-biz-tenant`
1. `TenantProperties` 用于定义多租户的配置
2. `TenantFrameworkService` 检验租户 id 是否合法、返回租户 id 集合
3. `TenantContextWebFilter` Web 请求中获取到请求头的租户 ID，并设置到 `TenantContextHolder` 中
4. `TenantIgnoreAspect` AOP 切面，为添加了 `@TenantIgnore` 注解的方法或者类会忽略多租户（通过 `TenantContextHolder` 设置忽略多租户）
5. `TenantDatabaseInterceptor` 关于数据库的多租户拦截器，通过修改 SQL 语句，实现多租户。（以 `TenantLineInnerInterceptor` 形式注入到 MyBatisPlus 的拦截器链中）
6. `TenantVisitContextInterceptor` 通过设置请求头的访问租户号，实现跨租户访问（修改 LoginUser 和 TenantContextHolder 的作用域租户号）
7. `TenantSecurityWebFilter` 做一些租户安全的检查，例如：当前登录用户的租户 ID 是否合法且与 Request 的租户 ID 相同；校验当前租户 ID 是否合法
8. `TenantJobAspect` 为 `@TenantJob` 注解提供支持：为所有租户 id 执行一次 Runnable（通过 `TenantUtils` 实现）
9. `RedisCacheManager` 引入租户缓存，Redis 实现真实的缓存 key = `<key>:<tenant_id>`
10. 为 MQ 提供多租户支持

---

## 5. `yudao-spring-boot-starter-security`

---

## 7. `yudao-spring-boot-starter-biz-data-permission`

---

## 8. `yudao-spring-boot-starter-protection`

---

## 9. `yudao-spring-boot-starter-monitor`

---

## 10. `yudao-spring-boot-starter-job`

---

## 11. `yudao-module-infra`

---

## 12. `yudao-module-system`

---

## 13. `yudao-spring-boot-starter-biz-ip`

---

## 14. `yudao-spring-boot-starter-job`

---

## 15. `yudao-spring-boot-starter-mq`

---

## 16. `yudao-spring-boot-starter-excel`

---

## 17. `yudao-spring-boot-starter-websocket`

---

## 18. `yudao-spring-boot-starter-test`



































