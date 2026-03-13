---
name: youlai-app
description: Uni-app mobile development guidelines for admin systems. Use this skill when working on youlai-app project, building mobile admin pages, implementing wot-design-uni components, or mobile UI/UX design.
---

# Youlai App - Mobile Development Guide

Development guidelines for Uni-app + Vue3 + wot-design-uni mobile admin application.

## Trigger Conditions

Use this skill when:
- Working on `youlai-app` project
- Building mobile admin pages
- Implementing wot-design-uni components
- Designing mobile UI/UX
- Handling mobile-specific features (camera, location, etc.)

---

## Part 0: Mandatory Rules (Must Follow)

### 0.1 Component Priority

**Use wot-design-uni official components, never custom when available**

| Need | Use | Don't |
|------|-----|-------|
| Button | `<wd-button>` | Custom `.btn` |
| Input | `<wd-input>` | Custom `.input` |
| Card | `<wd-card>` / `<wd-cell-group>` | Custom `.card` |
| Dialog | `<wd-popup>` / `<wd-dialog>` | Custom modal |
| List | `<wd-cell>` / `<wd-grid>` | Custom list |
| Loading | `<wd-loading>` / `<wd-skeleton>` | Custom animation |
| Empty | `<wd-status>` | Custom empty state |

### 0.2 CSS Unification Rule

**Never duplicate styles, always reuse**

```scss
// ❌ Wrong: Define in each component
.user-card { padding: 24rpx; background: var(--color-bg); }
.role-card { padding: 24rpx; background: var(--color-bg); }

// ✅ Correct: Unified base class
.base-card { padding: 24rpx; background: var(--color-bg); border-radius: 16rpx; }
```

### 0.3 Dark Mode Requirement

**All colors must use CSS variables, no hardcoded values**

```scss
// ❌ Wrong: Hardcoded colors
.card { background: #ffffff; color: #333333; }

// ✅ Correct: CSS variables
.card { background: var(--color-bg); color: var(--color-text); }
```

---

## Part 1: Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Vue | 3.4+ | Frontend framework |
| Uni-app | 3.0+ | Cross-platform framework |
| wot-design-uni | 1.3+ | UI component library |
| TypeScript | 5.0+ | Type safety |
| UnoCSS | 0.58+ | Atomic CSS |
| Pinia | 2.1+ | State management |

---

## Part 2: CSS Usage Boundary

**UnoCSS atomic classes + BEM naming hybrid approach**

| Atomic Class Count | Approach |
|-------------------|----------|
| ≤3 classes | UnoCSS atomic classes |
| >3 classes | BEM CSS class |

### Merge Rule (When Element Has Class Name)

```vue
<!-- ❌ Wrong: Class name + atomic classes mixed -->
<view class="filter-bar flex items-center mt-12rpx">

<!-- ✅ Correct: Merge atomic classes into CSS class -->
<view class="filter-bar">
```

```scss
.filter-bar {
  display: flex;      // merged flex
  align-items: center; // merged items-center
  margin-top: 12rpx;   // merged mt-12rpx
}
```

### Use Shortcuts When Available

| Atomic Classes | Shortcut | Reduction |
|---------------|----------|-----------|
| `flex items-center` | `flex-start` | 2→1 |
| `flex justify-between items-center` | `flex-between` | 3→1 |
| `flex justify-center items-center` | `flex-center` | 3→1 |

---

## Part 3: BEM Naming Convention

```
Block__Element--Modifier
```

```scss
.user-card {
  background-color: var(--color-bg);
  border-radius: 24rpx;
  padding: 32rpx;

  &__header {
    display: flex;
    align-items: center;
  }

  &__avatar {
    width: 96rpx;
    height: 96rpx;
    border-radius: 50%;
  }

  &__name {
    font-size: 32rpx;
    font-weight: 600;
    color: var(--color-text);
  }

  &--highlight {
    background: linear-gradient(135deg, #2563eb 0%, #3b82f6 100%);
  }
}
```

---

## Part 4: Page Templates

### List Page Template

```vue
<template>
  <view class="page">
    <!-- Search Bar -->
    <view class="search-bar">
      <wd-search v-model="keyword" placeholder="搜索" @search="handleSearch" />
    </view>

    <!-- Filter Tabs -->
    <view class="filter-tabs">
      <wd-tabs v-model="activeTab" @change="handleTabChange">
        <wd-tab name="all" title="全部" />
        <wd-tab name="enabled" title="启用" />
        <wd-tab name="disabled" title="禁用" />
      </wd-tabs>
    </view>

    <!-- List -->
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

    <!-- Floating Action Button -->
    <wd-fab icon="add" @click="handleAdd" />
  </view>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";
import { onLoad } from "@dcloudio/uni-app";

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

function handleSearch() { fetchList(true); }
function handleTabChange() { fetchList(true); }
function handleRefresh() { refreshing.value = true; fetchList(true); }
function handleLoadMore() { if (hasMore.value) { pageNum.value++; fetchList(); } }

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
</style>
```

---

## Part 5: Card Design Standards

### Information Hierarchy

**Mobile card ≠ Web table**

| Level | Content | Visual Weight | Example |
|-------|---------|---------------|---------|
| Primary | Identity | Most prominent | Avatar + Name + Status |
| Secondary | Position | Medium | Role + Department |
| Auxiliary | Contact | Weaker | Phone + Email |
| Meta | Timestamp | Weakest | Created time |

### Card Template

```vue
<template>
  <view class="user-card">
    <!-- Primary Info Row -->
    <view class="user-card__header">
      <wd-img :src="user.avatar" width="80rpx" height="80rpx" round />
      <view class="user-card__identity">
        <text class="user-card__name">{{ user.nickname }}</text>
        <text class="user-card__role">{{ user.roleName }} · {{ user.deptName }}</text>
      </view>
      <wd-tag :type="user.status === 1 ? 'success' : 'danger'" plain>
        {{ user.status === 1 ? '正常' : '禁用' }}
      </wd-tag>
    </view>

    <!-- Auxiliary Info Row -->
    <view class="user-card__contact">
      <text>📱 {{ user.mobile }}</text>
      <text v-if="user.email">📧 {{ user.email }}</text>
    </view>
  </view>
</template>

<style lang="scss" scoped>
.user-card {
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
}
</style>
```

### Card Rules

- **Keep cards under 3-4 lines**
- **Use `·` to separate secondary info**
- **Icon prefix for auxiliary info: 📱📧**
- **Lightweight actions: `···` more button**

---

## Part 6: Dark Mode CSS Variables

```scss
:root,
page {
  /* Background */
  --color-bg: #ffffff;
  --color-bg-secondary: #f5f5f5;
  --color-bg-tertiary: #ebebeb;

  /* Text */
  --color-text: #1f2937;
  --color-text-secondary: #6b7280;
  --color-text-placeholder: #9ca3af;

  /* Border */
  --color-border: #e5e7eb;
  --color-divider: #f3f4f6;

  /* Functional */
  --color-primary: #2563eb;
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;
  
  /* Light variants for gradients */
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

## Part 7: rpx Unit Standards

- Design base: 750rpx
- Font sizes: even values `24rpx`, `26rpx`, `28rpx`, `32rpx`

| Scenario | Size | Description |
|----------|------|-------------|
| Auxiliary | `24rpx` | Labels, descriptions, footer |
| Secondary | `26rpx` | Buttons, filters |
| Base | `28rpx` | Body text, list items |
| Title | `32rpx` | Page titles, important info |

- Spacing: multiples of 4 `16rpx`, `24rpx`, `32rpx`

---

## Part 8: Code Quality Checklist

- [ ] Use wot-design-uni components (not custom)
- [ ] CSS classes ≤3 atomic classes, >3 use BEM
- [ ] All colors use CSS variables
- [ ] Dark mode works correctly
- [ ] Cards under 3-4 lines
- [ ] Information hierarchy clear
- [ ] Actions use `···` more button
- [ ] Use UnoCSS shortcuts when available
- [ ] rpx units for all sizes
- [ ] TypeScript types defined
