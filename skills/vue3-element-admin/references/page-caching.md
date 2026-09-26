# 页面缓存

## 开启缓存

路由设置 `meta.keepAlive: true` 后，页面才会加入缓存候选：

```typescript
{
  path: "user",
  name: "User",
  component: () => import("@/views/system/user/index.vue"),
  meta: { title: "user", keepAlive: true },
}
```

## 项目机制

- `useTagsViewStore` 的 `cachedViews` 保存路由 `fullPath`。
- `LayoutMain.vue` 中的 `currentComponent()` 用 `fullPath` 生成包装组件名，使 `keep-alive` 的 `include` 能精确匹配。
- 同一路径的不同查询参数可能形成不同 `fullPath`；修改缓存策略前先确认产品是否需要分别缓存。
- `meta.affix: true` 的标签会固定在标签栏，不能由用户关闭。

## 刷新与移除

关闭标签时会从 `cachedViews` 移除。刷新当前页通过 redirect 路由完成：先删除缓存，再跳转到 `/redirect` + 原 `fullPath`，由 redirect 页面重新加载。

修改缓存逻辑时至少验证：

1. 首次进入和返回标签时状态是否符合预期。
2. 手动刷新是否真正重新请求数据。
3. 关闭标签后缓存是否释放。
4. 带 path 参数、query 和外链的页面是否误入缓存。

不要套用依赖固定组件 `name` 的通用方案而绕过当前 `fullPath` 包装机制。
