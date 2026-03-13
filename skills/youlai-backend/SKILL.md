---
name: youlai-backend
description: Youlai backend development guidelines and API protocol standards. Use this skill when working on any youlai backend project (youlai-boot, youlai-nest, youlai-gin, youlai-django, youlai-think, youlai-aspnet), implementing REST APIs, authentication, or permission systems.
---

# Youlai Backend - API Protocol & Development Guide

Common backend development guidelines for all Youlai backend services.

## Trigger Conditions

Use this skill when:
- Working on any youlai backend project
- Implementing REST APIs
- Setting up authentication (JWT/Redis-token)
- Implementing permission system
- Designing database schema

---

## Part 1: API Response Protocol

### Standard Response Format

```json
// Success response
{
  "code": 200,
  "msg": "操作成功",
  "data": { ... }
}

// Error response
{
  "code": 400,
  "msg": "参数错误",
  "data": null
}
```

### Paginated Response Format

```json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "list": [...],
    "total": 100
  }
}
```

### Response Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Error |

---

## Part 2: API Path Convention

### Path Style

- Use kebab-case: `/api/v1/user-profile`
- Version prefix: `/api/v1/`
- Resource naming: plural nouns `/users`, `/roles`

### Standard CRUD Paths

| Action | Method | Path |
|--------|--------|------|
| List (paginated) | GET | `/api/v1/users/page` |
| Get by ID | GET | `/api/v1/users/{id}` |
| Create | POST | `/api/v1/users` |
| Update | PUT | `/api/v1/users` |
| Delete | DELETE | `/api/v1/users/{id}` |
| Batch delete | DELETE | `/api/v1/users/batch` |
| Options (dropdown) | GET | `/api/v1/users/options` |
| Form data (for edit) | GET | `/api/v1/users/{id}/form` |

### Auth Paths

| Action | Method | Path |
|--------|--------|------|
| Login | POST | `/api/v1/auth/login` |
| Logout | POST | `/api/v1/auth/logout` |
| Refresh token | POST | `/api/v1/auth/refresh-token` |
| Current user info | GET | `/api/v1/users/me` |

### WeChat Miniapp Paths

| Action | Method | Path |
|--------|--------|------|
| Login | POST | `/api/v1/auth/wechat-miniapp/login` |
| Phone login | POST | `/api/v1/auth/wechat-miniapp/phone-login` |
| Bind mobile | POST | `/api/v1/auth/wechat-miniapp/bind-mobile` |

---

## Part 3: Authentication

### Session Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| JWT | Stateless, token contains user info | Distributed systems |
| Redis-token | Stateful, token stored in Redis | Single server, need revocation |

### Token Structure

```
Authorization: Bearer {access_token}
```

### JWT Claims

| Claim | Description |
|-------|-------------|
| `sub` | User ID |
| `username` | Username |
| `exp` | Expiration time |
| `iat` | Issued at time |
| `jti` | JWT ID (for revocation) |

### Token TTL

| Token Type | Default TTL |
|------------|-------------|
| Access Token | 2 hours |
| Refresh Token | 7 days |

---

## Part 4: Permission System

### Permission Model

```
User → Role → Permission (Menu + Button)
```

### Permission Types

| Type | Code | Description |
|------|------|-------------|
| Directory | `M` | Menu directory |
| Menu | `C` | Menu page |
| Button | `F` | Button permission |

### Permission Check

```java
// Button permission format
@PreAuthorize("hasAuthority('sys:user:add')")

// Data scope
@DataScope(deptAlias = "d", userAlias = "u")
```

### Data Scope Levels

| Level | Description |
|-------|-------------|
| 1 | All data |
| 2 | Custom scope |
| 3 | Department and below |
| 4 | Department only |
| 5 | Self only |

---

## Part 5: Request Parameters

### Pagination Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| pageNum | int | 1 | Page number |
| pageSize | int | 20 | Page size |
| keywords | string | - | Search keyword |

### Date Range Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| startDate | string | Start date (yyyy-MM-dd) |
| endDate | string | End date (yyyy-MM-dd) |

### Sort Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| sortBy | string | Sort field |
| sortOrder | string | ASC/DESC |

---

## Part 6: Database Conventions

### Table Naming

- Prefix: `sys_` for system tables
- Snake_case: `sys_user`, `sys_role_menu`

### Common Fields

| Field | Type | Description |
|-------|------|-------------|
| id | bigint | Primary key |
| create_time | datetime | Created time |
| update_time | datetime | Updated time |
| create_by | bigint | Creator ID |
| update_by | bigint | Updater ID |
| deleted | tinyint | Soft delete flag |

### SQL File Location

All backends share the same SQL schema:
```
youlai-boot/sql/mysql/youlai_admin.sql
```

---

## Part 7: Configuration

### Required Configuration

| Config | Description |
|--------|-------------|
| Database | MySQL connection |
| Redis | Redis connection |
| JWT | Secret key, TTL |
| CORS | Allowed origins |

### Environment Variables

```env
# Database
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=youlai_admin
DB_USER=root
DB_PASS=

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT
JWT_SECRET=SecretKey012345678901234567890123456789
JWT_ACCESS_TTL=7200
JWT_REFRESH_TTL=604800

# Session Mode
SESSION_MODE=jwt  # or redis-token
```

---

## Part 8: Error Handling

### Exception Response

```json
{
  "code": 400,
  "msg": "用户名已存在",
  "data": null
}
```

### Common Exceptions

| Exception | Code | Message |
|-----------|------|---------|
| BadRequestException | 400 | 参数错误 |
| UnauthorizedException | 401 | 未登录或token过期 |
| ForbiddenException | 403 | 无权限 |
| NotFoundException | 404 | 资源不存在 |
| BusinessException | 500 | 业务异常 |

---

## Part 9: Code Quality Checklist

- [ ] Follow API path convention
- [ ] Use standard response format
- [ ] Implement proper error handling
- [ ] Add permission checks
- [ ] Use pagination for list endpoints
- [ ] Validate input parameters
- [ ] Log important operations
- [ ] Handle soft delete
- [ ] Use transactions for multi-step operations
- [ ] Add API documentation
