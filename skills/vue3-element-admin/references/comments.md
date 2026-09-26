# 注释

注释形式按「是否对外暴露」选择，不按重要程度选择。`/** */`（JSDoc）有工具链语义（IDE 悬浮提示、文档生成），只写在会被外部消费的成员上；组件内部私有实现用单行 `//`，避免文档注释泛滥稀释公开 API。

## 形式选择

| 成员 | 注释形式 |
| --- | --- |
| 函数/方法（无论是否导出）：API 模块方法、composable 内部函数、页面 `<script setup>` 私有函数 | 多行 `/** */` |
| 字段与 props：props 属性、interface/type 字段、枚举成员 | 单行 `/** 说明 */` |
| 对外暴露：`export` 的类型 / 类 / 常量对象、`defineProps` / `defineEmits` / `defineExpose` / `defineModel` | 多行 `/** */` |
| 变量与行内说明：`computed`、`ref`、局部变量、执行逻辑说明 | `//` |
| 对象字面量属性：局部配置对象、请求参数对象等实现细节 | `//`（**不属于"字段"**，不加 JSDoc）|

> 只有**声明**（函数、类型、interface 字段、props 属性、枚举成员）才用 JSDoc；**对象字面量属性是运行时的局部数据**，JSDoc 文档工具与 IDE 签名提示都不消费它，用 `//` 即可（对齐 Google TypeScript Style Guide 对 documentation comment / implementation comment 的划分）。

```ts
// 正确：props（对外暴露）用 JSDoc
interface Props {
  /** 契约 images：图片数组，数组顺序即展示顺序，首图即默认封面 */
  images: ProductImage[];
}

// 正确：内部 computed 用单行注释
// 是否可提交：选齐 + 可售 + 库存已知且足量 + 数量合法
const canConfirm = computed(() => ...);

// 错误：私有成员写 JSDoc，公开 API 的文档提示被稀释
/** 是否可提交：选齐 + 可售 + 库存已知且足量 + 数量合法 */
const canConfirm = computed(() => ...);

// 正确：函数签名用多行 JSDoc，页面私有函数也不例外（IDE 悬停可读）
/**
 * 删除角色，删除后右侧联动到剩余角色
 *
 * @param roleId 角色 ID
 */
async function handleDelete(roleId?: string) { ... }

// 错误：函数签名用行注释，IDE 悬停无文档
// 删除角色，删除后右侧联动到剩余角色
async function handleDelete(roleId?: string) { ... }
```

本项目里「对外暴露」还包括下面几种，它们的成员同样用 JSDoc：

- API 模块：`const RoleAPI = {...}; export default RoleAPI`，对象内的方法在调用处有 IDE 提示。
- 聚合导出对象：`const WorkflowAPI = { model: ModelAPI, ... }; export default WorkflowAPI`，被它引用的 `ModelAPI` 等对象也算对外暴露。
- 导出类与导出枚举：`Storage` 的静态方法、`ThemeMode` 的成员。

符号级注释用标准多行格式（`/**` 首行、` * 内容`、 ` */` 尾行），字段与 props 用单行 `/** 说明 */` 置于上方。`@param` / `@returns` 只在类型没表达清楚时补，不复述参数名。

## 页面与组件文件

`src/views/**`、`src/components/**` 的 `<script setup>` 内部，按成员性质分流，跟「这个文件会不会被别的页面 import」无关：

| 成员 | 形式 |
| --- | --- |
| 函数/方法：`function fn()`、`const fn = () => {}`、`const fn = useDebounceFn(async () => {}, 300)` | 多行 `/** */` |
| 页面/组件内部状态：`ref`、`reactive`、`computed`、`watch`、局部常量 | `//` |
| `defineProps` / `defineEmits` / `defineExpose` | 多行 `/** */` |
| 模板节点、`onMounted` 等生命周期里的执行逻辑 | `//`，或直接不写 |

这里最容易出错的是把方法签名降级成 `//`，连写两三行充当天花板：

```ts
// 错误：行注释冒充方法文档，IDE 悬浮看不到签名说明
// 删除单个或批量用户
// 安全检查：禁止删除当前登录用户
// 参数 id：指定时删除单个用户；不指定时删除表格勾选项
async function handleDelete(id?: string) { ... }
```

```ts
/**
 * 删除单个或批量用户，删除前拦住"把自己删掉"这条路径
 *
 * @param id 指定时按单个用户删除，不指定时删除表格勾选项
 */
async function handleDelete(id?: string) { ... }
```

方法一律要写注释：标题说明方法干什么就够了，"打开新增弹窗""加载表单选项"这类动作描述是合格的，不因为方法名已经表达出来就省略。有前置条件、副作用或约束时，在标题下再补一行；没有就只留标题。

## 文案

- 解释"为什么、约束和副作用"，不复述代码；自解释的局部函数、赋值、模板节点不写注释。
- 实现内部的原因与陷阱用 `//` 就地压成一行；不用 JSDoc 给 `onMounted`、`watch` 等执行逻辑开场。
- 不维护 `@author`、`@since`、日期、变更流水；历史交给版本控制，开源协议与生成器要求的头注释保留。
- 行为变更时同步更新相关注释，删除过期说明；不保留注释掉的旧代码，演示页里用于讲解用法的示例除外。

## 批量整理

按形式批量改写历史注释时，先确认成员是否对外暴露：API 模块对象里的方法、导出类的成员、导出枚举的成员容易被误判成内部成员而降级为 `//`，导致 IDE 悬浮提示丢失。
