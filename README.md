# Project1

> 基于业界最佳实践的 AI 全自动代码开发项目模板

## 开发方法论

本项目采用 **Spec-Driven Development (SDD)** 方法论，**以 OpenSpec 为主线，Superpowers 为增强**：

| 工具/框架 | 用途 | 来源 |
|-----------|------|------|
| **OpenSpec OPSX** | 主线流程：规范驱动的提案/实施/归档工作流 | Fission-AI/OpenSpec |
| **Superpowers** | 增强节点：brainstorming→writing-plans→TDD→verification | obra/superpowers |
| **AGENTS.md** | 跨AI工具的项目级上下文指令 | Linux Foundation AAIF |
| **Cursor Rules (.mdc)** | IDE级别的规则约束和自动触发 | Cursor 官方规范 |
| **Playwright MCP** | AI驱动的浏览器自动化测试 | Microsoft |
| **TDD** | 测试驱动开发（RED-GREEN-REFACTOR） | 工程最佳实践 |

## 全自动开发工作流

```
用户提出需求 → AI 自动判断阶段 → 调用对应工具链

┌─ 1. 探索 ───────────────────────────────────────────────┐
│  Superpowers:brainstorming                               │
│  → 澄清需求 → 方案取舍 → explore.md                     │
│  🚫 用户确认                                             │
├──────────────────────────────────────────────────────────┤
│  2. 定义                                                 │
│  OpenSpec /opsx:propose（基于 explore.md）                │
│  → proposal.md → specs/ → design.md → tasks.md           │
│  🚫 用户确认                                             │
├──────────────────────────────────────────────────────────┤
│  3. 计划                                                 │
│  Superpowers:writing-plans（基于 tasks.md）               │
│  → plans.md（详细步骤，与 tasks.md 分离不覆盖）           │
│  🚫 用户确认                                             │
├──────────────────────────────────────────────────────────┤
│  4. 构建（全自动）                                        │
│  按 plans.md 执行 → 进度同步到 tasks.md ✓ + plans.md ✓   │
│  Superpowers:subagent-driven-development + TDD            │
├──────────────────────────────────────────────────────────┤
│  5. 验证 + 调试（全自动）                                 │
│  typecheck → lint → test → build → Playwright             │
│  失败 → systematic-debugging → TDD修复 → 重新验证        │
├──────────────────────────────────────────────────────────┤
│  6. 审查（全自动）                                        │
│  code-reviewer 子代理：规范合规 → 代码质量 → 安全         │
├──────────────────────────────────────────────────────────┤
│  7. 发布                                                 │
│  /opsx:sync + /opsx:archive + finishing-branch            │
│  🚫 用户确认                                             │
└──────────────────────────────────────────────────────────┘
```

## 项目结构

```
├── AGENTS.md                       # AI 代理指令
├── openspec/                       # 统一规范管理
│   ├── specs/                      # 主干规范
│   ├── adr/                        # 架构决策记录
│   └── changes/<name>/             # 变更工作单元
│       ├── explore.md → proposal.md → specs/ → design.md
│       ├── tasks.md（概要）→ plans.md（详细）
│       └── review.md
├── .cursor/rules/                  # 5 个精简规则文件
│   ├── 00-orchestrator             # 编排引擎
│   ├── 01-iron-laws                # 铁律+验证+调试
│   ├── 02-tdd-and-quality          # TDD+代码质量
│   ├── 03-ui-and-platform          # UI+API+平台
│   └── 04-git-and-cicd             # Git+CI/CD
├── src/                            # 源代码
└── tests/                          # 测试
```

## 快速开始

```bash
# 1. 安装依赖
pnpm install

# 2. 安装 OpenSpec（可选，增强规范工作流）
npm install -g @fission-ai/openspec@latest
openspec init

# 3. 安装 Playwright 浏览器
pnpm exec playwright install

# 4. 启动开发
pnpm dev
```

## 与 AI 交互

### Cursor Agent（推荐）

已安装 Superpowers 插件时，AI 自动遵循完整的 SDD 工作流。

**探索阶段**：
- "我想做一个..." / "帮我设计一个新功能" → brainstorming → explore.md

**定义阶段**：
- 确认 explore.md 后 → /opsx:propose → 全套产物

**计划阶段**：
- 确认定义产物后 → writing-plans → plans.md

**构建阶段**：
- 确认 plans.md 后 → subagent-driven-development → 按步执行

**调试/验证**：
- "修复这个bug" → systematic-debugging → TDD 修复
- "看看页面效果" → Playwright 浏览器测试

### OpenSpec 命令

```
/opsx:propose    # 基于 explore.md 创建变更提案及全套产物
/opsx:apply      # 按 plans.md 实施任务
/opsx:verify     # 对照 specs/ 验证实施
/opsx:sync       # 合并增量规范到主干
/opsx:archive    # 归档完成的变更
```

## 环境适配

本项目支持多部署目标，通过适配器模式实现跨平台：

- **Web 浏览器** — 标准部署，响应式设计
- **CEF 容器** — 桌面应用嵌入，通过 `window.__cef__` 桥接
- **Electron** — 桌面应用
- **Mobile WebView** — 移动端嵌入

详见 `.cursor/rules/03-ui-and-platform.mdc`

## 许可证

MIT
