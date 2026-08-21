# CURD 页面开发

> 此模式仅在明确指定使用 CURD 页面时参考。默认页面开发请参考 [new-page-guide.md](new-page-guide.md)。

项目采用配置驱动型 CURD 架构，由 `PageSearch` + `PageContent` + `PageModal` 三组件 + `usePage()` composable 组成。开发一个 CURD 页面只需写配置文件 + 组装组件。

## 目录结构

```
views/system/user/
├── index.vue          # 页面入口，组装三大组件
├── config/
│   ├── search.ts      # 搜索区配置 (ISearchConfig)
│   ├── content.ts     # 表格区配置 (IContentConfig)
│   ├── add.ts         # 新增弹窗配置 (IModalConfig)
│   ├── edit.ts        # 编辑弹窗配置 (IModalConfig)
│   └── options.ts     # 下拉选项初始化（可选）
└── components/        # 页面私有组件（如自定义列模板）
```

## index.vue 模板

```vue
<script setup lang="ts">
import { usePage } from "@/components/CURD/usePage";
import SearchConfig from "./config/search";
import ContentConfig from "./config/content";
import AddConfig from "./config/add";
import EditConfig from "./config/edit";
import UserAPI from "@/api/system/user";

const { searchRef, contentRef, addModalRef, editModalRef,
        handleQueryClick, handleResetClick, handleAddClick,
        handleEditClick, handleExportClick } = usePage();

/** 编辑时异步获取详情 */
const handleEdit = (row: any) => {
  handleEditClick(row, (data: any) => UserAPI.getFormData(row.id), editModalRef.value);
};
</script>

<template>
  <div class="page-container">
    <PageSearch ref="searchRef" :search-config="SearchConfig" @query-click="handleQueryClick" @reset-click="handleResetClick" />
    <PageContent ref="contentRef" :content-config="ContentConfig"
      @add-click="handleAddClick" @edit-click="handleEdit" @export-click="handleExportClick">
      <!-- 自定义列插槽 -->
      <template #status="{ row }">
        <el-tag :type="row.status === 1 ? 'success' : 'danger'">
          {{ row.status === 1 ? "启用" : "禁用" }}
        </el-tag>
      </template>
    </PageContent>
    <PageModal ref="addModalRef" :modal-config="AddConfig" />
    <PageModal ref="editModalRef" :modal-config="EditConfig" />
  </div>
</template>
```

## 搜索区配置（ISearchConfig）

```typescript
export default {
  permPrefix: "sys:user",
  formItems: [
    { type: "input", label: "关键词", prop: "keywords", attrs: { placeholder: "用户名/手机号", clearable: true } },
    { type: "select", label: "状态", prop: "status", attrs: { placeholder: "全部", clearable: true },
      options: [{ label: "启用", value: 1 }, { label: "禁用", value: 0 }] },
  ],
  showNumber: 3,        // 默认展示 3 项，超出折叠
  isExpandable: true,   // 支持展开/收起
} as ISearchConfig;
```

## 表格区配置（IContentConfig）

```typescript
export default {
  permPrefix: "sys:user",
  pk: "id",
  indexAction: (params) => UserAPI.getPage(params),    // 列表请求（必填）
  deleteAction: (ids) => UserAPI.deleteByIds(ids),     // 删除
  exportAction: (params) => UserAPI.export(params),     // 后端导出
  toolbar: ["add", "delete", "import", "export"],       // 左侧工具栏
  defaultToolbar: ["refresh", "filter", "search"],      // 右侧工具栏
  cols: [
    { type: "selection", width: 50 },
    { label: "用户名", prop: "username", width: 120 },
    { label: "状态", prop: "status", width: 100, templet: "custom", slotName: "status" },
    { label: "创建时间", prop: "createTime", width: 180, templet: "date" },
    { label: "操作", fixed: "right", width: 180, templet: "tool", operat: ["edit", "delete"] },
  ],
} as IContentConfig;
```

## 弹窗配置（IModalConfig）

```typescript
export default {
  permPrefix: "sys:user",
  component: "dialog",                  // "dialog" | "drawer"
  dialog: { title: "新增用户", width: "600px" },
  form: { labelWidth: 100 },
  formAction: (data) => UserAPI.create(data),
  beforeSubmit: (data) => {
    // 提交前数据处理
    return data;
  },
  formItems: [
    { type: "input", label: "用户名", prop: "username", rules: [{ required: true, message: "请输入用户名" }] },
    { type: "input", label: "密码", prop: "password", attrs: { type: "password", showPassword: true } },
    { type: "select", label: "角色", prop: "roleIds", attrs: { multiple: true },
      options: [], initFn: async (item) => { item.options = await RoleAPI.getOptions(); } },
  ],
} as IModalConfig;
```

## 内置列模板（templet）

| templet | 用途 | 关键参数 |
|---|---|---|
| `image` | 图片预览 | `imageWidth`, `imageHeight` |
| `list` | 字典映射显示 | `selectList` |
| `url` | 超链接 | — |
| `switch` | 开关（行内编辑） | `activeValue`, `inactiveValue` |
| `input` | 输入框（行内编辑） | `inputType` |
| `price` | 价格格式化 | `priceFormat` |
| `date` | 日期格式化 | `dateFormat`（默认 `YYYY-MM-DD HH:mm:ss`） |
| `icon` | 图标 | — |
| `tool` | 操作栏按钮 | `operat: ["edit", "delete"]` |
| `custom` | 自定义插槽 | `slotName` |

## 表单项类型

搜索区支持：`input`、`select`、`input-number`、`cascader`、`tree-select`、`date-picker`、`time-picker`、`time-select`、`input-tag`、`custom-tag`、`custom`

弹窗区额外支持：`radio`、`checkbox`、`switch`、`text`、`icon-select`

## 权限前缀机制

三个配置都有 `permPrefix`（如 `"sys:user"`），工具栏按钮内置 `perm` 映射：

| 按钮 | perm 拼接结果 |
|---|---|
| 新增 | `sys:user:create` |
| 删除 | `sys:user:delete` |
| 导入 | `sys:user:import` |
| 导出 | `sys:user:export` |
| 编辑 | `sys:user:update` |
| 查看 | `sys:user:view` |
| 刷新/筛选/搜索 | `*:*:*`（不限权限） |

`permPrefix` 未设置时不做权限校验。

## usePage() 返回方法

| 方法 | 用途 |
|---|---|
| `handleQueryClick` | 触发查询（合并 filter 参数） |
| `handleResetClick` | 重置搜索条件 |
| `handleAddClick` | 打开新增弹窗 |
| `handleEditClick(row, callback, ref?)` | 打开编辑弹窗，callback 用于异步获取详情 |
| `handleViewClick(row, callback, ref?)` | 打开查看弹窗（禁用表单） |
| `handleExportClick` | 导出数据 |
| `handleSearchClick` | 右侧搜索按钮 |
| `handleFilterChange` | 筛选条件变化 |
