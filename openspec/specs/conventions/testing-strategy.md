# 测试策略

## 测试金字塔

```
        ╱╲
       ╱E2E╲         5% — 仅覆盖关键用户路径
      ╱──────╲
     ╱ 集成   ╲      15% — 模块间交互
    ╱──────────╲
   ╱   单元     ╲    80% — 隔离的逻辑测试
  ╱──────────────╲
```

## 单元测试（Vitest）

### 测什么

- 纯函数
- 业务逻辑
- 数据转换
- 验证规则
- 错误处理路径
- 边界情况

### 不测什么

- 框架内部实现
- 第三方库的行为
- 无意义的 getter/setter
- 实现细节

### 约定

```typescript
import { describe, test, expect } from 'vitest';

describe('calculateDiscount', () => {
  test('非高级用户应返回 0', () => {
    const user = createUser({ isPremium: false });
    expect(calculateDiscount(user)).toBe(0);
  });

  test('会员时长超过 12 个月的高级用户应返回 10%', () => {
    const user = createUser({ isPremium: true, tenureMonths: 24 });
    expect(calculateDiscount(user)).toBe(10);
  });

  test('负数会员时长应抛出 ValidationError', () => {
    const user = createUser({ tenureMonths: -1 });
    expect(() => calculateDiscount(user)).toThrow(ValidationError);
  });
});
```

### 规则

- 每个测试只测一个行为
- 测试名称：`当 [条件] 时应该 [预期结果]`
- 使用工厂函数创建测试数据，而非原始字面量
- 不用 mock，除非依赖不可避免（网络、文件系统）
- 测试行为，而非实现

## 集成测试（Vitest）

### 测什么

- API 路由处理器
- 数据库操作
- 服务层交互
- 多模块工作流

### 约定

- 使用真实实现（不用 mock）
- 每个测试独立地设置和清理测试数据
- 测试契约，而非内部实现

## E2E 测试（Playwright）

### 测什么

- 关键用户流程（登录、结账等）
- 跨页面导航
- 表单提交和验证
- 不同视口下的响应式行为

### 不测什么

- 样式细节（使用视觉回归工具）
- 单元测试能轻松覆盖的错误状态
- 性能（使用专用性能工具）

### 约定

```typescript
import { test, expect } from '@playwright/test';

test.describe('用户认证', () => {
  test('使用有效凭据应成功登录', async ({ page }) => {
    await page.goto('/login');
    await page.getByLabel('邮箱').fill('user@example.com');
    await page.getByLabel('密码').fill('password123');
    await page.getByRole('button', { name: '登录' }).click();
    await expect(page.getByText('欢迎')).toBeVisible();
  });
});
```

## 覆盖率阈值

| 指标 | 最低要求 |
|------|----------|
| 分支覆盖率 | 80% |
| 函数覆盖率 | 80% |
| 行覆盖率 | 80% |
| 语句覆盖率 | 80% |

## 测试反模式（避免）

1. **测试 mock** — 断言真实行为，而非 mock 的设置
2. **滥用快照** — 只用于稳定的、充分理解的输出
3. **测试专用方法** — 不要为了测试而给生产代码加方法
4. **共享可变状态** — 每个测试使用新鲜的状态
5. **测试实现细节** — 测试"做什么"，不是"怎么做"
6. **不稳定的测试** — 修复或移除；禁止永久跳过
