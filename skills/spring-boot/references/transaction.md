# 事务边界

仅在一个用例包含多次写入、跨表一致性、事件发布或事务失效风险时读取本文件。

## 选择事务的位置

- 事务边界放在负责完整业务用例的 Service 方法，不放在 Controller。
- 单条独立 Mapper 写入通常不需要为了形式额外包事务。
- 一个用例需要多表写入、读后写一致性或“全部成功/全部失败”时使用事务。
- 只读事务可用于一组需要一致快照或明确只读优化的查询；不要求给每个简单查询机械添加 `readOnly = true`。

```java
@Transactional
public Long createUser(UserCreateForm form) {
    // 多表写入，RuntimeException 默认触发回滚。
}

@Transactional(readOnly = true)
public UserOverviewVO getOverview(Long userId) {
    // 多次读取需要同一事务上下文时使用。
}
```

## 回滚规则

Spring 默认对 `RuntimeException` 和 `Error` 回滚。只有受检异常可能逃出方法且业务要求回滚时，才显式扩大范围：

```java
@Transactional(rollbackFor = Exception.class)
```

不要把 `rollbackFor = Exception.class` 当成所有写方法的强制模板。更重要的是不要吞掉异常，也不要在已经标记回滚后假装返回成功。

## 常见失效场景

| 场景 | 风险 | 优先做法 |
|------|------|----------|
| 同类内部调用事务方法 | 调用未经过代理 | 抽取到独立 Service；简单场景可用 `TransactionTemplate` |
| 非代理管理的对象 | 注解不生效 | 让 Spring 管理该组件并通过 Bean 调用 |
| 异常被捕获后不再抛出 | 事务可能提交 | 转换并重新抛出，或显式标记 rollback-only |
| 异步线程 | 不继承调用线程事务 | 在异步消费者中建立自己的事务 |
| 远程调用处于长事务中 | 占用连接和锁，失败语义复杂 | 缩短本地事务，使用事件、Outbox 或补偿机制 |

不推荐通过“注入自身代理”作为默认答案；它容易隐藏职责和循环依赖。优先拆分职责或使用显式事务模板。

## 事件与外部副作用

- 需要数据库提交后执行的动作使用 `@TransactionalEventListener(phase = AFTER_COMMIT)`。
- 对可靠投递有要求时，仅靠内存事件不够，使用 Outbox 或等价持久化机制。
- 文件上传、短信、邮件、HTTP/RPC 等不可回滚副作用不要与数据库事务混成伪原子操作；定义重试、幂等或补偿策略。

## 批量操作

- 批大小根据 SQL 长度、锁时间、内存和数据库限制确定，不固定为 500。
- 大批量任务需要进度、断点续跑或部分失败语义时，不要用一个超长事务包住全部数据。
- 批次间提交前确认业务是否允许部分成功；不允许时需重新设计导入或预校验流程。
