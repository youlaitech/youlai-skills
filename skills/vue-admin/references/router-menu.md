# 路由与菜单配置

## 静态路由（constantRoutes）

定义在 `router/index.ts`，使用 `createWebHashHistory()`：

```typescript
export const constantRoutes: RouteRecordRaw[] = [
  { path: "/login", component: () => import("@/views/login/index.vue"), meta: { hidden: true } },
  { path: "/", component: Layout, redirect: "/dashboard", children: [
    {
      path: "dashboard",
      name: "Dashboard",
      component: () => import("@/views/dashboard/index.vue"),
      meta: { title: "dashboard", icon: "homepage", affix: true, keepAlive: true },
    },
  ]},
];
```

## Meta 字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `title` | `string` | 路由标题，同时是 i18n key |
| `icon` | `string` | 菜单图标（SVG 名或 `el-icon-xxx`） |
| `hidden` | `boolean` | 是否在侧边栏隐藏 |
| `keepAlive` | `boolean` | 是否开启页面缓存 |
| `affix` | `boolean` | 标签是否固定在标签栏 |
| `alwaysShow` | `boolean` | 目录只有一个子路由时是否始终显示为父级菜单 |
| `params` | `Record<string, unknown>` | 路由参数 |
| `externalUrl` | `string` | 外链地址 |
| `roles` | `string[]` | 角色集合 |

## 新增静态菜单

在 `constantRoutes` 中添加路由记录，`meta.title` 填写语言包 key（如 `"user"`），系统自动在语言包 `route` 命名空间查找翻译：

```typescript
{
  path: "/system/user",
  component: Layout,
  children: [
    {
      path: "",
      name: "User",
      component: () => import("@/views/system/user/index.vue"),
      meta: {
        title: "user",         // 语言包 key，对应 route.user
        icon: "user",
        keepAlive: true,
      },
    },
  ],
}
```

## 新增动态菜单

在后端菜单管理界面添加菜单，前端无需改代码。后端返回的 `component` 字段是相对于 `src/views/` 的路径（如 `"system/user/index"` 或 `"system/user"`），前端通过 `import.meta.glob("../views/**/*.vue")` 动态解析，自动匹配 `.vue` 或 `/index.vue`。

## 多级菜单

父路由用 `Layout` 组件，子路由 `path` 不带 `/`：

```typescript
{
  path: "/system",
  component: Layout,
  meta: { title: "system", icon: "setting" },
  children: [
    { path: "user", name: "User", component: () => import("@/views/system/user/index.vue"), meta: { title: "user" } },
    { path: "role", name: "Role", component: () => import("@/views/system/role/index.vue"), meta: { title: "role" } },
  ],
}
```

## 动态路由生成流程

1. 用户登录后首次导航，`isRouteGenerated === false`
2. 调用 `userStore.getUserInfo()` 获取用户信息（含 roles/perms）
3. `permissionStore.generateRoutes()`：`MenuAPI.getRoutes()` 从后端获取路由树
4. `transformRoutes()` 转换：顶层 `component: "Layout"` → `Layout` 组件；其他通过 `import.meta.glob` 动态导入
5. `router.addRoute()` 注册，`return { ...to, replace: true }` 重新导航
