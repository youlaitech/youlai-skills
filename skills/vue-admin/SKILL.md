---
name: vue-admin
description: Vue 3 + TypeScript development standards. Use this skill when building Vue 3 applications with Composition API, Pinia, UnoCSS and SCSS. Covers naming, CSS architecture, component structure, API patterns and code quality. UI-library agnostic (Element Plus, Ant Design Vue, etc).
---

# Vue 3 开发规范

## 技术栈

Vue 3 (Composition API) · TypeScript · Vite · Pinia · Vue Router · UnoCSS · SCSS

## 目录结构

```
src/
├── api/                        # API 请求层
│   ├── common.ts              # 公共类型（ApiResult, PageResult 等）
│   └── system/user/
│       ├── index.ts           # API 对象
│       └── types.ts           # 请求/响应类型
├── components/                 # 全局复用组件
├── composables/                # 组合式函数（use 前缀）
├── constants/                  # 常量
├── directives/                 # 自定义指令
├── enums/                      # 枚举
├── lang/                       # 国际化
├── layouts/                    # 布局组件
├── plugins/                    # 插件注册
├── router/                     # 路由 + 守卫
├── stores/                     # Pinia（扁平结构，无 modules/ 子目录）
├── styles/                     # 全局样式
├── utils/                      # 工具函数
├── views/                      # 页面（与路由对应）
│   └── system/user/
│       ├── index.vue
│       └── components/
└── settings.ts
```

## 命名规范

| 类型 | 风格 | 示例 | 理由 |
|------|------|------|------|
| 变量 | camelCase | `userName` | JS 惯例 |
| 常量 | UPPER_SNAKE_CASE | `MAX_COUNT` | 区分不可变值 |
| 函数 | camelCase，动词开头 | `fetchUserList` | 动词表行为 |
| 类/接口/类型 | PascalCase | `UserInfo` | 区分类型与实例 |
| 枚举 | PascalCase | `StatusEnum` | — |
| 枚举值 | UPPER_SNAKE_CASE | `StatusEnum.ACTIVE` | 区分枚举成员 |
| 布尔值 | is/has/can/should 前缀 | `isLoading` | 一眼识别为布尔判断，名词有歧义 |
| Vue 组件文件 | PascalCase | `UserForm.vue` | 与 HTML 原生元素区分 |
| 页面文件 | index.vue | `system/user/index.vue` | 路由目录入口 |
| TS/JS 模块 | kebab-case | `format-date.ts` | 文件系统大小写兼容 |
| Composables | use + camelCase | `usePageTable` | Vue 官方约定 |
| Store | use + 模块名 + Store | `useUserStore` | Pinia 约定 |

**禁止**：前端类型用 `VO/DTO` 后缀。前端无需区分数据传输层级，语义命名更直观：`UserItem`（列表项）、`UserForm`（表单）、`UserQueryParams`（查询参数）、`UserDetail`（详情）。

## 方法命名与 handle 规则

| 场景 | 命名 |
|------|------|
| 查询/加载 | `fetchList` / `loadOptions` |
| 打开/关闭弹窗 | `openDialog` / `closeDialog` |
| 提交/保存 | `submitForm` / `saveX` |
| 新增/编辑/删除 | `createX` / `updateX` / `deleteX` |
| 重置 | `resetForm` / `resetQuery` |
| 表单校验 | `validateForm` |
| 导入/导出 | `importX` / `exportX` |
| **事件入口（流程编排）** | `handleSubmit` / `handleDelete` / `handleEditClick` |

**handle 判断标准**：函数只做一件事 → 不用 handle（可复用）；组合多个动作 + 流程控制 → 用 handle（事件入口）。

```typescript
// ✅ 单一动作 → 不用 handle，可被多处复用
function openDialog() { dialogState.visible = true; }

// ✅ 流程编排 → 使用 handle，作为事件入口
async function handleSubmit() {
  const valid = await validateForm();
  if (!valid) return;
  await submitForm(formData);
  closeDialog();
  fetchList();
}
```

`fetch/load` 本身已隐含异步语义，不加 `Async` 后缀。单次调用的逻辑直接内联，不过度抽取。

## CSS / UnoCSS / SCSS 边界

**核心原则：UnoCSS 只处理无语义微调（间距、对齐），结构性样式归 BEM + SCSS。**

| 场景 | 方案 |
|------|------|
| 全局页面骨架 | 全局类（如 `page-*`），复用不重复造 |
| 有结构语义的元素 | BEM + SCSS |
| 无语义的布局微调 | UnoCSS（间距、flex 对齐等） |
| 穿透/动画/媒体查询 | SCSS（`:deep()`、`@keyframes`、`@media`） |

**BEM 元素上可附加少量无语义原子类（间距、对齐），但结构性样式（颜色、背景、圆角、阴影）必须收敛到 SCSS**：

```vue
<!-- ✅ 无 BEM 类 → 纯 UnoCSS -->
<div class="flex-y-center gap-10px">...</div>

<!-- ✅ BEM + 1-2 个无语义原子类（间距）可接受 -->
<div class="user-card mt-4">...</div>

<!-- ❌ BEM + 大量原子类（结构+语义混写） -->
<div class="user-card flex flex-col gap-4 p-4 bg-white rounded shadow">...</div>
```

- 同一元素原子类 ≤ 3 个，超过提取 BEM
- 颜色用 UI 库 CSS 变量或项目变量，禁止硬编码
- 状态用 `is-*`：`is-collapsed`、`is-loading`；变体用 BEM Modifier：`layout--top`、`todo-row--done`
- BEM 格式：`block__element--modifier`，kebab-case，Block 带页面前缀：`user-card`、`role-permission`

## 组件规范

- SFC 块顺序：`template` → `script setup` → `style scoped`
- script 内部顺序：导入（Vue → 第三方 → 类型 → 内部 → 相对路径）→ Props/Emits → 状态 → 计算属性 → 监听器 → 生命周期 → 方法 → defineExpose
- Props 优先 TypeScript 类型声明 + `withDefaults`，不与运行时声明混用
- 组件 ≤ 300 行，使用 `<script setup>`

## 类型与 API 约定

### 公共类型（api/common.ts）

```typescript
interface ApiResult<T = unknown> { code: string; data: T; msg: string; }
interface BaseQueryParams { pageNum: number; pageSize: number; sortBy?: string; order?: string; }
interface PageResult<T> { list: T[]; total: number; }
interface OptionItem { value: string | number; label: string; children?: OptionItem[]; }
```

### 类型命名

| 语义 | 命名 |
|------|------|
| 列表项 | `UserItem` |
| 表单对象 | `UserForm` |
| 查询参数 | `UserQueryParams extends BaseQueryParams` |
| 详情 | `UserDetail` |
| 登录用户信息 | `UserInfo` |

### API 定义

```typescript
const USER_BASE_URL = "/api/v1/users";

const UserAPI = {
  /** 获取用户分页列表。 */
  getPage(q: UserQueryParams) { return request<unknown, PageResult<UserItem>>({ url: USER_BASE_URL, method: "get", params: q }); },
  /** 获取表单详情。 */
  getFormData(id: string) { return request<unknown, UserForm>({ url: `${USER_BASE_URL}/${id}/form`, method: "get" }); },
  /** 新增。 */
  create(data: UserForm) { return request({ url: USER_BASE_URL, method: "post", data }); },
  /** 修改。 */
  update(id: string, data: UserForm) { return request({ url: `${USER_BASE_URL}/${id}`, method: "put", data }); },
  /** 删除。 */
  deleteByIds(ids: string) { return request({ url: `${USER_BASE_URL}/${ids}`, method: "delete" }); },
};

export default UserAPI;
export * from "./types";
```

| 操作 | 方法名 | HTTP |
|------|--------|------|
| 分页查询 | `getPage` | GET |
| 表单详情 | `getFormData` | GET |
| 新增 | `create` | POST |
| 修改 | `update` | PUT |
| 删除 | `deleteByIds` | DELETE |
| 导出/导入 | `export` / `import` | GET / POST |

## Store

Setup Store 写法，扁平目录（无 `modules/` 子目录）。组件外使用 `useXxxStoreHook()`。

```typescript
export const useUserStore = defineStore("user", () => {
  const userInfo = ref<UserInfo>({} as UserInfo);
  const isLoggedIn = computed(() => !!userInfo.value.username);
  async function login(payload: LoginRequest) { /* ... */ }
  return { userInfo, isLoggedIn, login };
});

// 组件外调用（路由守卫等）
export function useUserStoreHook() {
  return useUserStore(store);
}
```

## Composables

`use` 前缀 + camelCase。参数用 options 对象，返回响应式引用和方法，不需要外部修改的状态用 `readonly()` 包裹。

## 注释

注释只写有信息量的内容：补充代码未直接表达的背景、意图、约束和边界。不复述代码、不做视觉分隔。

| 场景 | 写法 | 说明 |
|------|------|------|
| 类型/属性/常量 | 单行 `/** ... */` | 说明用途、约束 |
| 函数/方法 | 多行 JSDoc | 用途、参数、返回值 |
| 业务规则/兼容策略 | 行内或块级 | 解释为什么这样做 |
| 大型配置分组 | 简短普通注释 | `// 认证`、`// UI` |
| 函数内部普通步骤 | 不写 | 通过命名和空行表达 |
| 纯视觉分隔标题 | 禁止 | 不用横线/等号包裹 |

**JSDoc 规则**：函数多行（至少 3 行），带 `@param`/`@returns` 必须多行。摘要句号结尾。不需要说明就不写。

```typescript
// ✅ 分组
// 认证
ACCESS_TOKEN: `${APP_PREFIX}:auth:access_token`,

// ✅ 解释边界
const keys = Object.keys(localStorage).filter(k => k.startsWith(prefix)); // 只清理当前应用写入的缓存

/**
 * 搜索仅匹配菜单标题，避免路径命中过多造成结果噪音。
 */
function searchByTitle() {}

/**
 * 格式化文件大小。
 *
 * @param bytes - 字节数。
 * @param decimals - 小数位数，默认 2。
 * @returns 格式化后的字符串，如 "1.50 MB"。
 */
export function formatFileSize(bytes: number, decimals = 2): string { /* ... */ }

// ❌ 复述代码
const index = list.findIndex(item => item.id === id); // 查找索引
```

行内注释解释“为什么”而非“是什么”，靠近相关代码。

## 反模式速查

| 反模式 | 正确做法 |
|--------|---------|
| 类型用 `VO/DTO` 后缀 | 语义命名：`UserItem`、`UserForm` |
| 硬编码颜色 `#409eff` | UI 库 CSS 变量或项目变量 |
| 原子类 > 3 个 | 提取 BEM 类 + SCSS |
| BEM + 大量原子类混写 | BEM 负责结构，原子类仅做无语义微调 |
| `:deep()` 滥用 | 优先用组件属性/插槽配置 |
| `height: 100vh` | `min-height: 100vh` |
| z-index 魔法数字 | CSS 变量分层管理 |
| `class UserAPI` 静态方法 | `const UserAPI = {}` 对象字面量 |
| `stores/modules/` 子目录 | 扁平 `stores/` 结构 |
| 单次调用过度抽取方法 | 直接内联 |
| 复述代码的注释 | 只写有信息量的注释 |
| `handle` 用于单一动作 | handle 仅用于流程编排 |

## 自查清单

- [ ] 类型无 `VO/DTO` 后缀
- [ ] 布尔值有 `is/has/can/should` 前缀
- [ ] `handle` 仅用于流程编排
- [ ] API 用 `const XXXAPI = {}` 对象字面量
- [ ] BEM 带页面/功能前缀
- [ ] BEM 元素上原子类仅做无语义微调，结构性样式收敛到 SCSS
- [ ] 颜色用 CSS 变量，不硬编码
- [ ] SFC 块顺序：template → script → style
- [ ] Store 用 Setup Store + `useXxxStoreHook()`
- [ ] 公共函数有 JSDoc，无复述性注释
- [ ] 组件 ≤ 300 行，使用 `<script setup>`
