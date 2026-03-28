---
name: uniapp
description: UniApp 移动端开发规范。当开发 UniApp + Vue3 移动应用、使用 wot-design-uni 组件、实现移动端 UI/UX、处理小程序兼容性时使用此 skill。
---

# UniApp 移动端开发规范

## 触发条件

- 开发 UniApp 移动端项目
- 使用 wot-design-uni 组件
- 实现移动端 UI/UX
- 处理小程序兼容性问题
- 使用 UnoCSS 原子类

---

## Part 0: 强制规则

### 0.1 组件优先原则

**能用 wot-design-uni 官方组件，绝不自定义**

| 需求   | 使用                             | 避免            |
| ------ | -------------------------------- | --------------- |
| 按钮   | `<wd-button>`                    | 自定义 `.btn`   |
| 输入框 | `<wd-input>`                     | 自定义 `.input` |
| 卡片   | `<wd-card>` / `<wd-cell-group>`  | 自定义 `.card`  |
| 弹窗   | `<wd-popup>` / `<wd-dialog>`     | 自定义弹窗      |
| 列表   | `<wd-cell>` / `<wd-grid>`        | 自定义列表      |
| 加载   | `<wd-loading>` / `<wd-skeleton>` | 自定义动画      |
| 空状态 | `<wd-status>`                    | 自定义空状态    |

### 0.2 CSS 统一原则

**禁止重复定义相同样式，必须复用**

```scss
// ❌ 错误：每个组件都定义一遍
.user-card {
  padding: 24rpx;
  background: var(--color-bg);
}
.role-card {
  padding: 24rpx;
  background: var(--color-bg);
}

// ✅ 正确：统一基础类
.base-card {
  padding: 24rpx;
  background: var(--color-bg);
  border-radius: 16rpx;
}
```

### 0.3 暗黑模式强制要求

**所有颜色必须使用 CSS 变量，禁止硬编码**

```scss
// ❌ 错误：硬编码颜色
.card {
  background: #ffffff;
  color: #333333;
}

// ✅ 正确：使用 CSS 变量
.card {
  background: var(--color-bg);
  color: var(--color-text);
}
```

---

## Part 1: 技术栈

- **Vue** - 前端框架
- **Uni-app** - 跨平台框架
- **wot-design-uni** - UI 组件库
- **TypeScript** - 类型安全
- **UnoCSS** - 原子 CSS
- **Pinia** - 状态管理

---

## Part 2: 目录结构

```
src/
├── api/                    # API 请求
│   ├── system/             # 系统模块
│   │   ├── user.ts
│   │   └── role.ts
│   └── index.ts
├── components/             # 全局组件
│   ├── UserCard/
│   │   └── index.vue
│   └── CustomTree/
│       └── index.vue
├── composables/            # 组合式函数
│   ├── useList.ts          # 列表逻辑
│   ├── useForm.ts          # 表单逻辑
│   └── useAuth.ts          # 认证逻辑
├── enums/                  # 枚举定义
│   ├── ResultCode.ts
│   └── Status.ts
├── pages/                  # 页面
│   ├── index/              # 首页
│   │   └── index.vue
│   ├── login/              # 登录
│   │   └── index.vue
│   ├── mine/               # 我的
│   │   └── index.vue
│   └── work/               # 工作台
│       ├── menu/
│       │   └── index.vue
│       └── user/
│           └── index.vue
├── static/                 # 静态资源
│   ├── images/
│   └── icons/
├── stores/                 # Pinia 状态
│   ├── modules/
│   │   ├── user.ts
│   │   └── app.ts
│   └── index.ts
├── styles/                 # 全局样式
│   ├── variables.scss      # CSS 变量
│   ├── index.scss          # 入口
│   └── uno.scss            # UnoCSS 配置
├── utils/                  # 工具函数
│   ├── request.ts          # 请求封装
│   ├── auth.ts             # Token 管理
│   ├── permission.ts       # 权限判断
│   └── format.ts           # 格式化
├── App.vue                 # 根组件
├── main.ts                 # 入口
├── manifest.json           # 应用配置
├── pages.json              # 页面配置
└── uni.scss                # UniApp 样式变量
```

---

## Part 3: CSS 使用边界

### UnoCSS 原子类 + BEM 混合方案

| 原子类数量 | 方案          |
| ---------- | ------------- |
| ≤3 个      | UnoCSS 原子类 |
| >3 个      | BEM CSS 类    |

### 合并原则（元素已有类名时）

```vue
<!-- ❌ 错误：类名 + 原子类混用 -->
<view class="filter-bar flex items-center mt-12rpx">

<!-- ✅ 正确：原子类合并到 CSS 类 -->
<view class="filter-bar">
```

```scss
.filter-bar {
  display: flex; // 合并 flex
  align-items: center; // 合并 items-center
  margin-top: 12rpx; // 合并 mt-12rpx
}
```

### 使用预设快捷类

| 原子类组合                          | 预设           | 减少 |
| ----------------------------------- | -------------- | ---- |
| `flex items-center`                 | `flex-start`   | 2→1  |
| `flex justify-between items-center` | `flex-between` | 3→1  |
| `flex justify-center items-center`  | `flex-center`  | 3→1  |

---

## Part 4: 命名规范

### 文件命名

| 类型       | 规范              | 示例               |
| ---------- | ----------------- | ------------------ |
| 组件       | PascalCase        | `UserCard.vue`     |
| 页面       | kebab-case 或小写 | `user-profile.vue` |
| 组合式函数 | camelCase + use   | `useUserStore.ts`  |
| 工具函数   | camelCase         | `formatDate.ts`    |
| 常量       | UPPER_SNAKE_CASE  | `API_PREFIX.ts`    |

### 变量命名

| 类型     | 规范             | 示例                    |
| -------- | ---------------- | ----------------------- |
| 变量     | camelCase        | `userList`              |
| 常量     | UPPER_SNAKE_CASE | `MAX_COUNT`             |
| 私有变量 | \_ 前缀          | `_internalState`        |
| 布尔值   | is/has/can 前缀  | `isLoading`, `hasToken` |

### 方法命名

| 动作      | 前缀        | 示例            |
| --------- | ----------- | --------------- |
| 获取/加载 | fetch/load  | `fetchUserList` |
| 提交/保存 | submit/save | `submitForm`    |
| 新增      | create      | `createUser`    |
| 更新      | update      | `updateUser`    |
| 删除      | delete      | `deleteUser`    |
| 打开/关闭 | open/close  | `openDialog`    |
| 事件处理  | handle      | `handleSubmit`  |

### CSS 命名（BEM）

```scss
.user-card {
  padding: 24rpx;
  background: var(--color-bg);
  border-radius: 16rpx;

  &__header {
    display: flex;
    align-items: center;
  }

  &__avatar {
    width: 80rpx;
    height: 80rpx;
    border-radius: 50%;
  }

  &__name {
    font-size: 28rpx;
    font-weight: 600;
    color: var(--color-text);
  }

  &__role {
    font-size: 24rpx;
    color: var(--color-text-secondary);
  }

  &--active {
    border: 2rpx solid var(--color-primary);
  }
}
```

---

## Part 5: 页面模板

### 列表页面模板

```vue
<template>
  <view class="page">
    <!-- 搜索栏 -->
    <view class="search-bar">
      <wd-search v-model="keyword" placeholder="搜索" @search="handleSearch" />
    </view>

    <!-- 筛选标签 -->
    <view class="filter-tabs">
      <wd-tabs v-model="activeTab" @change="handleTabChange">
        <wd-tab name="all" title="全部" />
        <wd-tab name="enabled" title="启用" />
        <wd-tab name="disabled" title="禁用" />
      </wd-tabs>
    </view>

    <!-- 列表 -->
    <scroll-view
      class="list-container"
      scroll-y
      :refresher-enabled="true"
      :refresher-triggered="refreshing"
      @refresherrefresh="handleRefresh"
      @scrolltolower="handleLoadMore"
    >
      <view v-for="item in list" :key="item.id" class="list-item">
        <user-card :user="item" @edit="handleEdit" @delete="handleDelete" />
      </view>
      <wd-loadmore :state="loadState" />
    </scroll-view>

    <!-- 悬浮按钮 -->
    <wd-fab icon="add" @click="handleAdd" />
  </view>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import { onLoad } from "@dcloudio/uni-app";
import type { UserVO } from "@/api/system/user/types";

const keyword = ref("");
const activeTab = ref("all");
const list = ref<UserVO[]>([]);
const loading = ref(false);
const refreshing = ref(false);
const hasMore = ref(true);
const pageNum = ref(1);

const loadState = computed(() => {
  if (loading.value) return "loading";
  if (!hasMore.value) return "finished";
  return "loading";
});

async function fetchList(isRefresh = false) {
  if (loading.value) return;
  loading.value = true;

  if (isRefresh) {
    pageNum.value = 1;
    list.value = [];
  }

  try {
    const res = await UserAPI.getPage({
      keyword: keyword.value,
      status: activeTab.value,
      pageNum: pageNum.value,
    });
    list.value = isRefresh ? res.data.list : [...list.value, ...res.data.list];
    hasMore.value = res.data.list.length >= 20;
  } finally {
    loading.value = false;
    refreshing.value = false;
  }
}

function handleSearch() {
  fetchList(true);
}

function handleTabChange() {
  fetchList(true);
}

function handleRefresh() {
  refreshing.value = true;
  fetchList(true);
}

function handleLoadMore() {
  if (hasMore.value) {
    pageNum.value++;
    fetchList();
  }
}

function handleAdd() {
  uni.navigateTo({ url: "/pages/work/user/edit" });
}

function handleEdit(id: number) {
  uni.navigateTo({ url: `/pages/work/user/edit?id=${id}` });
}

async function handleDelete(id: number) {
  const { confirm } = await uni.showModal({
    title: "确认删除？",
    content: "删除后不可恢复",
  });
  if (confirm) {
    await UserAPI.deleteById(id);
    uni.showToast({ title: "删除成功", icon: "success" });
    fetchList(true);
  }
}

onLoad(() => fetchList());
</script>

<style lang="scss" scoped>
.page {
  min-height: 100vh;
  background-color: var(--color-bg-secondary);
}

.search-bar {
  padding: 24rpx;
  background-color: var(--color-bg);
}

.filter-tabs {
  background-color: var(--color-bg);
}

.list-container {
  height: calc(100vh - 200rpx);
}
</style>
```

---

## Part 6: 卡片设计规范

### 信息层级原则

**移动端卡片 ≠ Web 表格**

| 层级     | 内容     | 视觉权重 | 示例                 |
| -------- | -------- | -------- | -------------------- |
| 主信息   | 身份识别 | 最突出   | 头像 + 用户名 + 状态 |
| 次信息   | 身份定位 | 中等     | 角色 + 部门          |
| 辅助信息 | 联系方式 | 较弱     | 手机号 + 邮箱        |
| 元信息   | 时间戳   | 最弱     | 创建时间             |

### 卡片模板

```vue
<template>
  <view class="user-card">
    <!-- 主信息行 -->
    <view class="user-card__header">
      <wd-img :src="user.avatar" width="80rpx" height="80rpx" round />
      <view class="user-card__identity">
        <text class="user-card__name">{{ user.nickname }}</text>
        <text class="user-card__role">{{ user.roleName }} · {{ user.deptName }}</text>
      </view>
      <wd-tag :type="user.status === 1 ? 'success' : 'danger'" plain>
        {{ user.status === 1 ? "正常" : "禁用" }}
      </wd-tag>
    </view>

    <!-- 辅助信息行 -->
    <view class="user-card__contact">
      <text>📱 {{ user.mobile }}</text>
      <text v-if="user.email">📧 {{ user.email }}</text>
    </view>

    <!-- 操作按钮 -->
    <view class="user-card__actions" @click.stop="showActions">
      <wd-icon name="more" />
    </view>
  </view>
</template>

<script setup lang="ts">
import type { UserVO } from "@/api/system/user/types";

defineProps<{ user: UserVO }>();
const emit = defineEmits<{ edit: [id: number]; delete: [id: number] }>();

function showActions() {
  uni.showActionSheet({
    itemList: ["编辑", "删除"],
    success: ({ tapIndex }) => {
      if (tapIndex === 0) emit("edit", props.user.id);
      if (tapIndex === 1) emit("delete", props.user.id);
    },
  });
}
</script>

<style lang="scss" scoped>
.user-card {
  position: relative;
  padding: 24rpx;
  background: var(--color-bg);
  border-radius: 16rpx;

  &__header {
    display: flex;
    align-items: center;
    gap: 16rpx;
  }

  &__identity {
    flex: 1;
    min-width: 0;
  }

  &__name {
    display: block;
    font-size: 28rpx;
    font-weight: 600;
    color: var(--color-text);
  }

  &__role {
    display: block;
    font-size: 24rpx;
    color: var(--color-text-secondary);
    margin-top: 4rpx;
  }

  &__contact {
    display: flex;
    gap: 24rpx;
    margin-top: 12rpx;
    font-size: 24rpx;
    color: var(--color-text-secondary);
  }

  &__actions {
    position: absolute;
    right: 24rpx;
    bottom: 24rpx;
    padding: 16rpx;
  }
}
</style>
```

### 卡片规则

- **控制在 3-4 行以内**
- **次信息用 `·` 分隔**
- **辅助信息用图标前缀：📱📧**
- **操作轻量化：`···` 更多按钮**

---

## Part 7: 暗黑模式 CSS 变量

```scss
:root,
page {
  /* 背景色 */
  --color-bg: #ffffff;
  --color-bg-secondary: #f5f5f5;
  --color-bg-tertiary: #ebebeb;

  /* 文字色 */
  --color-text: #1f2937;
  --color-text-secondary: #6b7280;
  --color-text-placeholder: #9ca3af;

  /* 边框色 */
  --color-border: #e5e7eb;
  --color-divider: #f3f4f6;

  /* 功能色 */
  --color-primary: #2563eb;
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;

  /* 渐变背景 */
  --color-primary-light: rgba(37, 99, 235, 0.1);
  --color-success-light: rgba(16, 185, 129, 0.1);
  --color-warning-light: rgba(245, 158, 11, 0.1);
  --color-danger-light: rgba(239, 68, 68, 0.1);
}

.wot-theme-dark {
  --color-bg: #1f2937;
  --color-bg-secondary: #111827;
  --color-bg-tertiary: #374151;
  --color-text: #f9fafb;
  --color-text-secondary: #9ca3af;
  --color-text-placeholder: #6b7280;
  --color-border: #374151;
  --color-divider: #1f2937;
}
```

---

## Part 8: rpx 单位规范

- 设计稿基准：750rpx
- 字体大小：偶数值

| 场景      | 字号  | 说明               |
| --------- | ----- | ------------------ |
| 辅助信息  | 24rpx | 描述、标签、页脚   |
| 按钮/次要 | 26rpx | 次要操作、筛选按钮 |
| 基础字号  | 28rpx | 正文、列表项       |
| 标题      | 32rpx | 页面标题、重要信息 |

- 间距：4 的倍数 `16rpx`、`24rpx`、`32rpx`

---

## Part 9: 小程序兼容性

### 避免使用伪元素

小程序不支持 `::before`、`::after`，改用真实元素：

```vue
<!-- ❌ 错误：使用伪元素 -->
<style>
.hero::after {
  content: "";
  position: absolute;
  /* ... */
}
</style>

<!-- ✅ 正确：使用真实元素 -->
<template>
  <view class="hero">
    <view class="hero-fade"></view>
  </view>
</template>
```

### 避免使用 DOM API

小程序不支持 `document`、`window`，使用 UniApp API：

```typescript
// ❌ 错误
document.getElementById("app");

// ✅ 正确
uni.createSelectorQuery().select("#app");
```

---

## Part 10: 代码质量检查清单

- [ ] 使用 wot-design-uni 组件（不自定义）
- [ ] 原子类 ≤3 个，>3 个用 BEM
- [ ] 所有颜色使用 CSS 变量
- [ ] 暗黑模式正常工作
- [ ] 卡片控制在 3-4 行
- [ ] 信息层级清晰
- [ ] 操作使用 `···` 更多按钮
- [ ] 使用 UnoCSS 预设快捷类
- [ ] 所有尺寸使用 rpx 单位
- [ ] 定义 TypeScript 类型
- [ ] 避免伪元素
- [ ] 避免使用 DOM API
