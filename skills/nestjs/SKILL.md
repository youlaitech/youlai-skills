---
name: nestjs
description: NestJS backend development standards. Use this skill when developing NestJS projects, implementing REST APIs, using TypeORM, JWT authentication, or permission control.
---

# NestJS 后端开发规范

## 触发条件

- Develop NestJS projects
- Implement REST APIs
- Use TypeORM for data access
- Implement JWT authentication
- Implement permission control

---

## Part 1: 技术栈

| 层 | 选型 | 说明 |
|----|------|------|
| 运行环境 | **Node.js 20+** | ES2022+、原生 fetch |
| 框架 | **NestJS** | 装饰器、模块化、依赖注入 |
| 语言 | **TypeScript** 5.x | 类型安全、装饰器元数据 |
| ORM | **TypeORM** | Active Record / Data Mapper |
| 数据库 | **MySQL 8.x** | InnoDB 引擎 |
| 缓存 | **Redis 7.x** (ioredis) | 分布式缓存 |
| 认证 | **@nestjs/jwt** + **@nestjs/passport** | JWT 签发/验签 |
| API 文档 | **@nestjs/swagger** | OpenAPI 自动生成 |

---

## Part 2: 目录结构

```
src/
├── common/                         # 公共模块
│   ├── constants/                  # metadata, redis, role, system
│   ├── decorators/                 # auth, current-user, data-permission, log
│   ├── dto/                        # base-query.dto
│   ├── entities/                   # base.entity
│   ├── enums/                      # action-type, data-scope, error-code, log-module
│   ├── exceptions/                 # business.exception
│   ├── filters/                    # http-exception.filter
│   ├── guards/                     # jwt-auth, permission, data-scope
│   ├── interceptors/               # response, request, data-permission
│   ├── interfaces/                 # current-user, response
│   ├── middleware/                  # logger, rate-limit, request-context
│   ├── models/                     # role-data-scope.model
│   ├── redis/                      # redis.module, redis.service
│   ├── subscribers/                # audit.subscriber
│   └── utils/                      # captcha, logger, serialize, user-agent
│
├── config/                         # jwt, oss, redis, typeorm
│
├── auth/                           # 认证模块
│   ├── auth.{controller,service,module}.ts
│   ├── wxma-auth.{controller,service}.ts
│   ├── strategies/jwt.strategy.ts
│   ├── guards/redis-token.guard.ts
│   └── dto/                        # login-request, login-result, wxma-login-result
│
├── system/                         # 系统管理（按实体分子模块）
│   ├── user/                       # {controller,service,module, entities/, dto/}
│   ├── role/                       # 同上 + role-permission.service
│   ├── menu/                       # 同上
│   ├── dept/ dict/ config/ notice/ log/
│
├── codegen/                        # 代码生成器
├── file/                           # 文件模块
├── message/                        # 消息模块（SSE）
├── app.module.ts                   # 根模块
└── main.ts                         # 入口
```

**设计原则**：
- `common/` 是公共组件，被所有模块共享
- `auth/` 是认证模块，含 JWT 策略 + 守卫
- `system/` 按实体分子模块，每子模块含 controller/service/module/entities/dto
- NestJS 用 Module 组织依赖，Decorator 声明路由/守卫/拦截器

---

## Part 3: 命名规范

### 3.1 文件命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 模块 | `{name}.module.ts` | `user.module.ts` |
| 控制器 | `{name}.controller.ts` | `user.controller.ts` |
| 服务 | `{name}.service.ts` | `user.service.ts` |
| 实体 | `{name}.entity.ts` | `sys-user.entity.ts` |
| DTO | `{name}.dto.ts` | `create-user.dto.ts` |
| 守卫 | `{name}.guard.ts` | `jwt-auth.guard.ts` |
| 策略 | `{name}.strategy.ts` | `jwt.strategy.ts` |

### 3.2 类命名

| 类型 | 规范 | 示例 |
|------|------|------|
| Controller | `{Entity}Controller` | `UserController` |
| Service | `{Entity}Service` | `UserService` |
| Module | `{Entity}Module` | `UserModule` |
| Entity | PascalCase | `SysUser` |
| DTO | 功能 + 类型后缀 | `CreateUserDto`, `UserPageDto`, `UserVO` |

### 3.3 方法命名

| 动作 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | get | `getById()` |
| 查询列表 | list | `listUsers()` |
| 分页查询 | page | `pageUsers()` |
| 新增 | create | `createUser()` |
| 更新 | update | `updateUser()` |
| 删除 | remove / delete | `removeById()` |

### 3.4 变量命名

| 类型 | 规范 | 示例 |
|------|------|------|
| 变量 | camelCase | `userList` |
| 常量 | UPPER_SNAKE_CASE | `MAX_SIZE` |
| 私有变量 | `_` 前缀 | `_userService` |
| 布尔值 | is/has/can 前缀 | `isDeleted`, `hasPermission` |

---

## Part 4: RESTful API 规范

### 4.1 标准 CRUD 路径

| 操作 | 方法 | 路径 |
|------|------|------|
| 分页列表 | `GET` | `/api/v1/users/page` |
| 详情 | `GET` | `/api/v1/users/:id` |
| 新增 | `POST` | `/api/v1/users` |
| 更新 | `PUT` | `/api/v1/users` |
| 删除 | `DELETE` | `/api/v1/users/:id` |
| 批量删除 | `DELETE` | `/api/v1/users/batch` |
| 下拉选项 | `GET` | `/api/v1/users/options` |

### 4.2 Controller 模板

```typescript
@Controller("api/v1/users")
@ApiTags("用户管理")
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get("page")
  @ApiOperation({ summary: "用户分页列表" })
  async page(@Query() query: UserPageDto) {
    return this.userService.pageUsers(query);
  }

  @Get(":id")
  @ApiOperation({ summary: "用户详情" })
  async getById(@Param("id") id: string) {
    return this.userService.getUserById(Number(id));
  }

  @Post()
  @ApiOperation({ summary: "新增用户" })
  @RequirePermission("sys:user:add")
  async create(@Body() form: CreateUserDto) {
    return this.userService.createUser(form);
  }

  @Put()
  @ApiOperation({ summary: "更新用户" })
  @RequirePermission("sys:user:edit")
  async update(@Body() form: UpdateUserDto) {
    return this.userService.updateUser(form);
  }

  @Delete(":id")
  @ApiOperation({ summary: "删除用户" })
  @RequirePermission("sys:user:delete")
  async remove(@Param("id") id: string) {
    this.userService.removeUserById(Number(id));
  }
}
```

---

## Part 5: 响应格式与异常处理

### 5.1 统一响应

```typescript
export interface Result<T> {
  code: number;
  msg: string;
  data: T;
}

export interface PageResult<T> {
  code: number;
  msg: string;
  data: { list: T[]; total: number };
}

// 响应拦截器自动包装
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Result<T>> {
  intercept(_context: ExecutionContext, next: CallHandler): Observable<Result<T>> {
    return next.handle().pipe(map(data => ({ code: 200, msg: "操作成功", data })));
  }
}
```

### 5.2 业务异常 + 全局过滤器

```typescript
export class BusinessException extends Error {
  constructor(public readonly message: string, public readonly code: number = 400) {
    super(message);
  }
}

@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();
    let code = 500, msg = "系统异常";
    if (exception instanceof BusinessException) { code = exception.code; msg = exception.message; }
    else if (exception instanceof HttpException) { code = exception.getStatus(); msg = exception.message; }
    response.status(200).json({ code, msg, data: null });
  }
}
```

---

## Part 6: 实体规范

```typescript
@Entity()
export abstract class BaseEntity {
  @PrimaryGeneratedColumn("increment", { type: "bigint" })
  id: string;

  @CreateDateColumn({ name: "create_time", comment: "创建时间" })
  createTime: Date;

  @UpdateDateColumn({ name: "update_time", comment: "更新时间" })
  updateTime: Date;

  @Column({ type: "tinyint", default: 0, comment: "删除标志(0未删除 1已删除)" })
  deleted: number;
}

@Entity("sys_user")
export class SysUser extends BaseEntity {
  @Column({ length: 50, unique: true, comment: "用户名" })
  username: string;

  @Column({ length: 100, select: false, comment: "密码" })
  password: string;

  @Column({ length: 50, comment: "昵称" })
  nickname: string;

  @Column({ type: "tinyint", default: 1, comment: "状态(1正常 0禁用)" })
  status: number;

  @Column({ name: "dept_id", type: "bigint", nullable: true, comment: "部门ID" })
  deptId: string;
}
```

---

## Part 7: Service 规范

```typescript
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(SysUser) private userRepo: Repository<SysUser>,
  ) {}

  async pageUsers(query: UserPageDto): Promise<PageResult<UserVO>> {
    const { pageNum, pageSize, keywords, status } = query;
    const qb = this.userRepo.createQueryBuilder("user");
    if (keywords) qb.andWhere("user.username LIKE :kw OR user.nickname LIKE :kw", { kw: `%${keywords}%` });
    if (status !== undefined) qb.andWhere("user.status = :status", { status });
    qb.orderBy("user.createTime", "DESC").skip((pageNum - 1) * pageSize).take(pageSize);
    const [list, total] = await qb.getManyAndCount();
    return { code: 200, msg: "操作成功", data: { list: list.map(this.toVO), total } };
  }

  async createUser(form: CreateUserDto): Promise<number> {
    if (await this.userRepo.findOne({ where: { username: form.username } }))
      throw new BusinessException("用户名已存在");
    const user = this.userRepo.create({ ...form, password: await bcrypt.hash(form.password, 10) });
    await this.userRepo.save(user);
    return Number(user.id);
  }

  private toVO(user: SysUser): UserVO {
    return { id: Number(user.id), username: user.username, nickname: user.nickname, status: user.status, createTime: user.createTime };
  }
}
```

---

## Part 8: 认证规范

### 8.1 JWT 策略

```typescript
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(configService: ConfigService) {
    super({ jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(), secretOrKey: configService.get("jwt.secret") });
  }
  async validate(payload: JwtPayload): Promise<CurrentUser> {
    return { userId: payload.sub, username: payload.username };
  }
}
```

### 8.2 权限装饰器 + 守卫

```typescript
export const RequirePermission = (...permissions: string[]) => SetMetadata("permissions", permissions);

@Injectable()
export class PermissionGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(context: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<string[]>("permissions", [context.getHandler(), context.getClass()]);
    if (!required) return true;
    const { permissions } = context.switchToHttp().getRequest().user;
    return required.some(p => permissions.includes(p));
  }
}
```

### 8.3 权限标识格式

```
模块:实体:操作
sys:user:create / sys:user:edit / sys:user:delete / sys:role:list
```

---

## Part 9: 注释规范

- **公共类/方法用 JSDoc** (`/** */`)，含 `@param`/`@returns`/`@throws`
- NestJS 装饰器参数用 `@ApiProperty({ description: "..." })`

```typescript
/**
 * 用户管理服务。
 * 负责用户 CRUD、密码管理、角色分配。
 */
@Injectable()
export class UserService {
  /**
   * 创建新用户。
   * @param form - 用户表单（username 必填且唯一）
   * @returns 新用户 ID
   * @throws BusinessException 用户名已存在
   */
  async createUser(form: CreateUserDto): Promise<number> { ... }
}
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应拦截器 + 全局异常过滤器
- [ ] 权限校验使用 `@RequirePermission` 装饰器
- [ ] DTO 使用 `class-validator` 校验
- [ ] 实体继承 `BaseEntity`
- [ ] 使用 TypeORM 逻辑删除（`deleted` 字段）
- [ ] API 添加 `@ApiOperation` 注解
- [ ] 模块按功能划分子模块
- [ ] 公共方法有 JSDoc 注释

### 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| Controller 中直接操作 Repository | 委托 Service 层 |
| `catch (e) { }` 空捕获 | 抛出 `BusinessException` 或记录日志 |
| TypeORM `findOne()` 不处理 null | 检查 null → `throw new BusinessException("不存在")` |
| `response.json({ code: 200 })` 散落各处 | 用拦截器统一包装 |
| `@Body() body: any` 无类型 | 定义 DTO 类 + `class-validator` 校验 |
