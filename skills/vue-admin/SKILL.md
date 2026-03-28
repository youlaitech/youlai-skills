---
name: vue-admin
description: Vue3 管理后台开发规范。当开发 Vue3 + Element Plus 管理后台、CRUD 页面、权限管理、自定义组件时使用此 skill。
---

# Vue3 管理后台开发规范

## 触发条件

- 开发 Vue3 管理后台项目
- 实现 CRUD 页面
- 使用 Element Plus 组件
- 实现权限管理
- 创建自定义组件

---

## Part 1: 技术栈

- **Vue** - 前端框架
- **Element Plus** - UI 组件库
- **TypeScript** - 类型安全
- **Vite** - 构建工具
- **Pinia** - 状态管理
- **Vue Router** - 路由
- **Axios** - HTTP 客户端

---

## Part 2: 目录结构

```
src/
├── api/                    # API 请求
│   ├── system/             # 系统模块 API
│   │   ├── user/           # 用户模块
│   │   │   ├── index.ts    # API 定义
│   │   │   └── types.ts    # 类型定义
│   │   └── role/           # 角色模块
│   └── index.ts            # API 统一导出
├── components/             # 全局组件
│   ├── Form/               # 表单组件
│   │   ├── UserSelect.vue
│   │   └── DeptTree.vue
│   ├── Table/              # 表格组件
│   │   └── ProTable.vue
│   └── Dialog/             # 弹窗组件
│       └── FormDialog.vue
├── composables/            # 组合式函数
│   ├── useTable.ts         # 表格逻辑
│   ├── useForm.ts          # 表单逻辑
│   └── useDialog.ts        # 弹窗逻辑
├── directives/             # 自定义指令
│   ├── permission.ts       # 权限指令
│   └── loading.ts          # 加载指令
├── enums/                  # 枚举定义
│   ├── ResultCode.ts       # 响应码
│   └── MenuType.ts         # 菜单类型
├── hooks/                  # 自定义 hooks
│   ├── useLoading.ts
│   └── usePermission.ts
├── layout/                 # 布局组件
│   ├── components/
│   │   ├── Sidebar.vue
│   │   ├── Navbar.vue
│   │   └── AppMain.vue
│   └── index.vue
├── router/                 # 路由配置
│   ├── index.ts
│   ├── staticRoutes.ts     # 静态路由
│   └── dynamicRoutes.ts    # 动态路由
├── stores/                 # Pinia 状态
│   ├── modules/
│   │   ├── user.ts
│   │   ├── permission.ts
│   │   └── app.ts
│   └── index.ts
├── styles/                 # 全局样式
│   ├── variables.scss      # CSS 变量
│   ├── reset.scss          # 重置样式
│   └── index.scss          # 入口
├── utils/                  # 工具函数
│   ├── request.ts          # Axios 封装
│   ├── auth.ts             # Token 管理
│   ├── permission.ts       # 权限判断
│   └── format.ts           # 格式化
└── views/                  # 页面组件
    ├── system/
    │   ├── user/
    │   │   ├── index.vue       # 列表页
    │   │   ├── components/     # 页面组件
    │   │   │   └── UserForm.vue
    │   │   └── hooks/          # 页面 hooks
    │   │       └── useUser.ts
    │   └── role/
    └── dashboard/
```

---

## Part 3: 命名规范

### 文件命名

| 类型       | 规范             | 示例               |
| ---------- | ---------------- | ------------------ |
| 组件       | PascalCase       | `UserSelect.vue`   |
| 页面       | kebab-case       | `user-profile.vue` |
| 组合式函数 | camelCase + use  | `useUserStore.ts`  |
| 工具函数   | camelCase        | `formatDate.ts`    |
| 常量       | UPPER_SNAKE_CASE | `API_PREFIX.ts`    |
| 类型       | PascalCase       | `UserVO.ts`        |

### 变量命名

| 类型     | 规范             | 示例                         |
| -------- | ---------------- | ---------------------------- |
| 变量     | camelCase        | `userList`                   |
| 常量     | UPPER_SNAKE_CASE | `MAX_COUNT`                  |
| 私有变量 | \_ 前缀          | `_internalState`             |
| 布尔值   | is/has/can 前缀  | `isLoading`, `hasPermission` |

### 方法命名

| 动作      | 前缀        | 示例            |
| --------- | ----------- | --------------- |
| 获取/加载 | fetch/load  | `fetchUserList` |
| 提交/保存 | submit/save | `submitForm`    |
| 新增      | create      | `createUser`    |
| 更新      | update      | `updateUser`    |
| 删除      | delete      | `deleteUser`    |
| 打开/关闭 | open/close  | `openDialog`    |
| 事件处理  | handle      | `handleSubmit`  |

**handle 使用规则：**

| 类型     | 特点         | 是否用 handle |
| -------- | ------------ | ------------- |
| 单一动作 | 只做一件事   | 否            |
| 流程编排 | 组合多个动作 | 是            |

```typescript
// ✅ 单一动作 → 不用 handle
function openDialog() {
  dialogState.visible = true;
}

// ✅ 流程编排 → 使用 handle
async function handleSubmit() {
  const { valid } = await formRef.value.validate();
  if (!valid) return;
  await submitForm();
  closeDialog();
}
```

### CSS 命名（BEM）

```
Block__Element--Modifier
```

```scss
.user-card {
  padding: 16px;
  background: var(--el-bg-color);
  border-radius: 8px;

  &__header {
    display: flex;
    align-items: center;
  }

  &__avatar {
    width: 48px;
    height: 48px;
    border-radius: 50%;
  }

  &__name {
    font-size: 16px;
    font-weight: 600;
  }

  &--active {
    border: 2px solid var(--el-color-primary);
  }
}
```

---

## Part 4: 组件规范

### 组件优先原则

| 需求   | 使用          | 避免            |
| ------ | ------------- | --------------- |
| 按钮   | `<el-button>` | 自定义 `.btn`   |
| 输入框 | `<el-input>`  | 自定义 `.input` |
| 表格   | `<el-table>`  | 自定义表格      |
| 弹窗   | `<el-dialog>` | 自定义弹窗      |
| 表单   | `<el-form>`   | 自定义表单      |

### CRUD 页面模板

```vue
<template>
  <div class="app-container">
    <!-- 搜索栏 -->
    <el-form :model="queryParams" ref="queryFormRef" :inline="true">
      <el-form-item label="关键词" prop="keywords">
        <el-input
          v-model="queryParams.keywords"
          placeholder="请输入"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleQuery"> <i-ep-search />搜索 </el-button>
        <el-button @click="handleReset"> <i-ep-refresh />重置 </el-button>
      </el-form-item>
    </el-form>

    <!-- 操作栏 -->
    <el-row :gutter="10" class="mb-[10px]">
      <el-col :span="1.5">
        <el-button type="primary" v-permission="['sys:user:add']" @click="handleAdd">
          <i-ep-plus />新增
        </el-button>
      </el-col>
    </el-row>

    <!-- 数据表格 -->
    <el-table v-loading="loading" :data="tableData" border>
      <el-table-column label="ID" prop="id" width="80" />
      <el-table-column label="用户名" prop="username" min-width="120" />
      <el-table-column label="状态" prop="status" width="100">
        <template #default="{ row }">
          <el-tag :type="row.status === 1 ? 'success' : 'danger'">
            {{ row.status === 1 ? "正常" : "禁用" }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="150" fixed="right">
        <template #default="{ row }">
          <el-button link type="primary" v-permission="['sys:user:edit']" @click="handleEdit(row)">
            编辑
          </el-button>
          <el-button
            link
            type="danger"
            v-permission="['sys:user:delete']"
            @click="handleDelete(row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-pagination
      v-model:current-page="queryParams.pageNum"
      v-model:page-size="queryParams.pageSize"
      :total="total"
      :page-sizes="[10, 20, 50, 100]"
      layout="total, sizes, prev, pager, next, jumper"
      @change="handleQuery"
    />
  </div>
</template>

<script setup lang="ts">
import type { UserVO, UserPageQuery } from "@/api/system/user/types";

defineOptions({ name: "UserList" });

const loading = ref(false);
const tableData = ref<UserVO[]>([]);
const total = ref(0);
const queryParams = reactive<UserPageQuery>({
  pageNum: 1,
  pageSize: 20,
  keywords: "",
});

function handleQuery() {
  loading.value = true;
  UserAPI.getPage(queryParams)
    .then(({ data }) => {
      tableData.value = data.list;
      total.value = data.total;
    })
    .finally(() => {
      loading.value = false;
    });
}

function handleReset() {
  queryFormRef.value?.resetFields();
  handleQuery();
}

function handleAdd() {
  // 打开新增弹窗
}

function handleEdit(row: UserVO) {
  // 打开编辑弹窗
}

async function handleDelete(row: UserVO) {
  const { confirm } = await ElMessageBox.confirm("确认删除该用户？", "警告", {
    type: "warning",
  });
  if (confirm) {
    await UserAPI.deleteById(row.id);
    ElMessage.success("删除成功");
    handleQuery();
  }
}

onMounted(() => handleQuery());
</script>
```

---

## Part 5: API 规范

### API 定义模板

```typescript
// api/system/user/types.ts
export interface UserVO {
  id: number;
  username: string;
  nickname: string;
  mobile: string;
  email: string;
  status: number;
  createTime: string;
}

export interface UserForm {
  id?: number;
  username: string;
  nickname: string;
  mobile: string;
  email: string;
  status: number;
  roleIds: number[];
}

export interface UserPageQuery extends PageQuery {
  username?: string;
  status?: number;
  deptId?: number;
}

// api/system/user/index.ts
import request from "@/utils/request";
import type { UserVO, UserForm, UserPageQuery } from "./types";

class UserAPI {
  /** 获取用户分页列表 */
  static getPage(params: UserPageQuery) {
    return request<PageResult<UserVO>>({
      url: "/api/v1/users/page",
      method: "get",
      params,
    });
  }

  /** 获取用户详情 */
  static getById(id: number) {
    return request<Result<UserVO>>({
      url: `/api/v1/users/${id}`,
      method: "get",
    });
  }

  /** 获取用户表单数据 */
  static getFormData(id: number) {
    return request<Result<UserForm>>({
      url: `/api/v1/users/${id}/form`,
      method: "get",
    });
  }

  /** 新增用户 */
  static create(data: UserForm) {
    return request<Result<number>>({
      url: "/api/v1/users",
      method: "post",
      data,
    });
  }

  /** 更新用户 */
  static update(data: UserForm) {
    return request<Result<void>>({
      url: "/api/v1/users",
      method: "put",
      data,
    });
  }

  /** 删除用户 */
  static deleteById(id: number) {
    return request<Result<void>>({
      url: `/api/v1/users/${id}`,
      method: "delete",
    });
  }

  /** 批量删除用户 */
  static batchDelete(ids: number[]) {
    return request<Result<void>>({
      url: "/api/v1/users/batch",
      method: "delete",
      data: { ids },
    });
  }

  /** 获取用户选项列表 */
  static getOptions() {
    return request<Result<OptionType[]>>({
      url: "/api/v1/users/options",
      method: "get",
    });
  }
}

export default UserAPI;
```

### 响应格式

```typescript
// 标准响应
interface Result<T> {
  code: number;
  msg: string;
  data: T;
}

// 分页响应
interface PageResult<T> {
  code: number;
  msg: string;
  data: {
    list: T[];
    total: number;
  };
}

// 分页查询参数
interface PageQuery {
  pageNum: number;
  pageSize: number;
  keywords?: string;
}

// 选项类型
interface OptionType {
  value: string | number;
  label: string;
  children?: OptionType[];
}
```

---

## Part 6: 权限控制

### 路由守卫

```typescript
// router/guard.ts
import { useUserStore } from "@/stores/modules/user";

router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore();

  // 登录页直接放行
  if (to.path === "/login") {
    return next();
  }

  // 未登录跳转登录页
  if (!userStore.token) {
    return next(`/login?redirect=${to.path}`);
  }

  // 已加载路由直接放行
  if (userStore.routes.length > 0) {
    return next();
  }

  // 加载动态路由
  await userStore.loadRoutes();
  next({ ...to, replace: true });
});
```

### 按钮权限

```vue
<template>
  <!-- 指令方式 -->
  <el-button v-permission="['sys:user:add']" type="primary">新增</el-button>

  <!-- 函数方式 -->
  <el-button v-if="hasPermission(['sys:user:edit'])">编辑</el-button>
</template>

<script setup lang="ts">
import { hasPermission } from "@/utils/permission";
</script>
```

### 权限指令

```typescript
// directives/permission.ts
import { useUserStore } from "@/stores/modules/user";

export const permission: Directive = {
  mounted(el, binding) {
    const userStore = useUserStore();
    const permissions = binding.value as string[];

    const hasPermission = permissions.some((p) => userStore.permissions.includes(p));

    if (!hasPermission) {
      el.parentNode?.removeChild(el);
    }
  },
};
```

---

## Part 7: 代码质量检查清单

- [ ] 使用 TypeScript 严格模式
- [ ] 为所有变量和函数定义类型
- [ ] 优先使用 Element Plus 组件
- [ ] 遵循命名规范
- [ ] 为公共函数添加 JSDoc 注释
- [ ] 处理加载和错误状态
- [ ] 实现权限检查
- [ ] 使用组合式函数复用逻辑
- [ ] 组件保持在 300 行以内
- [ ] 使用 `<script setup>` 语法
- [ ] API 按模块组织
- [ ] 使用 BEM 命名 CSS
