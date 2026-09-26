# 命名

## 文件与标识符

| 对象 | 约定 |
| --- | --- |
| 页面入口 | `index.vue`；已有路由路径和组件目录入口保持稳定 |
| Vue 文件、模板标签 | kebab-case；业务组件如 `user-card.vue`，新增基础 UI 包装用 `app-` 前缀 |
| 组件导入标识、类型 | PascalCase；类型不加 I/T 前缀 |
| 组合函数 | `useXxx.ts`，文件与导出函数同名 |
| Store | 业务文件 `user.ts`，导出 `useUserStore`；实例 `userStore`，Pinia 根实例 `pinia` |
| 工具、配置、类型文件 | 按职责命名，如 `storage.ts`、`user.types.ts`；工具目录 `index.ts` 仅聚合导出 |
| 变量、函数、props 声明 | camelCase；保留 ID、API、SMS 等通用缩写 |
| 固定配置常量 | UPPER_SNAKE_CASE；普通 const 不强制大写 |
| 内部布尔状态 | `isSaving` 表达状态、`hasMore` 表达存在、`canEdit` 表达能力 |

- 名称与数据形态一致：`selectedUserId`、`selectedUserIds`、`users`、`activeSlideIndex`。
- 易混单位写入名称，如 `durationSeconds`、`sizeBytes`；展示字符串用 `sizeText`。
- 同一概念使用同一词汇；同一作用域出现多个表单或列表时再加业务限定。
- 框架 API、已有组件 props 和服务端字段保留契约，不机械改大小写或追加布尔前缀。

## 函数职责

| 职责 | 命名 |
| --- | --- |
| UI 事件参数适配、表单校验、操作确认 | `handleStatusChange`、`handleSubmit` |
| 获取数据并更新页面/Store/组合函数状态 | `loadUsers` |
| 新增/编辑模式下的保存流程 | `saveUser` |
| 打开、关闭、重置状态 | `openEditor`、`closeEditor`、`resetFilters` |
| 纯转换、计算、校验 | `formatDate`、`parseQuery`、`buildOptions`、`validatePassword` |
| 指定值 / 两种状态反转 | `setThemeMode(mode)` / `toggleThemeMode()`，两者不合并 |
| 检查并补齐前置条件 | `ensureUserInfo`；说明可能发生的加载或跳转 |
| 初始化 / 根据已有状态同步 | `initializeSession` / `syncNavigationBarTheme` |

- `handle` 仅作 UI 入口，业务动作与 Store action 不加此前缀；生命周期调用业务动作。
- 事件只需调用已有动作时直接绑定；需要局部业务名时使用别名，不叠加空转调函数。
- `get` 不表示同步或无网络；API 读取方法见[类型与接口](types-api.md)。`set/clear/reset` 的宾语必须对应实际操作范围。
- 项目事件入口统一 `handleXxx`，不混用 `onXxx/xxxHandler`；框架生命周期保留原名。

## Store 与组合函数

- 登录态、资料已加载、资料完整性分别表达为 `isAuthenticated`、`hasUserInfo`、`isProfileComplete`；不得各页面同名不同义。
- Store 管理共享状态；只转调请求且无共享状态的流程放页面或组合函数，兼容/适配边界除外。
- 组合函数按实际能力命名；通用列表返回 `items`，页面可解构为 `users/roles`。

## 组件事件

使用 `defineEmits` 声明载荷；自定义事件用 kebab-case，不加 handle/on。

| 语义 | 事件名 |
| --- | --- |
| 请求执行动作，父组件处理 | `submit`、`delete` |
| 请求关闭，父组件可拒绝 | `request-close` |
| 操作已经完成 | `saved`、`deleted`、`closed` |
| 值选择/变化 | `select`、`change`，传明确的新值或业务对象 |
| 双向绑定 | `update:modelValue`，保留 Vue 约定 |

来源：[Vue 命名](https://vuejs.org/style-guide/rules-strongly-recommended.html)、[组合函数](https://vuejs.org/guide/reusability/composables.html)、[事件处理](https://vuejs.org/guide/essentials/event-handling.html)、[组件事件](https://vuejs.org/guide/components/events.html)。
