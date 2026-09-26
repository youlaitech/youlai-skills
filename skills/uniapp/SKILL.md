---
name: uniapp
description: uni-app（Vue 3、TypeScript、Wot UI、UnoCSS）的开发、重构与代码审查规范，涵盖命名、类型、注释、样式、页面分包与状态管理。适用于 Vue 页面，不直接用于 nvue 或 uni-app x。
---

# uni-app 开发规范

先核对安装版本、目标平台和改动范围，再按任务读取 reference。Wot v1/v2 不混用；不因执行规范擅自升级依赖或添加插件。

框架机制以所链接文档和安装版本为准；本文选定的命名、注释风格与样式分工属于项目约定，已有明确仓库规范优先。

## 按需读取

| 任务 | 规范 |
| --- | --- |
| 文件、变量、函数、Store、事件 | [命名](references/naming.md) |
| 请求与响应类型、API 边界 | [类型与接口](references/types-api.md) |
| 注释、TSDoc、TODO、编译指令 | [注释](references/comments.md) |
| UnoCSS、BEM、SCSS、主题 | [样式](references/styles.md) |
| 目录、分包、页面配置、布局 | [页面与布局](references/pages-layout.md) |
| 表单、分页、会话、缓存、验收 | [质量检查](references/quality.md) |

## 改动边界

- 规范应用于本次改动；不附带全仓改名或修改第三方、自动生成文件。
- 只更新规范时不修改应用代码；本 skill 不收录项目重构清单或业务伪代码。
