# youlai-skills

全栈开发 AI Agent Skills，包含 Vue3 管理后台、UniApp 移动端、6 大后端框架（Spring Boot、NestJS、Gin、Django、ThinkPHP、ASP.NET Core）及 MySQL 数据库设计规范。

## Skills 列表

### 前端开发

| Skill | 描述 | 技术栈 |
| --- | --- | --- |
| [`vue-admin`](https://skills.sh/youlaitech/youlai-skills/vue-admin) | Vue3 管理后台开发规范 | Vue3 + Element Plus + TypeScript |
| [`uniapp`](https://skills.sh/youlaitech/youlai-skills/uniapp) | UniApp 移动端开发规范 | UniApp + Vue3 + wot-design-uni |

### 后端开发

| Skill | 描述 | 技术栈 |
| --- | --- | --- |
| [`spring-boot`](https://skills.sh/youlaitech/youlai-skills/spring-boot) | Java 后端开发规范 | Spring Boot + MyBatis-Plus |
| [`nestjs`](https://skills.sh/youlaitech/youlai-skills/nestjs) | Node.js 后端开发规范 | NestJS + TypeORM |
| [`gin`](https://skills.sh/youlaitech/youlai-skills/gin) | Go 后端开发规范 | Gin + GORM |
| [`django`](https://skills.sh/youlaitech/youlai-skills/django) | Python 后端开发规范 | Django REST Framework |
| [`thinkphp`](https://skills.sh/youlaitech/youlai-skills/thinkphp) | PHP 后端开发规范 | ThinkPHP 8 |
| [`aspnet`](https://skills.sh/youlaitech/youlai-skills/aspnet) | .NET 后端开发规范 | ASP.NET Core 8 |

### 数据库

| Skill | 描述 | 技术栈 |
| --- | --- | --- |
| [`mysql-design`](https://skills.sh/youlaitech/youlai-skills/mysql-design) | MySQL 数据库设计规范 | MySQL 8 |

## 安装

```bash
npx skills add youlaitech/youlai-skills
```

## 使用方式

Skills 根据项目类型自动触发：

| 项目类型          | 触发 Skill     |
| ----------------- | -------------- |
| Vue3 管理后台     | `vue-admin`    |
| UniApp 移动端     | `uniapp`       |
| Java Spring Boot  | `spring-boot`  |
| Node.js NestJS    | `nestjs`       |
| Go Gin            | `gin`          |
| Python Django     | `django`       |
| PHP ThinkPHP      | `thinkphp`     |
| .NET ASP.NET Core | `aspnet`       |
| 数据库设计        | `mysql-design` |

## 每个 Skill 包含

- **技术栈版本** - 推荐的技术栈及版本
- **目录结构** - 标准的项目目录结构
- **命名规范** - 文件、类、方法、变量命名规则
- **RESTful API 规范** - 标准 CRUD 路径设计
- **响应格式** - 统一的 Result/PageResult 响应
- **实体/模型规范** - 基础实体、字段定义
- **认证规范** - JWT/Redis Token 认证
- **代码质量检查清单** - 开发完成检查项

## 发布到 skills.sh

1. 创建 GitHub 仓库 `youlaitech/youlai-skills`
2. 推送代码到 main 分支
3. 访问 `https://skills.sh/youlaitech/youlai-skills` 即可被索引

## 本地开发

```bash
# 克隆仓库
git clone https://github.com/youlaitech/youlai-skills.git

# 目录结构
youlai-skills/
├── README.md
└── skills/
    ├── vue-admin/SKILL.md
    ├── uniapp/SKILL.md
    ├── spring-boot/SKILL.md
    ├── nestjs/SKILL.md
    ├── gin/SKILL.md
    ├── django/SKILL.md
    ├── thinkphp/SKILL.md
    ├── aspnet/SKILL.md
    └── mysql-design/SKILL.md
```
