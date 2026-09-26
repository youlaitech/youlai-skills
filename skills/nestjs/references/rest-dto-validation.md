# HTTP、DTO 与校验

## 路由设计

如果应用在 `main.ts` 设置了全局前缀或 URI 版本，Controller 不再重复它：

```typescript
// main.ts
app.setGlobalPrefix("api");
app.enableVersioning({ type: VersioningType.URI, defaultVersion: "1" });

// user.controller.ts
@Controller("users")
export class UserController {}
```

常见资源路由：

| 目的 | 方法和路径 |
| --- | --- |
| 分页或筛选列表 | `GET /users?pageNum=1&pageSize=20` |
| 详情 | `GET /users/:id` |
| 创建 | `POST /users` |
| 完整替换 | `PUT /users/:id` |
| 部分更新 | `PATCH /users/:id` |
| 删除 | `DELETE /users/:id` |
| 批量删除 | 项目已有约定；使用专用 DTO 表达 IDs |
| 下拉选项 | `GET /users/options` |

静态路径应在参数路径之前声明，避免 `options` 或 `batch` 被当作 ID。更新时以 Path ID 为资源标识，不要同时要求 Body 提供另一个 ID。

密码、刷新令牌、短信验证码等敏感值不要放在 URL 查询参数中；根据威胁模型使用请求 Body、受保护 Cookie 或协议规定的 Header。

## 模型职责

| 类型 | 建议职责 | 规则 |
| --- | --- | --- |
| `Entity` | 持久化映射 | 仅在数据访问层使用，不直接对外返回 |
| `CreateXxxDto` | 创建输入 | 只允许客户端可写字段，提供完整运行时校验 |
| `UpdateXxxDto` | 更新输入 | 与 `PUT` / `PATCH` 语义一致，不包含 Path ID |
| `XxxQueryDto` | 非分页查询条件 | 字段可选性、枚举和转换规则明确 |
| `XxxPageQueryDto` | 分页查询条件 | 明确页码、页长、默认值和最大页长 |
| `XxxResponseDto` | 对外响应 | 只包含契约字段，敏感字段永不出现 |
| `XxxDetailDto` | 详情响应 | 仅在详情确有额外字段时使用 |
| `XxxFormResponseDto` | 编辑表单回显 | 只用于前端表单所需的只读投影 |
| `interface` / `type` | 进程内契约 | 用于内部对象和第三方结果，不承担运行时校验 |

NestJS 项目不必复制 Java 项目的 `VO / DTO / Form / Query` 全套命名。优先采用 Nest 社区熟悉的 `Dto` 后缀，并通过前缀表达用途。

## 输入校验

请求 DTO 必须是 class，因为 interface 在运行时被擦除：

```typescript
export class CreateUserDto {
  @ApiProperty({ description: "用户名", minLength: 3, maxLength: 50 })
  @IsString()
  @Length(3, 50)
  username!: string;

  @ApiProperty({ description: "部门 ID", required: false, type: String })
  @IsOptional()
  @IsString()
  @Matches(/^\d+$/)
  deptId?: string;
}
```

全局 ValidationPipe 应根据兼容性需求启用 `transform`、`whitelist` 和 `forbidNonWhitelisted`。不要在不检查现有客户端的情况下突然切换严格策略。

注意以下绕过点：

- `@Body("ticket") ticket: string` 不会执行类级 DTO 校验；使用专用 DTO。
- `@Body() ids: string[]` 不能仅靠 TypeScript 校验数组元素；使用 DTO 或 `ParseArrayPipe`。
- 查询字符串默认是 string；数字、布尔值和日期需显式转换并校验。
- 枚举字段同时使用 `@IsEnum` 和准确的 OpenAPI schema。
- 页长设置合理上限，防止一次查询过多记录。
- 自定义校验不得依赖可被客户端伪造的服务端字段。

## 显式映射

不要把请求对象直接扩展进实体：

```typescript
// 避免：服务端字段可能被意外写入
repository.create({ ...dto });

// 推荐：客户端可写字段白名单
repository.create({
  username: dto.username,
  nickname: dto.nickname,
  deptId: dto.deptId,
});
```

更新同样使用显式 allow-list。密码哈希、角色、租户、创建人、删除状态和审核状态由专门用例控制。

响应使用显式投影或 mapper。不要返回 Entity，也不要假定 `select: false` 是唯一防线。

## 响应与异常

统一成功包装只执行一次。Service 返回业务数据，如 `{ list, total }`；Interceptor 再转换为项目约定的 envelope，避免 Service 自己返回 `code/msg/data` 后被二次包装。

异常过滤器应保持 HTTP 语义：

| 情况 | 常见状态 |
| --- | --- |
| 输入或业务前置条件无效 | 400 / 422（遵循项目约定） |
| 未认证 | 401 |
| 无权限 | 403 |
| 资源不存在 | 404 |
| 唯一键或状态冲突 | 409 |
| 请求过多 | 429 |
| 未知服务端错误 | 500 |

业务错误应基于 `HttpException` 或映射到明确 HTTP 状态，不要仅继承 `Error` 后把所有异常用 HTTP 200 返回。

未知异常在服务端记录请求标识、路由和结构化上下文；客户端只接收稳定消息，不能暴露堆栈、SQL、文件绝对路径或上游密钥。
