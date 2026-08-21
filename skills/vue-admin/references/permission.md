# 权限控制

## v-hasPerm 指令

```vue
<!-- 单个权限 -->
<el-button v-hasPerm="'sys:user:create'">新增</el-button>

<!-- 多个权限（满足其一） -->
<el-button v-hasPerm="['sys:user:create', 'sys:user:update']">操作</el-button>

<!-- 超级管理员可见 -->
<el-button v-hasPerm="*:*:*">任意权限可见</el-button>
```

实现逻辑：`roles` 包含 `"ROOT"` → 超级管理员直接放行；权限标识包含 `"*:*:*"` → 直接放行；否则检查 `perms` 数组是否包含所需权限。无权限则移除 DOM 节点。

## hasPerm() 函数（JS 逻辑判断）

```typescript
import { hasPerm } from "@/utils/auth";

const canEdit = hasPerm("sys:user:update");
```

## 权限标识命名

格式：`模块:资源:操作`，如 `sys:user:create`、`sys:role:assign`。

| 操作 | 标识 |
|------|------|
| 列表 / 详情 | `list` / `view` |
| 新增 / 编辑 / 删除 | `create` / `update` / `delete` |
| 导入 / 导出 | `import` / `export` |
| 分配权限 | `assign` |

## CURD 中的权限自动化

CURD 配置的 `permPrefix` + 按钮内置 `perm` 自动拼接成完整权限标识，传给 `v-hasPerm` 控制按钮显隐。无需手动在每个按钮上加 `v-hasPerm`。

| 按钮 | perm | 拼接结果（permPrefix = `sys:user`） |
|---|---|---|
| 新增 | `create` | `sys:user:create` |
| 删除 | `delete` | `sys:user:delete` |
| 导入 | `import` | `sys:user:import` |
| 导出 | `export` | `sys:user:export` |
| 编辑 | `update` | `sys:user:update` |
| 查看 | `view` | `sys:user:view` |
| 刷新/筛选/搜索 | `*:*:*` | 不限权限 |
