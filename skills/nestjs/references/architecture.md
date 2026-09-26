# 架构与模块边界

## 先识别现状

检查根模块和目标模块的 `imports`、`providers`、`controllers`、`exports`，再判断依赖是否合理。不要只凭目录名推断领域边界，也不要仅为符合模板移动文件。

对于有来项目，下面的 feature-first 结构可以保留：

```text
src/
├── auth/
├── system/
│   ├── user/
│   ├── role/
│   ├── menu/
│   ├── dept/
│   ├── dict/
│   ├── config/
│   ├── notice/
│   └── log/
├── file/
├── message/
├── codegen/
├── common/
└── config/
```

这套目录已经体现业务语义。除非用户要做领域重构且收益明确，否则不要将 `dept` 改名为 `organization`，也不要将 `role/menu/permission/data-scope` 整体改名为 `authorization`。

## 模块职责

- 一个功能模块封装一个内聚的业务能力，而非机械对应一张表。
- Controller 负责路由、参数解析、授权元数据和响应契约，不直接拼装复杂查询或跨库写入。
- Application Provider 组织用例和事务；领域规则较复杂时可拆出策略、policy 或专用服务。
- Repository 或 adapter 负责数据访问细节，不把 HTTP DTO 传播到持久化边界。
- Module 的 `exports` 是公共 API，只暴露其他模块需要使用的 Provider。
- 根模块主要负责装配全局基础设施与功能模块，不承载业务实现。

可以增加聚合模块（如 `SystemModule`）简化根模块装配，但这不是必需条件，也不应触发整棵目录树重命名。

## 依赖方向

优先保持依赖单向：

```text
controller -> application provider -> repository / adapter
                              \----> domain policy
```

跨模块调用时依次考虑：

1. 调用方是否真的需要整个目标模块，还是只需查询一个实体仓库。
2. 是否可以抽出稳定接口并通过 injection token 注入实现。
3. 是否可以通过领域事件解耦提交后的通知。
4. 只有双方确实需要互相引用且短期无法拆分时才使用 `forwardRef()`。

发现 `forwardRef()` 时检查其两侧的真实使用。模块虽然被导入但 Provider 未被注入，通常可以直接删除该依赖。

## `common` 边界

适合放入 `common/`：

- 与业务无关的装饰器、管道、过滤器和拦截器。
- 稳定的基础接口、错误抽象、请求上下文和通用工具。
- 被多个模块使用且语义不属于某个业务域的基础设施适配。

不适合放入 `common/`：

- 用户、部门、角色、菜单等具体业务查询。
- 仅被一个模块使用的 helper。
- 为避开循环依赖而搬入的业务服务。

若通用守卫需要读取权限或数据范围，优先依赖一个窄接口或 token；避免 `common` 反向 import `system` 的具体模块。

## 拆分大型服务

按变化原因和一致性边界拆分，例如：

- 用户 CRUD 与密码策略。
- 角色授权与数据范围计算。
- 文件元数据与对象存储适配。
- 查询读取与跨表写入用例。

不要仅因文件超过某个行数就拆分。一个拆分应让职责、测试边界或事务范围更清晰。

## 架构评审问题

- 每个模块的公共能力是什么，`exports` 是否超过需要？
- 是否存在未使用 import、无意义的 `forwardRef()` 或双向 Provider 注入？
- `common` 是否正在了解具体业务实体？
- Controller 是否包含可复用业务规则或数据访问？
- 一次业务操作跨越哪些写入，事务边界在哪里？
- 新抽象是否解决了当前重复或耦合，而不是只增加层级？
