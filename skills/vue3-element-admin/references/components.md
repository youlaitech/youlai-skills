# 常用项目组件

组件 Props 和事件以 `src/components/` 中的当前实现为准。使用前先搜索现有调用，复用已验证的配置和交互方式。

## Upload

```vue
<SingleImageUpload v-model="avatarUrl" :max-file-size="2" accept="image/*" />
<MultiImageUpload v-model="imageUrls" :limit="5" :max-file-size="5" />
<FileUpload v-model="fileList" :limit="3" :max-file-size="10" />
```

| 组件                | `v-model`    | 主要约束                   |
| ------------------- | ------------ | -------------------------- |
| `SingleImageUpload` | `string`     | 单图 URL，无 `limit` Prop  |
| `MultiImageUpload`  | `string[]`   | 多图 URL，支持 `limit`     |
| `FileUpload`        | `FileInfo[]` | 文件信息列表，支持 `limit` |

三个组件均支持 `data`、`name`、`maxFileSize` 和 `accept`，并复用 `FileAPI.upload()`。不要绕过组件自行拼装重复的上传、进度或校验逻辑，除非需求超出当前组件能力。

## DictSelect 与 DictTag

```vue
<DictSelect v-model="status" code="sys_normal_disable" type="select" />
<DictTag :model-value="row.status" code="sys_normal_disable" />
```

- `DictSelect` 支持 `select`、`radio`、`checkbox`，值类型为字符串、数字或其数组。
- `DictTag` 用于只读展示，并根据字典项渲染标签文字和类型。
- 字典数据由 `useDictStore` 按需加载和缓存；不要在每个页面重复请求同一字典。

## TableSelect

`TableSelect` 不使用 `v-model`。选择结果通过 `confirm-click` 事件返回，`multiple` 位于 `selectConfig` 内：

```vue
<TableSelect
  :text="selectedText"
  :select-config="selectConfig"
  @confirm-click="handleConfirm"
/>
```

```typescript
import type { UserItem } from "@/api/system/user";
import UserAPI from "@/api/system/user";

const selectConfig = {
  indexAction: UserAPI.getPage,
  pk: "id",
  multiple: true,
  formItems: [{ type: "input", label: "用户名", prop: "username" }],
  tableColumns: [
    { label: "用户名", prop: "username" },
    { label: "昵称", prop: "nickname" },
  ],
};

function handleConfirm(selection: UserItem[]) {
  // 按业务需要保存 ID、行数据或展示文本
}
```

跨页多选依赖稳定且唯一的 `pk`。展示文字由调用方通过 `text` 提供。

## IconSelect

```vue
<IconSelect v-model="menuIcon" />
```

图标值使用项目现有命名体系。需要限制来源、清空或回显时先检查当前 Props 和已有菜单页面用法。
