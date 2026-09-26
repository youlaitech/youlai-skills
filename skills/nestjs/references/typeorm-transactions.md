# TypeORM、查询与事务

仅在项目使用 TypeORM 时应用本文。若仓库使用 Prisma、MikroORM、Drizzle 或其他方案，沿用其原生事务和迁移机制。

## Entity 边界

- Entity 描述表结构、关系和持久化约束，不作为请求 DTO 或响应 DTO。
- 数据库 `bigint` 在 TypeORM 中通常映射为 string；API 层也保持 string，不使用 `Number(id)`。
- 可空列的 TypeScript 类型同时包含 `null` 或项目约定的空值，避免类型与数据库不一致。
- 密码哈希、令牌摘要等敏感列使用 `select: false`，响应层仍需显式投影作为第二道防线。
- 唯一性、非空和关联完整性应由数据库约束兜底，Service 的预检查只用于改善错误提示。

不是所有 Entity 都必须继承同一个 `BaseEntity`。业务实体、关联表、日志表和元数据表可以有不同审计字段。只有字段和生命周期语义一致时才复用基类。

## 软删除策略

一个模型内选择并一致使用一种策略：

1. TypeORM 原生软删除：使用 `@DeleteDateColumn`、`softDelete` / `softRemove`，查询按 TypeORM 语义排除已删除数据。
2. 业务删除标志：使用 `isDeleted` / `deleted` 等字段，并通过明确的 repository helper 或统一查询入口附加条件。

不要把普通数值删除标志称为“TypeORM 逻辑删除”，也不要在部分查询使用原生软删除、另一些查询手动判断标志。管理端查看已删除数据时应有显式接口和权限。

## 查询规则

- 参数始终绑定，不拼接客户端输入。
- 动态排序使用字段白名单，不能直接采用请求提供的列名或方向。
- 分页设置最大页长；深分页或大表场景根据访问模式考虑游标分页。
- 关联加载以实际响应字段为依据，避免无界 relation 和 N+1 查询。
- 对常用过滤、排序、唯一约束和外键检查索引；不要凭感觉添加重复索引。
- 复杂联表可以直接投影为 Response DTO，只要查询边界、字段别名和测试清楚。

不要 monkey-patch `SelectQueryBuilder.prototype` 或其他全局 prototype 来自动追加数据权限、租户或软删除条件。这类隐式修改会污染所有查询且难以测试。优先使用：

- 显式查询 helper。
- 自定义 repository / adapter。
- 接收授权上下文的 query builder factory。
- ORM 官方提供且行为可验证的扩展点。

## 事务边界

以下操作通常需要同一事务：

- 主记录与关联表同时写入。
- 状态变化与审计记录必须一起成功。
- 角色授权、菜单授权或用户角色分配。
- 先检查当前状态再更新，且并发下必须保持不变量。

事务中的所有数据库调用必须使用同一个 transaction manager 或 QueryRunner，不要混入普通注入 Repository。

```typescript
await dataSource.transaction(async (manager) => {
  const userRepo = manager.getRepository(SysUser);
  const roleRepo = manager.getRepository(SysUserRole);

  await userRepo.save(user);
  await roleRepo.insert(roleRows);
});
```

缓存删除、消息发布、邮件、对象存储和 SSE 推送不是数据库事务的一部分。优先在提交成功后执行；要求可靠投递时采用 outbox 等可恢复机制。不要在事务尚未提交时通知其他组件读取新状态。

## 并发与幂等

- 依赖唯一键处理重复创建，并将冲突转换为 409 或项目约定的业务错误。
- 余额、库存、版本状态等并发更新使用原子 SQL、乐观锁或悲观锁，选择必须与冲突频率和事务范围匹配。
- 可重试接口定义幂等键或自然幂等条件，不以“前端不会重复提交”为保证。
- 批量写入要明确全部成功还是部分成功，并为失败项提供稳定反馈。

## 迁移与配置

- 生产环境关闭 `synchronize`，使用可审查的 migration 演进 schema。
- migration 同时考虑向前部署、回滚难度、历史数据回填和零停机兼容窗口。
- Entity 变更与 migration 一起验证，避免只修改装饰器。
- 连接信息来自经过校验的配置，不在源码或 Docker 镜像中写入真实凭据。
