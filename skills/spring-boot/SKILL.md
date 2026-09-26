---
name: spring-boot
description: Develop, review, and refactor Spring Boot backends, with concrete conventions for the youlai-boot project. Use for REST APIs, MyBatis-Plus persistence, model naming, validation, transactions, Spring Security, JWT/Redis tokens, or new backend modules. Outside youlai-boot, preserve the host project's established architecture instead of imposing youlai-specific packages or response types.
metadata:
  short-description: Spring Boot 与 youlai-boot 后端开发规范
---

# Spring Boot 后端开发

交付符合现有项目架构、接口契约和质量要求的 Spring Boot 代码。先识别当前仓库的真实约定，再决定是沿用、渐进迁移还是明确重构；不要把示例或偏好当成用户未要求的全局改造。

## 适用边界

- 在 `youlai-boot` 中使用本文件的项目约定。
- 在其他 Spring Boot 项目中，只采用通用原则；目录、命名、响应包装和权限表达式以目标仓库为准。
- 用户指令优先。修改前检查 `AGENTS.md`、构建文件、相关模块及未提交改动，避免覆盖用户工作。
- 普通功能开发不自动扩大为安全审计、全仓重命名、框架升级或多模块拆分。

## 项目基线

| 层 | 选型 |
|----|------|
| JDK | Java 17+ |
| 基础框架 | Spring Boot 4.x |
| ORM | MyBatis-Plus |
| 数据库 | MySQL 8.x |
| 缓存 | Redis 7.x |
| 认证 | Spring Security + JWT / Redis Token |
| API 文档 | Springdoc OpenAPI + Knife4j |
| 对象映射 | MapStruct，复杂联表可由 Mapper 直接投影 VO |

以目标仓库的 `pom.xml` 和配置类为最终事实；升级依赖或改变兼容基线前先验证官方兼容性。

## 架构边界

目标结构采用“业务域分包、域内分层”的单体架构：

```text
com.youlai.boot
├── common/       # 纯共享类型：基础对象、结果、常量、通用异常
├── framework/    # Web、Security、MyBatis、缓存、切面等基础设施
├── config/       # 应用装配；不承载业务规则
├── support/      # file、mail、sms、sse 等外部能力
├── auth/         # 认证业务
├── system/       # 用户、角色、菜单、部门、字典等
├── form/         # 动态表单
├── workflow/     # 工作流
└── codegen/      # 代码生成
```

迁移期间可暂存 `module/form`、`module/workflow`，但新增代码不要继续制造两套顶级目录语言。

遵守以下边界：

- `common` 不依赖 `framework`、`config` 或任何业务域。
- `framework` 不依赖具体业务域；通过 Port 或回调接口获取业务能力。
- `config` 可以作为组合根装配 framework、auth、system 等 Bean。
- 跨业务域调用使用意图明确的 Facade、Port 或 Command，不传递另一个域的 Entity。
- 业务 Service 接口不继承 MyBatis-Plus `IService<Entity>`；实现类可内部复用 `ServiceImpl`。
- 避免双向包依赖。必要时优先引入项目自有命令对象或领域事件，而不是移动 Entity 到 `common`。
- 单体项目先用包边界和 ArchUnit 固化依赖方向；没有独立部署需求时不主动拆微服务。

配置类与属性类的归属判定：

- `@Configuration` 装配类看 Bean 消费方：仅被单个业务域或 support 组件消费的 Bean，装配类随域放置（如 `auth/config/CaptchaConfig`、`support/mail/MailConfig`）；跨域或框架级 Bean 才留在全局 `config/`。检验法：删掉某业务域后 Bean 无人引用，装配类就应随该域移动。
- `@ConfigurationProperties` 属性类看包根形态：根上已有兄弟类的包（`support/mail`、`support/sms`、`framework/web/ratelimit`）贴根平铺；根下只有子目录的集合根（`auth`、`codegen`、`framework/security`、`support/file`）建 `property/` 子包。
- 属性类只挂 `@ConfigurationProperties`，不叠加 `@Configuration`；注册统一交给 `@ConfigurationPropertiesScan`，避免组件扫描与属性扫描双重注册。

## 模型职责与改进方向

| 类型 | 建议定义 | 当前项目的改进重点 |
|------|----------|--------------------|
| `Entity` | 仅持久化层使用 | 基本合理，但 `SysUser`/`SysLog` 与 `Role`/`Dept`/`Menu` 的 `Sys` 前缀不一致 |
| `Form` | 写请求输入 | 创建/更新可能共用、Body 内 `id` 与 Path ID 重复，部分入口校验不完整 |
| `Query` | 查询条件；分页类使用 `XxxPageQuery` | `UserQuery` 实际是分页查询，`DeptQuery` 又是不分页查询，语义不统一 |
| `VO` | 对外响应或只读投影 | 整体较规范；复杂联表由 Mapper 直接投影 VO 可以保留 |
| `DTO` | 模块内部或模块间传输 | 已收敛：推送载荷按 `DictChangeDTO` 进 `dto/`，事件语义由主题常量承载 |

新代码按以下规则收敛，历史代码除非用户要求，不做无收益的批量重命名：

- Entity 使用领域名，如 `User`、`Role`；表前缀由 `@TableName("sys_user")` 表达。保留现有 `SysUser`、`SysLog` 作为迁移兼容项。
- 写模型使用 `XxxForm`。创建与更新字段或校验不同时拆为 `XxxCreateForm`、`XxxUpdateForm`。
- Path 中已有资源 ID 时，以 Path 为唯一事实源，Form 不重复携带 ID。
- 分页查询使用 `XxxPageQuery`，非分页过滤使用 `XxxQuery`。时间范围优先使用 `createdFrom`、`createdTo` 等明确的时间类型。
- 不把权限范围、当前用户或租户等执行上下文塞进外部 Query；通过独立上下文参数或安全组件传递。
- VO 只用于输出，不复用为写模型。Mapper 可为复杂联表直接返回 VO，但不接收 Controller Form。
- DTO 必须具有明确的内部传输语义。SSE/消息推送载荷是跨层传输载体，按 `XxxDTO` 命名进 `dto/`，事件语义由主题常量（如 `SseTopics`）承载；会话或过程状态放 `context`。只有真正的 Spring 领域事件才建 `event/` 包，不为单个载荷新开分类包。
- auth 模块与其他业务域统一采用 `Form`/`Query`/`VO`，不要再并存 `req`/`resp` 两套后缀。

## 输入校验

- `@RequestBody` 输入使用 `@Valid`；Controller 类需要校验 Path/Query 标量时使用 `@Validated`。
- 分页参数约束 `pageNum >= 1`，并为 `pageSize` 设置合理上限。
- 排序字段必须映射到服务端白名单；排序方向使用枚举或明确的允许值，禁止直接拼接任意 SQL 字段。
- 状态、类型等有限集合优先使用枚举；为兼容数据库整数时，在边界完成转换和校验。
- 动态表单的 `Map<String, Object>` 无法只靠 Bean Validation 覆盖，必须执行字段白名单、类型、长度和业务规则校验。
- 不要只在新增接口校验；更新、状态变更、密码重置等写入口同样必须校验。

## API 与响应

- 资源路径使用复数名词，如 `/api/v1/users`；动作端点只在无法自然表达为资源状态时使用。
- Controller 返回 `Result<T>` 或 `PageResult<T>`；无响应数据使用 `Result<Void>`，避免 `Result<?>`。
- Controller 负责协议适配，Service 不返回 `Result`，Mapper 不返回 `Result` 或 `PageResult`。
- 现有模块可暂时让 Service 返回 `IPage<VO>`、Controller 调用 `PageResult.success(page)`；同一模块不要混用 `Page`、`IPage`、`PageResult` 三种服务契约。
- 新增成功优先返回资源 ID；更新、删除成功返回 `Result<Void>`。
- 业务码保留现有 `A****`、`B****`、`C****` 体系，同时使用合适的 HTTP 状态表达协议错误，如 400、401、403、404、409、429、500。
- 新增/更新接口按实际幂等风险使用 `@RepeatSubmit`；增删改和关键动作按审计需要添加 `@Log`，不要机械标注所有查询。

## 持久化与对象转换

- Entity 不直接从 Controller 返回，也不作为跨模块契约。
- 单表简单映射优先 MapStruct；复杂联表查询允许 Mapper 直接投影 VO。
- 禁止用 `BeanUtils.copyProperties` 或 JSON 序列化做常规对象转换；少量明确字段手写映射可接受。
- Entity 的主键策略和逻辑删除方式以项目全局 MyBatis-Plus 配置为准。不要同时编造一套冲突的 `@TableLogic` 或 ID 策略。

## 实施流程

1. 阅读目标模块、相邻模块、数据库映射、配置和测试，确认真实契约。
2. 判断改动属于输入 Form、查询 Query、输出 VO、内部 DTO/Event/Context 中的哪一类。
3. 先定义校验、授权、事务边界和失败语义，再实现 Mapper、Service、Controller。
4. 检查跨域依赖；若需要另一个域的数据或动作，调用窄接口，不依赖其 Entity 或通用 `IService`。
5. 编译并运行与改动相关的测试；外部数据库、Redis 或第三方服务不可用时，明确说明未验证部分。
6. 只修改任务范围内文件，报告兼容性影响和剩余迁移项。

## 注释原则

- 注释解释约束、原因、副作用、线程/事务要求和容易踩坑的行为，不重复类名、注解或代码表面含义。
- 公共扩展点、跨模块接口和存在非显然约束的方法应写 Javadoc；简单内部 POJO、显而易见的 getter/setter 不强制逐项注释。
- 需要稳定生成 Javadoc 摘要时优先使用 `{@summary ...}`。
- `@author` 沿用项目既有的作者约定，不要自创值；`@since` 写项目当前版本号，与 `pom.xml` 的 `version` 一致，升版本时同步更新，不写日期。
- API 字段优先由 `@Schema` 描述协议含义；Javadoc 补充领域约束，避免两处复制同一句话。
- 代码变化时同步更新注释；不确定是否仍成立的历史说明应删除或验证后改写。

详细规则见 [references/comments.md](references/comments.md)。

## 按需读取参考

- 认证、白名单、Token 或权限：读 [references/authentication.md](references/authentication.md)。
- 事务边界或事务失效：读 [references/transaction.md](references/transaction.md)。
- MapStruct 或查询投影：读 [references/mapstruct.md](references/mapstruct.md)。
- 日志级别、异常日志、脱敏：读 [references/logging.md](references/logging.md)。
- 新增业务资源或模块：读 [references/new-module.md](references/new-module.md)。
- 注释或 Javadoc 专项整理：读 [references/comments.md](references/comments.md)。
