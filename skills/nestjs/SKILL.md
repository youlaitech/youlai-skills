---
name: nestjs
description: Develop, review, diagnose, and refactor NestJS backends while respecting the repository's existing architecture. Use for NestJS modules, controllers, providers, DTO validation, persistence, authentication and authorization, configuration, security, and tests. Inspect the project first; do not force REST, TypeORM, JWT, or a new directory taxonomy when the codebase uses different choices.
---

# NestJS 工程实践

## 目标

在不破坏项目既有边界、命名和技术选型的前提下，交付可维护、可验证且安全的 NestJS 改动。

本 Skill 提供决策规则，不提供必须逐字复制的脚手架。用户要求、仓库内 `AGENTS.md` 与项目配置优先于本文示例。

## 工作流程

1. 先检查仓库，不根据 Skill 猜测技术栈：
   - 读取 `package.json`、锁文件、`nest-cli.json`、TypeScript 与 lint 配置。
   - 检查 `main.ts`、根模块、相关功能模块、DTO、实体或模型、测试和数据库迁移。
   - 查看 Git 状态，保留用户已有改动，不顺手重构无关代码。
2. 明确任务性质：新增功能、修复、诊断、评审、安全加固或结构调整。
3. 先确定对外契约和模块边界，再实现最小且完整的改动。
4. 复用项目现有框架与惯例；只有当前做法造成真实问题时才建议迁移。
5. 按风险验证，并如实报告执行的命令、失败原因和未覆盖项。

## 项目兼容原则

- 不强制 REST、TypeORM、JWT、MySQL、Redis 或 Swagger；仅在仓库已经采用或用户明确要求时使用。
- 不为追求“标准目录”批量改名。对于有来项目，保留 `auth/`、`system/user/`、`system/role/`、`system/menu/`、`system/dept/`、`file/`、`message/`、`codegen/`、`common/`、`config/` 等现有业务语义。
- 不要求把 `system/dept` 改成 `organization`，也不要求把角色、菜单和权限统一迁入 `authorization`。
- 沿用项目的包管理器、格式化规则、响应结构、异常基类和数据库访问模式；发现明显缺陷时说明风险并做局部修正。

## 核心规则

### 模块和依赖

- 按业务能力组织功能模块；Controller 处理协议适配，Provider 承担业务用例，持久化实现负责数据访问。
- 模块只导出外部确实需要的 Provider。优先单向依赖；先拆出共享能力或使用注入令牌，再考虑 `forwardRef()`。
- `common/` 只能依赖稳定的通用抽象，不反向依赖具体业务模块。
- 服务按职责和变化原因拆分，不以文件行数作为唯一依据。

### DTO、模型和接口

- HTTP 输入使用带 `class-validator` 规则的 class，不用 interface 充当运行时校验 DTO。
- 创建、更新、查询和响应模型职责分离；不要照搬 Java 项目的 `VO / Form` 分层。
- `Entity` 只表达持久化；不要直接作为外部响应，也不要接收整包请求数据做批量赋值。
- 服务端控制字段通过白名单显式映射。密码哈希、删除标记、租户、权限和审计字段不得由普通请求覆盖。
- 数据库 `bigint` 标识在 API 和 TypeScript 中保持为 string，避免超过安全整数范围。

### HTTP 契约和异常

- 全局前缀和版本只配置一次；Controller 使用相对业务路径。
- 列表可使用 `GET /users` 搭配分页参数；不强制额外的 `/page`。
- 资源更新使用 `/:id`；`PUT` 表示完整替换，`PATCH` 表示部分更新。不要让 Body ID 与 Path ID 形成两个真相来源。
- 成功响应可统一包装，但异常必须保留正确的 HTTP 状态，如 400、401、403、404、409、429 和 500。
- 未知异常记录 request ID 和服务端上下文，对客户端返回稳定的通用消息，不泄露堆栈、SQL 或密钥。

### 数据、安全和副作用

- 多表写入、状态变更与审计等原子操作使用事务；缓存失效、消息和 SSE 通知在事务提交后执行。
- 不修改 ORM 或框架全局 prototype 注入业务行为；使用显式 helper、repository wrapper 或查询构造器函数。
- 认证信息、密码、验证码、刷新令牌、上传路径和运行时密钥按敏感数据处理。
- 权限标识沿用项目既有命名，不在同一项目混用 `add/edit` 与 `create/update`。

### 注释

- 注释解释“为什么”、业务不变量、安全决策、边界条件和非显然副作用，不复述代码。
- 对外导出的契约、复杂规则和兼容性约束使用 JSDoc/TSDoc；类型已经表达清楚时不要机械补齐 `@param` 和 `@returns`。
- DTO 字段优先通过校验器与 OpenAPI 元数据共同表达约束。
- 注入成员使用 `private readonly userService`；不要为了表示私有而添加 `_` 前缀。

## 按需读取参考

只读取与当前任务相关的参考文件：

- 目录、模块边界、循环依赖和服务拆分：读取 [references/architecture.md](references/architecture.md)。
- Controller、路由、DTO、校验和响应契约：读取 [references/rest-dto-validation.md](references/rest-dto-validation.md)。
- TypeORM、实体、软删除、查询和事务：读取 [references/typeorm-transactions.md](references/typeorm-transactions.md)。
- 登录、JWT、RBAC、数据权限、文件和配置安全：读取 [references/auth-security.md](references/auth-security.md)。
- 类型质量、测试、注释和交付检查：读取 [references/testing-comments.md](references/testing-comments.md)。

## 完成标准

- 改动符合当前仓库的真实技术栈和命名，不引入无必要的新分层。
- 请求在运行时经过校验，输出不暴露实体内部字段或敏感信息。
- 依赖方向清晰；新增的跨模块依赖有明确理由且没有可避免的循环。
- 数据写入满足事务和并发要求，外部副作用的执行时机明确。
- 使用项目原生命令完成适量的 typecheck/build、lint、单元测试和相关 e2e 测试。
- 不通过放宽类型、关闭规则、删除断言或弱化测试来掩盖失败。
- 最终说明修改内容、验证结果、剩余风险以及是否存在与本次无关的基线失败。
