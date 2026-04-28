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

- **Node.js** - 运行环境
- **NestJS** - 后端框架
- **TypeScript** - 类型安全
- **TypeORM** - ORM 框架
- **MySQL** - 数据库
- **Redis** - 缓存
- **JWT** - Token 认证
- **Swagger** - API 文档

---

## Part 2: 目录结构

```
src/
├── common/                         # 公共模块（17 个子目录）
│   ├── constants/                  # 常量
│   │   ├── metadata.constant.ts
│   │   ├── redis.constants.ts
│   │   ├── role.constant.ts
│   │   └── system.constant.ts
│   ├── context/                    # 请求上下文
│   │   └── request-context.ts
│   ├── decorators/                 # 自定义装饰器
│   │   ├── auth.decorator.ts
│   │   ├── current-user.decorator.ts
│   │   ├── data-permission.decorator.ts
│   │   └── log.decorator.ts
│   ├── dto/                        # 基础 DTO
│   │   └── base-query.dto.ts
│   ├── entities/                   # 基础实体
│   │   └── base.entity.ts
│   ├── enums/                      # 枚举
│   │   ├── action-type.enum.ts
│   │   ├── data-scope.enum.ts
│   │   ├── error-code.enum.ts
│   │   └── log-module.enum.ts
│   ├── exceptions/                 # 业务异常
│   │   └── business.exception.ts
│   ├── filters/                    # 异常过滤器
│   │   └── http-exception.filter.ts
│   ├── guards/                     # 守卫
│   │   ├── jwt-auth.guard.ts
│   │   ├── permission.guard.ts
│   │   └── data-scope.guard.ts
│   ├── interceptors/               # 拦截器
│   │   ├── response.interceptor.ts
│   │   ├── request.interceptor.ts
│   │   └── data-permission.interceptor.ts
│   ├── interfaces/                 # 公共接口
│   │   ├── current-user.interface.ts
│   │   └── response.interface.ts
│   ├── middleware/                  # 中间件
│   │   ├── logger.middleware.ts
│   │   ├── rate-limit.middleware.ts
│   │   └── request-context.middleware.ts
│   ├── models/                     # 数据模型
│   │   └── role-data-scope.model.ts
│   ├── plugins/                    # 插件
│   │   └── data-permission.plugin.ts
│   ├── redis/                      # Redis 服务
│   │   ├── redis.module.ts
│   │   └── redis.service.ts
│   ├── subscribers/                # TypeORM 订阅者
│   │   └── audit.subscriber.ts
│   └── utils/                      # 工具类
│       ├── captcha.util.ts
│       ├── logger_utils.ts
│       ├── serialize.util.ts
│       └── user-agent.util.ts
│
├── config/                         # 配置
│   ├── jwt.config.ts
│   ├── oss.config.ts
│   ├── redis.config.ts
│   └── typeorm.config.ts
│
├── auth/                           # 认证模块
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── auth.module.ts
│   ├── wxma-auth.controller.ts     # 微信小程序认证
│   ├── wxma-auth.service.ts
│   ├── strategies/
│   │   └── jwt.strategy.ts
│   ├── guards/
│   │   └── redis-token.guard.ts
│   ├── interfaces/
│   │   └── user-session.interface.ts
│   └── dto/
│       ├── login-request.dto.ts
│       ├── login-result.dto.ts
│       └── wxma-login-result.dto.ts
│
├── system/                         # 系统管理模块（按实体划分子模块）
│   ├── user/                       # 用户管理
│   │   ├── user.controller.ts
│   │   ├── user.service.ts
│   │   ├── user.module.ts
│   │   ├── entities/
│   │   │   ├── sys-user.entity.ts
│   │   │   ├── sys-user-role.entity.ts
│   │   │   └── sys-user-social.entity.ts
│   │   ├── dto/
│   │   │   ├── create-user.dto.ts
│   │   │   ├── update-user.dto.ts
│   │   │   ├── user-form.dto.ts
│   │   │   ├── user-query.dto.ts
│   │   │   ├── current-user.dto.ts
│   │   │   ├── password-change.dto.ts
│   │   │   └── ...
│   │   └── interfaces/
│   │       └── user-auth-info.interface.ts
│   ├── role/                       # 角色管理
│   │   ├── role.controller.ts
│   │   ├── role.service.ts
│   │   ├── role-permission.service.ts
│   │   ├── role.module.ts
│   │   ├── entities/{sys-role,sys-role-menu,sys-role-dept}.entity.ts
│   │   └── dto/{create-role,update-role,role-query}.dto.ts
│   ├── menu/                       # 菜单管理
│   │   ├── menu.controller.ts
│   │   ├── menu.service.ts
│   │   ├── menu.module.ts
│   │   ├── entities/sys-menu.entity.ts
│   │   ├── dto/{create-menu,update-menu}.dto.ts
│   │   └── interfaces/menu.interface.ts
│   ├── dept/                       # 部门管理
│   ├── dict/                       # 字典管理（含 dict-item）
│   ├── config/                     # 系统配置
│   ├── notice/                     # 通知公告（含 user-notice）
│   └── log/                        # 日志管理
│       ├── log.controller.ts
│       ├── log.service.ts
│       ├── log.module.ts
│       ├── logging.interceptor.ts  # 日志拦截器
│       ├── entities/sys-log.entity.ts
│       └── dto/{log-page,log-query,visit-stats,visit-trend}.dto.ts
│
├── codegen/                        # 代码生成器
│   ├── codegen.controller.ts
│   ├── codegen.service.ts
│   ├── codegen.module.ts
│   ├── dto/{codegen-preview,gen-config-form,table-query}.dto.ts
│   └── templates/{backend,frontend}/
│
├── file/                           # 文件模块
│   ├── file.controller.ts
│   ├── file.service.ts
│   └── file.module.ts
│
├── message/                        # 消息模块（SSE 推送）
│   ├── sse.controller.ts
│   ├── sse.service.ts
│   ├── sse-session-registry.service.ts
│   └── sse.module.ts
│
├── types/                          # 第三方类型声明
│   └── third-party.d.ts
│
├── app.module.ts                   # 根模块
└── main.ts                         # 入口
```

---

## Part 3: 命名规范

### 文件命名

| 类型   | 规范       | 示例                  |
| ------ | ---------- | --------------------- |
| 模块   | kebab-case | `user.module.ts`      |
| 控制器 | kebab-case | `user.controller.ts`  |
| 服务   | kebab-case | `user.service.ts`     |
| 实体   | kebab-case | `user.entity.ts`      |
| DTO    | kebab-case | `user-form.dto.ts`    |
| 接口   | kebab-case | `result.interface.ts` |

### 类命名

| 类型   | 规范              | 示例                                |
| ------ | ----------------- | ----------------------------------- |
| 控制器 | 实体 + Controller | `UserController`                    |
| 服务   | 实体 + Service    | `UserService`                       |
| 模块   | 实体 + Module     | `UserModule`                        |
| 实体   | PascalCase        | `User` 或 `SysUser`                 |
| DTO    | 功能 + DTO        | `UserVO`, `UserForm`, `UserPageDto` |

### 方法命名

| 动作     | 前缀          | 示例           |
| -------- | ------------- | -------------- |
| 查询单个 | get           | `getById()`    |
| 查询列表 | list          | `listUsers()`  |
| 分页查询 | page          | `pageUsers()`  |
| 新增     | create        | `createUser()` |
| 更新     | update        | `updateUser()` |
| 删除     | remove/delete | `removeById()` |

### 变量命名

| 类型     | 规范             | 示例                         |
| -------- | ---------------- | ---------------------------- |
| 变量     | camelCase        | `userList`                   |
| 常量     | UPPER_SNAKE_CASE | `MAX_SIZE`                   |
| 私有变量 | \_ 前缀          | `_internalState`             |
| 布尔值   | is/has/can 前缀  | `isDeleted`, `hasPermission` |

---

## Part 4: RESTful API 规范

### 标准 CRUD 路径

| 操作     | 方法   | 路径                    |
| -------- | ------ | ----------------------- |
| 分页列表 | GET    | `/api/v1/users/page`    |
| 详情     | GET    | `/api/v1/users/:id`     |
| 新增     | POST   | `/api/v1/users`         |
| 更新     | PUT    | `/api/v1/users`         |
| 删除     | DELETE | `/api/v1/users/:id`     |
| 批量删除 | DELETE | `/api/v1/users/batch`   |
| 下拉选项 | GET    | `/api/v1/users/options` |

### Controller 模板

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
  async create(@Body() form: UserForm) {
    return this.userService.createUser(form);
  }

  @Put()
  @ApiOperation({ summary: "更新用户" })
  @RequirePermission("sys:user:edit")
  async update(@Body() form: UserForm) {
    this.userService.updateUser(form);
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

## Part 5: 响应格式

### 统一响应

```typescript
// common/interfaces/result.interface.ts
export interface Result<T> {
  code: number;
  msg: string;
  data: T;
}

export interface PageResult<T> {
  code: number;
  msg: string;
  data: {
    list: T[];
    total: number;
  };
}

// common/interceptors/transform.interceptor.ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Result<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Result<T>> {
    return next.handle().pipe(
      map((data) => ({
        code: 200,
        msg: "操作成功",
        data,
      })),
    );
  }
}
```

### 分页查询参数

```typescript
// common/dto/page.dto.ts
export class PageDto {
  @ApiProperty({ description: "页码", default: 1 })
  @IsInt()
  @Min(1)
  @Type(() => Number)
  pageNum: number;

  @ApiProperty({ description: "每页数量", default: 20 })
  @IsInt()
  @Min(1)
  @Max(100)
  @Type(() => Number)
  pageSize: number;

  @ApiProperty({ description: "关键词", required: false })
  @IsString()
  @IsOptional()
  keywords?: string;
}
```

---

## Part 6: 实体规范

### 基础实体

```typescript
// common/entities/base.entity.ts
@Entity()
export abstract class BaseEntity {
  @PrimaryGeneratedColumn("increment", { type: "bigint" })
  id: string;

  @CreateDateColumn({ name: "create_time", comment: "创建时间" })
  createTime: Date;

  @UpdateDateColumn({ name: "update_time", comment: "更新时间" })
  updateTime: Date;

  @Column({ name: "create_by", type: "bigint", nullable: true, comment: "创建人" })
  createBy: string;

  @Column({ name: "update_by", type: "bigint", nullable: true, comment: "更新人" })
  updateBy: string;

  @Column({ type: "tinyint", default: 0, comment: "删除标志(0未删除 1已删除)" })
  deleted: number;
}
```

### 实体示例

```typescript
// modules/system/user/entities/user.entity.ts
@Entity("sys_user")
@ApiModel("用户实体")
export class SysUser extends BaseEntity {
  @Column({ length: 50, unique: true, comment: "用户名" })
  @ApiProperty({ description: "用户名" })
  username: string;

  @Column({ length: 100, select: false, comment: "密码" })
  password: string;

  @Column({ length: 50, comment: "昵称" })
  @ApiProperty({ description: "昵称" })
  nickname: string;

  @Column({ length: 20, nullable: true, comment: "手机号" })
  @ApiProperty({ description: "手机号" })
  mobile: string;

  @Column({ length: 100, nullable: true, comment: "邮箱" })
  @ApiProperty({ description: "邮箱" })
  email: string;

  @Column({ type: "tinyint", default: 1, comment: "状态(1正常 0禁用)" })
  @ApiProperty({ description: "状态" })
  status: number;

  @Column({ name: "dept_id", type: "bigint", nullable: true, comment: "部门ID" })
  @ApiProperty({ description: "部门ID" })
  deptId: string;
}
```

---

## Part 7: Service 规范

```typescript
// modules/system/user/user.service.ts
@Injectable()
export class UserService {
  constructor(
    @InjectRepository(SysUser)
    private userRepository: Repository<SysUser>,
  ) {}

  async pageUsers(query: UserPageDto): Promise<PageResult<UserVO>> {
    const { pageNum, pageSize, keywords, status } = query;
    const qb = this.userRepository.createQueryBuilder("user");

    if (keywords) {
      qb.andWhere("user.username LIKE :keywords OR user.nickname LIKE :keywords", {
        keywords: `%${keywords}%`,
      });
    }

    if (status !== undefined) {
      qb.andWhere("user.status = :status", { status });
    }

    qb.orderBy("user.createTime", "DESC");
    qb.skip((pageNum - 1) * pageSize).take(pageSize);

    const [list, total] = await qb.getManyAndCount();
    return {
      code: 200,
      msg: "操作成功",
      data: {
        list: list.map(this.toVO),
        total,
      },
    };
  }

  async getUserById(id: number): Promise<UserVO> {
    const user = await this.userRepository.findOne({ where: { id: String(id) } });
    if (!user) {
      throw new BusinessException("用户不存在");
    }
    return this.toVO(user);
  }

  async createUser(form: UserForm): Promise<number> {
    const exists = await this.userRepository.findOne({ where: { username: form.username } });
    if (exists) {
      throw new BusinessException("用户名已存在");
    }

    const user = this.userRepository.create({
      ...form,
      password: await bcrypt.hash(form.password, 10),
    });
    await this.userRepository.save(user);
    return Number(user.id);
  }

  private toVO(user: SysUser): UserVO {
    return {
      id: Number(user.id),
      username: user.username,
      nickname: user.nickname,
      mobile: user.mobile,
      email: user.email,
      status: user.status,
      createTime: user.createTime,
    };
  }
}
```

---

## Part 8: 认证规范

### JWT 策略

```typescript
// modules/auth/strategies/jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(configService: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get("jwt.secret"),
    });
  }

  async validate(payload: JwtPayload): Promise<CurrentUser> {
    return {
      userId: payload.sub,
      username: payload.username,
    };
  }
}
```

### 权限装饰器

```typescript
// common/decorators/permission.decorator.ts
export const RequirePermission = (...permissions: string[]) =>
  SetMetadata("permissions", permissions);

// common/guards/permission.guard.ts
@Injectable()
export class PermissionGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<string[]>("permissions", [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!required) {
      return true;
    }

    const { permissions } = context.switchToHttp().getRequest().user;
    return required.some((p) => permissions.includes(p));
  }
}
```

---

## Part 9: 异常处理

```typescript
// common/filters/http-exception.filter.ts
@Catch()
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();

    let code = 500;
    let msg = "系统异常";

    if (exception instanceof BusinessException) {
      code = exception.code;
      msg = exception.message;
    } else if (exception instanceof HttpException) {
      code = exception.getStatus();
      msg = exception.message;
    }

    response.status(200).json({
      code,
      msg,
      data: null,
    });
  }
}

// common/exceptions/business.exception.ts
export class BusinessException extends Error {
  constructor(
    public readonly message: string,
    public readonly code: number = 400,
  ) {
    super(message);
  }
}
```

---

## Part 10: 代码质量检查清单

- [ ] 遵循 RESTful API 路径规范
- [ ] 使用统一响应格式
- [ ] 实现全局异常过滤器
- [ ] 添加权限装饰器
- [ ] 分页接口使用 PageResult
- [ ] DTO 使用 class-validator 校验
- [ ] 实体继承 BaseEntity
- [ ] 使用 TypeORM 逻辑删除
- [ ] API 添加 @ApiOperation 注解
- [ ] 模块按功能划分
- [ ] 使用依赖注入
- [ ] 使用拦截器统一响应格式
