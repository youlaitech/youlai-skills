---
name: vue3-element-admin
description: Vue3 Element Plus admin dashboard development guidelines. Use this skill when working on vue3-element-admin project, building admin dashboards, implementing CRUD pages, permission management, or Element Plus components.
---

# Vue3 Element Admin - Web Frontend Development Guide

Development guidelines for Vue3 + Element Plus + TypeScript admin dashboard system.

## Trigger Conditions

Use this skill when:
- Working on `vue3-element-admin` project
- Building admin dashboard pages
- Implementing CRUD operations with Element Plus
- Setting up permission management
- Creating custom components

---

## Part 1: Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Vue | 3.4+ | Frontend framework |
| Element Plus | 2.5+ | UI component library |
| TypeScript | 5.0+ | Type safety |
| Vite | 5.0+ | Build tool |
| Pinia | 2.1+ | State management |
| Vue Router | 4.2+ | Routing |
| Axios | 1.6+ | HTTP client |

---

## Part 2: Project Structure

```
src/
├── api/              # API requests
├── components/       # Global components
│   ├── Form/         # Form components
│   ├── Table/        # Table components
│   └── Dialog/      # Dialog components
├── composables/      # Composition functions
├── directives/       # Custom directives
├── enums/            # Enum definitions
├── hooks/            # Custom hooks
├── layout/           # Layout components
├── router/           # Route configuration
├── stores/           # Pinia stores
├── styles/           # Global styles
├── utils/            # Utility functions
└── views/            # Page components
```

---

## Part 3: Naming Conventions

### File Naming

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `UserSelect.vue` |
| Views | kebab-case | `user-profile.vue` |
| Composables | camelCase + use | `useUserStore.ts` |
| Utils | camelCase | `formatDate.ts` |
| Constants | UPPER_SNAKE_CASE | `API_PREFIX.ts` |

### Variable Naming

| Type | Convention | Example |
|------|------------|---------|
| Variables | camelCase | `userList` |
| Constants | UPPER_SNAKE_CASE | `MAX_COUNT` |
| Private variables | _prefix | `_internalState` |
| Boolean | is/has/can prefix | `isLoading`, `hasPermission` |

### Function Naming

| Action | Prefix | Example |
|--------|--------|---------|
| Fetch/Load | fetch/load | `fetchUserList` |
| Submit/Save | submit/save | `submitForm` |
| Create/Update/Delete | create/update/delete | `createUser` |
| Open/Close | open/close | `openDialog` |
| Event handlers | handle | `handleSubmit` |

---

## Part 4: Component Standards

### CRUD Page Template

```vue
<template>
  <div class="app-container">
    <!-- Search Bar -->
    <el-form :model="queryParams" ref="queryFormRef" :inline="true">
      <el-form-item label="关键词" prop="keywords">
        <el-input v-model="queryParams.keywords" placeholder="请输入" clearable />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleQuery">
          <i-ep-search />搜索
        </el-button>
        <el-button @click="handleReset">
          <i-ep-refresh />重置
        </el-button>
      </el-form-item>
    </el-form>

    <!-- Action Bar -->
    <el-row :gutter="10" class="mb-[10px]">
      <el-col :span="1.5">
        <el-button type="primary" @click="handleAdd">
          <i-ep-plus />新增
        </el-button>
      </el-col>
    </el-row>

    <!-- Data Table -->
    <el-table v-loading="loading" :data="tableData" border>
      <el-table-column label="ID" prop="id" width="80" />
      <el-table-column label="名称" prop="name" min-width="120" />
      <el-table-column label="操作" width="150" fixed="right">
        <template #default="{ row }">
          <el-button link type="primary" @click="handleEdit(row)">编辑</el-button>
          <el-button link type="danger" @click="handleDelete(row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- Pagination -->
    <el-pagination
      v-model:current-page="queryParams.pageNum"
      v-model:page-size="queryParams.pageSize"
      :total="total"
      @change="handleQuery"
    />
  </div>
</template>

<script setup lang="ts">
defineOptions({ name: "UserList" });

const loading = ref(false);
const tableData = ref<UserVO[]>([]);
const total = ref(0);
const queryParams = reactive<PageQuery>({
  pageNum: 1,
  pageSize: 20,
  keywords: "",
});

function handleQuery() {
  loading.value = true;
  UserAPI.getPage(queryParams).then(({ data }) => {
    tableData.value = data.list;
    total.value = data.total;
  }).finally(() => {
    loading.value = false;
  });
}

function handleAdd() {
  // Open add dialog
}

function handleEdit(row: UserVO) {
  // Open edit dialog
}

async function handleDelete(row: UserVO) {
  const { confirm } = await ElMessageBox.confirm("确认删除?", "警告");
  if (confirm) {
    await UserAPI.deleteById(row.id);
    ElMessage.success("删除成功");
    handleQuery();
  }
}

onMounted(() => handleQuery());
</script>
```

### Custom Component Guidelines

**Use Element Plus components first:**

| Need | Use | Don't |
|------|-----|-------|
| Button | `<el-button>` | Custom `.btn` |
| Input | `<el-input>` | Custom `.input` |
| Table | `<el-table>` | Custom table |
| Dialog | `<el-dialog>` | Custom modal |
| Form | `<el-form>` | Custom form |

**When to create custom components:**
- Repeated patterns across pages (e.g., `<UserSelect />`)
- Complex logic that needs encapsulation
- Element Plus doesn't provide the component

---

## Part 5: API Standards

### Request Format

```typescript
// api/user/index.ts
import request from "@/utils/request";

export interface UserVO {
  id: number;
  name: string;
  email: string;
}

export interface UserPageQuery extends PageQuery {
  name?: string;
  status?: number;
}

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
}

export default UserAPI;
```

### Response Format

```typescript
// Standard response
interface Result<T> {
  code: number;
  msg: string;
  data: T;
}

// Paginated response
interface PageResult<T> {
  code: number;
  msg: string;
  data: {
    list: T[];
    total: number;
  };
}
```

---

## Part 6: Permission Control

### Route Permission

```typescript
// router/guard.ts
router.beforeEach(async (to, from, next) => {
  const userStore = useUserStore();
  
  if (to.path === "/login") {
    return next();
  }
  
  if (!userStore.token) {
    return next("/login");
  }
  
  // Load dynamic routes
  if (!userStore.routes.length) {
    await userStore.loadRoutes();
    return next({ ...to, replace: true });
  }
  
  next();
});
```

### Button Permission

```vue
<template>
  <!-- v-permission directive -->
  <el-button v-permission="['sys:user:add']" type="primary">新增</el-button>
  
  <!-- v-if with permission check -->
  <el-button v-if="hasPermission(['sys:user:edit'])">编辑</el-button>
</template>

<script setup>
import { hasPermission } from "@/utils/permission";
</script>
```

---

## Part 7: Code Quality Checklist

- [ ] Use TypeScript strict mode
- [ ] Define proper types for all variables and functions
- [ ] Use Element Plus components instead of custom ones
- [ ] Follow naming conventions
- [ ] Add JSDoc comments for public functions
- [ ] Handle loading and error states
- [ ] Implement permission checks
- [ ] Use composables for reusable logic
- [ ] Keep components under 300 lines
- [ ] Use `<script setup>` syntax
