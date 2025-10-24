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

## 1. yudao-common
在 `yudao-common` 最重要的包即 `pojo`、`exception`、`enums`。\

**pojo**:
1. 定义了 `CommonResult<T>` 用于封装接口返回数据，`CommonResult` 封装了 `code`、`msg`、`data` 字段。
2. 定义了分页参数和分页结果，分页参数 `PageParam`、分页结果 `PageResult`。若需要对分页的结果进行排序，则也提供了 `SortablePageParam`、`SortingField`。

**exception**:
1. 定义了表示异常的 `RuntimeException`：`ServerException` 服务器异常、`ServiceException` 业务逻辑异常。
2. 定义了全局错误码 `GlobalErrorCodeConstants` 枚举，用于定义全局错误的各种类型。
3. 提供 `ServiceExceptionUtil` 工具类，方便创建 `ServiceException`。

**enums**:
1. 通用状态枚举 `CommonStatusEnum`：ENABLE 启用、DISABLE 禁用
2. 终端的枚举 `TerminalEnum`：UNKNOWN、WECHAT_MINI_PROGRAM、WECHAT_WAP、H5、APP
3. 用户类型枚举 `UserTypeEnum`：ADMIN 管理员、MEMBER 会员

---
## 2. yudao-spring-boot-starter-web









