# 配置驱动 CRUD

组件目录为 `src/components/Crud`，由 `CrudSearch`（查询表单）、`CrudTable`（列表）、`CrudModal`（新增/编辑弹窗）三个组件与 `useCrudPage` 组合逻辑构成。

仅在用户明确选择配置驱动模式，或维护已有配置驱动页面时读取本文。存在复杂交互、大量自定义状态或领域流程时优先使用普通页面和 Composables。

## 先读真实实现

实现前检查：

- `src/components/Crud/types.ts`
- `src/components/Crud/useCrudPage.ts`
- `src/components/Crud/CrudSearch.vue`
- `src/components/Crud/CrudTable.vue`
- `src/components/Crud/CrudModal.vue`
- `src/views/demo/table/crud/`

这些文件是类型、事件名和默认行为的事实来源。文档示例与当前类型冲突时，以代码为准。

## 目录结构

```text
views/system/user/
├── index.vue
├── config/
│   ├── search.ts
│   ├── content.ts
│   ├── add.ts
│   ├── edit.ts
│   └── options.ts
└── components/
```

只创建任务真正需要的配置和私有组件，不机械复制全部文件。

## 页面组装

`useCrudPage` 是默认导出。页面通过组件事件调用它返回的编排方法：

```vue
<template>
  <div class="page-container">
    <crud-search
      ref="searchRef"
      :search-config="searchConfig"
      @query-click="handleQueryClick"
      @reset-click="handleResetClick"
    />

    <crud-table
      ref="contentRef"
      :content-config="contentConfig"
      @add-click="handleAddClick"
      @operate-click="handleOperateClick"
      @filter-change="handleFilterChange"
    />

    <crud-modal
      ref="addModalRef"
      :modal-config="addModalConfig"
      @submit-click="handleSubmitClick"
    />
    <crud-modal
      ref="editModalRef"
      :modal-config="editModalConfig"
      @submit-click="handleSubmitClick"
    />
  </div>
</template>

<script setup lang="ts">
import type { IOperateData } from "@/components/Crud/types";
import useCrudPage from "@/components/Crud/useCrudPage";

import addModalConfig from "./config/add";
import contentConfig from "./config/content";
import editModalConfig from "./config/edit";
import searchConfig from "./config/search";

const {
  searchRef,
  contentRef,
  addModalRef,
  editModalRef,
  handleQueryClick,
  handleResetClick,
  handleAddClick,
  handleEditClick,
  handleSubmitClick,
  handleFilterChange,
} = useCrudPage();

function handleOperateClick(data: IOperateData) {
  if (data.name === "edit") {
    handleEditClick(data.row);
  }
}
</script>
```

不要改写成不存在的 `@edit-click` 事件，也不要把 `useCrudPage` 当作命名导入。
需要通过回调异步加载表单详情时，先检查 `useCrudPage.ts` 的当前回调签名并保持类型一致，不要用 `any` 或断言掩盖返回类型错误。

## 搜索配置

```typescript
import type { ISearchConfig } from "@/components/Crud/types";

const searchConfig: ISearchConfig = {
  permPrefix: "sys:user",
  formItems: [
    {
      type: "input",
      label: "关键词",
      prop: "keywords",
      attrs: { placeholder: "用户名/手机号", clearable: true },
    },
    {
      type: "select",
      label: "状态",
      prop: "status",
      attrs: { clearable: true },
      options: [
        { label: "启用", value: 1 },
        { label: "禁用", value: 0 },
      ],
    },
  ],
};

export default searchConfig;
```

异步选项需要复用时集中初始化；只在单个配置项使用时可通过其 `initFn` 加载。避免页面打开后重复请求同一组选项。

## 内容配置

使用泛型绑定查询和行数据类型：

```typescript
import type { UserItem, UserQueryParams } from "@/api/system/user";
import UserAPI from "@/api/system/user";
import type { ICrudTableConfig } from "@/components/Crud/types";

const contentConfig: ICrudTableConfig<UserQueryParams, UserItem> = {
  permPrefix: "sys:user",
  pk: "id",
  indexAction: UserAPI.getPage,
  deleteAction: UserAPI.deleteByIds,
  toolbar: ["add", "delete", "import", "export"],
  defaultToolbar: ["refresh", "filter", "search"],
  cols: [
    { type: "selection", width: 50, align: "center" },
    { label: "用户名", prop: "username" },
    { label: "状态", prop: "status", templet: "custom", slotName: "status" },
    {
      label: "操作",
      fixed: "right",
      templet: "tool",
      operat: ["edit", "delete"],
    },
  ],
};

export default contentConfig;
```

常用 `templet` 包括 `image`、`list`、`url`、`switch`、`input`、`price`、`percent`、`icon`、`date`、`tool`、`custom`。使用前仍需检查当前 `ICrudTableConfig` 联合类型。

## 弹窗配置

```typescript
import UserAPI from "@/api/system/user";
import type { UserForm } from "@/api/system/user";
import type { IModalConfig } from "@/components/Crud/types";

const addModalConfig: IModalConfig<UserForm> = {
  permPrefix: "sys:user",
  dialog: { title: "新增用户", width: 600 },
  formAction: UserAPI.create,
  formItems: [
    {
      type: "input",
      label: "用户名",
      prop: "username",
      rules: [{ required: true, message: "请输入用户名", trigger: "blur" }],
    },
  ],
};

export default addModalConfig;
```

- `beforeSubmit` 直接修改传入表单数据；当前实现不使用其返回值。
- 编辑配置的 `formAction` 通常需要从表单数据中取得主键，再调用更新 API。
- 只有配置会被异步修改时才用 `reactive` 包裹；纯静态配置直接导出。

## 权限和验证

`permPrefix` 与内置操作的权限拼接规则（含自定义按钮）见 [permission.md](permission.md)。

完成前确认：

- 配置中的方法、事件和模板名存在于当前 CRUD 类型与组件中。
- `indexAction` 返回 `PageResult<T>` 或组件明确支持的数组。
- 主键类型、批量删除格式和权限字符串与真实 API 一致。
- 自定义插槽名同时存在于配置和模板中。
- 新代码没有为了迁就旧框架内部的 `any` 而扩散无类型数据。
