---
name: git-version
description: Git version control automation standards. Use this skill when generating commit messages, managing version numbers, creating tags, or generating changelogs and release notes.
---

# Git 版本管理自动化规范

## 触发条件

- Generate conventional commit messages
- Manage semantic versioning
- Create Git tags
- Generate changelogs
- Generate release notes
- Automate version release workflows

---

## Part 1: Commit Message 规范

### 1.1 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 1.2 Type 类型

| Type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat(user): 添加用户登录功能` |
| `fix` | Bug 修复 | `fix(auth): 修复 Token 过期验证问题` |
| `docs` | 文档更新 | `docs(readme): 更新安装说明` |
| `style` | 代码格式（不影响功能） | `style: 格式化代码缩进` |
| `refactor` | 重构（非新功能、非 Bug 修复） | `refactor(user): 重构用户服务层` |
| `perf` | 性能优化 | `perf(query): 优化分页查询性能` |
| `test` | 测试相关 | `test(user): 添加用户服务单元测试` |
| `chore` | 构建/工具相关 | `chore: 更新依赖版本` |
| `ci` | CI/CD 相关 | `ci: 添加 GitHub Actions 工作流` |
| `build` | 构建系统相关 | `build: 升级 webpack 配置` |
| `revert` | 回滚提交 | `revert: 回滚用户登录功能` |

### 1.3 Scope 范围

| Scope | 说明 |
|-------|------|
| `auth` | 认证授权 |
| `user` / `role` / `menu` / `dept` / `dict` / `notice` | 业务模块 |
| `log` / `file` / `api` | 日志/文件/接口 |
| `db` / `config` / `deps` | 数据库/配置/依赖 |

### 1.4 自动生成流程

1. 分析变更：`git diff --cached --stat && git diff --cached --name-status`
2. 识别类型：新增文件 → `feat`，修改配置 → `chore`，删除文件 → `refactor`
3. 生成 Message 示例：

```
feat(user): 添加用户头像上传功能

- 支持 JPG/PNG 格式
- 限制文件大小 2MB
- 自动压缩图片
```

---

## Part 2: 语义化版本规范

### 2.1 版本格式

```
MAJOR.MINOR.PATCH[-PRERELEASE][+BUILD]
```

| 部分 | 说明 | 更新时机 |
|------|------|----------|
| MAJOR | 主版本号 | 不兼容的 API 变更 |
| MINOR | 次版本号 | 向后兼容的功能新增 |
| PATCH | 修订号 | 向后兼容的问题修复 |
| PRERELEASE | 预发布标识 | `alpha`, `beta`, `rc` |

### 2.2 版本递增规则

| Commit Type | 版本递增 |
|-------------|----------|
| `feat` | MINOR +1, PATCH = 0 |
| `fix` | PATCH +1 |
| `feat!` 或 `BREAKING CHANGE` | MAJOR +1, MINOR = 0, PATCH = 0 |
| `docs` / `style` / `refactor` / `test` / `chore` | 不递增 |

### 2.3 预发布版本

| 阶段 | 版本示例 | 说明 |
|------|----------|------|
| Alpha | `1.0.0-alpha.1` | 内部测试 |
| Beta | `1.0.0-beta.1` | 公开测试 |
| Release Candidate | `1.0.0-rc.1` | 候选发布 |
| Release | `1.0.0` | 正式发布 |

### 2.4 版本文件同步

| 项目类型 | 版本文件 | 更新方式 |
|----------|----------|----------|
| Node.js | `package.json` | 修改 `version` 字段 |
| Java Maven | `pom.xml` | 修改 `<version>` 标签 |
| Python | `pyproject.toml` | 修改 `version` 字段 |
| Go | 可选或跳过 | — |

---

## Part 3: Git Tag 规范

### 3.1 命名格式

| 类型 | 格式 | 示例 |
|------|------|------|
| 正式版本 | `v{VERSION}` | `v1.0.0` |
| 预发布版本 | `v{VERSION}-{PRERELEASE}.{N}` | `v1.0.0-beta.1` |

### 3.2 创建带注释 Tag

```bash
git tag -a v1.0.0 -m "Release v1.0.0

Features:
- 用户登录功能
- 角色权限管理

Fixes:
- 修复 Token 过期问题"
```

### 3.3 常用命令

```bash
git tag -l                              # 列出所有 Tag
git tag -l "v1.*"                       # 匹配 Tag
git show v1.0.0                         # 查看 Tag 详情
git tag -d v1.0.0                       # 删除本地 Tag
git push origin --delete v1.0.0        # 删除远程 Tag
git push origin v1.0.0                  # 推送单个 Tag
git push origin --tags                  # 推送所有 Tag
```

---

## Part 4: Changelog 规范

### 4.1 格式模板

基于 [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)：

```markdown
# Changelog

## [Unreleased]
### Added / Changed / Deprecated / Removed / Fixed / Security

## [1.0.0] - 2024-01-15
### Added
- 用户登录功能
- 角色权限管理

### Fixed
- 修复 Token 过期验证问题 (#123)

### Security
- 升级 JWT 库修复安全漏洞
```

### 4.2 变更分类

| 分类 | 说明 |
|------|------|
| Added | 新增功能 |
| Changed | 功能变更 |
| Deprecated | 即将废弃 |
| Removed | 已移除 |
| Fixed | Bug 修复 |
| Security | 安全相关 |

### 4.3 自动生成

```bash
git log v0.9.0..v1.0.0 --pretty=format:"%s" | grep "^feat" | sed 's/feat/### Added/'
git log v0.9.0..v1.0.0 --pretty=format:"%s" | grep "^fix" | sed 's/fix/### Fixed/'
```

---

## Part 5: Release Notes 规范

```markdown
# Release v1.0.0

## 🎉 新功能
### 用户管理
- **用户登录** - 支持 JWT 和 Redis Token 两种认证模式
- **角色权限** - 基于 RBAC 的细粒度权限控制

## 🐛 Bug 修复
- 修复 Token 过期后前端未正确跳转登录页的问题 (#123)

## 💥 重大变更
- API 路径统一调整为 `/api/v1/` 前缀

## 📦 依赖更新
- 升级 Spring Boot 至 3.2.0

## 完整变更日志
查看 [CHANGELOG.md](./CHANGELOG.md) 获取完整变更记录。
```

---

## Part 6: 自动化工作流

### 6.1 标准发布流程

```
1. git status                  — 检查工作区状态
2. git checkout main && git pull — 确保在主分支
3. git add . && git commit -m "..." — 提交变更
4. git push origin main        — 推送
5. 确定版本号 → 更新版本文件    — version bump
6. git add . && git commit -m "chore: release vX.X.X"
7. git tag -a vX.X.X -m "Release vX.X.X"
8. git push origin vX.X.X      — 推送 Tag
9. git push origin main         — 推送版本更新提交
```

> ⚠️ 注意顺序：先提交代码 → 创建 Tag → 更新版本文件 → 提交版本更新 → 推送

### 6.2 版本发布检查清单

- [ ] 所有测试通过
- [ ] 代码已 Review
- [ ] CHANGELOG.md 已更新
- [ ] 版本号已更新
- [ ] Tag 已创建并推送

---

## Part 7: Windows PowerShell 兼容性

| Bash | PowerShell | 说明 |
|------|-----------|------|
| `cmd1 && cmd2` | `cmd1; cmd2` 或分开执行 | PowerShell 不支持 `&&` |
| `head -5` | `Select-Object -First 5` | 无 `head` |
| `grep` | `Select-String` 或 `findstr` | 无 `grep` |
| `sed` | `-replace` | 无 `sed` |

```powershell
# Windows 示例
git add .
git commit -m "feat: 新功能"
git push

# 查看最近 5 个 Tag
git tag -l "v*" --sort=-v:refname | Select-Object -First 5
```

---

## Part 8: 常见问题

### 推送被拒绝

```bash
# Rebase（推荐，保持历史线性）
git pull --rebase origin master
git push origin master
```

### Rebase 后提交丢失

```bash
git reflog -10                      # 查找丢失的提交
git cherry-pick <commit-hash>       # 恢复
git push origin master
```

### Tag 创建在错误提交上

```bash
git tag -d v1.0.0                   # 删本地
git push origin --delete v1.0.0    # 删远程
git tag -a v1.0.0 <correct-hash> -m "Release v1.0.0"
git push origin v1.0.0
```

---

## Part 9: 常见反模式

| 反模式 | 正确做法 |
|--------|----------|
| `git add -A && git commit -m "update"` | 分步暂存，写有意义 Message |
| `git push --force` 到 main/master | 用 `--force-with-lease` 或禁止 |
| Tag 无 `-a` 无 message | `git tag -a v1.0.0 -m "Release v1.0.0"` |
| commit message `fix bug` 无 scope/详情 | `fix(auth): 修复 Token 过期后无限循环刷新` |
| 版本号跳跃不递增 | `feat` → MINOR+1, `fix` → PATCH+1 |
| 不更新 CHANGELOG 就发版 | Changelog 与 Tag 同步更新 |
