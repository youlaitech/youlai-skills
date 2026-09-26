# 权限控制

权限控制用于界面可见性，不替代后端鉴权。权限标识和角色名称以服务端契约为准。

## `v-hasPerm`

```vue
<!-- 单个权限 -->
<el-button v-hasPerm="'sys:user:create'">新增</el-button>

<!-- 多个权限，满足任意一个即可 -->
<el-button v-hasPerm="['sys:user:create', 'sys:user:update']">操作</el-button>

<!-- 通配权限必须作为字符串表达式传入 -->
<el-button v-hasPerm="'*:*:*'">不限权限</el-button>
```

不要写 `v-hasPerm="*:*:*"`；这不是合法的 Vue 表达式。

指令复用 `@/utils/auth` 的 `hasPerm()`：

- 用户角色包含 `ROOT` 时直接放行。
- 所需权限包含 `*:*:*` 时直接放行。
- 数组权限采用“满足任意一个”的语义。
- 无权限时元素会从 DOM 移除。

脚本逻辑直接调用相同实现：

```typescript
import { hasPerm } from "@/utils/auth";

const canEdit = hasPerm("sys:user:update");
```

角色可见性使用已有 `v-hasRole`，不要用角色名冒充权限字符串。

## 权限命名

沿用后端的 `模块:资源:操作` 格式，例如：

- `sys:user:list`
- `sys:user:create`
- `sys:user:update`
- `sys:user:delete`
- `sys:role:assign`

不要仅为追求统一而重命名现有权限；它们是前后端契约。

## 配置驱动 CRUD

设置 `permPrefix: "sys:user"` 后，`CrudTable` 会把内置按钮操作拼接为完整权限：

| 按钮               | 操作                           | 结果                 |
| ------------------ | ------------------------------ | -------------------- |
| 新增 / 编辑 / 删除 | `create` / `update` / `delete` | `sys:user:create` 等 |
| 导入 / 导出        | `import` / `export`            | `sys:user:import` 等 |
| 搜索               | `list`                         | `sys:user:list`      |
| 刷新 / 筛选        | `*:*:*`                        | 不限制               |

自定义按钮的 `perm` 可以填写操作名或完整权限。没有 `permPrefix` 时，操作名不会自动产生权限限制；实现前检查 `CrudTable.vue` 的当前拼接逻辑。
