# 常用组件使用

## Upload 文件上传

三个上传组件，统一通过 `FileAPI.upload(formData)` 上传到 `/api/v1/files`：

```vue
<!-- 单图上传 -->
<SingleImageUpload v-model="avatarUrl" :max-file-size="2" accept="image/*" />

<!-- 多图上传 -->
<MultiImageUpload v-model="imageUrls" :limit="5" :max-file-size="5" />

<!-- 文件上传 -->
<FileUpload v-model="fileList" :limit="3" :max-file-size="10" />
```

| 组件 | v-model 类型 | 说明 |
|---|---|---|
| `SingleImageUpload` | `string` | 图片 URL |
| `MultiImageUpload` | `string[]` | URL 数组 |
| `FileUpload` | `FileInfo[]` | `{ name, url }` 数组 |

公共 Props：`data`（额外参数）、`name`（字段名默认 `"file"`）、`maxFileSize`（MB）、`accept`、`limit`。

## Dict 字典组件

```vue
<!-- 字典下拉 -->
<DictSelect v-model="status" code="sys_normal_disable" type="select" />

<!-- 字典标签（只读展示） -->
<DictTag :model-value="row.status" code="sys_normal_disable" />
```

| 组件 | Props | 说明 |
|---|---|---|
| `DictSelect` | `code`（必填）、`type`（`"select"` / `"radio"` / `"checkbox"`） | 按需加载并缓存 |
| `DictTag` | `code`、`modelValue`、`size` | 根据值渲染 `el-tag` |

字典数据通过 `useDictStore` 统一缓存到 localStorage（key: `vea:system:dict_cache`），内置请求去重队列防止并发重复请求。

## TableSelect 表格选择器

Popover + 表格 + 分页的选择器，支持单选/多选和跨页选择：

```vue
<TableSelect
  v-model="selectedIds"
  :select-config="{
    indexAction: (params) => UserAPI.getPage(params),
    pk: 'id',
    formItems: [{ type: 'input', label: '用户名', prop: 'username' }],
    tableColumns: [
      { label: '用户名', prop: 'username' },
      { label: '昵称', prop: 'nickname' },
    ],
  }"
  text="选择用户"
  :multiple="true"
/>
```

## IconSelect 图标选择器

```vue
<IconSelect v-model="menuIcon" />
```
