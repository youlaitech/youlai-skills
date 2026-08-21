# 新增接口与页面

## 页面模式对比

默认使用 Composables 模式或简单页面。仅当明确指定使用 CURD 页面时，才参考 Config 驱动模式：

| 模式 | 适用场景 | 示例 | 参考文档 |
|------|----------|------|----------|
| Composables 模式（默认） | 列表页、表单页、带自定义交互的页面 | `views/system/role/` | 本文 |
| 简单页面 | 仪表盘、设置页、详情页 | `views/dashboard/`、`views/profile/` | 本文 |
| Config 驱动 CURD（需指定） | 标准增删改查，无特殊交互 | `views/demo/curd/` | [curd-development.md](curd-development.md) |

---

## 创建 API 模块

以角色模块为例。每个 API 模块遵循 `index.ts` + `types.ts` 结构。

### Step 1: 创建类型定义

```
src/api/system/role/
├── index.ts    # API 对象
└── types.ts    # 请求/响应类型
```

```typescript
// src/api/system/role/types.ts

import type { BaseQueryParams } from "@/api/common";

/** 角色分页查询参数 */
export interface RoleQueryParams extends BaseQueryParams {
  keywords?: string;
}

/** 角色列表项 */
export interface RoleItem {
  id?: string;
  code?: string;
  name?: string;
  sort?: number;
  status?: number;
  dataScope?: number;
  updateTime?: Date;
}

/** 角色表单对象 */
export interface RoleForm {
  id?: string;
  code?: string;
  name?: string;
  sort?: number;
  dataScope?: number;
  status?: number;
  remark?: string;
}
```

### Step 2: 创建 API 对象

```typescript
// src/api/system/role/index.ts

import request from "@/utils/request";
import type { RoleQueryParams, RoleItem, RoleForm } from "./types";
import type { OptionItem, PageResult } from "@/api/common";

const ROLE_BASE_URL = "/api/v1/roles";

const RoleAPI = {
  /** 获取角色分页数据 */
  getPage(queryParams?: RoleQueryParams) {
    return request<unknown, PageResult<RoleItem>>({
      url: ROLE_BASE_URL,
      method: "get",
      params: queryParams,
    });
  },

  /** 获取角色下拉数据源 */
  getOptions() {
    return request<unknown, OptionItem[]>({ url: `${ROLE_BASE_URL}/options`, method: "get" });
  },

  /** 获取角色表单数据 */
  getFormData(id: string) {
    return request<unknown, RoleForm>({ url: `${ROLE_BASE_URL}/${id}/form`, method: "get" });
  },

  /** 新增角色 */
  create(data: RoleForm) {
    return request({ url: ROLE_BASE_URL, method: "post", data });
  },

  /** 更新角色 */
  update(id: string, data: RoleForm) {
    return request({ url: `${ROLE_BASE_URL}/${id}`, method: "put", data });
  },

  /** 批量删除角色，多个以英文逗号分割 */
  deleteByIds(ids: string) {
    return request({ url: `${ROLE_BASE_URL}/${ids}`, method: "delete" });
  },
};

export default RoleAPI;
export * from "./types";
```

### API 方法命名约定

| 方法 | HTTP | 路径 | 说明 |
|------|------|------|------|
| `getPage` | GET | `/api/v1/roles` | 分页列表 |
| `getFormData` | GET | `/api/v1/roles/{id}/form` | 表单回显数据 |
| `getOptions` | GET | `/api/v1/roles/options` | 下拉选项 |
| `create` | POST | `/api/v1/roles` | 新增 |
| `update` | PUT | `/api/v1/roles/{id}` | 修改 |
| `deleteByIds` | DELETE | `/api/v1/roles/{ids}` | 删除（逗号分隔） |
| `export` | GET | `/api/v1/roles/export` | 导出（`responseType: "blob"`） |

自定义方法按业务语义命名，如 `getRoleMenuIds`、`updateRoleMenus`。

---

## 创建 CURD 页面（Composables 模式）

角色模块采用此模式：`usePageTable` 管理分页表格，`useTableSelection` 管理选择，弹窗逻辑内联。

### 页面结构

```
views/system/role/
└── index.vue    # 单文件包含搜索+表格+弹窗
```

### 核心模板

```vue
<script setup lang="ts">
import { ElMessage, ElMessageBox, type FormInstance, type FormRules } from "element-plus";
import RoleAPI from "@/api/system/role";
import type { RoleForm, RoleItem, RoleQueryParams } from "@/api/system/role";
import { usePageTable, useTableSelection } from "@/composables";
import { CommonStatus } from "@/enums";

defineOptions({ name: "Role", inheritAttrs: false });

// 分页表格
const { loading, list, total, params, fetchData, handleQuery, handleResetQuery } = usePageTable<
  RoleItem,
  RoleQueryParams
>({
  initialParams: { pageNum: 1, pageSize: 10, keywords: "" },
  request: RoleAPI.getPage,
});

// 表格选择
const { selectedIds, hasSelection, handleSelectionChange } = useTableSelection<RoleItem>();

// 弹窗状态
const dialogState = reactive({ title: "", visible: false });
const formData = reactive<RoleForm>({ sort: 1, status: CommonStatus.ENABLED });
const roleFormRef = ref<FormInstance>();

const rules: FormRules<RoleForm> = {
  name: [{ required: true, message: "请输入角色名称", trigger: "blur" }],
  code: [{ required: true, message: "请输入角色编码", trigger: "blur" }],
};

/** 打开新增角色弹窗 */
function handleCreateClick() {
  dialogState.title = "新增角色";
  dialogState.visible = true;
}

/**
 * 打开编辑角色弹窗
 *
 * @param roleId 角色 ID
 */
async function handleEditClick(roleId: string) {
  dialogState.title = "修改角色";
  const data = await RoleAPI.getFormData(roleId);
  Object.assign(formData, data);
  dialogState.visible = true;
}

/**
 * 校验并提交角色表单
 *
 * 非自定义数据权限时丢弃部门 ID
 */
async function handleSubmit() {
  const valid = await roleFormRef.value?.validate().then(() => true, () => false);
  if (!valid) return;

  loading.value = true;
  try {
    if (formData.id) {
      await RoleAPI.update(formData.id, formData);
      ElMessage.success("修改成功");
    } else {
      await RoleAPI.create(formData);
      ElMessage.success("新增成功");
    }
    dialogState.visible = false;
    handleResetQuery();
  } finally {
    loading.value = false;
  }
}

/**
 * 删除单个或批量角色
 *
 * @param roleId 指定时删除单个角色；不指定时删除表格勾选项
 */
async function handleDelete(roleId?: string) {
  const ids = roleId ?? selectedIds.value.join(",");
  if (!ids) return;

  await ElMessageBox.confirm("确认删除已选中的数据项?", "警告", { type: "warning" });
  loading.value = true;
  try {
    await RoleAPI.deleteByIds(ids);
    ElMessage.success("删除成功");
    handleResetQuery();
  } finally {
    loading.value = false;
  }
}

onMounted(() => handleQuery());
</script>

<template>
  <div class="page-container">
    <!-- 搜索区 -->
    <el-card class="page-search" shadow="never">
      <el-form :model="params" :inline="true">
        <el-form-item prop="keywords" label="关键字">
          <el-input v-model="params.keywords" placeholder="角色名称" clearable @keyup.enter="handleQuery" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="handleQuery">搜索</el-button>
          <el-button @click="handleResetQuery">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <!-- 表格区 -->
    <el-card class="page-content" shadow="never">
      <div class="page-toolbar">
        <div class="page-toolbar__left">
          <el-button type="primary" @click="handleCreateClick">新增</el-button>
          <el-button type="danger" :disabled="!hasSelection" @click="handleDelete()">删除</el-button>
        </div>
      </div>

      <el-table v-loading="loading" :data="list" border @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column label="角色名称" prop="name" min-width="100" />
        <el-table-column label="角色编码" prop="code" width="150" />
        <el-table-column label="状态" align="center" width="100">
          <template #default="{ row }">
            <el-tag :type="row.status === 1 ? 'success' : 'info'">
              {{ row.status === 1 ? "正常" : "禁用" }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" width="180" align="center">
          <template #default="{ row }">
            <el-button type="primary" size="small" link @click="handleEditClick(row.id)">编辑</el-button>
            <el-button type="danger" size="small" link @click="handleDelete(row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>

      <pagination
        v-if="total > 0"
        v-model:total="total"
        v-model:page="params.pageNum"
        v-model:limit="params.pageSize"
        @pagination="fetchData"
      />
    </el-card>

    <!-- 弹窗 -->
    <el-dialog v-model="dialogState.visible" :title="dialogState.title" width="600px">
      <el-form ref="roleFormRef" :model="formData" :rules="rules" label-width="100px">
        <el-form-item label="角色名称" prop="name">
          <el-input v-model="formData.name" placeholder="请输入角色名称" />
        </el-form-item>
        <el-form-item label="角色编码" prop="code">
          <el-input v-model="formData.code" placeholder="请输入角色编码" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button type="primary" @click="handleSubmit">确定</el-button>
        <el-button @click="dialogState.visible = false">取消</el-button>
      </template>
    </el-dialog>
  </div>
</template>
```

### usePageTable 参数

```typescript
const { loading, list, total, params, fetchData, handleQuery, handleResetQuery } = usePageTable<
  ItemType,
  QueryType
>({
  initialParams: { pageNum: 1, pageSize: 10, /* 其他初始查询参数 */ },
  request: XxxAPI.getPage,  // 传入 API 方法引用
  onBeforeReset: () => queryFormRef.value?.resetFields(),  // 可选：重置前回调
});
```

### useTableSelection

```typescript
const { selectedIds, hasSelection, handleSelectionChange } = useTableSelection<ItemType>();
// selectedIds: Ref<string[]>  选中的 ID 数组
// hasSelection: ComputedRef<boolean>  是否有选中项
// handleSelectionChange: (rows: ItemType[]) => void  绑定到 el-table @selection-change
```

---

## 创建简单页面（非 CURD）

仪表盘、设置页、详情页等无分页表格的页面，直接使用 API 调用 + 组件组装。

### 基本结构

```vue
<script setup lang="ts">
import LogAPI from "@/api/log";
import type { VisitTrendData } from "@/api/log";

defineOptions({ name: "Dashboard" });

const loading = ref(false);
const statsData = ref({
  onlineUsers: 0,
  todayVisits: 0,
  totalUsers: 0,
});

/** 获取访问统计数据 */
async function fetchStats() {
  loading.value = true;
  try {
    const data = await LogAPI.getVisitOverview();
    statsData.value = data;
  } finally {
    loading.value = false;
  }
}

onMounted(() => fetchStats());
</script>

<template>
  <div class="page-container">
    <el-row :gutter="16" v-loading="loading">
      <el-col :span="6">
        <el-card shadow="never">
          <div class="stat-card">
            <span class="stat-card__label">在线用户</span>
            <span class="stat-card__value">{{ statsData.onlineUsers }}</span>
          </div>
        </el-card>
      </el-col>
      <!-- 更多卡片... -->
    </el-row>
  </div>
</template>

<style scoped lang="scss">
.stat-card {
  display: flex;
  flex-direction: column;
  gap: 8px;

  &__label {
    font-size: 14px;
    color: var(--el-text-color-secondary);
  }

  &__value {
    font-size: 28px;
    font-weight: 600;
    color: var(--el-color-primary);
  }
}
</style>
```

### 页面布局约定

所有页面根节点使用 `class="page-container"`，内部按功能分区：

| CSS 类 | 用途 |
|--------|------|
| `page-container` | 页面根容器 |
| `page-search` | 搜索区卡片 |
| `page-content` | 内容区卡片 |
| `page-toolbar` | 工具栏（`__left` + `__right`） |
| `page-table-wrapper` | 表格包裹层（自适应高度） |
| `page-pagination` | 分页区 |

---

## 何时用哪种模式

默认使用 Composables 模式。仅当明确要求使用 CURD 页面时，才使用 Config 驱动模式：

| 场景 | 推荐模式 |
|------|----------|
| 列表页、表单页、带自定义交互（权限分配、拖拽排序） | Composables 模式（默认），单文件 |
| 仪表盘、数据展示 | 简单页面，API 调用 + 组件组装 |
| 设置页、个人中心 | 简单页面，表单 + API 调用 |
| 明确指定使用 CURD 页面的标准增删改查 | Config 驱动 CURD，配置拆分到 `config/` |

Composables 模式是项目实际业务页面采用的模式（角色、用户、部门等），灵活性高，适合大多数场景。Config 驱动模式开发效率高但灵活性受限，仅在明确指定时使用。
