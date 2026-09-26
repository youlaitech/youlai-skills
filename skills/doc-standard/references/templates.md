# 栏目模板

每种文档类型对应一个模板。模板定义结构、写作规则和字数控制。

## Diátaxis 模式映射

| 模板 | Diátaxis 模式 | 读者问题 |
|------|--------------|----------|
| 操作指南 | Tutorial | "教我在界面上一步步做" |
| 开发指南 | Tutorial / Reference / Explanation | "教我怎么写代码" / "这个配置在哪改" / "团队约定是什么" |
| 组件文档 | Reference | "这个组件有哪些 Props" |
| Composable 文档 | Reference | "这个函数签名是什么" |
| 项目部署 | How-to | "帮我发布上线" |
| 进阶定制 | How-to | "帮我完成这个裁剪任务" |
| FAQ | Reference | "这个报错怎么解决" |

> 一篇文档只做一件事。操作指南不混入原理，参考文档不混入教程。混合模式的页面是文档质量下降的开始。

---

## 操作指南模板

适用于「操作指南」栏：新增菜单等在管理界面点击操作的纯界面任务。

**定位**：读者带明确任务来（「我要加一个菜单」），文档要让他照着界面 3 分钟内做完。

**结构**：

```markdown
---
title: 新增菜单
---

# 新增菜单

在后台「菜单管理」中维护菜单数据。改完后到「角色管理」分配权限并重新登录生效。

## 新增目录

目录用于组织菜单，本身不对应页面。选择顶级菜单作为父级菜单，菜单类型选「目录」，路由路径以 `/` 开头。

![新增目录](/images/router-menu/new-directory.png)

## 新增菜单

在目录下新增菜单，填写菜单名称、路由路径和页面组件。页面组件与实际文件路径一致，省略 `src/views/` 和 `.vue`。

## 开启页面缓存

需同时满足两个条件：

1. 菜单配置中开启「页面缓存」
2. 「页面标识」填写与组件 `name` 一致的值

\`\`\`vue
<script setup lang="ts">
// 页面标识填写 User，组件 name 必须同为 User
defineOptions({ name: "User" });
</script>
\`\`\`
```

**写作规则**：

1. 开头一句话：说明在哪个界面操作、改完如何生效
2. 章节按操作命名：`## 新增目录` `## 开启页面缓存`，不写 `## 菜单体系` `## 路由机制`
3. 纯界面操作，不出现路由配置、路由守卫源码等前端代码
4. 例外：操作必须的前端配合（如页面缓存的 `defineOptions`）保留最小代码块
5. 截图配关键步骤，不每步都配
6. 细节交叉引用开发指南：权限标识、国际化等不在本文档展开
7. 末尾不加「下一步」「相关链接」

**字数控制**：单篇 80~150 行。

---

## 组件文档模板

适用于「组件」栏：表单组件、展示组件、导航、CRUD、Composables。

**定位**：读者想知道「这个组件怎么用、有哪些 Props」。文档要让他看到代码示例和 API 表格。

**结构**：

```markdown
---
title: Upload 文件上传
---

# Upload 文件上传

基于 el-upload 封装，支持七牛云/阿里云/本地存储切换。

## 基础用法

\`\`\`vue
<script setup lang="ts">
import Upload from "@/components/Upload/index.vue";

const fileUrl = ref("");
\`\`\`

<template>
  <Upload v-model="fileUrl" />
</template>
\`\`\`

## 限制类型

\`\`\`vue
<Upload v-model="fileUrl" :accept="['image/png', 'image/jpeg']" />
\`\`\`

## Props

| 属性 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `v-model` | `string` | `—` | 文件访问地址（必填） |
| `accept` | `string[]` | `[]` | 允许的文件 MIME 类型 |
| `maxSize` | `number` | `10` | 文件大小上限（MB） |
| `type` | `"qiniu" \| "aliyun" \| "local"` | `"local"` | 存储方式 |
```

**写作规则**：

1. 开头一句话：说明组件基于什么封装、核心能力是什么
2. 章节顺序固定：`## 基础用法` → `## 进阶用法`（可选）→ `## Props`
3. 进阶用法按功能命名：`## 限制类型` `## 多文件上传`，不写 `## 使用场景`
4. 代码示例值要具体：`ref(1)` 不写 `ref('')`，`v-model="fileUrl"` 不写 `v-model="value"`
5. Props 表格：类型用反引号、必填标注「（必填）」、无默认值写 `—`
6. 多组件合并页面：每个组件用 `##` 分隔，下接一句话说明 + `### 基础用法` + `### Props`
7. 不写：功能特性列表、使用场景、emoji、`::: info` 在线演示块、相关链接
8. 在线演示：合并到首句内联链接，如 `基于 el-upload 封装的单图上传组件。[在线演示](url)`
9. 多页组件（如 CRUD 有子页面）可保留模块索引导航表，不算「先看什么」导航表

**字数控制**：单篇 60~120 行。Props 多的可放宽到 200 行。

---

## Composable 文档模板

适用于「Composables」栏：`useDictSync`、`useSse`、`useTableSelection` 等组合式函数。

**定位**：读者想知道「这个函数签名是什么、怎么调用、返回什么」。文档要让他直接看到完整类型定义和调用示例。

**结构**：

```markdown
---
title: useDictSync 字典同步
---

# useDictSync 字典同步

通过 SSE 实时接收字典变更并自动更新本地缓存。

## 基础用法

\`\`\`typescript
import { useDictSync } from "@/composables/sse/useDictSync";

const { connected, dictVersion } = useDictSync();
\`\`\`

## 参数

| 参数 | 类型 | 默认值 | 说明 |
|---|---|---|---|
| `autoConnect` | `boolean` | `true` | 是否自动连接 SSE |

## 返回值

| 属性 | 类型 | 说明 |
|---|---|---|
| `connected` | `Ref<boolean>` | SSE 连接状态 |
| `dictVersion` | `Ref<number>` | 当前字典版本号 |
| `reconnect` | `() => void` | 手动重连 |

## 类型定义

\`\`\`typescript
interface UseDictSyncOptions {
  autoConnect?: boolean;
  onMessage?: (data: DictSyncData) => void;
}
\`\`\`
```

**写作规则**：

1. 开头一句话：说明函数做什么、基于什么实现
2. 章节顺序固定：`## 基础用法` → `## 参数` → `## 返回值` → `## 类型定义`（可选）
3. 必须有完整类型签名：参数和返回值的类型定义不能省略
4. 代码示例可直接运行：包含 import 语句
5. 代码示例值要具体：`useDictSync()` 不写 `useHook()`，`connected` 不写 `state`
6. 不写：实现原理、SSE 协议解释、功能特性列表、使用场景、emoji、「先看什么」表
7. 末尾不加「相关链接」「下一步」

**字数控制**：单篇 50-100 行。参数多的可到 120 行。

---

## 开发指南模板

适用于「开发指南」栏，按内容分三种变体：

| 变体 | 适用文档 | Diátaxis 模式 |
|------|---------|--------------|
| 教程类 | 权限控制、国际化、Mock 数据、AI 助手 | Tutorial |
| 规范类 | 代码规范 | Explanation |
| 参考类 | 系统设置 | Reference |

**定位**：读者要写代码完成任务（教程类）、了解团队约定（规范类）或找到配置字段（参考类）。

**结构（教程类）**：

```markdown
---
title: Mock 数据
---

# Mock 数据

后端接口未就绪时，通过 Mock 在本地模拟接口返回。

## 开启 Mock

`.env.development` 中设置：

\`\`\`bash
VITE_MOCK_DEV_SERVER = true
\`\`\`

## 创建 Mock 文件

在 `mock/` 目录下新建 `xxx.mock.ts`：

\`\`\`typescript
import { defineMock } from "vite-plugin-mock-dev-server";

export default defineMock([
  {
    url: "/api/v1/users",
    method: "GET",
    response: () => ({ code: "00000", data: [], msg: "成功" }),
  },
]);
\`\`\`
```

**结构（规范类）**：

```markdown
---
title: 代码规范
---

# 代码规范

命名、文件、组件、导入、类型和注释约定。

## 命名

| 类型 | 规则 | 示例 |
|---|---|---|
| 组件 | PascalCase | `UserProfile.vue` |
| Composable | `use` 开头 camelCase | `useDictSync` |
| 常量 | UPPER_SNAKE_CASE | `MAX_PAGE_SIZE` |
```

**结构（参考类）**：

```markdown
---
title: 系统设置
---

# 系统设置

配置分两层：`settings.ts` 控制功能开关，`.env` 控制环境变量。

## 环境变量

\`\`\`bash
VITE_APP_TITLE=vue3-element-admin    # 项目名称
VITE_APP_BASE_API=/dev-api           # 代理前缀
\`\`\`
```

**写作规则**：

1. 教程类：章节按任务命名，代码可复制即用，不写实现原理
2. 规范类：章节按维度命名（`## 命名` `## 文件命名` `## 组件规范`），规则用「维度 + 规则 + 示例」三列表格
3. 参考类：章节按配置载体命名（`## settings.ts` `## 环境变量` `## 主题色`），代码块直接列字段 + 注释
4. 代码必须可运行：复制粘贴就能用，不省略 import、不写 `// ...`
5. 横切主题按内容归属规则交叉引用，不重复
6. 末尾不加「相关链接」「下一步」

**字数控制**：教程类 60~120 行。规范类 100~200 行。参考类 80~150 行。

---

## 项目部署模板

适用于「项目部署」栏：构建生产环境产物并部署上线。

**定位**：读者要发布上线。文档要让他直接复制构建命令和部署配置。

**结构**：

```markdown
---
title: 项目部署
---

# 项目部署

构建生产环境产物，部署到 Nginx 或 Docker。

## 构建生产环境

\`\`\`bash
pnpm run build
\`\`\`

构建产物在 `dist/` 目录。

## Nginx 部署

\`\`\`nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API 代理，与 .env.production 的 VITE_APP_BASE_API 保持一致
    location /prod-api/ {
        proxy_pass http://api.youlai.tech/;
    }
}
\`\`\`
```

**写作规则**：

1. 开头一句话：说明文档覆盖什么
2. 章节按步骤命名：`## 构建生产环境` `## Nginx 部署` `## Docker 部署`
3. 命令和配置必须可直接执行：不省略参数、不写 `<your-domain>`
4. 代理前缀与 `.env.production` 的 `VITE_APP_BASE_API` 保持一致
5. 不写：「先看什么」表、介绍段、设计思路
6. 不写：「常见问题」放 FAQ 栏
7. 末尾不加「相关链接」「下一步」

**字数控制**：单篇 60~100 行。

---

## 进阶定制模板

适用于「进阶定制」栏：移除动态路由、移除登录等对项目的裁剪/改造。

**定位**：读者已经跑通项目，现在要按需裁剪。文档要让他清楚知道「改哪些文件、删哪些代码、怎么验证」。

**结构**：

```markdown
---
title: 移除动态路由
---

# 移除动态路由

将后端驱动的动态路由改为前端静态路由，适用于内网系统或无权限管理的场景。

## 改动点

| 文件 | 操作 |
|---|---|
| `src/router/guards/permission.ts` | 删除动态路由生成逻辑 |
| `src/store/modules/permission-store.ts` | 删除整个文件 |
| `src/router/index.ts` | 把所有路由写进 `constantRoutes` |

## 具体步骤

### 1. 路由改为静态

把原动态路由全部移到 `constantRoutes`：

\`\`\`typescript
export const constantRoutes: RouteRecordRaw[] = [
  { path: "/login", ... },
  { path: "/", component: Layout, children: [
    { path: "dashboard", ... },
    { path: "system/user", ... },
  ]},
]
\`\`\`

### 2. 移除路由守卫拦截

`src/router/guards/permission.ts` 中删除：

\`\`\`typescript
// 删除以下逻辑
if (!permissionStore.isRouteGenerated) {
  const dynamicRoutes = await permissionStore.generateRoutes()
  ...
}
\`\`\`

### 3. 删除权限 Store

删除 `src/store/modules/permission-store.ts`，并移除 `src/store/index.ts` 中的引用。

## 验证

1. `pnpm run dev` 启动无报错
2. 直接访问 `/system/user` 能进入页面
3. 侧边栏菜单完整显示
```

**写作规则**：

1. 开头一句话：说明改完后达到什么效果、适用什么场景
2. 必须有「改动点」表：让读者一眼看到影响范围
3. 步骤按序号命名：`### 1. 路由改为静态` `### 2. 移除路由守卫拦截`
4. 删除的代码要写明：用注释标注 `// 删除以下逻辑`，不写「删除相关代码」这种模糊表述
5. 必须有「验证」章节：列出 2~3 条可执行的验证步骤
6. 不写：为什么这么设计、动态路由原理、改造前后对比图
7. 不写：「先看什么」表、介绍段、常见问题
8. 末尾不加「相关链接」「下一步」

**字数控制**：单篇 80~150 行。改动点多的可到 200 行。

---

## FAQ 模板

适用于「常见问题」栏：启动报错、配置冲突、行为异常等排障型文档。

**定位**：读者遇到问题来查解决方案。文档要让他直接看到答案，不看原理。

**结构**：

```markdown
---
title: 常见问题
---

# 常见问题

## 启动报错 `ERR_OSSL_EVP_UNSUPPORTED`

Node.js 17+ 的 OpenSSL 兼容问题。

\`\`\`bash
export NODE_OPTIONS=--openssl-legacy-provider
\`\`\`

## variables.scss 与 variables.module.scss 的区别

`variables.scss` 供 SCSS 文件使用，`variables.module.scss` 供 JS/TS 导入使用（SCSS 变量无法直接被 JS 读取）。

在 JS 中：

\`\`\`typescript
import { $colors } from "@/styles/variables.module.scss";
\`\`\`

## 菜单保存后侧边栏不显示

检查菜单类型是否正确：
- 目录用 `M` 类型
- 菜单用 `C` 类型
- 按钮用 `F` 类型
```

**写作规则**：

1. 问题即标题：`## 启动报错 ERR_OSSL_EVP_UNSUPPORTED`，不写 `## 问题一`
2. 答案直接给解决方案：第一句话就是答案，不写「原因与设计」「为什么不能」「结论」等原理段落
3. 原理压缩为 1-2 句：如需解释原因，用一句话带过
4. 代码命令可直接执行：不省略参数
5. 不写：「获取帮助」「相关链接」尾巴
6. 不写：emoji、「先看什么」表

**字数控制**：单个问题 5-15 行。总篇数不限。
