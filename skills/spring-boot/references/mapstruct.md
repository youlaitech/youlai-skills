# 对象转换与查询投影

仅在 Entity、Form、VO 之间转换，或设计复杂查询返回结构时读取本文件。

## 决策规则

| 场景 | 推荐方式 |
|------|----------|
| 单表 Entity ↔ Form/VO，字段大体同名 | MapStruct |
| 多对象组装、少量字段重命名 | MapStruct + `@Mapping` |
| 复杂联表、聚合统计、分页读模型 | Mapper 直接投影 VO |
| 只有两三个明确字段 | 手写构造或 setter 可接受 |
| 反射复制或 JSON 中转 | 禁止用于常规转换 |

不要为了“所有转换必须经过 Converter”而先查完整 Entity 再二次组装复杂列表；也不要让 Mapper 接收 Controller 的 Form。

## MapStruct 约定

```java
@Mapper(componentModel = "spring")
public interface UserConverter {

    UserVO toVO(User entity);

    User toEntity(UserCreateForm form);

    UserEditVO toEditVO(User entity);

    @Mapping(target = "id", ignore = true)
    void updateEntity(UserUpdateForm form, @MappingTarget User entity);
}
```

- Converter 放在所属业务域的 `converter/`。
- 更新已有 Entity 时优先使用 `@MappingTarget`，显式忽略 ID、审计字段和不能由客户端修改的字段。
- 字段名不同、枚举转换或时间格式转换必须显式声明。
- 对敏感字段采取显式忽略策略，避免新字段加入后意外进入 VO。
- 修改模型后检查 MapStruct 编译警告，不要全局关闭 unmapped 警告来掩盖契约变化。

## Mapper 直接投影

复杂联表可以直接返回只读 VO：

```java
IPage<UserPageVO> selectUserPage(
        Page<?> page,
        @Param("query") UserPageQuery query
);
```

约束：

- VO 只表达读模型，不包含写入行为。
- SQL 别名与 VO 字段保持清晰映射。
- Mapper 不返回 Controller 响应包装 `Result`/`PageResult`。
- Query 可以传给查询 Mapper，但 Form 不直接传给写 Mapper；Service 先转换成 Entity 或明确的持久化参数。

## 不推荐做法

```java
BeanUtils.copyProperties(source, target);

UserVO vo = objectMapper.convertValue(entity, UserVO.class);

int update(UserUpdateForm form);
```

前两种方式隐藏类型和字段错误；最后一种让持久化层直接依赖 HTTP 输入契约。
