# AGENTS.md

> 跨工具 AI 代理指令。支持：Cursor、GitHub Copilot、Codex CLI、Windsurf、Amp、Devin、Gemini CLI 等。

## 项目概述

**规范驱动开发（SDD）**，以 **OpenSpec 为主线**，**Superpowers 为增强**。
工作流：**探索 → 定义 → 计划 → 构建 → 验证 → 审查 → 发布**。

## 技术栈

- **运行时**：Node.js 20+
- **语言**：TypeScript 5.x（严格模式）
- **框架**：可适配（Next.js / React / Vue / 原生 — 按项目定义）
- **测试**：Vitest（单元/集成）+ Playwright（E2E/浏览器）
- **代码检查**：ESLint + Prettier
- **包管理器**：pnpm
- **版本控制**：Git（主干开发模式）

## 全自动工作流

编排引擎 `.cursor/rules/00-orchestrator.mdc` 驱动全自动阶段流转。

### 阶段流转图

```
用户提需求
    │
    ▼
┌─ 探索 ─────────────────────────────────────────────────────────┐
│  Superpowers:brainstorming → explore.md                         │
│  澄清需求、方案取舍、技术方向                                    │
└────────────┬──── 🚫 用户确认 explore.md ─────────────────────────┘
             ▼
┌─ 定义 ─────────────────────────────────────────────────────────┐
│  OpenSpec /opsx:propose（读取 explore.md）                       │
│  → proposal.md → specs/ → design.md → tasks.md                  │
└────────────┬──── 🚫 用户确认全套产物 ───────────────────────────┘
             ▼
┌─ 计划 ─────────────────────────────────────────────────────────┐
│  Superpowers:writing-plans（读取 tasks.md + specs + design）    │
│  → plans.md（详细步骤，与 tasks.md 分离不覆盖）                  │
└────────────┬──── 🚫 用户确认 plans.md ──────────────────────────┘
             ▼
┌─ 构建 ─────────────────────────────────────────────────────────┐
│  按 plans.md 执行 → 进度同步到 tasks.md ✓ + plans.md ✓          │
│  Superpowers:subagent-driven-development + TDD                  │
├─────────────────────────────────────────────────────────────────┤
│  验证 → 自动调试 → 重新验证 → 审查 → 自动修复 → 重新审查         │
└────────────┬──── 🚫 用户确认发布 ───────────────────────────────┘
             ▼
┌─ 发布 ─────────────────────────────────────────────────────────┐
│  /opsx:sync（合并规范）→ /opsx:archive（归档变更）              │
│  Superpowers:finishing-a-development-branch                     │
└─────────────────────────────────────────────────────────────────┘
```

### 每个阶段的工具调度

| 阶段 | OpenSpec | Superpowers | 产物 |
|------|---------|-------------|------|
| **探索** | — | `brainstorming` | `explore.md` |
| **定义** | `/opsx:propose`（基于 explore.md） | — | `proposal + specs + design + tasks` |
| **计划** | — | `writing-plans`（基于 tasks.md） | `plans.md`（与 tasks.md 分离） |
| **构建** | `/opsx:apply` | `subagent-driven-dev` + `TDD` | 源码 + 进度同步 |
| **验证** | `/opsx:verify` | `verification` | 命令行证据 |
| **调试** | — | `systematic-debugging` | — |
| **审查** | — | `code-review` | `review.md` |
| **发布** | `/opsx:sync` + `/opsx:archive` | `finishing-branch` | 归档 |

### 意图自动识别

AI 根据用户消息自动路由：
- "我想做一个..." / "新功能" → 探索阶段（brainstorming → explore.md）
- "开始定义" / 确认 explore.md 后 → 定义阶段（/opsx:propose）
- "开始计划" / 确认定义产物后 → 计划阶段（writing-plans → plans.md）
- "开始实施" / "执行计划" → 构建阶段（按 plans.md 执行）
- "修复这个bug" / "测试不过" → 调试阶段
- "看看效果" / "打开页面" → Playwright 浏览器测试
- "做完了" / "提交" → 发布阶段
- 紧急修复 → 跳过定义，直接调试→TDD修复→验证

## 关键命令

```bash
pnpm install          # 安装依赖
pnpm dev              # 启动开发服务器
pnpm build            # 生产构建
pnpm test             # 运行单元/集成测试（Vitest）
pnpm test:e2e         # 运行 E2E 测试（Playwright）
pnpm lint             # ESLint 检查
pnpm lint:fix         # 自动修复 lint 问题
pnpm typecheck        # TypeScript 类型检查
pnpm format           # Prettier 格式化
```

## 项目结构

```
├── AGENTS.md                         # AI 代理指令（本文件）
├── openspec/                         # 统一规范管理（OpenSpec 主线）
│   ├── config.yaml                   # OpenSpec 项目配置
│   ├── specs/                        # 主干规范（权威的、已合并的）
│   │   ├── features/                 # 功能规范
│   │   ├── api/                      # API 契约
│   │   ├── security/                 # 安全基线
│   │   └── conventions/              # 编码规范 + 测试策略
│   ├── adr/                          # 架构决策记录
│   │   └── NNNN-<标题>.md
│   ├── changes/                      # 进行中的变更
│   │   ├── .templates/               # 产物模板（explore/tasks/plans）
│   │   ├── <change-name>/            # 单个变更工作单元
│   │   │   ├── explore.md            # 探索记录（brainstorming 产出）
│   │   │   ├── proposal.md           # 提案（/opsx:propose 产出）
│   │   │   ├── specs/                # 增量规范
│   │   │   ├── design.md             # 设计文档
│   │   │   ├── tasks.md              # 概要任务清单（/opsx:propose 产出）
│   │   │   ├── plans.md              # 详细实施计划（writing-plans 产出）
│   │   │   └── review.md             # 审查记录
│   │   └── archive/                  # 已归档的变更
│   └── schemas/                      # 自定义 workflow schema（可选）
├── src/                              # 源代码
│   ├── components/                   # UI 组件
│   ├── lib/                          # 共享工具
│   ├── services/                     # 业务逻辑
│   ├── hooks/                        # 自定义 Hook（React）
│   └── types/                        # TypeScript 类型定义
├── tests/                            # 测试文件
│   ├── unit/                         # 单元测试
│   ├── integration/                  # 集成测试
│   └── e2e/                          # 端到端测试（Playwright）
├── .cursor/                          # Cursor IDE 配置
│   ├── rules/                        # .mdc 规则文件
│   └── mcp.json                      # MCP 服务器配置
└── public/                           # 静态资源
```

## 编码约定

详细规范见各 `.cursor/rules/*.mdc` 文件，本节为摘要索引：

| 编号 | 文件 | 职责 | alwaysApply |
|------|------|------|-------------|
| 00 | `00-orchestrator.mdc` | 全自动编排引擎、阶段流转 | ✅ |
| 01 | `01-engineering-process.mdc` | TDD、验证门禁、系统化调试、代码审查、Agent 行为纪律 | ✅ |
| 02 | `02-code-standards.mdc` | 架构、命名、错误处理、导入、注释 | ✅ |
| 03 | `03-frontend.mdc` | TypeScript 规则、组件架构、状态管理、无障碍、响应式、浏览器测试 | 按 glob |
| 04 | `04-api-design.mdc` | API 设计、REST 约定、响应结构、输入验证 | 按 glob |
| 05 | `05-platform-adaptation.mdc` | 多平台适配器模式、特性检测、CEF/Electron/WebView | 按 glob |
| 06 | `06-security.mdc` | OWASP Top 10、密钥管理、认证授权、数据保护 | ✅ |
| 07 | `07-git-workflow.mdc` | 分支策略、提交约定、防御性提交、PR 模板 | 按 glob |
| 08 | `08-cicd.mdc` | 质量门禁流水线、GitHub Actions、提交钩子、功能开关 | 按 glob |
| 09 | `09-dependency-discipline.mdc` | 安装前评估、版本锁定、依赖卫生 | 按 glob |
| 10 | `10-taste-codification.mdc` | 用户偏好持续捕获与持久化 | ✅ |

**当 `.cursor/rules/` 中的规则与本文件有冲突时，以 `.cursor/rules/` 为准。**

## 边界

### 始终

- 先写测试再写实现（TDD）
- 声称完成前先验证（展示命令输出）
- 遵循代码库中的现有模式
- 使用 TypeScript 严格模式
- 显式处理错误
- 导入包前验证其存在于 package.json（反幻觉）

### 先询问

- 添加新依赖
- 变更项目结构
- 修改 CI/CD 配置
- 架构变更
- 数据库 schema 变更
- 删除配置文件

### 绝不

- 直接推送到 main/master
- 以"改动简单"为由跳过测试
- 使用 `any` 类型
- 提交密钥、API key 或凭据
- 不理解代码用途就删除（切斯特顿栅栏）
- 未运行验证就声称工作已完成
- 在测试中使用 mock（除非绝对不可避免）
- 输出 `// TODO: implement` 或占位代码（零占位符策略）
- 凭空捏造函数签名或 API（反幻觉）
- 未经确认执行破坏性命令（Agent 安全边界）

## 环境目标

本项目支持多种部署目标：

### Web（浏览器）

- 现代浏览器（Chrome 90+、Firefox 90+、Safari 15+、Edge 90+）
- 响应式设计（移动优先）
- Core Web Vitals 目标：LCP < 2.5s，FID < 100ms，CLS < 0.1

### CEF 容器（桌面嵌入）

- Chromium Embedded Framework 集成
- 通过 `window.__cef__` 或消息传递实现桥接 API
- CEF API 不可用时优雅降级
- 通过适配器模式与 Web 目标共享代码库

### 通用模式

- 使用特性检测，而非 User-Agent 嗅探
- 平台 API 通过接口抽象
- 发布前在所有目标平台测试
