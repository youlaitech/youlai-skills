---
name: uniapp
description: UniApp + Vue3 mobile development standards. Use this skill when developing UniApp mobile apps, using wot-design-uni components, implementing mobile UI/UX, or handling mini-program compatibility.
---

# UniApp 移动端开发规范

## 触发条件

- Develop UniApp mobile projects
- Use wot-design-uni components
- Implement mobile UI/UX
- Handle mini-program compatibility issues
- Use UnoCSS atomic classes

---

## Part 0: 强制规则

### 核心分层原则

**BEM 定义语义，UnoCSS 补充微调，SCSS 只做脏活。**

| 层级 | 职责 | 示例 |
|------|------|------|
| **BEM** | 组件语义锚点 | `profile-card`、`profile-card__avatar` |
| **UnoCSS** | 简单布局微调（≤3 个属性） | `flex items-center gap-16rpx` |
| **SCSS** | 伪类/动画/穿透等原子类做不了的事 | `@keyframes`、复杂选择器 |

### 选型决策

| 场景 | 方案 | 示例 |
|------|------|------|
| 简单布局（1-2 个属性） | 原子类 | `class="flex items-center"` |
| 组件样式（3+ 属性） | BEM | `class="profile-card"` |
| 重复出现的组合 | Shortcuts 收敛 | `class="flex-center"` |
| 主题相关 | CSS 变量 + BEM | `background: var(--color-primary)` |

```vue
<!-- ✅ 组件样式归 BEM，微调用原子类 -->
<view class="profile-card">
  <image class="profile-card__avatar" :src="user.avatar" />
  <text class="profile-card__name">{{ user.name }}</text>
</view>

<!-- ✅ 简单间距直接用原子类 -->
<view class="mt-16rpx p-4">
  <wd-search v-model="keywords" placeholder="搜索" />
</view>

<!-- ❌ 同一元素混用过多原子类 + BEM -->
<view class="profile-card flex flex-col p-28rpx bg-white rounded-28rpx shadow-sm">
</view>
```

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

```
block__element--modifier
 │      │          │
 │      │          └── 状态/变体（双连字符）
 │      └── 组成部分（双下划线）
 └── 页面前缀 + 功能实体
```

**Block 必须带页面前缀**：

```text
首页(index)  → home-hero / home-nav / home-stat
我的(mine)   → mine-hero / mine-profile / mine-menu
登录(login)  → login-form / login-brand / login-nav
工作台(work)  → work-menu / work-user / work-card
```

**Element 通用词汇**：

| 语义 | 标准词汇 | 禁止 |
|------|----------|------|
| 头部 | `__header` | ~~head~~ / ~~top~~ |
| 底部 | `__footer` | ~~bottom~~ |
| 标题 | `__title` | ~~name~~ / ~~heading~~ |
| 副标题 | `__subtitle` | ~~desc~~（描述时用 desc） |
| 描述 | `__desc` | ~~text~~ / ~~detail~~ |
| 图标 | `__icon` | ~~img~~ / ~~pic~~ |
| 图片 | `__image` | ~~img~~ / ~~bg~~ |
| 头像 | `__avatar` | ~~photo~~ |
| 标签 | `__tag` | ~~badge~~（徽章时用 badge） |
| 徽章 | `__badge` | ~~dot~~（圆点时用 dot） |
| 操作按钮 | `__action` | ~~action-btn~~ |
| 操作区 | `__actions` | ~~btns~~ |
| 内容 | `__body` | ~~content~~ |
| 输入框 | `__input` | ~~field~~（行容器时用 field） |
| 表单行 | `__field` | ~~row~~ |
| 列表 | `__list` | ~~items~~ |
| 列表项 | `__item` | ~~row~~ |

**SCSS 嵌套**：

```scss
.login {
  background: linear-gradient(135deg, var(--color-bg-tertiary), var(--color-primary-light));

  &__field {
    display: flex;
    align-items: center;

    &:focus-within {
      box-shadow: 0 0 0 2px var(--color-primary);
    }
  }

  &__code-btn {
    height: 64rpx;
    border-radius: 16rpx;

    &--active { color: var(--color-primary); background: var(--color-primary-light); }
    &--disabled { color: var(--color-text-placeholder); background: var(--color-bg-tertiary); }
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

## Part 7: 主题变量系统

### 变量速查

**主题色**：

| 语义 | Light | Dark | 变量 |
|------|-------|------|------|
| 主色 | `#4d80f0` | `#3b82f6` | `--color-primary` |
| 主色浅 | `#e8f0fe` | `#1e3a5f` | `--color-primary-light` |
| 主色深 | `#2563eb` | `#60a5fa` | `--color-primary-dark` |
| 成功 | `#34d19d` | `#34d399` | `--color-success` |
| 成功浅 | `#e6f7f1` | `#064e3b` | `--color-success-light` |
| 警告 | `#f0883a` | `#fbbf24` | `--color-warning` |
| 警告浅 | `#fff4e6` | `#78350f` | `--color-warning-light` |
| 危险 | `#ff4757` | `#f87171` | `--color-danger` |
| 危险浅 | `#fff1f2` | `#7f1d1d` | `--color-danger-light` |

**背景色**：

| 语义 | Light | Dark | 变量 |
|------|-------|------|------|
| 主背景 | `#ffffff` | `#1f2937` | `--color-bg` |
| 次背景 | `#f5f5f5` | `#111827` | `--color-bg-secondary` |
| 三级背景 | `#f1f5f9` | `#1e293b` | `--color-bg-tertiary` |

**文字色**：

| 语义 | Light | Dark | 变量 |
|------|-------|------|------|
| 主文字 | `#1f2937` | `#f9fafb` | `--color-text` |
| 次文字 | `#6b7280` | `#9ca3af` | `--color-text-secondary` |
| 占位文字 | `#9ca3af` | `#6b7280` | `--color-text-placeholder` |
| 禁用文字 | `#d1d5db` | `#4b5563` | `--color-text-disabled` |

**边框色**：

| 语义 | Light | Dark | 变量 |
|------|-------|------|------|
| 默认 | `#e5e7eb` | `#374151` | `--color-border` |
| 浅色 | `#f3f4f6` | `#1f2937` | `--color-border-light` |

### 使用原则

```scss
// ✅ 使用变量
.card { background: var(--color-bg); border: 1rpx solid var(--color-border); }

// ❌ 禁止硬编码
.card { background: #ffffff; border: 1rpx solid #e5e7eb; }
```

### Wot Design 变量桥接

项目在 `theme.scss` 中已桥接 Wot Design 变量：

```scss
--wot-color-theme:   var(--color-primary);
--wot-color-success: var(--color-success);
--wot-color-warning: var(--color-warning);
--wot-color-danger:  var(--color-danger);
--wot-color-bg:      var(--color-bg);
--wot-color-text:    var(--color-text);
--wot-color-text-secondary: var(--color-text-secondary);
--wot-color-text-placeholder: var(--color-text-placeholder);
--wot-color-border:  var(--color-border);
```

### z-index 层级

只有 `position: fixed/sticky/absolute` 且存在层叠竞争的元素才设 z-index，禁止魔法数字。

```scss
--z-dropdown: 100;   // 下拉菜单、Popover
--z-sticky:   200;   // 吸顶栏、吸底栏
--z-overlay:  300;   // 遮罩层
--z-popup:    400;   // Popup
--z-dialog:   500;   // Dialog / MessageBox
--z-toast:    600;   // Toast
--z-navbar:   700;   // 固定导航栏
--z-fab:      800;   // 浮动按钮
```

### 间距系统

项目在 `theme.scss` 中定义了语义间距变量：

| 语义 | 值 | 场景 |
|------|-----|------|
| xxs | `4rpx` | 极小间距 |
| xs | `8rpx` | 图标与文字 |
| sm | `16rpx` | 模块间分隔 |
| md | `24rpx` | 卡片内边距 |
| lg | `32rpx` | 页面左右边距 |
| xl | `40rpx` | 大区块内边距 |
| xxl | `48rpx` | 大区块分隔 |

语义别名：`--spacing-page: 32rpx`、`--spacing-card: 24rpx`、`--spacing-section: 24rpx`、`--spacing-element: 16rpx`、`--spacing-compact: 8rpx`

### 圆角

| 场景 | 值 |
|------|-----|
| 小元素（按钮、标签） | `8rpx` |
| 中元素（输入框） | `16rpx` |
| 大卡片 | `24rpx` |
| 圆形 | `50%` |

### 阴影

| 级别 | Light | Dark |
|------|-------|------|
| sm | `0 2rpx 4rpx rgba(0,0,0,0.05)` | `0 2rpx 4rpx rgba(0,0,0,0.2)` |
| md | `0 8rpx 12rpx rgba(0,0,0,0.1)` | `0 8rpx 12rpx rgba(0,0,0,0.3)` |
| lg | `0 20rpx 30rpx rgba(0,0,0,0.1)` | `0 20rpx 30rpx rgba(0,0,0,0.4)` |

### 字体

| 场景 | 字号 | 字重 | 行高 |
|------|------|------|------|
| 辅助信息 | `24rpx` | 400 | 1.4 |
| 次要文字 | `26rpx` | 400 | 1.5 |
| 正文 | `28rpx` | 400 | 1.6 |
| 小标题 | `30rpx` | 600 | 1.4 |
| 标题 | `32rpx` | 600 | 1.3 |
| 大标题 | `36rpx` | 700 | 1.2 |

### UnoCSS Shortcuts

| 快捷类 | 展开 | 用途 |
|--------|------|------|
| `flex-center` | `flex justify-center items-center` | 居中 |
| `flex-between` | `flex justify-between items-center` | 两端对齐 |
| `flex-start` | `flex justify-start items-center` | 起始对齐 |
| `flex-col-center` | `flex flex-col items-center` | 纵向居中 |

**新增 Shortcut 条件**：全局出现 ≥ 5 次 + 有明确语义 + 不含业务色值。

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

### WXSS 限制

| 特性 | 状态 | 替代 |
|------|------|------|
| `:deep()` | 编译后残留，WXSS 报错 | 组件 `custom-class` + 非 scoped style |
| `[attr*="val"]` | 不支持 | 具体类名 |
| `:has()` / `:is()` | 不支持 | 具体选择器 |
| `::before` / `::after` | 小程序不支持 | 真实元素 |

### 覆盖组件样式

```vue
<!-- ✅ custom-class + 非 scoped 覆盖组件样式 -->
<wd-card custom-class="my-card">
  <template #title>标题</template>
</wd-card>

<style lang="scss">
.my-card { margin: 0 !important; }
.my-card .wd-card__body { padding: 24rpx !important; }
</style>
```

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

## Part 10: 反模式

| 反模式 | 正确做法 |
|--------|---------|
| 硬编码颜色值 | 使用 `var(--color-*)` CSS 变量 |
| 普通流内容设 z-index | 只有定位元素才设 |
| `::deep([attr*="val"])` | 使用具体类名 |
| Block 名过于通用（如 `.card`） | 使用页面/功能前缀（如 `.mine-card`） |
| `height: 100vh` | 使用 `min-height: 100vh` |
| z-index 魔法数字 | 使用 `var(--z-*)` |
| SCSS 大量 `@apply` | 原子类留在模板 |
| AI 蓝紫渐变 | 纯色或品牌色 |
| Emoji 当图标 | SVG 图标或 Wot Design Icon |
| 过度装饰（阴影+渐变+动画） | 最多一个装饰效果 |
| 同一元素原子类 >3 个 | 归入 BEM 类 |

---

## Part 11: 代码质量检查清单

- [ ] 页面根元素 `class="page"` + `min-height: 100vh`
- [ ] 使用 wot-design-uni 组件（不自定义）
- [ ] BEM 类名带页面前缀
- [ ] 原子类 ≤3 个，>3 个用 BEM
- [ ] 所有颜色使用 CSS 变量
- [ ] 所有尺寸使用 rpx 单位
- [ ] 暗黑模式正常工作
- [ ] 卡片控制在 3-4 行
- [ ] 信息层级清晰
- [ ] 操作使用 `···` 更多按钮
- [ ] 使用 UnoCSS 预设快捷类
- [ ] z-index 使用 `var(--z-*)`
- [ ] 定义 TypeScript 类型
- [ ] 避免伪元素和 DOM API
