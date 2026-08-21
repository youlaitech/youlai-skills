# 项目配置

## settings.ts

配置分两层：`appConfig` 控制功能开关（从环境变量读取），`defaults` 控制 UI 默认值（用户可覆盖，持久化到 localStorage）。

```typescript
// 应用级配置（不可变，从环境变量读取）
export const appConfig = {
  name: pkg.name,
  version: pkg.version,
  title: env.VITE_APP_TITLE || pkg.name,
  tenantEnabled: env.VITE_APP_TENANT_ENABLED === "true",
} as const;

// 默认设置（用户可覆盖，持久化到 localStorage）
export const defaults = {
  theme: ThemeMode.LIGHT,           // light | dark
  themePalette: defaultThemePalette.id,  // 预设 id：arco | ant-design | element-plus
  layout: LayoutMode.LEFT,          // left | top | mix | double
  size: ComponentSize.DEFAULT,      // default | large | small
  language: LanguageEnum.ZH_CN,     // zh-cn | en
  showTagsView: true,
  tagsViewStyle: TagsViewStyle.CARD, // line | card
  showAppLogo: true,
  showWatermark: false,
  pageSwitchingAnimation: "fade-slide", // none | fade | fade-slide | fade-scale
} as const;
```

## 环境变量

| 变量 | 说明 |
|---|---|
| `VITE_APP_TITLE` | 应用标题 |
| `VITE_APP_BASE_API` | 接口基础路径 |
| `VITE_APP_TENANT_ENABLED` | 是否开启多租户 |
| `VITE_MOCK_DEV_SERVER` | 是否开启 Mock |

## StorageKey 管理

统一在 `constants/index.ts` 管理，前缀 `vea:`，分类管理：

```typescript
const APP_PREFIX = "vea";
export const STORAGE_KEYS = {
  // 认证
  ACCESS_TOKEN: `${APP_PREFIX}:auth:access_token`,
  REFRESH_TOKEN: `${APP_PREFIX}:auth:refresh_token`,
  // 租户
  TENANT_ID: `${APP_PREFIX}:tenant:id`,
  // 系统
  DICT_CACHE: `${APP_PREFIX}:system:dict_cache`,
  // UI
  SHOW_TAGS_VIEW: `${APP_PREFIX}:ui:show_tags_view`,
  TAGS_VIEW_STYLE: `${APP_PREFIX}:ui:tags_view_style`,
} as const;
```

`ROLE_ROOT = "ROOT"` 是超级管理员角色标识。

## 配置持久化

`useSettingsStore` 通过 `useStorage(STORAGE_KEYS.XXX, defaults.XXX)` 将每个配置项持久化到 localStorage，并设置 watcher 实时应用主题变化。

## 主题预设

三套配色：ArcoD（默认）、AntD、ElementD。通过 `themePalette` 切换。
