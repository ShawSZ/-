# 电影购票系统 PRD v1.0

> 版本：v1.0  
> 日期：2026-04-27  
> 状态：初稿  

---

## 1. 项目背景与目标

### 1.1 背景
随着电影消费市场的持续增长，用户对便捷、高效的线上购票体验需求日益提高。同时，热门影片的"秒杀抢票"场景对系统的高并发处理能力提出了极高要求。参考**猫眼、大麦、淘票票**等主流平台的核心能力，构建一套功能完善、高可用的电影购票系统。

### 1.2 目标
- 构建**后台管理系统**，支持影院方/运营方对电影、场次、座位、价格、秒杀活动等进行全生命周期管理。
- 构建**客户端购票系统**，提供流畅的选座、下单、支付体验。
- 构建**秒杀抢票引擎**，支撑热门影片首映/节假日等高并发抢票场景。

### 1.3 核心指标
| 指标 | 目标值 |
|------|--------|
| 秒杀峰值 QPS | ≥ 5000 |
| 下单成功率（正常场景） | ≥ 99.9% |
| 秒杀下单响应时间（P99） | ≤ 500ms |
| 系统可用性 | ≥ 99.9% |

---

## 2. 用户角色

### 2.1 客户端用户

| 角色 | 描述 |
|------|------|
| 未注册用户（游客） | 浏览电影、查看影院、搜索，购票前须注册/登录 |
| 普通用户 | 购票、选座、支付、查看订单、发表影评 |
| 会员用户 | 享有折扣价、优先购权限、专属座位区域、生日福利等权益 |

### 2.2 管理端角色与权限矩阵

后台系统采用 **RBAC（基于角色的访问控制）** 模型，权限精确到按钮级。

| 角色 | 可访问模块 | 分配方式 |
|------|-----------|---------|
| **超级管理员** | 全部模块（含权限管理、操作审计、系统配置） | 系统初始化内置 |
| **影院管理员** | 电影管理、影院/影厅管理、排片管理、价格管理、订单管理、本影院数据统计 | 由超级管理员分配 |
| **运营管理员** | 电影管理（仅编辑/上下架）、秒杀活动管理、用户管理、内容审核 | 由超级管理员分配 |
| **财务管理员** | 订单管理（仅查看和退款）、数据统计（仅销售/票房报表） | 由超级管理员分配 |
| **审核管理员** | 电影审核、影评审核、用户举报处理 | 由超级管理员分配 |

---

## 3. 功能需求

### 3.1 后台管理系统

#### 3.1.1 电影管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 电影信息录入 | 新增电影：片名（中/英文）、海报、预告片、导演、演员、类型、时长、上映日期、剧情简介、评分 | P0 |
| 电影信息编辑 | 支持修改已录入的电影信息 | P0 |
| 电影上下架 | 控制电影在前台的可展示状态 | P0 |
| 批量导入 | 通过 Excel/API 批量导入影片信息 | P2 |
| 标签管理 | 自定义标签（如 IMAX、3D、4DX、杜比全景声等） | P1 |

#### 3.1.2 影院与影厅管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 影院信息管理 | 名称、地址、联系电话、交通指引、影院图片 | P0 |
| 影厅管理 | 影厅名称、类型（普通/IMAX/杜比），座位排布图 | P0 |
| 座位模板 | 支持自定义座位图：行/列数、特殊座位（情侣座、无障碍座）、不可售座位 | P0 |
| 座位分组 | 将座位划分为不同区域（如黄金区、普通区）以支持分区定价 | P1 |

#### 3.1.3 排片管理（场次管理）
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 场次创建 | 选择电影 + 影厅 + 时间，自动计算结束时间 | P0 |
| 场次编辑 | 修改场次价格、时间等 | P0 |
| 批量排片 | 支持按模板批量生成一周排片计划 | P1 |
| 排片日历 | 以日历视图展示和管理排片 | P1 |
| 冲突检测 | 同一影厅同一时间段不可重复排片，自动告警 | P0 |

#### 3.1.4 价格管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 基础定价 | 按场次设定标准票价 | P0 |
| 分区定价 | 同一影厅不同区域不同价格 | P1 |
| 时段定价 | 早场/下午场/晚场/午夜场差异化定价 | P1 |
| 特殊场次定价 | IMAX/3D/杜比等加价策略 | P1 |
| 价格日历 | 按日期维度查看/编辑未来 N 天的票价 | P2 |

#### 3.1.5 秒杀活动管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 秒杀活动创建 | 选择场次、设置秒杀价、秒杀库存量、活动时间 | P0 |
| 秒杀时段配置 | 预热期 → 抢购期 → 售罄/结束 | P0 |
| 秒杀限购规则 | 每用户限购张数、每订单限购张数 | P0 |
| 秒杀风控配置 | 同一 IP 限购、设备限购、账号等级要求 | P1 |
| 秒杀活动看板 | 实时展示秒杀进度、库存消耗、并发量 | P1 |

#### 3.1.6 订单管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 订单列表 | 按时间/状态/影院/用户筛选，支持导出 | P0 |
| 订单详情 | 查看订单完整信息：用户、电影、座位、价格、支付状态 | P0 |
| 订单退款 | 支持手动退款/取消订单 | P0 |
| 异常订单处理 | 标记和处理支付超时、重复支付等异常 | P1 |

#### 3.1.7 用户管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 用户列表 | 查看注册用户信息、注册时间、购票记录 | P0 |
| 用户等级 | 设置会员等级及对应权益 | P1 |
| 黑名单 | 对恶意刷票、频繁退票用户进行限制 | P1 |

#### 3.1.8 系统管理（权限与鉴权）

##### 3.1.8.1 管理员账号管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 管理员列表 | 展示所有后台管理员账号：用户名、姓名、角色、所属影院、状态、最后登录时间/IP | P0 |
| 创建管理员 | 超级管理员创建新的后台账号，分配初始密码，首次登录强制修改密码 | P0 |
| 编辑管理员 | 修改管理员信息、角色、状态 | P0 |
| 启用/停用 | 停用后该管理员无法登录后台 | P0 |
| 重置密码 | 超级管理员可重置指定管理员密码，重置后下次登录强制修改 | P1 |

##### 3.1.8.2 管理员登录与鉴权
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 后台登录 | 独立登录页：用户名 + 密码 + 图形验证码 | P0 |
| 登录失败锁定 | 连续 5 次登录失败，账号临时锁定 30 分钟 | P0 |
| 双因素认证（2FA） | 可选开启 TOTP/短信验证码二次认证 | P2 |
| Session 管理 | 后台采用 JWT + Redis Session，空闲 30 分钟自动过期 | P0 |
| 多端互斥 | 同一管理员账号不可同时在多台设备登录，新登录挤掉旧登录 | P1 |
| 退出登录 | 清除服务端 Session 和客户端 Token | P0 |

##### 3.1.8.3 角色与权限管理（RBAC）
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 角色列表 | 展示所有角色：角色名、描述、关联用户数 | P0 |
| 创建角色 | 自定义角色名称、描述 | P0 |
| 权限树配置 | 以树形结构展示所有权限点（模块 → 菜单 → 操作按钮），支持勾选分配 | P0 |
| 权限粒度 | 权限至少控制到以下级别：<br>- 模块级：电影管理、排片管理、订单管理等<br>- 操作级：新增/编辑/删除/查看/审核/导出 | P0 |
| 角色模板 | 预设"影院管理员""运营管理员""财务管理员""审核管理员"角色模板，开箱即用 | P1 |
| 数据权限 | 影院管理员仅能查看和管理所属影院的数据（行级数据权限） | P1 |

##### 3.1.8.4 菜单与路由管理
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 动态菜单 | 登录后根据角色权限动态生成左侧导航菜单，无权限菜单不可见 | P0 |
| 菜单管理 | 超级管理员可自定义菜单名称、图标、排序、路由地址、是否可见 | P2 |
| 按钮级权限 | 页面内的新增/编辑/删除等按钮根据权限动态展示/隐藏 | P0 |

##### 3.1.8.5 操作审计日志
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 操作日志列表 | 记录所有管理员的关键操作：操作人、IP、操作时间、操作模块、操作类型、操作详情、操作结果（成功/失败） | P0 |
| 操作详情 | 记录操作前后的数据变更对比（如"将票价从 45 元改为 55 元"） | P1 |
| 日志查询 | 按时间范围、操作人、操作模块、操作类型筛选 | P0 |
| 日志导出 | 支持将审计日志导出为 Excel | P1 |
| 日志不可删除 | 审计日志仅可查看不可修改或删除，保证可追溯性 | P0 |

##### 3.1.8.6 数据安全与合规
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 密码策略 | 密码至少 8 位，必须包含大小写字母+数字+特殊字符，每 90 天强制更换 | P0 |
| 敏感数据脱敏 | 后台展示用户手机号时中间 4 位脱敏（138****1234），查看完整信息需二次授权 | P0 |
| 登录 IP 白名单 | 可配置允许登录后台的 IP 段，非白名单 IP 拒绝访问 | P1 |

#### 3.1.9 数据统计
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 销售概览 | 日/周/月/自定义区间票房总额、出票量 | P0 |
| 电影票房排行 | 按电影维度统计票房 | P0 |
| 影院销售报表 | 按影院维度统计销售数据 | P1 |
| 上座率统计 | 每场次/每日上座率分析 | P1 |
| 秒杀数据报表 | 秒杀活动参与人数、转化率、库存消耗速度 | P1 |
| 用户行为分析 | 浏览→购票转化漏斗、复购率 | P2 |

---

### 3.2 客户端购票系统

#### 3.2.1 用户认证与账号管理

| 功能 | 描述 | 优先级 |
|------|------|--------|
| **注册** | 支持手机号验证码注册（国际手机号支持）、微信/支付宝一键授权注册 | P0 |
| **密码登录** | 手机号 + 密码登录，支持图形验证码防暴力破解（连续 3 次失败弹出验证码，5 次失败临时锁定 15 分钟） | P0 |
| **验证码登录** | 手机号 + 短信验证码登录，验证码有效期 60 秒，60 秒内不可重发 | P0 |
| **第三方登录** | 微信授权登录、支付宝授权登录，首次绑定自动创建账号 | P0 |
| **退出登录** | 清除当前设备 token，支持"退出所有设备" | P0 |
| **忘记密码** | 手机号验证身份 → 设置新密码 → 重新登录，新密码不可与前 3 次重复 | P0 |
| **修改密码** | 原密码验证 → 设置新密码，需短信二次确认 | P1 |
| **Token 管理** | JWT 双 Token 机制（Access Token 2h 过期 + Refresh Token 7d 过期），Refresh Token 支持滚动续期 | P0 |
| **多设备登录** | 同一账号最多同时在 3 台设备登录，超出时最早登录的设备被踢下线 | P1 |
| **账号注销** | 用户可自助注销账号，进入 7 天冷静期后可撤销，到期后自动删除数据 | P2 |
| **个人中心** | 头像、昵称、会员等级、观影记录、常用观影人管理 | P0 |
| **观影人管理** | 添加/删除常用观影人（姓名+身份证号，用于购票），最多添加 5 人 | P1 |

#### 3.2.2 首页与发现
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 首页推荐 | 正在热映 + 即将上映，Banner 轮播 | P0 |
| 电影搜索 | 按片名、演员、导演、类型搜索 | P0 |
| 电影筛选 | 按类型、地区、年代、IMAX/3D 等维度筛选 | P0 |
| 榜单 | 票房榜、评分榜、期待榜 | P1 |
| 附近影院 | 基于 LBS 展示附近影院及对应排片 | P1 |

#### 3.2.3 电影详情
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 基本信息 | 海报、评分、导演、演员、类型、时长、上映日期 | P0 |
| 预告片 | 在线播放预告片/花絮 | P1 |
| 剧情简介 | 完整剧情介绍 | P0 |
| 影评 | 用户评价列表，支持点赞/举报 | P1 |
| 想看/看过 | 标记想看/看过，统计"想看"人数 | P1 |

#### 3.2.4 选座购票
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 场次列表 | 选择日期 → 选择影院 → 选择场次 | P0 |
| 座位选择 | 可视化座位图，显示已售/锁定/可选座位 | P0 |
| 座位交互 | 点选/取消座位、显示座位价格 | P0 |
| 座位锁定 | 选座后临时锁定（5分钟倒计时），超时释放 | P0 |
| 确认订单 | 确认电影、场次、座位、数量、总价 | P0 |
| 加购服务 | 爆米花/可乐等卖品加购（可选） | P2 |

#### 3.2.5 支付
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 支付方式 | 微信支付、支付宝、银行卡 | P0 |
| 支付倒计时 | 下单后 15 分钟内未支付自动取消 | P0 |
| 支付结果 | 支付成功/失败页面，失败引导重试 | P0 |
| 支付记录 | 查看历史支付明细 | P1 |

#### 3.2.6 订单与售后
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 订单列表 | 全部/待支付/已支付/已取消/已退款 | P0 |
| 订单详情 | 二维码取票凭证、电影信息、座位信息 | P0 |
| 退票/改签 | 支持开场前规定时间内退票/改签（按规则扣手续费） | P1 |

#### 3.2.7 秒杀抢票
| 功能 | 描述 | 优先级 |
|------|------|--------|
| 秒杀入口 | 首页秒杀专区、影片详情页秒杀提示 | P0 |
| 秒杀倒计时 | 活动开始前显示倒计时，开始时自动触发 | P0 |
| 实时库存展示 | 显示剩余票数，营造紧迫感 | P0 |
| 秒杀结果 | 抢票成功 → 跳转支付；失败 → 提示已售罄 | P0 |
| 候补提醒 | 秒杀售罄后用户可登记候补，有余票时推送通知 | P2 |

#### 3.2.8 用户行为事件追踪（Kafka 驱动）
> 此功能专为发挥 Kafka 的消息持久化与高吞吐特性而设计，将用户在前端的所有关键操作以事件流形式上报至 Kafka，供实时分析和离线挖掘使用。

| 功能 | 描述 | 优先级 |
|------|------|--------|
| 埋点事件采集 | 前端自动采集以下事件并上报至 Kafka：页面浏览(PV/UV)、电影详情查看、搜索关键词、选座点击、下单提交、支付结果、秒杀点击与结果 | P1 |
| 事件上报 SDK | 前端提供统一埋点 SDK，批量上报（每 5 秒或每 30 条打包发送一次），避免高频请求阻塞业务接口 | P1 |
| 实时热力图 | 基于 Kafka Streams 实时聚合热门电影点击量、区域搜索热度，30 秒刷新一次看板 | P2 |
| 用户行为分析 | 通过 Kafka 长期留存的事件日志，分析浏览→购票转化漏斗、流失节点、复购周期 | P2 |
| 个性化推荐数据源 | 将用户标签(偏好类型、常去影院、消费能力)产出到 Redis，供首页推荐和榜单排序使用 | P2 |

---

## 4. 秒杀抢票专项需求

### 4.1 业务规则
| 规则 | 描述 |
|------|------|
| 限购 | 每用户限购 2 张，同一手机号/设备 ID 视为同一用户 |
| 预热 | 活动开始前 15 分钟进入预热期，可查看活动不可下单 |
| 超时释放 | 下单后 10 分钟未支付，订单取消，库存回滚 |
| 风控 | 禁止脚本刷票，同一 IP 每秒请求超过阈值自动限流 |

### 4.2 技术要点
| 模块 | 要求 |
|------|------|
| 静态化 | 秒杀页面提前静态化，静态资源部署 CDN |
| 限流 | 接入层 Nginx 限流 + 应用层分布式限流（令牌桶/Sentinel） |
| 防超卖 | Redis 原子扣减库存 + 数据库乐观锁双重校验 |
| 队列削峰 | 用户请求先写入 RocketMQ，利用 RocketMQ 的事务消息保障扣库存→下单的最终一致性，Worker 异步消费处理下单 |
| 缓存 | 商品信息、库存信息预热至 Redis，降低 DB 压力 |
| 降级 | 秒杀期间非核心功能（影评、榜单等）降级或熔断 |

### 4.3 秒杀流程
```
用户进入秒杀页
    ↓
展示倒计时/库存（Redis 读取）
    ↓
倒计时归零 → 点击抢票
    ↓
### 接入层限流校验 ###  ← Nginx + Sentinel
    ↓
### 风控规则校验 ###     ← IP/设备/用户维度
    ↓
### RocketMQ 事务消息 ###
    ① Redis 原子扣减库存（DECR）
    ② 扣减成功 → 发送半消息（Half Message）到 RocketMQ
    ③ 执行本地事务（创建订单写入 DB）
    ④ 本地事务成功 → Commit，消费者可见
    ⑤ 本地事务失败 → Rollback，消息丢弃，库存回滚（INCR）
    ↓
### 返回用户排队中 ###   ← 即时响应，后续异步处理
    ↓
### Worker 消费下单 ###  ← RocketMQ 消费者处理订单
    ↓
### 支付倒计时 ###       ← RocketMQ 延时消息，15分钟后自动检查支付状态，超时取消+库存回滚
```

---

## 5. 非功能需求

### 5.1 性能需求
| 指标 | 要求 |
|------|------|
| 页面首屏加载 | ≤ 1.5s |
| 选座页面渲染 | ≤ 1s |
| 下单接口响应（非秒杀） | ≤ 800ms |
| 秒杀接口响应（P99） | ≤ 500ms |
| 并发用户支持 | 正常场景 1000 并发，秒杀场景 5000 并发 |

### 5.2 安全需求

#### 5.2.1 认证安全
- 客户密码加密存储（bcrypt/Argon2），不可逆加密
- 管理端密码强度策略：8 位以上+大小写+数字+特殊字符，90 天强制更换
- 管理端登录失败锁定：连续 5 次失败临时锁定 30 分钟
- JWT Token 签名密钥定期轮换，泄露可立即吊销
- Refresh Token 滚动续期，单次使用即失效
- 管理端支持登录 IP 白名单配置

#### 5.2.2 接口安全
- 支付接口防重放攻击（nonce + timestamp 机制）
- 秒杀接口防刷（图形验证码/滑块验证、IP 限流、设备指纹）
- API 签名校验，防止请求被篡改
- 接口频率限制：按用户/按 IP 两级限流

#### 5.2.3 数据安全
- 敏感数据脱敏：后台默认脱敏展示（手机号中间 4 位、身份证号前后各 2 位）
- 数据库敏感字段加密存储
- 审计日志不可删除不可篡改
- SQL 注入/XSS/CSRF 防护
- HTTPS 全站加密传输

### 5.3 可用性需求
- 核心服务集群部署，单点故障自动切换
- 秒杀期间支持服务降级（非核心功能不可用不影响购票）
- 数据定时备份，RPO ≤ 15 分钟
- 系统容灾：同城双活或两地三中心（视成本而定）

### 5.4 兼容性需求
| 端 | 要求 |
|----|------|
| Web 端 | 兼容 Chrome/Firefox/Safari/Edge 最新两个大版本 |
| 移动端 H5 | 适配 iOS Safari、Android 各主流浏览器 |
| 小程序 | 微信小程序基础库 3.0+ |

---

## 6. 产品架构

### 6.1 业务架构示意
```
┌─────────────────────────────────────────────────────────────────┐
│                          客户端入口                               │
│          Web App  │  移动端 H5  │  小程序  │  API 开放          │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                        API 网关层                                │
│             认证鉴权 / 限流 / 路由 / 日志 / 跨域                   │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌──────────┬──────────┬──────┴──────┬──────────┬──────────┬───────┐
│  认证服务  │  用户服务 │  电影服务   │  订单服务 │  秒杀服务 │  事件管道 │
│  管理员登录│  注册/登录│  CRUD/搜索  │  下单/支付 │  抢票引擎 │  埋点采集 │
│  权限管理  │  会员等级 │  排片管理   │  退款/售后│  库存管理 │  Kafka 流 │
│  RBAC    │  风控模块 │  座位管理   │  订单查询 │  队列削峰 │  实时聚合 │
│  审计日志  │         │  影院管理   │          │           │  推荐引擎 │
└──────────┴──────────┴─────────────┴──────────┴──────────┴───────┘
                             │
┌──────────────────────────────▼──────────────────────────────────┐
│                          基础设施层                               │
│  MySQL(分库分表) │ Redis Cluster │ RocketMQ │ Kafka │ ES │ OSS │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 模块划分建议
建议采用微服务架构，核心服务如下：

| 服务 | 职责 |
|------|------|
| gateway | API 网关 / 统一入口 / 认证鉴权 / 限流 |
| auth-service | 后台管理员登录/登出 / RBAC 权限管理 / 菜单权限 / 操作审计日志 |
| user-service | 客户端用户注册/登录 / Token 管理 / 会员等级 / 风控 |
| movie-service | 电影 / 影院 / 影厅 / 排片 / 座位管理 |
| order-service | 订单创建 / 支付 / 退款 / 改签 |
| seckill-service | 秒杀活动 / 库存扣减 / RocketMQ 事务消息 / 排队削峰 |
| order-service | 订单创建 / RocketMQ 异步消费 / 支付 / 退款 / 改签 |
| payment-service | 支付通道对接 / 对账 |
| statistics-service | 数据统计 / 报表 |
| tracking-service | 用户行为事件接收 / Kafka 生产 / Kafka Streams 实时聚合 / 推荐标签产出 |

---

## 7. 数据实体与建表定义

> 以下为每个实体的详细列定义，作为 DDL 生成的唯一依据。字段类型、长度、约束均在此定义，DDL 直接以此为准产出。

### 7.1 管理端实体

#### admin_user（管理员）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| username | VARCHAR | 50 | NOT NULL, UK | 登录用户名 |
| real_name | VARCHAR | 50 | NOT NULL | 真实姓名 |
| password_hash | VARCHAR | 255 | NOT NULL | bcrypt 哈希 |
| role_id | BIGINT | — | NOT NULL, INDEX | 关联 role.id |
| cinema_id | BIGINT | — | NULL, INDEX | 关联 cinema.id，为空则全局 |
| status | TINYINT | — | NOT NULL, DEFAULT 1 | 1=启用, 0=停用 |
| last_login_ip | VARCHAR | 45 | NULL | 支持 IPv6 |
| last_login_time | DATETIME | — | NULL | 最后登录时间 |
| pwd_expire_time | DATETIME | — | NOT NULL | 密码过期时间 |
| created_at | DATETIME | — | NOT NULL, DEFAULT CURRENT_TIMESTAMP | 创建时间 |
| updated_at | DATETIME | — | NOT NULL, DEFAULT CURRENT_TIMESTAMP ON UPDATE | 更新时间 |

索引：
- UK: `uk_username` (username)
- IDX: `idx_role_id` (role_id), `idx_cinema_id` (cinema_id)

---

#### role（角色）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| role_name | VARCHAR | 50 | NOT NULL | 角色名称 |
| role_code | VARCHAR | 50 | NOT NULL, UK | 角色编码，如 `ADMIN` |
| description | VARCHAR | 255 | NULL | 角色描述 |
| status | TINYINT | — | NOT NULL, DEFAULT 1 | 1=启用, 0=停用 |
| is_builtin | TINYINT | — | NOT NULL, DEFAULT 0 | 1=内置(不可删除), 0=自定义 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_role_code` (role_code)

---

#### permission（权限点）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| perm_name | VARCHAR | 50 | NOT NULL | 权限名称 |
| perm_code | VARCHAR | 100 | NOT NULL, UK | 权限编码，如 `movie:create` |
| perm_type | VARCHAR | 20 | NOT NULL | module/menu/button |
| parent_id | BIGINT | — | NULL, INDEX | 父权限 ID |
| sort | INT | — | NOT NULL, DEFAULT 0 | 排序号 |
| icon | VARCHAR | 100 | NULL | 菜单图标 |
| route_path | VARCHAR | 200 | NULL | 前端路由路径 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_perm_code` (perm_code); IDX: `idx_parent_id` (parent_id)

---

#### role_permission（角色-权限关联）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| role_id | BIGINT | — | NOT NULL | 关联 role.id |
| permission_id | BIGINT | — | NOT NULL | 关联 permission.id |

索引：UK: `uk_role_perm` (role_id, permission_id); IDX: `idx_permission_id` (permission_id)

---

#### operate_log（审计日志）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| admin_id | BIGINT | — | NOT NULL, INDEX | 操作人 ID |
| username | VARCHAR | 50 | NOT NULL | 操作人用户名（冗余） |
| ip | VARCHAR | 45 | NOT NULL | 操作 IP |
| module | VARCHAR | 50 | NOT NULL, INDEX | 操作模块 |
| operation_type | VARCHAR | 30 | NOT NULL | CREATE/UPDATE/DELETE/EXPORT |
| target_id | VARCHAR | 64 | NULL | 操作对象 ID |
| detail | VARCHAR | 500 | NULL | 操作详情 |
| old_value | TEXT | — | NULL | 修改前值（JSON） |
| new_value | TEXT | — | NULL | 修改后值（JSON） |
| result | TINYINT | — | NOT NULL | 1=成功, 0=失败 |
| create_time | DATETIME | — | NOT NULL, INDEX | 日志时间 |

索引：IDX: `idx_admin_id` (admin_id), `idx_module` (module), `idx_create_time` (create_time)

---

#### login_log（登录日志）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| admin_id | BIGINT | — | NULL, INDEX | 关联 admin_user.id |
| username | VARCHAR | 50 | NOT NULL | 登录用户名 |
| ip | VARCHAR | 45 | NOT NULL | 登录 IP |
| login_time | DATETIME | — | NOT NULL | 登录时间 |
| login_type | VARCHAR | 20 | NOT NULL | PASSWORD/SMS/2FA |
| status | TINYINT | — | NOT NULL | 1=成功, 0=失败 |
| fail_reason | VARCHAR | 200 | NULL | 失败原因 |
| created_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_admin_id` (admin_id), `idx_login_time` (login_time)

---

### 7.2 公共业务实体

#### movie（电影）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| title | VARCHAR | 200 | NOT NULL | 片名 |
| title_en | VARCHAR | 200 | NULL | 英文片名 |
| poster | VARCHAR | 500 | NULL | 海报 URL |
| trailer | VARCHAR | 500 | NULL | 预告片 URL |
| duration | INT | — | NOT NULL | 时长（分钟） |
| genre | VARCHAR | 100 | NOT NULL | 类型，逗号分隔 |
| director | VARCHAR | 200 | NOT NULL | 导演 |
| cast | TEXT | — | NOT NULL | 主演，逗号分隔 |
| release_date | DATE | — | NOT NULL, INDEX | 上映日期 |
| synopsis | TEXT | — | NULL | 剧情简介 |
| rating | DECIMAL(2,1) | — | NULL, DEFAULT 0.0 | 评分 |
| status | TINYINT | — | NOT NULL, DEFAULT 0 | 0=待上映, 1=上映中, 2=已下架 |
| tags | VARCHAR | 200 | NULL | IMAX/3D/4DX 等，逗号分隔 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_release_date` (release_date), `idx_status` (status); FULLTEXT: `ft_title` (title)

---

#### cinema（影院）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| name | VARCHAR | 200 | NOT NULL | 影院名称 |
| address | VARCHAR | 500 | NOT NULL | 地址 |
| phone | VARCHAR | 20 | NOT NULL | 联系电话 |
| longitude | DECIMAL(10,7) | — | NULL | 经度 |
| latitude | DECIMAL(10,7) | — | NULL | 纬度 |
| images | TEXT | — | NULL | 影院图片 URL 列表（JSON） |
| status | TINYINT | — | NOT NULL, DEFAULT 1 | 1=营业, 0=停业 |
| admin_id | BIGINT | — | NULL, INDEX | 关联 admin_user.id（影院管理员） |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_admin_id` (admin_id)

---

#### hall（影厅）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| cinema_id | BIGINT | — | NOT NULL, INDEX | 关联 cinema.id |
| name | VARCHAR | 50 | NOT NULL | 影厅名称，如 1 号厅 |
| type | VARCHAR | 20 | NOT NULL, DEFAULT 'STANDARD' | STANDARD/IMAX/杜比 |
| seat_rows | INT | — | NOT NULL | 座位行数 |
| seat_cols | INT | — | NOT NULL | 座位列数 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_cinema_id` (cinema_id); UK: `uk_cinema_name` (cinema_id, name)

---

#### seat（座位）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| hall_id | BIGINT | — | NOT NULL, INDEX | 关联 hall.id |
| row_label | VARCHAR | 5 | NOT NULL | 行标签，如 A/B/C |
| col_index | INT | — | NOT NULL | 列号 |
| area_code | VARCHAR | 20 | NULL | 区域编码（用于分区定价） |
| type | TINYINT | — | NOT NULL, DEFAULT 0 | 0=普通, 1=情侣座, 2=无障碍座 |
| status | TINYINT | — | NOT NULL, DEFAULT 0 | 0=可售, 1=不可售 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_hall_row_col` (hall_id, row_label, col_index)

---

#### show（场次）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| movie_id | BIGINT | — | NOT NULL, INDEX | 关联 movie.id |
| hall_id | BIGINT | — | NOT NULL, INDEX | 关联 hall.id |
| start_time | DATETIME | — | NOT NULL | 开始时间 |
| end_time | DATETIME | — | NOT NULL | 结束时间 |
| base_price | DECIMAL(8,2) | — | NOT NULL | 基础票价 |
| status | TINYINT | — | NOT NULL, DEFAULT 0 | 0=待放映, 1=售票中, 2=已售罄, 3=已取消 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_movie_id` (movie_id), `idx_hall_id` (hall_id), `idx_start_time` (start_time);
UK: `uk_hall_time` (hall_id, start_time) — 冲突检测

---

### 7.3 订单与支付实体

#### order（订单）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| order_no | VARCHAR | 32 | NOT NULL, UK | 订单号 |
| user_id | BIGINT | — | NOT NULL, INDEX | 关联 user.id |
| show_id | BIGINT | — | NOT NULL, INDEX | 关联 show.id |
| total_price | DECIMAL(10,2) | — | NOT NULL | 总价 |
| status | TINYINT | — | NOT NULL, DEFAULT 0 | 0=待支付, 1=已支付, 2=已完成, 3=已取消, 4=已退款 |
| payment_method | VARCHAR | 20 | NULL | WECHAT/ALIPAY |
| paid_at | DATETIME | — | NULL | 支付时间 |
| created_at | DATETIME | — | NOT NULL, INDEX | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_order_no` (order_no); IDX: `idx_user_id` (user_id), `idx_show_id` (show_id), `idx_created_at` (created_at), `idx_status` (status)

---

#### order_item（订单明细）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| order_id | BIGINT | — | NOT NULL, INDEX | 关联 order.id |
| seat_id | BIGINT | — | NOT NULL | 关联 seat.id |
| seat_label | VARCHAR | 10 | NOT NULL | 冗余座位标签，如 A-05 |
| price | DECIMAL(8,2) | — | NOT NULL | 该座位单价 |
| created_at | DATETIME | — | NOT NULL | — |

索引：IDX: `idx_order_id` (order_id); UK: `uk_order_seat` (order_id, seat_id)

---

### 7.4 秒杀实体

#### seckill_activity（秒杀活动）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| show_id | BIGINT | — | NOT NULL, UK | 关联 show.id，一场次一个活动 |
| seckill_price | DECIMAL(8,2) | — | NOT NULL | 秒杀价 |
| total_stock | INT | — | NOT NULL | 秒杀总库存 |
| available_stock | INT | — | NOT NULL | 当前可用库存 |
| start_time | DATETIME | — | NOT NULL, INDEX | 活动开始时间 |
| end_time | DATETIME | — | NOT NULL, INDEX | 活动结束时间 |
| limit_per_user | INT | — | NOT NULL, DEFAULT 2 | 每用户限购 |
| status | TINYINT | — | NOT NULL, DEFAULT 0 | 0=未开始, 1=预热中, 2=进行中, 3=已售罄, 4=已结束, 5=已关闭 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_show_id` (show_id); IDX: `idx_start_time` (start_time), `idx_end_time` (end_time), `idx_status` (status)

---

#### transaction_log（RocketMQ 事务日志）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| tx_id | VARCHAR | 64 | NOT NULL, UK | RocketMQ 事务 ID |
| show_id | BIGINT | — | NOT NULL | 关联 show.id |
| user_id | BIGINT | — | NOT NULL | 关联 user.id |
| seat_ids | VARCHAR | 255 | NOT NULL | 座位 ID 列表，逗号分隔 |
| status | VARCHAR | 20 | NOT NULL, DEFAULT 'pending' | pending/committed/rollback |
| create_time | DATETIME | — | NOT NULL | — |
| update_time | DATETIME | — | NULL | — |

索引：UK: `uk_tx_id` (tx_id)

---

### 7.5 客户端实体

#### user（客户端用户）

| 字段 | 类型 | 长度 | 约束 | 说明 |
|------|------|------|------|------|
| id | BIGINT | — | PK, AUTO_INCREMENT | 主键 |
| phone | VARCHAR | 20 | NOT NULL, UK | 手机号 |
| password_hash | VARCHAR | 255 | NOT NULL | bcrypt 哈希 |
| nickname | VARCHAR | 50 | NULL | 昵称 |
| avatar | VARCHAR | 500 | NULL | 头像 URL |
| level | TINYINT | — | NOT NULL, DEFAULT 0 | 会员等级 |
| status | TINYINT | — | NOT NULL, DEFAULT 1 | 1=正常, 0=禁用 |
| last_login_ip | VARCHAR | 45 | NULL | 最后登录 IP |
| last_login_time | DATETIME | — | NULL | 最后登录时间 |
| register_time | DATETIME | — | NOT NULL | 注册时间 |
| created_at | DATETIME | — | NOT NULL | — |
| updated_at | DATETIME | — | NOT NULL | — |

索引：UK: `uk_phone` (phone); IDX: `idx_register_time` (register_time)

---

### 7.6 Kafka 事件实体（不入 MySQL，仅作定义）

#### user_event（用户行为事件 → Kafka Topic: user_behavior_raw）

| 字段 | 类型 | 说明 |
|------|------|------|
| event_id | VARCHAR(64) | 事件唯一 ID（UUID） |
| user_id | BIGINT | 用户 ID（未登录为 0） |
| event_type | VARCHAR(30) | 事件类型：page_view / search / seat_click / order_submit / payment_result / seckill_click |
| event_time | BIGINT | 事件时间戳（毫秒） |
| page | VARCHAR(100) | 页面路径 |
| extra_data | TEXT(JSON) | 附加数据 |
| device_id | VARCHAR(64) | 设备 ID |
| ip | VARCHAR(45) | 客户端 IP |
| ua | VARCHAR(500) | User-Agent |

> user_event 数据直接写入 Kafka，不经过 MySQL。此定义仅用于字段对齐。

---

## 8. 里程碑规划

| 阶段 | 内容 | 建议周期 |
|------|------|---------|
| **M1 - 基础搭建** | 项目初始化，数据库设计，**后台 RBAC 权限系统（管理员/角色/权限/登录鉴权）**，客户端用户注册/登录/Token 鉴权 | 2 周 |
| **M2 - 排片与选座** | 排片管理、可视化选座、订单流程打通 | 2 周 |
| **M3 - 支付与订单** | 支付对接、订单管理、退款流程 | 1.5 周 |
| **M4 - 秒杀抢票** | 秒杀活动管理、Redis 库存、**RocketMQ 事务消息**、抢票引擎 | 2 周 |
| **M5 - 事件管道与统计** | **用户行为埋点 SDK + Kafka 事件采集 + Kafka Streams 聚合**、操作审计日志、数据报表 | 2 周 |
| **M6 - 性能与安全** | 压测优化、安全加固（**2FA、IP 白名单、密码策略**）、限流降级、CDN 部署 | 1 周 |
| **M7 - 上线与监控** | 灰度发布、全量上线、监控告警接入 | 1 周 |

> 总计约 **11 周**

---

## 9. 风险与应对

| 风险 | 影响 | 应对方案 |
|------|------|---------|
| 秒杀超卖 | 资损 + 用户体验差 | Redis 原子扣减 + DB 乐观锁双重校验 |
| 支付超时/重复支付 | 订单状态不一致 | 支付回调幂等处理 + 定时对账任务 |
| 排片冲突 | 同一影厅同一时间排多场 | 排片冲突检测算法，前端实时告警 |
| 高并发打垮 DB | 系统不可用 | 缓存预热、限流、MQ 削峰、读写分离 |
| 恶意刷票 | 正常用户抢不到票 | 风控系统 + 限流 + 设备指纹 + 验证码 |
| 后台账号泄露 | 数据泄露、非法操作 | RBAC 最小权限原则 + 登录 IP 白名单 + 操作审计日志 + 2FA |
| 双 MQ 运维复杂度 | RocketMQ + Kafka 两套消息系统需独立维护，故障排查面扩大 | Docker Compose 统一编排；日志采集和监控（Prometheus + SkyWalking）覆盖两套 MQ；明确职责边界防止场景误用 |

---

## 10. 附录

### 10.1 参考平台
- **猫眼电影**：选座交互、排片日历、电影详情页
- **大麦**：秒杀抢票流程、限购规则、候补机制
- **淘票票**：会员体系、选座交互

### 10.2 客户端认证流程
```
### 注册流程 ###
用户输入手机号 → 获取短信验证码 → 设置密码 → 填写昵称 → 注册成功 → 自动登录

### 登录流程 ###
用户输入手机号+密码 → 校验图形验证码（触发时） → 验证身份 → 签发 Access Token + Refresh Token → 返回用户信息
    ↓ 连续 3 次失败 → 弹出图形验证码
    ↓ 连续 5 次失败 → 临时锁定 15 分钟

### Token 鉴权流程 ###
请求头携带 Access Token → 网关验证 Token → 有效则放行
    ↓ Access Token 过期 → 客户端用 Refresh Token 换取新的 Access Token → 重试原请求
    ↓ Refresh Token 也过期 → 跳转登录页

### 管理端登录流程 ###
管理员访问后台 → 跳转独立登录页 → 输入用户名+密码+验证码 → 校验身份
    ↓ 首次登录/密码已过期 → 强制修改密码
    ↓ 连续 5 次失败 → 账号锁定 30 分钟
    ↓ IP 不在白名单 → 拒绝访问
验证通过 → 签发 JWT Token → 加载角色权限 → 动态生成菜单 → 进入后台首页
```

### 10.3 权限校验流程
```
用户请求某个 API → 网关校验 Token 有效性 → 获取用户角色和权限
    → 判断是否有该 API 的访问权限 → 有则放行，无则返回 403
    → 按钮级别权限由前端根据权限列表动态控制显隐
```

### 10.4 技术选型

#### 10.4.1 后端框架

| 选型 | 方案 |
|------|------|
| **所选** | **Java + Spring Boot 3.x** |
| 备选 | Go (Gin/GoFrame) / Python (FastAPI) / Node.js (NestJS) |

**选择理由**：
- Spring Boot 生态成熟，Spring Security + Spring Data Redis + Spring Cloud Gateway 全家桶无缝集成，减少选型拼接成本
- Java 拥有最丰富的秒杀/高并发案例库和中文资料，排查问题时社区支持度最高
- 独立的个人开发者用 Java 更易招聘后续协作人员（若项目发展需要）
- Spring Boot 3.x 基于 GraalVM 支持 AOT 编译，启动速度和内存已大幅改善

**不选理由**：
- **Go**：虽然并发性能更优、编译产物更小，但 ORM 生态（GORM）远不如 JPA/MyBatis 成熟，RBAC 权限体系需完全自建，开发效率会下降
- **FastAPI**：Python 的 GIL 不适合秒杀这种 CPU 密集型+高并发场景，强类型校验在复杂业务聚合层维护成本高
- **NestJS**：Node.js 单线程模型在 5000 QPS 秒杀场景需要额外 Worker Thread 管理，且编译型语言在工具链（IDE 重构、编译期检查）方面差距明显

#### 10.4.2 核心语言与 JDK

| 选型 | 方案 |
|------|------|
| **所选** | **Java 21 (LTS)** |
| 备选 | Java 17 / Java 11 |

**选择理由**：
- Java 21 引入 **虚拟线程（Virtual Threads）**，在秒杀场景中可以大幅简化异步编程（不再需要 CompletableFuture 回调堆叠）
- 虚拟线程让每个请求占用极低开销，同等内存下支撑更高并发
- 记录类型（Record）、模式匹配（Pattern Matching）提升代码可读性
- LTS 版本，生产环境长期支持

**不选理由**：
- **Java 17**：缺少虚拟线程，秒杀场景下需要用 WebFlux 或手动线程池管理，代码复杂度明显增加
- **Java 11**：已接近 EOL，且缺少密封类、Record 等现代语言特性

#### 10.4.3 数据库

| 选型 | 方案 |
|------|------|
| **所选** | **MySQL 8.0 + MyBatis + ShardingSphere（分库分表）** |
| 备选 | PostgreSQL 16 / TiDB |

**选择理由**：
- MySQL 在互联网行业票务系统的覆盖率最高（猫眼、淘票票早期均基于 MySQL），踩坑方案可直接复用
- MyBatis 作为轻量级 ORM，SQL 完全手写可控，在分库分表场景下可精确优化每一句 SQL 的执行计划，避免 ORM 自动生成 SQL 导致的性能偏差
- 配合 ShardingSphere-JDBC 实现分库分表，将订单、用户、场次等核心表按业务维度水平拆分，提升写入吞吐和单表查询性能，同时通过主从读写分离保障高可用
- InnoDB 的行锁机制在座位锁定场景下表现可靠

**不选理由**：
- **PostgreSQL**：虽然 JSON 查询、窗口函数更强，但对标秒杀场景优势不明显；PG 在相同硬件下读性能不如 MySQL，且运维工具生态（如 binlog 解析、闪回）不如 MySQL 丰富
- **TiDB**：引入 ShardingSphere 分库分表后，已能满足项目初期的水平扩展需求；TiDB 分布式架构仍需至少 3 节点部署，运维成本仍偏高

#### 10.4.4 缓存

| 选型 | 方案 |
|------|------|
| **所选** | **Redis 7.x Cluster（集群模式）** |
| 备选 | Memcached / Hazelcast |

**选择理由**：
- Redis 的原子操作（`DECR`/`INCR`、Lua 脚本）是秒杀扣库存的核心保障，Memcached 无法提供
- Redis 数据结构丰富：String 存计数器、Hash 存座位状态、Sorted Set 做秒杀排队、Stream 做消息队列
- Redisson 框架提供成熟的分布式锁（`RLock`），在订单防重入场景直接可用
- Redis 7.x 新增的 `Function` 特性允许将秒杀脚本托管在服务端，减少网络开销
- Redis Cluster 模式提供数据分片 + 主从自动故障转移，单节点宕机不影响整体缓存服务，满足系统 99.9% 可用性目标

**不选理由**：
- **Memcached**：不支持持久化、不支持复杂数据结构、不支持 Lua 脚本，无法支撑秒杀扣库存的原子性要求，仅适合简单 KV 缓存，功能上不是一个量级
- **Hazelcast**：作为 Java 内嵌式数据网格，与应用耦合太紧，无法独立扩展缓存集群，且社区版缺少持久化和高可用特性

#### 10.4.5 消息队列

本系统采用**双 MQ 架构**，RocketMQ 负责业务消息，Kafka 负责事件管道，各取所长。

---

| 选型 | 方案 |
|------|------|
| **业务消息（所选）** | **RocketMQ 5.x** |
| **事件管道（所选）** | **Apache Kafka 3.x** |
| 备选 | RabbitMQ |

**RocketMQ — 业务消息（秒杀削峰 / 订单 / 事务消息）**

**选择理由**：
- 秒杀场景要求削峰填谷 + **事务消息**（扣库存成功才发下单消息，失败则回滚），RocketMQ 对事务消息的支持最成熟，提供 Half Message + 回查机制保证最终一致性
- 支持**延时消息**，可用于实现"下单后 15 分钟未支付自动取消"的场景，无需额外开发定时任务
- 阿里巴巴大规模电商秒杀的长期验证，国内社区和中文文档非常丰富
- 支持**消息轨迹**，方便在秒杀高峰排查具体订单的消息链路

**不选理由**（业务消息场景）：
- **RabbitMQ**：基于 Erlang，运维排查难度高；消息堆积时性能急剧下降，不适合秒杀这种瞬时海量消息场景；事务消息支持较弱

---

**Apache Kafka — 事件管道（用户行为追踪 / 3.2.8）**

**选择理由**：
- Kafka 的消息持久化到磁盘，天然适合用户行为事件流的长期留存和回溯消费
- Kafka Streams 可直接在消息管道内做实时聚合（如 PV/UV、热门电影排行），无需额外计算服务
- 分区内严格有序，配合时间戳保证用户行为序列的正确还原
- `spring-kafka` 提供 `@KafkaListener`、`KafkaTemplate` 等开箱集成

**不选理由**（事件管道场景）：
- **RocketMQ**：设计理念偏向业务消息的可靠投递，消息堆积和回溯消费能力不如 Kafka，不适合大量小体量事件日志的长期存储和流式处理

> **架构边界**：RocketMQ 和 Kafka 职责清晰、互不替代。秒杀请求走 RocketMQ 事务消息保障一致性；用户行为事件走 Kafka 保障吞吐和持久化。两套 MQ 独立部署，不共享 Topic。

#### 10.4.6 搜索

| 选型 | 方案 |
|------|------|
| **所选** | **Elasticsearch 8.x** |
| 备选 | MeiliSearch / Solr |

**选择理由**：
- ES 在电影搜索场景下分词插件丰富（IK 中文分词、拼音搜索），支持"演员名搜电影"、"模糊片名匹配"等典型需求
- ES 的聚合分析能力可直接支撑"票房排行、评分排行"等统计需求，减少统计服务的开发量
- 与 Spring Boot 集成度高（`spring-boot-starter-data-elasticsearch`），Repository 模式即可完成 CRUD

**不选理由**：
- **MeiliSearch**：虽然轻量、开箱即用，但分词能力和聚合分析远不如 ES，不支持嵌套聚合（票房分组+时间维度下钻），且社区生态较小
- **Solr**：项目活跃度已明显落后于 ES，且 Solr 的实时搜索（NRT）性能不如 ES，对中文分词支持需要额外插件配置

#### 10.4.7 对象存储

| 选型 | 方案 |
|------|------|
| **所选** | **MinIO（自建）+ 阿里云 OSS（生产）** |
| 备选 | 腾讯云 COS / AWS S3 / CloudFlare R2 |

**选择理由**：
- 开发/测试阶段使用 MinIO 自建存储，零成本，兼容 S3 API，代码迁移无缝
- 上线后切至阿里云 OSS，CDN 加速海报和预告片的分发，国内节点覆盖最广
- 阿里云 OSS 提供图片处理（缩放、裁剪、水印）功能，直接在前端控制海报展示样式，无需后端处理

**不选理由**：
- **腾讯云 COS**：功能和 OSS 基本对齐，但 API 的生态集成工具（如 SDK 文档丰富度、社区插件）略逊于阿里云
- **CloudFlare R2**：免费额度大但国内访问延迟高（无国内节点），不适合面向国内用户的电影票系统
- **AWS S3**：开发体验最好，但国内用户访问受限，且账单复杂不适合独立开发者做成本控制

#### 10.4.8 应用框架组件

| 组件 | 选型 | 理由 |
|------|------|------|
| 权限框架 | **Spring Security + Sa-Token** | Sa-Token 相比 Shiro 更轻量（零配置启动），相比 Spring Security OAuth2 学习曲线更低；提供开箱即用的 RBAC、Token 自动续期、Redis 集成 |
| 接口文档 | **SpringDoc OpenAPI (Swagger)** | SpringFox 已停止维护，SpringDoc 是社区推荐的替代方案，支持 OpenAPI 3.0，且与 Spring Boot 3.x 兼容性最好 |
| 参数校验 | **Jakarta Validation + 分组校验** | 内置`@NotBlank`、`@Pattern`等注解满足大多数场景；相比手动 `if/else` 校验，代码缩减 60% 以上 |
| 映射工具 | **MapStruct** | 编译期生成 Bean 映射代码，零反射开销；相比 BeanUtils（运行时反射，频繁拷贝时性能差）在秒杀场景有明显优势 |
| 分布式锁 | **Redisson** | 提供可重入锁、公平锁、读写锁等，相比手写 `SETNX+Lua` 方案更可靠，自动处理锁续期和看门狗机制 |

#### 10.4.9 前端

| 选型 | 方案 |
|------|------|
| **后台管理** | **Vue 3 + Element Plus + Vite** |
| **客户端 H5** | **Vue 3 + Vant + Pinia** |
| 备选 | React 18 + Ant Design / Next.js |

**选择理由**：
- Vue 3 的 Composition API + Pinia 状态管理在复杂交互（选座组件、秒杀倒计时）场景下逻辑复用度高
- Element Plus 和 Vant 均为饿了么团队维护，组件风格统一，后台和移动端之间的切换学习成本为零
- Vite 的开发热更新速度远快于 Webpack 打包方案

**不选理由**：
- **React + Ant Design**：同样成熟，但 React 的 JSX 对独立开发者而言模板可读性不如 Vue 单文件组件直观；且你需要同时掌握后台 (AntD) 和移动端 (Ant Design Mobile) 两套框架，累加学习成本更高
- **Next.js**：面向 SSR/SEO 场景，而电影购票系统是强交互 SPA，不需要 SSR，Next.js 的 Server Component 等服务端特性在此场景没有用武之地

#### 10.4.10 部署与运维

| 组件 | 选型 | 理由 |
|------|------|------|
| 容器化 | **Docker + Docker Compose** | 独立开发者前期不需要 K8s 的编排能力，Docker Compose 的 `docker-compose.yml` 单文件即可定义 MySQL + Redis + RocketMQ + Kafka + ES + 微服务集群，一键启动 |
| CDN | **阿里云 CDN** | 海报、预告片等静态资源加速，与 OSS 同厂商延迟最低，配置简单 |
| 监控 | **Prometheus + Grafana** | 秒杀期间实时监控 JVM 内存、接口 QPS、响应时间、Redis 命中率；Grafana 提供现成的 Spring Boot 监控大屏模板 |
| 日志 | **ELK（Elasticsearch + Logstash + Kibana）** | 与搜索共用 ES 集群；秒杀故障时可快速通过 TraceId 串联全链路日志 |
| 链路追踪 | **SkyWalking** | 秒杀请求跨 4 个服务（网关 → 秒杀服务 → Redis → MQ → 订单服务），SkyWalking 可视化追踪调用链和耗时瓶颈 |

#### 10.4.11 技术选型总结

```
┌─────────────────────────────────────────────────────────┐
│                    技术栈一览                              │
├─────────────────────────────────────────────────────────┤
│ 后端框架    │ Java 21 + Spring Boot 3.x + MyBatis + ShardingSphere    │
│ API 网关    │ Spring Cloud Gateway + Nginx                              │
│ 数据库      │ MySQL 8.0（分库分表）+ Redis 7.x Cluster（缓存/秒杀）     │
│ 消息队列    │ RocketMQ 5.x (业务消息/事务消息/延时消息)       │
│ 事件管道    │ Apache Kafka 3.x (用户行为事件流)              │
│ 搜索        │ Elasticsearch 8.x (电影搜索/统计)            │
│ 存储        │ MinIO (开发) / 阿里云 OSS (生产) + CDN       │
│ 权限        │ Sa-Token + RBAC                             │
│ 后台前端    │ Vue 3 + Element Plus + Vite                 │
│ 客户端 H5   │ Vue 3 + Vant + Pinia                        │
│ 监控        │ Prometheus + Grafana + SkyWalking           │
│ 部署        │ Docker Compose                              │
│ ─────────── │ 面向独立开发者, 平衡开发效率与生产性能        │
└─────────────────────────────────────────────────────────┘
```

### 10.5 Kafka 事件管道架构
```
┌─────────────────────────────────────────────────────────────────┐
│                        埋点数据生产侧                              │
│   Web App ──┐                                                    │
│   H5 ───────┤──→ 埋点 SDK（批量5s/30条）──→ HTTP 上报 Gateway      │
│   小程序 ───┘                                                    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────┴───────────────────────────────────┐
│                    tracking-service                               │
│   接收 → 校验 → 补充（IP/UA/设备指纹）→ 序列化 → Kafka Producer   │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────┴───────────────────────────────────┐
│                    Kafka Topic 分层                               │
│                                                                │
│  Topic: user_behavior_raw   (保留7天，所有原始事件)               │
│    ├─ partition: page_view                                     │
│    ├─ partition: search                                        │
│    ├─ partition: click                                          │
│    ├─ partition: order_event                                    │
│    └─ partition: seckill_event                                  │
│                                                                │
│  Topic: user_behavior_agg    (保留30天，聚合后数据)               │
│    ← Kafka Streams 消费 raw → 窗口聚合产出 agg                   │
│                                                                │
│  Topic: user_profile_tag    (保留7天，用户标签)                   │
│    ← Kafka Streams 消费 agg → 计算用户偏好 → 写入 Redis           │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────┬───────────────────────────────────┐
│         实时消费              │           离线消费                  │
│  Kafka Streams → 热力图看板   │  事件日志 → 每日 ETL → 数据仓库    │
│  Kafka Streams → 热门排行     │  → 转化漏斗分析 / 复购率 /         │
│  Redis ← 用户标签 → 首页推荐  │  → 用户画像 / 冷启动推荐           │
└─────────────────────────────┴───────────────────────────────────┘
```

### 10.6 术语表
| 术语 | 说明 |
|------|------|
| P0/P1/P2 | 优先级定义：P0 必须有，P1 建议有，P2 可以迭代 |
| 秒杀 | 限定时间内以超低价格限量抢购 |
| 削峰 | 通过队列缓冲系统瞬时高并发流量 |
| 风控 | 风险控制，防止恶意行为和欺诈交易 |
| RPO | Recovery Point Objective，数据恢复点目标 |

---

> **文档维护者**：产品团队  
> **更新记录**：
> - v1.0（2026-04-27）：初稿创建，涵盖核心功能与非功能需求
> - v1.1（2026-04-27）：补充客户端用户认证体系（注册/登录/Token/忘记密码/多设备），新增后台管理系统权限模块（RBAC/管理员管理/动态菜单/操作审计日志/安全合规）
> - v1.2（2026-04-27）：移除优惠营销相关功能（优惠券管理、营销活动、promotion-service、Coupon实体等），相应调整编号与架构图
> - v1.3（2026-04-27）：新增技术选型章节（10.4），覆盖后端框架/数据库/缓存/消息队列/搜索/存储/前端/部署等全栈选型及替代方案对比
> - v1.4（2026-04-27）：调整为双 MQ 架构——RocketMQ 负责业务消息（秒杀/订单/事务消息），Kafka 专用于用户行为事件追踪（3.2.8）；新增用户行为事件追踪功能及事件管道架构（10.5）；新增 TransactionLog 实体；补充双 MQ 运维风险
> - v1.5（2026-04-28）：根据技术团队评审意见，ORM 框架替换为 MyBatis（移除 MyBatis-Plus）；新增 ShardingSphere-JDBC 实现 MySQL 分库分表；升级 Redis 为集群模式（Redis Cluster）保障高可用
