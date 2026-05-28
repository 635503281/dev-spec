# 安全基线

## OWASP Top 10 防护

### 1. 注入攻击（SQL、NoSQL、命令注入）

- 仅使用参数化查询
- 禁止将用户输入拼接到查询中
- 使用 Zod schema 验证所有输入

### 2. 认证失效

- 使用成熟的认证库（better-auth、next-auth）
- 强制执行强密码策略
- 认证端点实施限流
- 使用安全的会话管理

### 3. 敏感数据暴露

- 禁止在日志中记录密码、令牌或个人信息
- 全面使用 HTTPS
- 对静态敏感数据加密
- 实施正确的密钥管理

### 4. XML 外部实体（XXE）

- 禁用 XML 外部实体处理
- 尽可能使用 JSON API 代替 XML

### 5. 访问控制失效

- 每个受保护操作都检查授权
- 默认拒绝 — 明确授予访问权限
- 修改操作前验证资源所有权

### 6. 安全配置错误

- 删除默认凭据
- 生产环境禁用调试/详细错误信息
- 设置安全头（CSP、HSTS、X-Frame-Options）
- 保持依赖更新

### 7. 跨站脚本（XSS）

- 渲染前转义所有用户生成内容
- 使用 Content Security Policy 头
- 利用框架内置转义（React 自动转义 JSX）

### 8. 不安全的反序列化

- 使用 schema 验证所有序列化数据
- 禁止在未验证的情况下反序列化不可信数据

### 9. 使用含已知漏洞的组件

- 定期运行 `pnpm audit`
- 保持依赖更新
- 关注安全公告

### 10. 日志与监控不足

- 记录所有认证事件
- 记录所有授权失败
- 使用结构化日志（用户、操作、资源）
- 禁止记录敏感数据

## 安全响应头

```typescript
// 所有响应必须包含的安全头
const securityHeaders = {
  'Content-Security-Policy': "default-src 'self'; script-src 'self'",
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
  'X-Content-Type-Options': 'nosniff',
  'X-Frame-Options': 'DENY',
  'X-XSS-Protection': '0',
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
};
```

## 环境变量

- 禁止提交 `.env` 文件
- 使用 `.env.example` 包含占位值
- 应用启动时用 Zod 验证所有环境变量
- 区分构建时和运行时变量
