# 编码规范

## TypeScript

### 严格规则

- `strict: true` — 始终启用
- 禁止 `any` 类型 — 使用 `unknown` + 类型守卫
- 禁止 `@ts-ignore` — 修复类型问题
- 禁止 `as` 断言 — 使用类型守卫或泛型
- 禁止 `!` 非空断言 — 正确处理 null/undefined

### 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 变量和函数 | camelCase | `getUserById` |
| 类型和接口 | PascalCase | `UserProfile` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 文件名 | kebab-case | `user-profile.ts` |
| 枚举 | PascalCase（成员也是） | `UserRole.Admin` |

### 导出

- 只使用命名导出（禁止 `export default`）
- 公共模块 API 使用 Barrel 导出（`index.ts`）
- 类型重导出使用 `export type`

### 函数

```typescript
// 推荐：公共函数显式声明返回类型
export function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// 推荐：回调使用箭头函数
const activeUsers = users.filter((user) => user.isActive);

// 推荐：提前返回代替嵌套条件
export function getDiscount(user: User): number {
  if (!user.isPremium) return 0;
  if (user.tenure < 12) return 5;
  return 10;
}
```

### 错误处理

```typescript
// 定义领域错误
export class NotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} 未找到: ${id}`);
    this.name = 'NotFoundError';
  }
}

// 对预期失败使用 Result 类型
export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// 禁止吞掉错误
try {
  await operation();
} catch (error: unknown) {
  if (error instanceof NotFoundError) {
    logger.warn('资源不存在', { error });
    return { ok: false, error };
  }
  throw error; // 重新抛出未预期的错误
}
```

## 文件组织

### 大小限制

- 每个文件最多 300 行
- 每个函数最多 50 行
- 每个组件最多 150 行

### 结构

- 每个文件一个导出（组件、服务、工具函数）
- 测试与源码放在一起
- 按功能分组，而非按技术层分组

```
src/
├── features/
│   └── auth/
│       ├── login-form.tsx          # 组件
│       ├── login-form.test.tsx     # 组件测试
│       ├── auth-service.ts         # 业务逻辑
│       ├── auth-service.test.ts    # 服务测试
│       ├── auth-types.ts           # 类型定义
│       └── index.ts                # Barrel 导出
└── lib/
    └── http/
        ├── client.ts
        ├── client.test.ts
        └── index.ts
```

## 依赖管理

### 添加新依赖

1. 先检查现有代码或标准库能否解决问题
2. 评估：包体积、维护状态、许可证、安全性
3. 优先选择零依赖的库
4. 生产依赖始终使用精确版本

### 禁止的模式

- 禁止为单个工具函数引入 `lodash`（使用原生 JS）
- 禁止使用 `moment.js`（使用 `date-fns` 或 `Intl`）
- 新代码禁止使用 CSS-in-JS 运行时库（使用 CSS modules 或 Tailwind）
