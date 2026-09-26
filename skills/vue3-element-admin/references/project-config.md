# 项目配置

修改配置时先读取当前 `src/settings.ts`、`src/constants/index.ts`、环境文件和对应 Store。本文说明职责，不复制完整配置清单。

## `settings.ts`

- `appConfig` 保存来自构建信息和环境变量的应用级配置，例如名称、版本、标题和功能开关。
- `defaults` 保存主题、布局、组件尺寸、语言、标签栏、水印和动画等 UI 默认值。
- 用户可修改的设置由 Store 持久化；不要直接修改 `defaults` 充当运行时状态。
- 增加设置项时同步处理类型、默认值、存储键、Store、设置界面以及实际应用逻辑。

## 环境变量

当前项目常用变量包括：

| 变量                      | 用途               |
| ------------------------- | ------------------ |
| `VITE_APP_PORT`           | 开发服务端口       |
| `VITE_APP_TITLE`          | 应用标题           |
| `VITE_APP_BASE_API`       | 请求前缀           |
| `VITE_APP_API_URL`        | 代理目标或接口地址 |
| `VITE_MOCK_DEV_SERVER`    | 开发 Mock 开关     |
| `VITE_APP_TENANT_ENABLED` | 多租户开关         |

新增变量时同时更新环境类型声明和需要的环境文件；不要把密钥或生产凭据提交到 Skill、源码或示例中。

## 存储键

所有本地和会话存储键集中在 `STORAGE_KEYS`，以 `APP_PREFIX` 构造。新增持久化项时复用该常量，不在业务代码中散落字符串；变更既有键时考虑旧数据迁移或兼容清理。

`ROLE_ROOT` 等跨模块常量同样从 `src/constants/index.ts` 引用，不重复声明。

## 主题

主题调色板集中在 `settings.ts`，主题相关组件样式优先使用项目或 Element Plus 变量。品牌色、图表色和状态色可以在集中配置中使用明确颜色，不要把“禁止硬编码颜色”理解成禁止定义主题色值。
