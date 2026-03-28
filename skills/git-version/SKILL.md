---
name: git-version
description: Git 版本管理自动化规范。当需要自动生成 Commit Message、管理版本号、创建 Tag、生成 Changelog 或 Release Notes 时使用此 skill。
---

# Git 版本管理自动化规范

## 触发条件

- 需要生成规范的 Commit Message
- 需要管理语义化版本号
- 需要创建 Git Tag
- 需要生成 Changelog
- 需要生成 Release Notes
- 需要自动化版本发布流程

---

## Part 1: Commit Message 规范

### 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 类型

| Type       | 说明                          | 示例                                 |
| ---------- | ----------------------------- | ------------------------------------ |
| `feat`     | 新功能                        | `feat(user): 添加用户登录功能`       |
| `fix`      | Bug 修复                      | `fix(auth): 修复 Token 过期验证问题` |
| `docs`     | 文档更新                      | `docs(readme): 更新安装说明`         |
| `style`    | 代码格式（不影响功能）        | `style: 格式化代码缩进`              |
| `refactor` | 重构（非新功能、非 Bug 修复） | `refactor(user): 重构用户服务层`     |
| `perf`     | 性能优化                      | `perf(query): 优化分页查询性能`      |
| `test`     | 测试相关                      | `test(user): 添加用户服务单元测试`   |
| `chore`    | 构建/工具相关                 | `chore: 更新依赖版本`                |
| `ci`       | CI/CD 相关                    | `ci: 添加 GitHub Actions 工作流`     |
| `build`    | 构建系统相关                  | `build: 升级 webpack 配置`           |
| `revert`   | 回滚提交                      | `revert: 回滚用户登录功能`           |

### Scope 范围

按模块/功能区域划分：

| Scope    | 说明         |
| -------- | ------------ |
| `auth`   | 认证授权模块 |
| `user`   | 用户模块     |
| `role`   | 角色模块     |
| `menu`   | 菜单模块     |
| `dept`   | 部门模块     |
| `dict`   | 字典模块     |
| `notice` | 通知公告模块 |
| `log`    | 日志模块     |
| `file`   | 文件模块     |
| `api`    | API 接口     |
| `db`     | 数据库       |
| `config` | 配置         |
| `deps`   | 依赖         |

### 自动生成 Commit Message 流程

1. **分析变更**

   ```bash
   git diff --cached --stat
   git diff --cached --name-status
   ```

2. **识别变更类型**
   - 新增文件 → `feat`
   - 修改业务逻辑 → `feat` 或 `fix`
   - 修改配置/文档 → `docs` 或 `chore`
   - 删除文件 → `refactor` 或 `fix`

3. **生成 Message**

   ```
   feat(user): 添加用户头像上传功能

   - 支持 JPG/PNG 格式
   - 限制文件大小 2MB
   - 自动压缩图片
   ```

---

## Part 2: 语义化版本规范

### 版本格式

```
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

| 部分       | 说明       | 更新时机              |
| ---------- | ---------- | --------------------- |
| MAJOR      | 主版本号   | 不兼容的 API 变更     |
| MINOR      | 次版本号   | 向后兼容的功能新增    |
| PATCH      | 修订号     | 向后兼容的问题修复    |
| PRERELEASE | 预发布标识 | `alpha`, `beta`, `rc` |
| BUILD      | 构建元数据 | 构建信息              |

### 版本递增规则

| Commit Type                                  | 版本递增 |
| -------------------------------------------- | -------- |
| `feat`                                       | MINOR    |
| `fix`                                        | PATCH    |
| `feat!` 或 `BREAKING CHANGE`                 | MAJOR    |
| `docs`, `style`, `refactor`, `test`, `chore` | 不递增   |

### 预发布版本

| 阶段              | 版本示例        | 说明         |
| ----------------- | --------------- | ------------ |
| Alpha             | `1.0.0-alpha.1` | 内部测试版本 |
| Beta              | `1.0.0-beta.1`  | 公开测试版本 |
| Release Candidate | `1.0.0-rc.1`    | 候选发布版本 |
| Release           | `1.0.0`         | 正式发布版本 |

### 版本号来源

优先级（从高到低）：

1. **手动指定** - 用户明确指定版本号
2. **package.json** - 前端/Node.js 项目（`version` 字段）
3. **pom.xml** - Java Maven 项目（`<version>` 标签）
4. **Cargo.toml** - Rust 项目
5. **pyproject.toml** - Python 项目
6. **git tag** - 最新 Tag 推算（无版本文件时）

### 自动版本递增

```bash
# 获取当前版本（优先从 package.json）
cat package.json | grep '"version"'

# 或从 git tag 获取
git describe --tags --abbrev=0 2>/dev/null || echo "0.0.0"

# 根据 Commit 分析递增
# feat → MINOR+1, PATCH=0
# fix → PATCH+1
# feat! 或 BREAKING CHANGE → MAJOR+1, MINOR=0, PATCH=0
```

### 版本文件更新

创建 Tag 后必须更新版本文件：

| 项目类型   | 版本文件          | 更新方式              |
| ---------- | ----------------- | --------------------- |
| Node.js    | `package.json`    | 修改 `version` 字段   |
| Java Maven | `pom.xml`         | 修改 `<version>` 标签 |
| Python     | `pyproject.toml`  | 修改 `version` 字段   |
| Go         | `version.go` 或无 | 可选或跳过            |

---

## Part 3: Git Tag 规范

### Tag 命名格式

| 类型       | 格式                          | 示例            |
| ---------- | ----------------------------- | --------------- |
| 正式版本   | `v{VERSION}`                  | `v1.0.0`        |
| 预发布版本 | `v{VERSION}-{PRERELEASE}.{N}` | `v1.0.0-beta.1` |

### Tag 注解

```bash
# 创建带注释的 Tag
git tag -a v1.0.0 -m "Release v1.0.0

Features:
- 用户登录功能
- 角色权限管理

Fixes:
- 修复 Token 过期问题
"
```

### Tag 操作

```bash
# 列出所有 Tag
git tag -l

# 列出匹配的 Tag
git tag -l "v1.*"

# 查看 Tag 详情
git show v1.0.0

# 删除本地 Tag
git tag -d v1.0.0

# 删除远程 Tag
git push origin --delete v1.0.0

# 推送 Tag 到远程
git push origin v1.0.0

# 推送所有 Tag
git push origin --tags
```

---

## Part 4: Changelog 规范

### 文件命名

- `CHANGELOG.md` - 主变更日志
- `CHANGES.md` - 备选命名

### 格式模板

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- 待发布的新功能

### Changed

- 待发布的变更

### Deprecated

- 即将废弃的功能

### Removed

- 已移除的功能

### Fixed

- 待发布的修复

### Security

- 安全相关修复

## [1.0.0] - 2024-01-15

### Added

- 用户登录功能
- 角色权限管理
- 菜单动态路由

### Fixed

- 修复 Token 过期验证问题 (#123)

### Security

- 升级 JWT 库修复安全漏洞

## [0.9.0] - 2024-01-01

### Added

- 初始版本发布
```

### 变更分类

| 分类       | 说明     |
| ---------- | -------- |
| Added      | 新增功能 |
| Changed    | 功能变更 |
| Deprecated | 即将废弃 |
| Removed    | 已移除   |
| Fixed      | Bug 修复 |
| Security   | 安全相关 |

### 自动生成 Changelog

从 Git 历史提取：

```bash
# 获取两个版本之间的提交
git log v0.9.0..v1.0.0 --pretty=format:"%s"

# 按 Type 分组
git log v0.9.0..v1.0.0 --pretty=format:"%s" | grep "^feat" | sed 's/feat/### Added/'
git log v0.9.0..v1.0.0 --pretty=format:"%s" | grep "^fix" | sed 's/fix/### Fixed/'
```

---

## Part 5: Release Notes 规范

### 格式模板

````markdown
# Release v1.0.0

## 🎉 新功能

### 用户管理

- **用户登录** - 支持 JWT 和 Redis Token 两种认证模式
- **角色权限** - 基于 RBAC 的细粒度权限控制
- **菜单路由** - 动态加载用户菜单权限

## 🐛 Bug 修复

- 修复 Token 过期后前端未正确跳转登录页的问题 (#123)
- 修复分页查询时总数计算错误的问题

## 💥 重大变更

- API 路径统一调整为 `/api/v1/` 前缀
- 数据库字段命名改为 `snake_case`

## 📦 依赖更新

- 升级 Spring Boot 至 3.2.0
- 升级 Vue 至 3.4.0

## 🔒 安全更新

- 升级 JWT 库修复 CVE-2024-XXXX

## 📝 文档更新

- 新增 API 文档
- 更新部署指南

## 贡献者

感谢以下贡献者的付出：@user1, @user2

## 安装/升级

### Docker

```bash
docker pull youlai/admin:v1.0.0
```
````

### 源码

```bash
git checkout v1.0.0
```

## 完整变更日志

查看 [CHANGELOG.md](./CHANGELOG.md) 获取完整变更记录。

```

---

## Part 6: 自动化工作流

### 标准发布流程

```

1. 检查工作区状态└─ git status

2. 确保在主分支└─ git checkout main && git pull

3. 生成 Commit Message └─ 分析 staged 变更 → 生成规范 Message

4. 提交变更└─ git commit -m "..."

5. 推送到远程└─ git push origin main

6. 确定版本号 └─ 分析 Commits → 递增版本号 └─ 来源优先级：手动指定 > package.json > pom.xml > git tag

7. 更新版本文件 └─ package.json / pom.xml 等

8. 提交版本更新 └─ git add . && git commit -m "chore: release vX.X.X" └─ Windows: 分开执行 git add . 然后 git commit

9. 创建 Tag └─ git tag -a vX.X.X -m "Release vX.X.X"

10. 推送 Tag └─ git push origin vX.X.X

11. 推送版本更新提交└─ git push origin main

12. 生成 Release Notes └─ 基于 Changelog → 格式化输出

```

> ⚠️ **注意顺序**：先提交代码变更 → 创建 Tag → 更新版本文件 → 提交版本更新 → 推送

### 快速提交流程

```

1. 分析变更 → 生成 Commit Message
2. git add -A
3. git commit -m "..."
4. git push

````

### 版本发布检查清单

- [ ] 所有测试通过
- [ ] 代码已 Review
- [ ] CHANGELOG.md 已更新
- [ ] 版本号已更新
- [ ] 文档已同步
- [ ] Tag 已创建
- [ ] Tag 已推送

---

## Part 7: 命令速查

### Commit

```bash
# 查看待提交变更
git diff --cached

# 提交
git commit -m "feat(user): 添加用户登录功能"

# 修改最近一次提交
git commit --amend -m "新消息"

# 交互式暂存
git add -p
````

### Tag

```bash
# 创建 Tag
git tag -a v1.0.0 -m "Release v1.0.0"

# 推送 Tag
git push origin v1.0.0

# 删除本地 Tag
git tag -d v1.0.0

# 删除远程 Tag
git push origin --delete v1.0.0
```

### 版本

```bash
# 查看最新 Tag
git describe --tags --abbrev=0

# 查看所有 Tag
git tag -l

# 查看两个版本间的变更
git log v0.9.0..v1.0.0 --oneline
```

### Windows PowerShell 兼容性

> ⚠️ Windows PowerShell 与 Bash 命令有差异，注意以下问题：

| Bash           | Windows PowerShell           | 说明                          |
| -------------- | ---------------------------- | ----------------------------- |
| `cmd1 && cmd2` | `cmd1; cmd2` 或分开执行      | PowerShell 不支持 `&&` 分隔符 |
| `head -5`      | `Select-Object -First 5`     | 无 `head` 命令                |
| `grep`         | `Select-String` 或 `findstr` | 无 `grep` 命令                |
| `sed`          | `-replace`                   | 无 `sed` 命令                 |

**Windows 示例：**

```powershell
# 分开执行而非 &&
git add .
git commit -m "feat: 新功能"
git push

# 查看最近 5 个 Tag
git tag -l "v*" --sort=-v:refname | Select-Object -First 5
```

---

## Part 8: 常见问题与解决方案

### 问题 1: 推送被拒绝（远程有新提交）

**现象：**

```
! [rejected] master -> master (fetch first)
error: failed to push some refs
```

**原因：** 远程仓库有本地没有的新提交

**解决方案：**

```bash
# 方案 1: Rebase（推荐，保持提交历史线性）
git pull --rebase origin master
git push origin master

# 方案 2: Merge（保留合并历史）
git pull origin master
git push origin master
```

### 问题 2: Rebase 后提交丢失

**现象：** `git pull --rebase` 后发现本地提交不见了

**原因：** 远程仓库被强制更新（forced update），rebase 时丢弃了本地提交

**解决方案：**

```bash
# 1. 用 reflog 查找丢失的提交
git reflog -10

# 输出示例：
# 834df6cd HEAD@{2}: commit: feat(profile): 重构个人中心页面
# 这里的 834df6cd 就是丢失的提交

# 2. 用 cherry-pick 恢复提交
git cherry-pick 834df6cd

# 3. 如果有冲突，解决后继续
git add <冲突文件>
git cherry-pick --continue
# 或直接提交
git commit --no-edit

# 4. 推送
git push origin master
```

### 问题 3: Cherry-pick 冲突

**现象：**

```
CONFLICT (content): Merge conflict in <file>
error: could not apply <commit>
```

**解决方案：**

```bash
# 1. 查看冲突文件
git diff --name-only --diff-filter=U

# 2. 手动解决冲突（编辑文件，删除 <<<<<<< HEAD 等标记）

# 3. 标记冲突已解决
git add <冲突文件>

# 4. 继续 cherry-pick
git cherry-pick --continue

# 或跳过此提交
git cherry-pick --skip

# 或放弃 cherry-pick
git cherry-pick --abort
```

### 问题 4: 多远程仓库推送部分失败

**现象：** 项目配置了多个远程仓库（Gitee/GitHub/GitCode），部分推送成功，部分失败

**原因：** 网络问题、认证问题或仓库配置差异

**解决方案：**

```bash
# 查看所有远程仓库
git remote -v

# 单独推送到特定远程
git push gitee master
git push github master
git push gitcode master

# 推送所有 tag 到特定远程
git push gitee --tags
```

**建议：** 如果某个远程持续失败，可以：

1. 检查网络连接
2. 更新认证凭据
3. 暂时移除该远程：`git remote remove <name>`

### 问题 5: 忘记创建 Tag

**现象：** 提交已推送，但忘记创建版本 Tag

**解决方案：**

```bash
# 1. 确认当前版本（从 package.json 或 git log）
git log --oneline -5

# 2. 创建 Tag（可以指定历史提交）
git tag -a v1.0.0 -m "Release v1.0.0"
# 或指定提交
git tag -a v1.0.0 <commit-hash> -m "Release v1.0.0"

# 3. 推送 Tag
git push origin v1.0.0
# 或推送所有 Tag
git push origin --tags
```

### 问题 6: Tag 创建在错误的提交上

**解决方案：**

```bash
# 1. 删除本地 Tag
git tag -d v1.0.0

# 2. 删除远程 Tag
git push origin --delete v1.0.0

# 3. 在正确的提交上重新创建
git tag -a v1.0.0 <correct-commit> -m "Release v1.0.0"

# 4. 推送新 Tag
git push origin v1.0.0
```

---

## Part 9: AI 辅助场景

### 场景 1: 自动生成 Commit Message

**用户操作**: 暂存变更后请求 AI 帮忙提交

**AI 执行**:

1. 运行 `git diff --cached` 分析变更
2. 识别变更类型和范围
3. 生成规范的 Commit Message
4. 执行 `git commit -m "..."`
5. 询问是否推送

### 场景 2: 发布新版本

**用户操作**: 请求发布 v1.0.0

**AI 执行**:

1. 检查工作区是否干净
2. 分析自上个 Tag 以来的 Commits
3. 确认版本号递增是否正确
4. 更新版本文件
5. 生成/更新 CHANGELOG.md
6. 提交版本更新
7. 创建 Tag
8. 推送 Commit 和 Tag
9. 生成 Release Notes

### 场景 3: 查看版本历史

**用户操作**: 询问最近几个版本的变更

**AI 执行**:

1. 运行 `git tag -l` 获取版本列表
2. 运行 `git log v1...v2` 查看变更
3. 格式化输出版本历史

### 场景 4: 回滚版本

**用户操作**: 请求回滚到上个版本

**AI 执行**:

1. 确认当前版本
2. 列出可回滚的版本
3. 确认回滚操作
4. 执行 `git revert` 或 `git reset`
5. 更新 CHANGELOG.md

---

## Part 10: 配置建议

### Git Hooks (Husky)

```json
{
  "scripts": {
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{js,ts,vue}": ["eslint --fix", "git add"],
    "*.{java}": ["./gradlew spotlessApply", "git add"]
  }
}
```

### Commitlint 配置

```javascript
// commitlint.config.js
module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "chore",
        "ci",
        "build",
        "revert",
      ],
    ],
    "scope-enum": [
      2,
      "always",
      [
        "auth",
        "user",
        "role",
        "menu",
        "dept",
        "dict",
        "notice",
        "log",
        "file",
        "api",
        "db",
        "config",
        "deps",
      ],
    ],
  },
};
```

### 标准版本工具

```json
{
  "scripts": {
    "release": "standard-version",
    "release:minor": "standard-version --release-as minor",
    "release:major": "standard-version --release-as major",
    "release:first": "standard-version --first-release"
  }
}
```

---

## Part 11: 质量检查清单

- [ ] Commit Message 符合规范格式
- [ ] Type 正确反映变更类型
- [ ] Scope 准确描述变更范围
- [ ] 版本号符合语义化规范
- [ ] Tag 使用 `v` 前缀
- [ ] Tag 包含注释说明
- [ ] CHANGELOG.md 分类清晰
- [ ] Release Notes 包含必要信息
- [ ] 重大变更已明确标注
- [ ] 贡献者已正确列出
