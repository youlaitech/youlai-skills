# 事务规范

## 基本原则

```java
// 必须指定 rollbackFor = Exception.class
@Transactional(rollbackFor = Exception.class)

// 只读查询加 readOnly = true
@Transactional(readOnly = true)

// 错误：未指定 rollbackFor，受检异常不回滚
@Transactional
```

## 事务失效场景（高频踩坑）

| 场景 | 原因 | 解决方案 |
|------|------|----------|
| 类内部方法调用 | AOP 代理失效，`this.method()` 不走代理 | 注入自身代理：`self.method()` |
| 方法非 public | Spring AOP 只代理 public 方法 | 改为 public |
| 异常被 catch 吞掉 | 未抛出 | catch 块中 `throw new RuntimeException(e)` |
| rollbackFor 未指定 | 受检异常不回滚 | 指定 `rollbackFor = Exception.class` |

## 事务边界

- Service 层加事务，Controller 和 Mapper 层不加
- 只读查询标记 `readOnly = true`
- 批量操作分批提交（每 500 条），避免长事务锁表
- 远程调用（RPC/HTTP）不在事务内，避免事务过长
- 异步操作用 `@TransactionalEventListener` 事务提交后触发
