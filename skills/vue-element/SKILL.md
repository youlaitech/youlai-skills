---
name: vue-element
description: Vue3 + Element Plus admin development standards. Use this skill when developing Vue3 admin dashboards, CRUD pages, permission management, or custom components with Element Plus.
---

# Vue3 管理后台开发规范

## 触发条件

- Develop Vue3 admin dashboard projects
- Implement CRUD pages
- Use Element Plus components
- Implement permission management
- Create custom components

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
block__element--modifier
 │      │          │
 │      │          └── 状态/变体（双连字符）
 │      └── 组成部分（双下划线）
 └── 页面前缀 + 功能实体
```

**Block 必须带页面前缀**：

```text
用户管理(user)    → user-card / user-form / user-toolbar
角色管理(role)    → role-card / role-permission / role-dialog
设备授权(device)  → device-auth / device-auth__panel
仪表盘(dashboard) → dashboard-stat / dashboard-chart
```

**Element 通用词汇**：

| 语义 | 标准词汇 | 禁止 |
|------|----------|------|
| 头部 | `__header` | ~~head~~ / ~~top~~ |
| 底部 | `__footer` | ~~bottom~~ |
| 标题 | `__title` | ~~name~~ / ~~heading~~ |
| 描述 | `__desc` | ~~text~~ / ~~detail~~ |
| 图标 | `__icon` | ~~img~~ / ~~pic~~ |
| 操作按钮 | `__action` | ~~action-btn~~ |
| 操作区 | `__actions` | ~~btns~~ |
| 内容 | `__body` | ~~content~~ |
| 列表 | `__list` | ~~items~~ |
| 列表项 | `__item` | ~~row~~ |
| 面板 | `__panel` | ~~box~~ |
| 标签 | `__tag` | ~~badge~~（徽章时用 badge） |

**SCSS 嵌套**：

```scss
.user-card {
  padding: 16px;
  background: var(--el-bg-color);
  border-radius: 8px;

  &__header {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  &__avatar {
    width: 48px;
    height: 48px;
    border-radius: 50%;
  }

  &__name {
    font-size: 16px;
    font-weight: 600;
    color: var(--el-text-color-primary);
  }

  &__actions {
    display: flex;
    gap: 8px;
  }

  &--active {
    border: 2px solid var(--el-color-primary);
  }

  &--disabled {
    opacity: 0.5;
    pointer-events: none;
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

## Part 8: 样式规范

### 样式分层原则

**BEM 定义语义，UnoCSS 补充微调，SCSS 只做脏活。**

| 层级 | 职责 | 示例 |
|------|------|------|
| **BEM** | 组件语义锚点 | `user-card`、`user-card__header` |
| **UnoCSS** | 简单布局微调（≤3 个属性） | `flex items-center gap-2` |
| **SCSS** | 伪类/动画/穿透等原子类做不了的事 | `:deep()`、`@keyframes` |

### 选型决策

| 场景 | 方案 | 示例 |
|------|------|------|
| 简单布局（1-2 个属性） | 原子类 | `class="flex items-center"` |
| 组件样式（3+ 属性） | BEM | `class="user-card"` |
| 重复出现的组合 | UnoCSS Shortcuts 收敛 | `class="flex-center"` |
| 主题相关 | CSS 变量 + BEM | `background: var(--el-color-primary-light-9)` |

```vue
<!-- ✅ 组件样式归 BEM，微调用原子类 -->
<div class="user-card">
  <el-avatar class="user-card__avatar" :src="user.avatar" />
  <span class="user-card__name">{{ user.name }}</span>
</div>

<!-- ✅ 简单间距直接用原子类 -->
<div class="mt-4 p-2">
  <el-input v-model="keyword" placeholder="搜索" />
</div>

<!-- ❌ 同一元素混用过多原子类 + BEM -->
<div class="user-card flex flex-col p-4 bg-white rounded-lg shadow-sm">
</div>
```

### 主题变量

**优先使用 Element Plus 内置 CSS 变量**，无需自定义主题色体系：

| 类别 | 变量 | 说明 |
|------|------|------|
| 主色 | `--el-color-primary` | 主题蓝 |
| 主色浅 | `--el-color-primary-light-3/5/7/8/9` | Element Plus 自动生成 |
| 成功 | `--el-color-success` / `--el-color-success-light-*` | 绿色系 |
| 警告 | `--el-color-warning` / `--el-color-warning-light-*` | 橙色系 |
| 危险 | `--el-color-danger` / `--el-color-danger-light-*` | 红色系 |
| 错误 | `--el-color-error` / `--el-color-error-light-*` | 红色系 |
| 背景色 | `--el-bg-color` / `--el-bg-color-overlay` / `--el-bg-color-page` | 页面/浮层/底色 |
| 文字色 | `--el-text-color-primary` / `--el-text-color-regular` / `--el-text-color-secondary` / `--el-text-color-placeholder` / `--el-text-color-disabled` | 层级递减 |
| 边框色 | `--el-border-color` / `--el-border-color-light` / `--el-border-color-lighter` / `--el-border-color-extra-light` | 层级递减 |
| 填充色 | `--el-fill-color` / `--el-fill-color-light` / `--el-fill-color-lighter` / `--el-fill-color-blank` | 背景填充 |

**自定义变量**（项目级，定义在 `_theme.scss`）：

| 变量 | 用途 |
|------|------|
| `--menu-background` | 侧边栏背景色 |
| `--menu-text` | 菜单文字色 |
| `--menu-active-text` | 菜单激活文字色 |
| `--menu-hover` | 菜单 hover 背景色 |

```scss
// ✅ 使用 Element Plus 变量
.card { background: var(--el-bg-color); border: 1px solid var(--el-border-color-lighter); }

// ❌ 禁止硬编码
.card { background: #ffffff; border: 1px solid #e5e7eb; }
```

### 布局类（全局）

项目在 `_layout.scss` 中定义了标准布局类，**CRUD 页面必须使用**：

| 类名 | 作用 | CSS |
|------|------|-----|
| `.page-container` | 页面容器 | `padding: 16px` |
| `.page-content` | 内容区（el-card 根元素） | `display: flex; flex: 1; flex-direction: column` |
| `.page-search` | 搜索栏 | 卡片样式 + 内边距覆盖 |
| `.page-toolbar` | 工具栏 | `flex + space-between + gap` |
| `.page-toolbar__left` | 工具栏左侧 | `flex + gap` |
| `.page-toolbar__right` | 工具栏右侧 | `flex + gap` |

**⚠️ 不要移除 `.page-content`**：该类提供 `display: flex; flex: 1; flex-direction: column`，删除会导致表格高度塌陷。

### z-index 层级

只有 `position: fixed/sticky/absolute` 且存在层叠竞争的元素才设 z-index，禁止魔法数字。

```scss
--z-dropdown: 100;   // 下拉菜单、Popover
--z-sticky:   200;   // 吸顶栏
--z-overlay:  300;   // 遮罩层
--z-dialog:   500;   // Dialog / Drawer
--z-toast:    600;   // Message / Notification
```

### 间距与圆角

**间距**（4px 网格基数）：

| 语义 | 值 | 场景 |
|------|-----|------|
| xs | `4px` | 图标与文字间距 |
| sm | `8px` | 按钮间距、紧凑内边距 |
| md | `16px` | 卡片内边距、模块间距 |
| lg | `24px` | 区块间距 |

**圆角**：

| 场景 | 值 |
|------|-----|
| 小元素（Tag、Badge） | `4px` |
| 按钮、输入框 | `var(--el-border-radius-base)` |
| 卡片、Dialog | `var(--el-border-radius-base)` |

### 阴影

| 级别 | 值 | 场景 |
|------|-----|------|
| sm | `0 1px 2px rgba(0,0,0,0.05)` | 卡片 |
| md | `0 4px 6px rgba(0,0,0,0.1)` | 弹窗 |
| lg | `0 10px 15px rgba(0,0,0,0.1)` | 模态框 |

### 反模式

| 反模式 | 正确做法 |
|--------|---------|
| 硬编码颜色值 | 使用 `var(--el-*)` 变量 |
| 移除 `.page-content` 类 | 保留，它提供关键 flex 布局 |
| `:deep()` 滥用 | 优先使用 Element Plus 属性/插槽 |
| Block 名过于通用（如 `.card`） | 使用页面/功能前缀（如 `.user-card`） |
| `height: 100vh` | 使用 `min-height: 100vh` |
| z-index 魔法数字 | 使用 `var(--z-*)` |
| SCSS 大量 `@apply` | 原子类留在模板 |
| 同一元素原子类 >3 个 | 归入 BEM 类 |
| `:deep(.el-card__body)` 加 flex | el-card__body 保持默认 block 布局 |

### 样式自查清单

- [ ] BEM 类名带页面前缀
- [ ] 颜色全部使用 CSS 变量
- [ ] `.page-content` 保留（提供 flex 布局）
- [ ] 模板中原子类 ≤ 3 个，超过则归入 BEM
- [ ] z-index 使用 `var(--z-*)`，普通流内容未设 z-index
- [ ] 不对 `.el-card__body` 加 flex 覆盖

---

## Part 9: 代码质量检查清单

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
- [ ] BEM 命名带页面前缀
- [ ] 颜色使用 `var(--el-*)` 变量
- [ ] 保留 `.page-content` 布局类
- [ ] 原子类 ≤3 个，超过用 BEM
