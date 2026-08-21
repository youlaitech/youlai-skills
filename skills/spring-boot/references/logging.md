# 日志规范

## 日志框架

- 门面：SLF4J（注解 `@Slf4j` 替代 `LoggerFactory.getLogger()`）
- 实现：Logback（Spring Boot 默认）

## 日志级别

| 级别 | 使用场景 | 示例 |
|------|----------|------|
| `ERROR` | 影响运行的错误，需人工介入 | 数据库连接失败、第三方 API 异常 |
| `WARN` | 非预期但可降级 | 缓存未命中降级、限流触发、登录失败 |
| `INFO` | 关键业务节点 | 用户登录、订单创建、定时任务开始/结束 |
| `DEBUG` | 调试信息（生产关闭） | SQL 语句、请求参数 |

## 打印规范

```java
// 错误：字符串拼接（即使日志级别关闭也会执行）
log.debug("用户信息: " + user.toString());

// 错误：用 System.out
System.out.println("用户创建成功");

// 正确：占位符 {}
log.info("用户创建成功: username={}, userId={}", form.getUsername(), user.getId());

// 正确：异常日志含堆栈（e 作为最后参数）
try {
    userMapper.insert(user);
} catch (Exception e) {
    log.error("保存用户失败: username={}", form.getUsername(), e);
    throw new BusinessException(ResultCode.SYSTEM_ERROR);
}

// 正确：敏感信息脱敏
log.info("用户登录: username={}, mobile={}", username, maskMobile(mobile));
// 禁止：日志输出密码、Token、身份证号明文
```

## 内容要求

| 要求 | 说明 |
|------|------|
| 携带业务标识 | `userId={}` `orderNo={}`，方便检索 |
| 异常含堆栈 | `log.error(msg, e)`，异常对象作为最后参数 |
| 禁止敏感信息 | 密码、Token、身份证号必须脱敏 |
| 条件日志 | 复杂计算用 `if (log.isDebugEnabled())` 包裹 |
