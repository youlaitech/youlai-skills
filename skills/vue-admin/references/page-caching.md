# 页面缓存

## 开启缓存

路由 `meta.keepAlive: true` 即可开启页面缓存：

```typescript
{
  path: "user",
  component: () => import("@/views/system/user/index.vue"),
  meta: { title: "user", keepAlive: true },
}
```

## 缓存机制

`keep-alive` 的 `include` 匹配的是组件 name。项目通过 `currentComponent()` 创建包装组件，其 `name` 设为 `route.fullPath`，确保缓存按完整路径精确匹配。

`useTagsViewStore` 维护 `cachedViews: string[]`（存 `fullPath`），仅当 `meta.keepAlive === true` 时添加到缓存列表。

```typescript
function addCachedView({ fullPath, keepAlive }: TagView) {
  if (cachedViews.value.includes(fullPath)) return;
  if (keepAlive) cachedViews.value.push(fullPath);
}
```

## 刷新当前页

关闭当前标签时自动从 `cachedViews` 移除。手动刷新通过 redirect 路由实现：`delCachedView(tag)` → `router.replace("/redirect" + tag.fullPath)` → redirect 组件重新加载原路由。

## 固定标签

`meta.affix: true` 的路由在 `onMounted` 时通过 `extractAffixTags()` 从路由树提取并初始化到标签栏，用户无法关闭。
