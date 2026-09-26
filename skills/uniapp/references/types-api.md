# 类型与接口

## 类型命名

手写类型按用途命名；DTO/VO 并非前端禁用词，但本项目不另建这套后缀。生成 SDK 保留原名。

| 用途 | 名称 |
| --- | --- |
| 列表记录 / 详情 | `UserItem` / `UserDetail` |
| 查询 / 分页查询 | `UserQuery` / `UserPageQuery` |
| 页面编辑模型 | `UserForm` |
| 新增 / 修改提交参数 | `CreateUserParams` / `UpdateUserParams` |
| 操作结果 | `LoginResult` |
| 业务对象 | `UserInfo`、`UserProfile` |
| 选项 / 树节点 | `SelectOption` / `DeptNode` |
| 分页结果 / 接口响应 | `PageResult<T>` / `ApiResponse<T>`，字段对应真实协议 |

用途或约束相同时共用类型，不造空壳；新增与更新参数相同可用 `SaveUserParams`。同类类型不混用 Data/Resp/Response 表达同一用途。

## 声明与存放

- 对象字段、Props、API 参数和响应使用 `interface`；联合、字面量、派生及 schema 推导类型使用 `type`。
- 必填不加问号；可选、null 分别遵守接口。有限状态用字面量联合或现有枚举。
- 外部未知值用 `unknown` 并收窄；不靠 `any` 跳过检查。局部变量保留合理推断。
- ID、枚举、时间格式遵守后端契约；字符串 Long ID 不转 Number，时间字符串不声明为 Date。
- 对外 API 标明参数与返回类型；类型使用 `import type` 导入，业务类型不声明为全局 namespace。

| 范围 | 位置 |
| --- | --- |
| 单个 API 模块请求/响应 | 与 `api/user.ts` 同文件 |
| 同域多个 API 文件共用 | `api/user.types.ts` |
| 页面私有编辑模型 | 页面内 `types.ts` |
| 跨业务协议 | `types/api.ts` |
| 环境、平台、生成声明 | 工程指定的 `.d.ts` 文件 |

## API 边界

- 读取使用 `getList/getPage/getOptions/getDetail`；编辑回填统一沿用 `getFormData` 或既有 `getForm`，不得与详情混同。
- 写入使用 `create/update/delete`；按 ID 批量删除用 `deleteByIds`，同类方法不混用同义词。
- 方法省去 API 对象已有的业务前缀；当前用户与其他对象、多种登录方式等范围必须区分。
- 业务 API 只负责通信；提示、导航和业务状态更新归调用流程。请求封装统一解包，页面不重复读取 data。
- 提交显式选择可写字段；不提交表单中的确认项、展示名称等额外字段。
- 前端改名不改变 URL、JSON 字段和枚举值；转换必须显式映射，不能只改类型声明。
- 请求与上传使用相同平台策略：H5 可走部署代理，App/小程序使用完整服务地址。

依据：[TypeScript 类型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)。后缀和 interface/type 分工采用本项目约定。
