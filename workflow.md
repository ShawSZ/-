# 电影购票系统 — 开发流程与先后顺序

> 参考 PRD v1.0，仅定义开发顺序与各模块的依赖关系

---

## 1. 核心原则

1. **先后端，后前端** — 后端定义 API 契约，前端基于接口联调，避免返工
2. **基础设施先行** — 项目骨架、数据库、中间件一步到位，后续不做基建停歇
3. **先网关，后登录** — 网关是所有流量的统一入口，就位后续服务才能注册路由
4. **先管理端，后客户端** — 管理端录入数据，客户端才能消费展示
5. **先核心链路，后辅助功能** — P0 优先：排片→选座→下单→支付 跑通后再做秒杀/统计/事件管道

---

## 2. 开发阶段全景

```
阶段0: 基础设施搭建
  │
  ▼
阶段1: 后端核心服务（6 个模块）
  │
  ▼
阶段2: 管理后台前端
  │
  ▼
阶段3: 客户端前端（H5 + 小程序）
  │
  ▼
阶段4: 秒杀抢票引擎
  │
  ▼
阶段5: 数据管道与统计
  │
  ▼
阶段6: 性能压测与安全加固
  │
  ▼
阶段7: 上线与监控
```

---

## 3. 阶段 0 — 基础设施搭建

**内容**（不做业务代码）：

| 步骤 | 内容 |
|------|------|
| 0.1 | Git 多模块 Maven 工程（`pom.xml` 聚合） |
| 0.2 | 数据库全部 DDL 设计与产出 |
| 0.3 | Docker Compose 编排（MySQL + Redis + RocketMQ + Kafka + ES + MinIO） |
| 0.4 | 各微服务空项目 + 统一版本管理 |
| 0.5 | `common` 公共模块（统一返回体、异常处理、工具类、日志切面） |
| 0.6 | MyBatis 逆向工程生成 Mapper + POJO |
| 0.7 | 多环境配置（dev/prod） |

### 数据库建表流程（0.2 详述）

DDL 严格依据 prd.md 第 7 章的数据实体定义逐表产出，不允许跳过规范直接凭经验建表。

**第 1 步：确定实体与所属微服务** — 按微服务边界拆分为多组 DDL 文件：
  - `db/auth/`：admin_user, role, permission, role_permission, operate_log, login_log
  - `db/user/`：user
  - `db/movie/`：movie, cinema, hall, seat, show
  - `db/order/`：order, order_item
  - `db/seckill/`：seckill_activity, transaction_log

**第 2 步：逐表产出 DDL** — 每张表按照 prd.md 中该实体的列定义表，依次确定：
  1. 字段名与类型、长度
  2. NOT NULL / NULL
  3. 默认值（DEFAULT）
  4. 主键（PK, AUTO_INCREMENT）
  5. 唯一约束（UK）
  6. 索引（INDEX / FULLTEXT）
  7. 外键关系（外键字段 + REFERENCES，实际建表时评估是否需要物理外键）

**第 3 步：DDL 示例模板**
```sql
CREATE TABLE `order` (
    `id`            BIGINT          NOT NULL AUTO_INCREMENT  COMMENT '主键',
    `order_no`      VARCHAR(32)     NOT NULL                 COMMENT '订单号',
    `user_id`       BIGINT          NOT NULL                 COMMENT '用户 ID',
    `show_id`       BIGINT          NOT NULL                 COMMENT '场次 ID',
    `total_price`   DECIMAL(10,2)   NOT NULL                 COMMENT '总价',
    `status`        TINYINT         NOT NULL DEFAULT 0       COMMENT '0=待支付,1=已支付,2=已完成,3=已取消,4=已退款',
    `payment_method` VARCHAR(20)    NULL                     COMMENT 'WECHAT/ALIPAY',
    `paid_at`       DATETIME        NULL                     COMMENT '支付时间',
    `created_at`    DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    `updated_at`    DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk_order_no` (`order_no`),
    KEY `idx_user_id` (`user_id`),
    KEY `idx_show_id` (`show_id`),
    KEY `idx_created_at` (`created_at`),
    KEY `idx_status` (`status`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';
```

**第 4 步：合并与验证** — 所有 DDL 合并写入 `db/init.sql`。在 Docker Compose 的 MySQL 容器首次启动时自动执行该文件。通过 `docker-compose logs mysql` 确认无建表报错。

**第 5 步：物理外键评估原则**
  - 同微服务内的强关联表（如 order ↔ order_item）可使用物理外键
  - 跨微服务的关联（如 order.show_id → show.id）不使用物理外键，仅保留逻辑外键（索引字段），由应用层保证一致性
  - 秒杀等高频写入场景全部避免物理外键，避免插入校验开销和锁竞争

---

## 4. 阶段 1 — 后端核心服务（★ 按此顺序开发）

### 4.1 gateway（第 1 个）

**依赖**：阶段 0

**开发内容（P0）**：
- Spring Cloud Gateway 基础路由配置（断言 + 过滤器）
- 全局过滤器：统一日志、TraceId 注入、CORS
- 全局异常处理 + 统一响应格式
- Nginx 反向代理前置配置
- 健康检查端点

**原因**：所有流量的统一入口，后续加服务只需加一条 route 配置。

---

### 4.2 auth-service（第 2 个）

**依赖**：gateway 就绪

**开发内容（P0）**：
- 管理员登录（用户名 + 密码 + 图形验证码）
- JWT 签发 + Redis Session（30 分钟空闲过期）
- 登录失败锁定（5 次失败锁定 30 分钟）
- RBAC 权限框架：角色 CRUD、权限树、角色-权限关联
- 管理员 CRUD（创建/编辑/启用/停用）
- 动态菜单生成 API
- 按钮级权限校验过滤器
- 操作审计日志
- 退出登录

**原因**：管理端每个操作都需要鉴权，login 是所有管理功能的入口。

---

### 4.3 user-service（第 3 个）

**依赖**：gateway 就绪

**开发内容（P0）**：
- 客户端用户注册（手机号 + 验证码）
- 密码登录 + 验证码登录
- JWT 双 Token（Access 2h + Refresh 7d 滚动续期）
- 第三方登录（微信/支付宝，首次绑定自动创建账号）
- 忘记密码 + 修改密码
- 个人中心
- 登录失败策略（3 次→验证码，5 次→锁定 15 分钟）
- 多设备登录（最多 3 台在线）

---

### 4.4 movie-service（第 4 个）

**依赖**：gateway + auth-service

**开发内容（P0）**：
- 电影 CRUD + 上下架
- 影院 CRUD
- 影厅 CRUD + 座位模板（行/列/特殊座位）
- 排片 CRUD + 冲突检测（同影厅同时间不可重复排片）
- 座位状态管理（锁定/释放）
- 基础定价管理
- ES 电影搜索集成（模糊匹配、中文分词）

---

### 4.5 order-service（第 5 个）

**依赖**：user-service + movie-service

**开发内容（P0）**：
- 订单创建（选座→锁定座位 5 分钟→生成订单）
- 订单列表/详情（按用户/时间/状态筛选）
- 订单取消（超时自动取消 + 座位释放）
- 订单退款（管理端手动退款）
- 订单状态机（待支付→已支付→已完成→已取消→已退款）

---

### 4.6 payment-service（第 6 个）

**依赖**：order-service

**开发内容（P0）**：
- 微信支付对接（JSAPI / Native）
- 支付宝对接
- 支付回调处理（幂等性设计）
- 支付结果查询
- 支付倒计时（15 分钟未支付自动取消）

---

### 后端模块依赖总图

```
阶段0 (基础设施)
   │
   ▼
gateway
   │
   ├───────────────────┐
   ▼                   ▼
auth-service     user-service
   │                   │
   ▼                   │
movie-service ────────┤
   │                   │
   ▼                   ▼
order-service
   │
   ▼
payment-service
```

### 后端开发顺序总结

1 → 2 → 3 → 4 → 5 → 6

```
gateway → auth-service → user-service → movie-service → order-service → payment-service
```

完成第 6 步后核心购票链路（排片→选座→下单→支付）跑通。

---

## 5. 阶段 2 — 管理后台前端

**前提**：阶段 1 完成，后端 API 可用。

| 顺序 | 内容 | 对应后端 |
|------|------|---------|
| 2.1 | Vue 3 + Element Plus + Vite 骨架、Axios 封装、路由配置 | — |
| 2.2 | 登录页 + 动态路由 + 权限指令 | auth-service |
| 2.3 | 电影管理（列表/新增/编辑/上下架） | movie-service |
| 2.4 | 影院管理 + 影厅管理 + 座位模板 | movie-service |
| 2.5 | 排片管理（场次创建/编辑/冲突提示） | movie-service |
| 2.6 | 订单管理（列表/详情/退款） | order-service |
| 2.7 | 系统管理（管理员管理、角色管理、权限配置） | auth-service |
| 2.8 | 数据统计看板 | statistics-service |

---

## 6. 阶段 3 — 客户端前端（H5 / 小程序）

**前提**：阶段 1 + 2 完成，管理端已录入数据，核心 API 稳定。

| 顺序 | 内容 | 对应后端 |
|------|------|---------|
| 3.1 | Vue 3 + Vant + Pinia 骨架 | — |
| 3.2 | 注册/登录页 | user-service |
| 3.3 | 首页（Banner + 热映 + 即将上映 + 搜索入口） | movie-service |
| 3.4 | 电影详情页（海报、评分、简介） | movie-service |
| 3.5 | 影院列表 + 场次列表 | movie-service |
| 3.6 | 可视化选座页（座位图、5 分钟锁定） | movie-service + order-service |
| 3.7 | 确认订单 + 支付 | order-service + payment-service |
| 3.8 | 订单列表/详情（含取票凭证） | order-service |
| 3.9 | 个人中心 + 观影人管理 | user-service |
| 3.10 | 搜索页（模糊搜索 + 筛选） | movie-service + ES |

---

## 7. 阶段 4 — 秒杀抢票引擎

**前提**：阶段 1-3 完成，正常购票链路已跑通。

### 7.1 后端（seckill-service）

| 顺序 | 内容 |
|------|------|
| 4.1 | 秒杀活动管理 CRUD（场次、秒杀价、库存、限购规则） |
| 4.2 | Redis 库存预热 + 原子扣减（DECR + Lua 脚本） |
| 4.3 | RocketMQ 事务消息（半消息→本地事务→Commit/Rollback） |
| 4.4 | Nginx + Sentinel 分布式限流 |
| 4.5 | 风控校验（IP 限流、设备指纹、用户限购） |
| 4.6 | RocketMQ 延时消息（15 分钟未支付取消 + 库存回滚） |
| 4.7 | 秒杀实时看板 |

### 7.2 管理端前端

| 顺序 | 内容 |
|------|------|
| 4.8 | 秒杀活动管理页面 |
| 4.9 | 秒杀实时看板 |

### 7.3 客户端前端

| 顺序 | 内容 |
|------|------|
| 4.10 | 秒杀入口（首页专区、详情页提示） |
| 4.11 | 秒杀倒计时 + 实时库存展示 |
| 4.12 | 抢票按钮 + 排队等待动画 |
| 4.13 | 秒杀结果页（成功→支付 / 售罄） |

---

## 8. 阶段 5 — 数据管道与统计

**前提**：核心链路 + 秒杀链路稳定。

| 顺序 | 内容 |
|------|------|
| 5.1 | tracking-service：事件接收、校验、Kafka Producer |
| 5.2 | Kafka Topic 创建（user_behavior_raw / agg / user_profile_tag） |
| 5.3 | Kafka Streams 实时聚合（PV/UV、热门排行、转化漏斗） |
| 5.4 | 客户端埋点 SDK（批量 5s/30 条，自动采集关键事件） |
| 5.5 | statistics-service：销售报表、票房排行、上座率 |
| 5.6 | 审计日志补充（操作详情、数据变更对比） |
| 5.7 | 管理端报表页面 |

---

## 9. 阶段 6 — 性能压测与安全加固

| 顺序 | 内容 |
|------|------|
| 6.1 | JMeter 压测：正常链路 + 秒杀链路 |
| 6.2 | 慢 SQL 优化 + 索引优化 + Redis 缓存优化 |
| 6.3 | 安全加固：2FA、IP 白名单、密码策略 |
| 6.4 | Sentinel 限流降级规则调优 |
| 6.5 | 前端静态资源 CDN 部署 |
| 6.6 | 第二轮压测验证（QPS ≥ 5000, P99 ≤ 500ms） |

---

## 10. 阶段 7 — 上线与监控

| 顺序 | 内容 |
|------|------|
| 7.1 | Prometheus + Grafana 大盘（JVM、QPS、RT、Redis 命中率） |
| 7.2 | SkyWalking 链路追踪接入 |
| 7.3 | ELK 日志采集（TraceId 全链路串联） |
| 7.4 | 灰度发布（10% → 50% → 100%） |
| 7.5 | 全量上线 + 持续监控 |

---

## 11. Java 开发代码规范

### 11.1 命名规范

| 元素 | 规范 | 示例 |
|------|------|------|
| 类名 | UpperCamelCase | `OrderService`, `SeckillController` |
| 方法名 | lowerCamelCase | `createOrder()`, `getUserById()` |
| 常量 | UPPER_SNAKE_CASE | `MAX_LOGIN_ATTEMPTS`, `TOKEN_EXPIRE_SECONDS` |
| 包名 | 全小写 | `com.movieticket.order.service` |
| 布尔变量 | 否定词禁止用 `is` 开头 | `failed` 而非 `isFailed`，`deleted` 而非 `isDeleted` |
| 异常类 | 以 `Exception` 结尾 | `SeckillStockException`, `PaymentTimeoutException` |
| 枚举类 | 以 `Enum` 结尾或 Security 层级用 `Type` | `OrderStatusEnum` |
| 抽象类 | 以 `Abstract` 开头 | `AbstractBaseController` |
| 实现类 | 以 `Impl` 结尾 | `OrderServiceImpl` |
| 测试类 | 以 `Test` 结尾 | `OrderServiceTest` |
| Controller 映射 | RESTful 复数名词 | `POST /orders`, `GET /orders/{id}` |

### 11.2 分层约定

| 层 | 类后缀 | 职责 | 禁止行为 |
|----|--------|------|---------|
| Controller | `Controller` | 接收请求、参数校验、调用 Service | 不能包含业务逻辑，不能直接操作 DAO |
| Service | `Service` / `ServiceImpl` | 业务逻辑编排、事务管理 | 不能直接处理 HTTP 请求/响应 |
| Repository / Mapper | `Mapper` / `Repository` | 数据库操作、SQL 映射 | 不能包含业务逻辑 |
| DTO | `Req` / `Resp` / `DTO` | 接口入参/出参 | 不能包含业务方法 |
| Entity / POJO | `PO` / `Entity` | 数据库表映射 | 不能包含业务方法 |
| Convert | `Converter` / `Mapper` | PO ↔ DTO ↔ VO 转换 | 用 MapStruct 编译期生成，禁止运行时反射拷贝 |

**分层调用链**：`Controller → Service → Mapper`，严禁跨层调用（Controller 直接调 Mapper）。

### 11.3 异常处理

- 业务异常继承自定义 `BusinessException`（含 `code` + `message`），由全局 `GlobalExceptionHandler` 统一捕获
- 禁止在 Controller 内 try-catch 后返回 `Result.error()`
- 禁止在循环内抛异常作为流程控制手段
- 校验异常（参数校验失败）由 `@Validated` + `@RestControllerAdvice` 统一处理

### 11.4 日志规范

- 使用 Lombok `@Slf4j`，禁止直接使用 `System.out.println`
- 日志级别使用规则：`ERROR`（需人工介入的异常）> `WARN`（不该发生的但可自动恢复）> `INFO`（关键流程节点，如订单创建/支付回调）> `DEBUG`（开发调试，上线后关闭）
- 日志必须包含 `traceId`（MDC 自动注入），串联全链路
- 禁止打印用户密码、身份证号、支付卡号等敏感信息
- 若需打印请求/响应体，必须脱敏处理

### 11.5 事务与 SQL

- `@Transactional` 仅标注在 Service 方法上，不标注在 Controller
- `@Transactional` 内避免 RPC 调用、MQ 发送等长耗时操作（会占用数据库连接）
- MQ 发送应放在事务之后（`TransactionSynchronizationManager.registerSynchronization`）
- SQL 必须走 MyBatis Mapper 的 parameterType 占位符（`#{}`），禁止拼接字符串防注入
- 禁止在循环内逐条执行 SQL（应使用批量操作或 foreach）

### 11.6 并发与线程安全

- 秒杀场景用 Redisson `RLock`（可重入锁 + 看门狗自动续期），禁止手写 `SETNX` + `Lua`
- 分布式锁的 key 应包含业务前缀 + 资源 ID，如 `seckill:stock:showId:{showId}`
- Controller 层禁止手动创建线程或使用 `new Thread(() -> ...)`
- 使用 Java 21 虚拟线程（`Executors.newVirtualThreadPerTaskExecutor()`），禁止 `new FixedThreadPool` 除非有明确隔离需求
- 多线程传递上下文用 `@Async` + 自定义 `TaskDecorator` 传递 MDC TraceId

### 11.7 接口设计

- 统一返回值格式：`Result<T>` 包含 `code`、`message`、`data`、`traceId`
- 分页接口统一使用 `PageReq`（pageNum + pageSize）→ `PageResp<T>`（list + total + pages）
- 新增/修改操作返回受影响记录 ID，列表接口返回分页，详情接口返回完整 DTO
- API 路径统一前缀：管理端 `/admin/v1/...`，客户端 `/api/v1/...`

### 11.8 依赖注入规范

- 使用构造器注入（`private final` + `@RequiredArgsConstructor`），禁止 `@Autowired` 字段注入
- 一个构造器的参数不超过 5 个，超出的说明该类职责过多，需拆分
- 禁止循环依赖，Service 之间调用通过接口而非实现类注入

### 11.9 枚举规范

- 枚举须包含 `code`（int）和 `desc`（String）字段，数据库存储 code
- 枚举序列化统一用 `@JsonValue`（输出 desc）和 `@JsonCreator`（反序列化 code）
- 所有枚举实现统一接口 `BaseEnum`（`getCode()` + `getDesc()`）

### 11.10 TODO 与 FIXME

```java
// TODO(yourname): 后续需要支持批量取消  —— 功能未完成
// FIXME(yourname): 极端情况下库存可能扣为负数  —— 已知缺陷
```

- 提交代码前必须清理自己名下的 TODO/FIXME，或在任务管理系统创建对应任务后保留
- 不允许有未处理且无任务关联的 `// TODO` 进入主分支

### 11.11 代码格式

- 缩进：4 空格，禁止 Tab
- 左大括号不换行
- 单行最长 120 字符，超出必须换行
- import 禁止使用通配符（`*`），IDE 设置：`java.*` 和 `javax.*` 用星号除外
- 方法体不超过 80 行，超过说明需拆分
- 类文件不超过 500 行（不含 POJO 和 Mapper）

---