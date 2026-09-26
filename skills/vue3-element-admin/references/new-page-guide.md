# 新增 API 与页面

在新增或重构业务页面时读取本文。先检查同领域 API、相邻页面和后端契约；本文示例不是必须复制的脚手架。

## 选择页面模式

| 场景                                   | 推荐模式      | 现有参考                                                       |
| -------------------------------------- | ------------- | -------------------------------------------------------------- |
| 列表、表单、权限分配、拖拽等自定义交互 | Composables   | `src/views/system/role/index.vue`                              |
| 仪表盘、详情、设置                     | 简单页面      | `src/views/dashboard/index.vue`、`src/views/profile/index.vue` |
| 标准增删改查且明确采用配置驱动         | 配置驱动 CRUD | `src/views/demo/table/crud/`                                   |

维护既有页面时保留当前模式。只有新增页面且没有相邻模式可复用时，才默认选择 Composables。

## API 模块

业务 API 通常采用以下结构：

```text
src/api/system/role/
├── index.ts
└── types.ts
```

复用公共类型，不重复声明分页结构：

```typescript
import type { BaseQueryParams } from "@/api/common";

export interface RoleQueryParams extends BaseQueryParams {
  keywords?: string;
}

export interface RoleItem {
  id: string;
  code: string;
  name: string;
  status: number;
}

export interface RoleForm {
  id?: string;
  code: string;
  name: string;
  status: number;
}
```

字段是否可选由真实接口和表单生命周期决定，不要为了省事把所有字段都标成可选。

API 使用项目现有的对象字面量风格：

```typescript
import type { OptionItem, PageResult } from "@/api/common";
import request from "@/utils/request";

import type { RoleForm, RoleItem, RoleQueryParams } from "./types";

const ROLE_BASE_URL = "/api/v1/roles";

const RoleAPI = {
  /** 获取角色分页数据 */
  getPage(params: RoleQueryParams) {
    return request<unknown, PageResult<RoleItem>>({
      url: ROLE_BASE_URL,
      method: "get",
      params,
    });
  },

  /** 获取角色选项 */
  getOptions() {
    return request<unknown, OptionItem[]>({
      url: `${ROLE_BASE_URL}/options`,
      method: "get",
    });
  },

  /** 获取角色表单数据 */
  getFormData(id: string) {
    return request<unknown, RoleForm>({
      url: `${ROLE_BASE_URL}/${id}/form`,
      method: "get",
    });
  },

  /** 新增角色 */
  create(data: RoleForm) {
    return request({ url: ROLE_BASE_URL, method: "post", data });
  },

  /** 更新角色 */
  update(id: string, data: RoleForm) {
    return request({ url: `${ROLE_BASE_URL}/${id}`, method: "put", data });
  },

  /** 批量删除角色 */
  deleteByIds(ids: string) {
    return request({ url: `${ROLE_BASE_URL}/${ids}`, method: "delete" });
  },
};

export default RoleAPI;
export * from "./types";
```

遵守以下边界：

- `request` 已在业务成功时返回响应中的 `data`，调用端不要再访问 `.data`。
- 方法名和 URL 以现有后端契约为准；不要从示例臆造批量删除、导出或表单回显接口。
- 导出文件时复用相邻 API 的 `responseType`、下载和错误处理方式。
- 后端返回的标识类型可能是 `string` 或 `number`；串联、路由和批量删除前确认实际类型。

## Composables 列表页

`usePageTable` 只负责分页请求、查询参数和加载状态；`useTableSelection` 只负责选中 ID。弹窗、表单和业务副作用留在页面或领域 Composable 中。

```typescript
const queryFormRef = ref<FormInstance>();

const {
  loading,
  list,
  total,
  params,
  fetchData,
  handleQuery,
  handleResetQuery,
} = usePageTable<RoleItem, RoleQueryParams>({
  initialParams: { pageNum: 1, pageSize: 10, keywords: "" },
  request: RoleAPI.getPage,
  onBeforeReset: () => queryFormRef.value?.resetFields(),
});

const { selectedIds, hasSelection, handleSelectionChange } =
  useTableSelection<RoleItem>();

onMounted(handleQuery);
```

行为约定：

- `handleQuery()` 回到第一页后请求；搜索按钮和回车查询使用它。
- `fetchData()` 保留当前页；分页、刷新和保存后保持页码时使用它。
- `handleResetQuery()` 恢复初始参数并请求；表单字段需要同步重置时传入 `onBeforeReset`。
- `selectedIds` 的元素类型为 `string | number`，提交前按 API 契约转换。
- 弹窗打开前重置表单和校验状态；编辑数据异步返回后再显示或填充表单。
- `loading` 必须在 `finally` 中恢复。取消确认框不应显示成功消息或继续删除。

完整 SFC 遵循项目块顺序：

```vue
<template>
  <div class="page-container">
    <el-card class="page-search" shadow="never">...</el-card>
    <el-card class="page-content" shadow="never">...</el-card>
  </div>
</template>

<script setup lang="ts">
// 导入、状态和业务逻辑
</script>

<style scoped lang="scss">
/* 仅放页面专用的语义样式 */
</style>
```

## 简单页面

没有分页表格时直接组合 API、状态和组件：

- 为异步状态提供加载、空数据和失败反馈。
- 并行且相互独立的请求使用 `Promise.all`；存在依赖时保持顺序。
- 页面复杂后按业务区块或可复用逻辑拆分，不按固定行数拆分。
- 优先复用 `page-container` 和现有布局类；仅为页面专用视觉新增局部样式。

## 完成检查

- API 类型与实际响应一致，调用端没有重复剥壳。
- 搜索、分页、重置、保存和删除后的刷新行为符合产品预期。
- 表单初始化、回显、关闭和校验不会残留上次数据。
- 权限、路由、缓存和语言包已按任务需要同步。
- SFC 顺序、TypeScript 和修改文件的 lint 通过。
