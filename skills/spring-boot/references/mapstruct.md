# MapStruct 对象转换规范

## 命名与目录

| 规范 | 说明 |
|------|------|
| 接口名 | `{Entity}Converter`，如 `UserConverter` |
| 存放位置 | `{module}/converter/` |
| 注解 | `@Mapper(componentModel = "spring")` |

## 标准模板

```java
@Mapper(componentModel = "spring")
public interface UserConverter {

    // Entity → VO
    UserVO toVO(SysUser entity);
    List<UserVO> toVOList(List<SysUser> entities);

    // Form → Entity
    SysUser toEntity(UserForm form);

    // Entity → Form（编辑回显）
    UserForm toForm(SysUser entity);

    // 字段名不同时用 @Mapping
    @Mapping(source = "dept.name", target = "deptName")
    UserVO toVOWithDept(SysUser entity);
}
```

## MapStruct vs BeanUtils

| 方案 | 性能 | 类型安全 | 编译期检查 | 推荐度 |
|------|------|----------|-----------|:---:|
| MapStruct | 极高（编译期生成） | 是 | 是 | 推荐 |
| 手写 getter/setter | 极高 | 是 | 是 | 可接受 |
| BeanUtils.copyProperties | 低（反射） | 否 | 否 | 禁止 |

强制使用 MapStruct，禁止 `BeanUtils.copyProperties` 和 JSON 中转。
