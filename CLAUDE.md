# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目参考

- **workflow.md** — 开发阶段全景和模块依赖关系。开始新模块前先查阅，确认当前所处阶段、前置依赖是否已完成。
- **prd.md** — 完整产品需求文档，包含功能定义、优先级（P0/P1/P2）、数据实体、业务规则。涉及具体功能开发或数据模型设计时查阅。

## 项目约束

### 禁止执行的危险操作

- **不可逆操作须先备份影响范围**：rm -rf、DROP TABLE、TRUNCATE、git reset --hard、git push --force、git checkout -- 覆盖文件
- **禁止无 WHERE 条件的 UPDATE/DELETE**：必须先 SELECT 确认影响行数
- **禁止生产高峰期执行 ALTER TABLE**：锁表会导致业务中断
- **禁止硬编码密码/Token/密钥**：泄露即失控，环境变量或配置中心管理
- **禁止 git commit --no-verify 或 --no-gpg-sign**：绕过安全检查
- **禁止将 .env 等敏感配置 git add**：防止凭据泄露到仓库
- **禁止将 .claude 文件夹上传到仓库**：.claude 为本地 Claude Code 配置目录，含用户偏好和会话数据，不应纳入版本控制
- **禁止密码明文存储**：必须 bcrypt/Argon2 哈希
- **禁止日志输出密码/身份证/银行卡等敏感信息**
- **禁止将调试堆栈直接返回前端**：信息泄露
- **禁止注释或绕过鉴权代码**（如临时去掉 @PreAuthorize）：合入即成漏洞
- **禁止 curl | bash 执行不可信脚本**
- **禁止代码中注释掉鉴权/安全校验逻辑并提交**
- **Schema 变更先在测试环境验证，执行时用 BEGIN + ROLLBACK 确认无副作用后再提交**

## 开发规范

### Skill 使用

- **前端开发**：调用 `frontend-design` skill。管理后台用 Vue 3 + Element Plus，客户端 H5 用 Vue 3 + Vant。
- **后端开发**：不调用特定 skill，直接编写代码。Java 21 + Spring Boot 3.x + MyBatis + ShardingSphere，按分层结构（Controller → Service → Mapper）逐层生成。
- **数据库设计**：不调用特定 skill，直接编写 DDL。
- **部署与 Docker**：不调用特定 skill，直接编写 Dockerfile 和 docker-compose.yml。

### 文档生成时机

- **API 文档**：后端每个微服务的 Controller 层完成后，立即确认 SpringDoc OpenAPI 注解完整（`@Schema`、`@Operation`、`@Parameter`），保证 Swagger UI 可读可用。
- **数据库文档**：表结构稳定后，在 `db/` 目录维护一份 `init.sql` 并添加必要的注释。
- **README**：每个微服务根目录下的 README 在服务首次提交时创建，说明服务职责、启动方式、依赖的中间件。
- **接口变更**：修改已有接口的请求/响应结构时，同步更新对应的 DTO 和注解，确保文档与代码一致。

### 功能拆分与代码生成批量

- **每个大功能必须拆分为若干小功能**（垂直切片），每个小功能是一个可独立测试的完整单元。例如"订单管理"拆分为：创建订单 → 查询订单 → 取消订单 → 退款，逐个开发。
- **每次生成的代码量**以一个垂直切片为单位：一个小功能的完整分层（DTO → Controller → Service → Mapper → SQL  +  对应的单元测试），而非逐文件生成。
- **单次生成的参考量**：1 个小功能的 5-8 个文件（含测试）。一次生成超过 8 个文件时先问是否要再拆分。
- **每个小功能开发完成后，立即编写并执行单元测试**，测试通过后再进入下一个小功能。
- **禁止一次生成整个微服务所有文件**，严格按小功能逐个开发、逐个测试。
- **修改代码时**，一次只改一个小功能的相关文件，不改外围无关代码。

### 测试规范

#### 测试策略

- **测试金字塔**：以 Service 层单元测试为主体（覆盖核心业务逻辑），Controller 层写轻量集成测试，Mapper 层写 SQL 映射测试。
- **每个小功能开发完成后必须先写测试再提交**，不允许积攒多个功能后统一补测试。
- **目标指标**：Service 层测试覆盖率 ≥ 90%，核心业务方法（订单状态流转、秒杀扣库存、支付回调）要求 100% 覆盖所有分支。

#### 测试范围与分工

| 测试类型 | 范围 | 框架 | 特点 |
|---------|------|------|------|
| 单元测试 | Service 层 + 工具类 + 转换器 | JUnit 5 + Mockito + AssertJ | 纯内存运行，不加载 Spring 容器，毫秒级执行 |
| 集成测试 | Controller 层 + Repository 层 | `@SpringBootTest` + `@MybatisTest` | 加载部分容器，测试接口完整调用链 |
| SQL 测试 | Mapper XML 映射 | `@MybatisTest` + H2 内嵌数据库 | 验证 SQL 语法和映射关系 |

#### 测试代码规范

- **测试类位置**：`src/test/java`，包路径与被测类一致
- **测试类命名**：`{被测类名}Test`（如 `OrderServiceTest`）
- **测试方法命名**：`{被测试方法}_should_{预期行为}`（如 `createOrder_should_return_order_when_seats_available`）
- **禁止**：测试方法名使用中文、测试逻辑依赖外部数据库或网络、测试之间互相依赖（每个测试独立可运行）

#### 测试内容要求

每个方法的单元测试必须覆盖：

1. **正常路径**：输入合法，验证返回值正确
2. **异常路径**：非法参数、资源不存在、重复操作等，验证正确抛出 `BusinessException` 并携带正确 `code`
3. **边界条件**：空值、极限值、超长字符串等

示例模式：
```java
// 正常路径
@Test
void createOrder_should_return_order_when_seats_available() {
    // given
    
    // when
    
    // then
}

// 异常路径
@Test
void createOrder_should_throw_when_seat_already_sold() {
    // given
    
    // when & then
    assertThrows(BusinessException.class, () -> orderService.createOrder(req));
}

// 边界条件
@Test
void createOrder_should_throw_when_exceeds_max_tickets() {
    // given
    
    // when & then
    assertThrows(BusinessException.class, () -> orderService.createOrder(req));
}
```

#### 禁止行为

- 禁止测试中连接真实的 Redis / RocketMQ / Kafka（用 Mock 或 Embedded 替代）
- 禁止测试中访问生产或开发数据库（用 H2 内存数据库或 Mock DAO）
- 禁止 `System.out.println` 验证结果（必须用 `assertThat` / `assertEquals` / `verify`）
- 禁止 `Thread.sleep` 等待异步结果（用 `Awaitility` 或 `CountDownLatch`）
- 禁止将多个测试塞入一个 `@Test` 方法（一个方法只测一个行为）
- 禁止提交不可重复执行的测试（如依赖固定的时间戳或自增 ID）
