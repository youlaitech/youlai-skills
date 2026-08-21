---
name: vue-admin
description: This skill should be used when developing Vue 3 applications with Composition API, creating admin pages with usePageTable composables, configuring routes/menus/permissions/page caching, implementing API request layers with axios, or using Element Plus components (Upload, Dict, TableSelect) in the vue3-element-admin project.
---

# Vue 3 开发规范

## 技术栈

Vue 3 (Composition API) · TypeScript · Vite · Pinia · Vue Router · UnoCSS · SCSS · Element Plus

## 目录结构

```
src/
├── api/                        # API 请求层（index.ts + types.ts）
├── components/                 # 全局复用组件
├── composables/                # 组合式函数（usePageTable 等）
├── constants/                  # 常量（StorageKeys 等）
├── directives/                 # 自定义指令（v-hasPerm 等）
├── lang/                       # 国际化（index.ts + package/）
├── layouts/                    # 布局组件
├── router/                     # 路由 + 守卫
├── stores/                     # Pinia（扁平结构，无 modules/）
├── styles/                     # 全局样式
├── utils/                      # 工具函数（request.ts, auth.ts）
├── views/                      # 页面（与路由对应）
│   └── system/role/
│       └── index.vue          # 单文件：搜索+表格+弹窗
└── settings.ts                # 项目配置
```

## 命名规范

| 类型 | 风格 | 示例 |
|------|------|------|
| 变量 | camelCase | `userName` |
| 常量 | UPPER_SNAKE_CASE | `MAX_COUNT` |
| 函数 | camelCase，动词开头 | `fetchUserList` |
| 类/接口/类型 | PascalCase | `UserInfo` |
| 枚举 | PascalCase | `StatusEnum` |
| 枚举值 | UPPER_SNAKE_CASE | `StatusEnum.ACTIVE` |
| 布尔值 | is/has/can/should 前缀 | `isLoading` |
| Vue 组件文件 | PascalCase | `UserForm.vue` |
| 页面文件 | index.vue | `system/user/index.vue` |
| TS/JS 模块 | kebab-case | `format-date.ts` |
| Composables | use + camelCase | `usePageTable` |
| Store | use + 模块名 + Store | `useUserStore` |

禁止前端类型用 `VO/DTO` 后缀。语义命名：`UserItem`（列表项）、`UserForm`（表单）、`UserQueryParams`（查询参数）、`UserDetail`（详情）。

## 方法命名与 handle 规则

| 场景 | 命名 |
|------|------|
| 查询/加载 | `fetchList` / `loadOptions` |
| 打开/关闭弹窗 | `openDialog` / `closeDialog` |
| 提交/保存 | `submitForm` / `saveX` |
| 新增/编辑/删除 | `createX` / `updateX` / `deleteX` |
| 重置 | `resetForm` / `resetQuery` |
| 事件入口（流程编排） | `handleSubmit` / `handleEditClick` |

`handle` 判断标准：函数只做一件事 → 不用 handle（可复用）；组合多个动作 + 流程控制 → 用 handle（事件入口）。

```typescript
// 单一动作 → 不用 handle
function openDialog() { dialogState.visible = true; }

// 流程编排 → 使用 handle
async function handleSubmit() {
  const valid = await validateForm();
  if (!valid) return;
  await submitForm(formData);
  closeDialog();
  fetchList();
}
```

`fetch/load` 已隐含异步语义，不加 `Async` 后缀。

## CSS / UnoCSS / SCSS 边界

UnoCSS 只处理无语义微调（间距、对齐），结构性样式归 BEM + SCSS。

| 场景 | 方案 |
|------|------|
| 全局页面骨架 | 全局类（如 `page-*`） |
| 有结构语义的元素 | BEM + SCSS |
| 无语义布局微调 | UnoCSS |
| 穿透/动画/媒体查询 | SCSS |

规则：
- 同一元素原子类 ≤ 3 个，超过提取 BEM
- 颜色用 CSS 变量，禁止硬编码
- BEM 格式：`block__element--modifier`，kebab-case，带页面前缀
- 状态用 `is-*`，变体用 BEM Modifier：`layout--top`

## 组件规范

- SFC 块顺序：`template` → `script setup` → `style scoped`
- script 内部顺序：导入 → Props/Emits → 状态 → 计算属性 → 监听器 → 生命周期 → 方法 → defineExpose
- Props 优先 TypeScript 类型声明 + `withDefaults`
- 组件 ≤ 300 行

## 类型与 API 约定

### 公共类型

```typescript
interface ApiResult<T = unknown> { code: string; data: T; msg: string; }
interface BaseQueryParams { pageNum: number; pageSize: number; sortBy?: string; order?: string; }
interface PageResult<T> { list: T[]; total: number; }
```

### API 定义

```typescript
const USER_BASE_URL = "/api/v1/users";

const UserAPI = {
  /** 获取用户分页数据 */
  getPage(q: UserQueryParams) { return request<unknown, PageResult<UserItem>>({ url: USER_BASE_URL, method: "get", params: q }); },
  /** 获取用户表单数据 */
  getFormData(id: string) { return request<unknown, UserForm>({ url: `${USER_BASE_URL}/${id}/form`, method: "get" }); },
  /** 新增用户 */
  create(data: UserForm) { return request({ url: USER_BASE_URL, method: "post", data }); },
  /** 更新用户 */
  update(id: string, data: UserForm) { return request({ url: `${USER_BASE_URL}/${id}`, method: "put", data }); },
  /** 批量删除用户 */
  deleteByIds(ids: string) { return request({ url: `${USER_BASE_URL}/${ids}`, method: "delete" }); },
};

export default UserAPI;
export * from "./types";
```

响应拦截器自动剥壳：`code === "00000"` 时返回 `response.data.data`。Token 过期（`A0230`）时 `refreshTokenOnce()` 单飞刷新。

## 页面开发模式

默认使用 Composables 模式开发页面。仅当明确指定使用 CURD 页面时，才使用 Config 驱动模式：

| 模式 | 适用场景 | 参考 |
|------|----------|------|
| Composables 模式（默认） | 列表页、表单页、带自定义交互的页面 | `references/new-page-guide.md` |
| 简单页面 | 仪表盘、设置页、详情页 | `references/new-page-guide.md` |
| Config 驱动 CURD（需指定） | 标准增删改查，无特殊交互 | `references/curd-development.md` |

页面根节点统一用 `class="page-container"`，内部分区：`page-search`（搜索区）、`page-content`（内容区）、`page-toolbar`（工具栏）。

## Store

Setup Store 写法，扁平目录。组件外使用 `useXxxStoreHook()` 避免 Pinia 未初始化。

```typescript
export const useUserStore = defineStore("user", () => {
  const userInfo = ref<UserInfo>({} as UserInfo);
  return { userInfo };
});

export function useUserStoreHook() {
  return useUserStore(store);
}
```

## Composables

`use` 前缀 + camelCase。参数用 options 对象，返回响应式引用和方法。

## 注释

注释写"为什么"和"踩坑点"，不写代码已经在说的。函数用 `/** */`，不用 `//`。

**格式**：
- 函数/方法 → `/** */` JSDoc
- 类型/接口/属性 → 单行 `/** */`
- 函数内部"为什么" → `//` 行内
- 配置分组 → `//` 简短

**函数注释看情况**：

```typescript
/** 打开角色表单弹窗 */
function openDialog() { ... }

/**
 * 打开编辑角色弹窗
 *
 * @param roleId 角色 ID
 */
async function handleEditClick(roleId: string) { ... }

/**
 * 校验并提交角色表单
 *
 * 非自定义数据权限时丢弃部门 ID
 */
async function handleSubmit() { ... }
```

简单函数一行够了。有参数加 `@param`。有踩坑点就多行补一句——只补"为什么"，不复述函数名已经能看出来的。

**常见错误**：

```typescript
// ❌ 复述代码：函数名已经说了"打开弹窗"，注释是废话
/** 打开弹窗 */
function openDialog() { ... }

// ❌ 描述"做什么"而非"为什么"
/** 遍历列表并过滤状态为启用的项 */
const enabledList = list.filter(item => item.status === 1);

// ✅ 写"为什么"
// status=1 是启用，数据库默认值是 0（禁用）
const enabledList = list.filter(item => item.status === 1);

// ❌ 每个函数都写注释，哪怕函数名一目了然
/** 获取用户信息 */
function getUserInfo() { ... }

// ✅ 函数名能说清的不写
function getUserInfo() { ... }
```

## 反模式速查

| 反模式 | 正确做法 |
|--------|---------|
| 类型用 `VO/DTO` 后缀 | 语义命名：`UserItem`、`UserForm` |
| 硬编码颜色 | CSS 变量 |
| 原子类 > 3 个 | 提取 BEM + SCSS |
| `class UserAPI` 静态方法 | `const UserAPI = {}` 对象字面量 |
| `stores/modules/` 子目录 | 扁平 `stores/` 结构 |
| 组件外直接用 `useXxxStore()` | 用 `useXxxStoreHook()` |
| 手动给 CURD 按钮加 `v-hasPerm` | 用 `permPrefix` 自动拼接 |
| `meta.title` 直接写中文 | 写语言包 key |
| 函数用 `//` 注释 | 用 `/** */` JSDoc 格式 |
| 注释复述"做什么" | 写"为什么"和约束 |

## 自查清单

- [ ] 类型无 `VO/DTO` 后缀
- [ ] 布尔值有 `is/has/can/should` 前缀
- [ ] `handle` 仅用于流程编排
- [ ] API 用 `const XXXAPI = {}` 对象字面量
- [ ] BEM 带页面前缀，原子类 ≤ 3 个
- [ ] 颜色用 CSS 变量
- [ ] SFC 块顺序：template → script → style
- [ ] Store 用 Setup Store + `useXxxStoreHook()`
- [ ] 组件 ≤ 300 行
- [ ] `meta.title` 使用语言包 key
- [ ] 权限标识遵循 `模块:资源:操作` 格式
- [ ] API 模块遵循 `index.ts` + `types.ts` 结构
- [ ] 函数用 `/** */` 注释，不用 `//`
- [ ] 注释写"为什么"，不复述"做什么"

## 参考文档

以下文档按需加载，涵盖具体开发场景的详细指南：

| 参考文件 | 适用场景 |
|----------|----------|
| [references/new-page-guide.md](references/new-page-guide.md) | 新增接口与页面：创建 API 模块、Composables 模式 CURD、简单页面、模式选择 |
| [references/curd-development.md](references/curd-development.md) | Config 驱动 CURD：配置接口、列模板、表单项、usePage() |
| [references/router-menu.md](references/router-menu.md) | 配置路由和菜单：静态/动态路由、Meta 字段、多级菜单 |
| [references/permission.md](references/permission.md) | 权限控制：v-hasPerm 指令、hasPerm() 函数、权限标识命名 |
| [references/page-caching.md](references/page-caching.md) | 页面缓存：keepAlive 配置、缓存机制、刷新当前页 |
| [references/i18n.md](references/i18n.md) | 菜单国际化：translateRouteTitle、语言包配置、语言切换 |
| [references/project-config.md](references/project-config.md) | 项目配置：settings.ts、环境变量、StorageKey 管理 |
| [references/components.md](references/components.md) | 常用组件：Upload、Dict、TableSelect、IconSelect |
