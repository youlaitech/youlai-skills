# 日志规范

仅在新增业务日志、异常处理、认证或外部调用时读取本文件。

## 日志级别

| 级别 | 使用场景 |
|------|----------|
| `ERROR` | 当前操作失败且需要排查，或发生未预期系统异常 |
| `WARN` | 可恢复的异常状态、限流、降级、重试耗尽或可疑行为 |
| `INFO` | 有审计或运营价值的关键业务节点，不记录每个普通 CRUD |
| `DEBUG` | 开发诊断信息，生产通常关闭 |

参数校验失败、资源不存在等预期客户端错误通常使用 `WARN` 或 `DEBUG`，不要全部记录为 `ERROR` 并打印完整堆栈。

## 写法

```java
log.info("用户创建成功: username={}, userId={}", form.getUsername(), userId);

try {
    remoteClient.send(command);
} catch (RemoteException e) {
    log.error("通知发送失败: noticeId={}, provider={}", noticeId, provider, e);
    throw new BusinessException(ResultCode.THIRD_PARTY_SERVICE_ERROR);
}
```

- 使用 SLF4J 占位符，不拼接日志字符串。
- 异常对象作为最后一个参数，以保留堆栈。
- 日志包含可检索的业务标识，如 `userId`、`formKey`、`instanceId`。
- 复杂参数序列化前检查日志级别，并限制长度。
- 不使用 `System.out`、`printStackTrace`。

## 敏感信息

禁止记录以下明文：

- 密码、验证码、access token、refresh token、JWT 密钥；
- 数据库、Redis、邮件、对象存储凭据；
- 身份证、银行卡和完整手机号等个人信息；
- 完整请求体中无法确认安全的动态字段。

只记录必要的脱敏值或不可逆摘要。认证头即使哈希后也不应作为跨系统用户标识长期保存。

## 异常边界

- 在能够补充业务上下文且最终处理异常的边界记录一次，不要每层重复打印同一异常。
- 重新抛出时保留 cause。
- 返回给客户端的消息不要包含 SQL、文件路径、第三方响应体或堆栈内容。
- 审计日志与诊断日志分开：`@Log` 表达谁在何时做了什么，应用日志解释系统为什么失败。
