# 页面与布局

## 目录责任

| 位置 | 内容 |
| --- | --- |
| `pages` / 既有分包目录 | 主包 / 分包页面入口 |
| 页面内 `components`、`composables`、`schemas.ts` | 私有组件、状态流程、校验规则 |
| 全局 `components` / `composables` | 已有跨页面调用的共享能力 |
| `store` | 会话、主题等跨页状态；页面临时状态留在本地 |
| `utils` | 纯转换与通用底层函数 |

页面组织交互；展示组件接收 props、发出事件，不直接修改 props；组合函数封装有状态流程。业务依赖显式导入。

## 页面配置

已安装 uni-pages 时遵守下表；其他工程继续维护 pages.json。

| 内容 | 维护入口 |
| --- | --- |
| 扫描、排除、分包根、声明输出 | Vite 中的 UniPages 配置 |
| 全局窗口、TabBar、预下载 | `pages.config.ts` |
| 页面标题、导航、刷新、layout、meta | 页面 `definePage`；已有 route-block 沿用原方式 |
| pages.json、自动导入声明 | 生成结果，不手改 |

- `definePage` 不引用运行时 ref/store；一个页面只维护一套元信息。
- 私有组件排除页面扫描；路由引用和 TabBar 路径一致，声明的 name 唯一。
- 迁移生成方式保留原配置；类型检查前通过工程脚本生成必需声明，不依赖手动启动 dev。
- 权限 meta 必须有类型及守卫消费，不视为框架自动鉴权。

依据：[uni-pages 配置](https://uni-helper.js.org/vite-plugin-uni-pages/option/)、[definePage](https://uni-helper.js.org/vite-plugin-uni-pages/definepage/)。

## 分包

- 以配置 root 为准，父子目录不得重复注册为分包根。
- 普通微信小程序 TabBar 页面放主包，普通分包之间不直接引用私有资源。
- 公共组件不反向依赖分包页面；构建后核对共享依赖位置、包体积及目标平台限制。
- 预下载只覆盖明确的后续路径；H5 独立检查 chunk，独立分包遵守平台规则。

依据：[uni-app 分包](https://uniapp.dcloud.net.cn/collocation/pages.html#subpackages)。

## 布局与导航

- Provider 提供 Wot 配置及反馈宿主；layout 管理页面边距和底栏占位；navbar 管理顶部几何和占位；弹层处理自身底部安全区。同一空间只预留一次。
- 使用自动布局后不再手工重复包裹；关闭布局时补齐必需宿主。
- 普通固定导航启用占位，沉浸式页面显式关闭；原生、自定义导航不重复显示。
- 页面或导航服务负责返回确认、空栈回退；自定义返回接管后，组件不得再无条件跳转。
- 导航测量处理无效值、窗口变化及监听清理，标题避开两侧操作与胶囊。
- 页面滚动与 scroll-view 选一个主滚动容器；刷新/触底绑定对应容器，纵向 scroll-view 具备确定高度约束。
- 长表单在键盘弹出后仍能访问输入与提交区；验证系统返回、浏览器返回和安全区。

依据：[uni-layouts](https://github.com/uni-helper/vite-plugin-uni-layouts)。
