# CI/CD 流水线设计方案文档

## 目录

- [1. 项目架构梳理](#1-项目架构梳理)
- [2. Git Hooks 设计方案](#2-git-hooks-设计方案)
- [3. CI/CD 流水线设计](#3-cicd-流水线设计)
- [4. 实施步骤](#4-实施步骤)
- [5. 工具配置说明](#5-工具配置说明)

---

## 1. 项目架构梳理

### 1.1 项目概述

**zhuyi-blog** 是一个基于 Nuxt.js 4 构建的现代化博客/作品集网站，支持多语言（中文、英文、法文）和内容管理系统。

### 1.2 技术栈

#### 核心框架
- **Nuxt.js 4.2.2** - Vue.js 全栈框架
- **Vue 3** - 渐进式 JavaScript 框架
- **TypeScript** - 类型安全的 JavaScript 超集
- **Node.js 24.13.0** - 运行时环境

#### 主要依赖
- **@nuxt/content 3.11.0** - 内容管理系统
- **@nuxt/ui 4.4.0** - UI 组件库
- **@nuxtjs/i18n 10.2.1** - 国际化支持
- **@nuxtjs/seo 3.3.0** - SEO 优化
- **@nuxt/image 2.0.0** - 图片优化
- **nuxt-studio 1.1.1** - 可视化内容编辑器
- **better-sqlite3 12.6.2** - SQLite 数据库
- **resend 6.8.0** - 邮件服务

#### 开发工具
- **pnpm 10.28.1** - 包管理器
- **ESLint 9.39.2** - 代码检查工具
- **@nuxt/eslint-config 1.12.1** - Nuxt ESLint 配置
- **TypeScript Compiler** - 类型检查

### 1.3 项目结构

```
zhuyi-blog/
├── app/                          # 应用代码
│   ├── app.config.ts            # 应用配置
│   ├── app.vue                  # 根组件
│   ├── assets/                  # 静态资源
│   │   ├── icons/               # 图标文件
│   │   └── style/               # 样式文件
│   ├── components/              # Vue 组件
│   │   ├── about/               # 关于页面组件
│   │   ├── content/             # 内容组件
│   │   ├── home/                 # 首页组件
│   │   ├── layout/              # 布局组件
│   │   └── ...
│   ├── composables/             # 组合式函数
│   ├── layouts/                 # 布局文件
│   ├── pages/                   # 页面路由
│   │   ├── [...slug].vue        # 动态路由
│   │   └── articles/            # 文章页面
│   └── utils/                   # 工具函数
├── content/                      # 内容文件（Markdown/JSON）
│   ├── en/                      # 英文内容
│   ├── fr/                      # 法文内容
│   └── zh/                      # 中文内容
├── i18n/                         # 国际化配置
│   ├── i18n.config.ts
│   └── locales/                 # 语言文件
├── server/                       # 服务端代码
│   ├── api/                     # API 路由
│   ├── emails/                  # 邮件服务
│   └── routes/                  # 服务端路由
├── public/                       # 公共静态资源
├── .github/                      # GitHub 配置
│   ├── workflows/               # GitHub Actions
│   └── ISSUE_TEMPLATE/          # Issue 模板
├── nuxt.config.ts               # Nuxt 配置
├── nuxt.schema.ts               # Nuxt 配置模式
├── tsconfig.json                # TypeScript 配置
├── eslint.config.mjs            # ESLint 配置
├── Dockerfile                   # Docker 构建文件
└── package.json                 # 项目依赖配置
```

### 1.4 构建与部署

#### 构建命令
- `pnpm dev` - 开发模式
- `pnpm build` - 生产构建
- `pnpm generate` - 静态站点生成
- `pnpm preview` - 预览构建结果
- `pnpm start` - 启动生产服务器

#### 部署方式
- **Docker** - 容器化部署（已有 Dockerfile）
- **Vercel/Netlify** - 无服务器部署
- **NuxtHub** - Nuxt 官方托管平台

### 1.5 现有 CI/CD 状态

#### 已存在的 GitHub Actions
1. **build-image.yml** - Docker 镜像构建和推送
   - 触发条件：标签推送（v*）或手动触发
   - 功能：构建 Docker 镜像并推送到 GitHub Container Registry

2. **semantic-pull-request.yml** - PR 标题验证
   - 触发条件：PR 创建/更新
   - 功能：验证 PR 标题是否符合 Conventional Commits 规范

3. **label-pr.yml** - PR 标签自动添加
   - 触发条件：PR 创建
   - 功能：根据 PR 标题自动添加相应标签

#### 缺失的功能
- ❌ 代码质量检查（Lint）
- ❌ 类型检查（TypeScript）
- ❌ 单元测试/集成测试
- ❌ 构建验证
- ❌ Git Hooks 配置
- ❌ 提交信息验证
- ❌ 代码格式化检查

---

## 2. Git Hooks 设计方案

### 2.1 设计目标

在本地提交前进行代码质量检查，确保：
- 代码符合 ESLint 规范
- TypeScript 类型检查通过
- 提交信息符合 Conventional Commits 规范
- 避免将问题代码推送到远程仓库

### 2.2 工具选择

推荐使用 **Husky** + **lint-staged** 组合：

- **Husky** - Git Hooks 管理工具
- **lint-staged** - 仅对暂存文件运行检查，提高效率
- **commitlint** - 提交信息规范检查

### 2.3 Git Hooks 配置

#### 2.3.1 pre-commit Hook

**功能**：在提交前执行代码检查

**检查项**：
1. ESLint 检查（仅检查暂存文件）
2. TypeScript 类型检查
3. 代码格式化检查（可选）

**配置示例**：

```json
// package.json
{
  "scripts": {
    "prepare": "husky install",
    "lint-staged": "lint-staged"
  },
  "lint-staged": {
    "*.{js,ts,vue}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{ts,vue}": [
      "bash -c 'pnpm typecheck'"
    ]
  }
}
```

#### 2.3.2 commit-msg Hook

**功能**：验证提交信息格式

**规则**：遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范

**格式要求**：
```
<type>(<scope>): <subject>

<body>

<footer>
```

**允许的类型**：
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档更新
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `perf`: 性能优化
- `test`: 测试相关
- `build`: 构建系统
- `ci`: CI 配置
- `chore`: 其他更改
- `revert`: 回滚提交

**配置示例**：

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert',
        'breaking',
        'enhancement',
      ],
    ],
    'subject-case': [0],
  },
}
```

#### 2.3.3 pre-push Hook（可选）

**功能**：在推送前执行完整检查

**检查项**：
1. 运行完整的 ESLint 检查
2. 运行完整的 TypeScript 类型检查
3. 运行构建测试（确保代码可以正常构建）

---

## 3. CI/CD 流水线设计

### 3.1 流水线架构图

```
┌─────────────────────────────────────────────────────────────┐
│                     代码提交阶段                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  pre-commit  │→ │ commit-msg   │→ │   pre-push   │     │
│  │   (本地)     │  │   (本地)     │  │   (本地)     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                  GitHub Actions CI 阶段                       │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Job 1: 代码质量检查 (Code Quality)                  │   │
│  │  - ESLint 检查                                        │   │
│  │  - TypeScript 类型检查                                │   │
│  │  - 代码格式化检查                                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Job 2: 构建验证 (Build Verification)                │   │
│  │  - 安装依赖                                           │   │
│  │  - 生产构建测试                                        │   │
│  │  - 静态生成测试                                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Job 3: PR 验证 (Pull Request Validation)            │   │
│  │  - PR 标题规范检查 (已有)                             │   │
│  │  - PR 标签自动添加 (已有)                             │   │
│  │  - 代码覆盖率检查 (可选)                              │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                  CD 部署阶段 (可选)                            │
│                                                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Job 4: Docker 镜像构建 (已有)                        │   │
│  │  - 构建 Docker 镜像                                   │   │
│  │  - 推送到 GitHub Container Registry                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Job 5: 部署到生产环境 (可选)                          │   │
│  │  - 部署到 Vercel/Netlify                             │   │
│  │  - 部署到 NuxtHub                                    │   │
│  │  - 部署到自托管服务器                                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 CI 流水线详细设计

#### 3.2.1 触发条件

**Pull Request 触发**：
- PR 创建
- PR 更新（代码推送）
- PR 同步

**Push 触发**：
- 推送到 `main` 分支
- 标签推送（v*）

**手动触发**：
- 通过 GitHub Actions UI 手动运行

#### 3.2.2 Job 1: 代码质量检查

**名称**：`code-quality-check`

**运行环境**：`ubuntu-latest`

**步骤**：

1. **检出代码**
   ```yaml
   - uses: actions/checkout@v4
   ```

2. **设置 pnpm**
   ```yaml
   - uses: pnpm/action-setup@v3
     with:
       version: 10.28.1
   ```

3. **设置 Node.js**
   ```yaml
   - uses: actions/setup-node@v4
     with:
       node-version: '24.13.0'
       cache: 'pnpm'
   ```

4. **安装依赖**
   ```yaml
   - run: pnpm install --frozen-lockfile
   ```

5. **ESLint 检查**
   ```yaml
   - run: pnpm lint
   ```

6. **TypeScript 类型检查**
   ```yaml
   - run: pnpm typecheck
   ```

**缓存策略**：
- 缓存 `node_modules` 和 `pnpm` 缓存以加速构建

#### 3.2.3 Job 2: 构建验证

**名称**：`build-verification`

**运行环境**：`ubuntu-latest`

**依赖**：Job 1（代码质量检查通过后）

**步骤**：

1. **检出代码**
2. **设置 pnpm 和 Node.js**（同 Job 1）
3. **安装依赖**
4. **生产构建测试**
   ```yaml
   - run: pnpm build
   ```
5. **静态生成测试**（可选）
   ```yaml
   - run: pnpm generate
   ```

**构建产物**：
- 上传构建产物作为 artifact（用于后续部署或测试）

#### 3.2.4 Job 3: PR 验证（增强）

**名称**：`pr-validation`

**运行环境**：`ubuntu-latest`

**功能**：
- 继承现有的 PR 标题验证和标签添加功能
- 添加代码覆盖率检查（如果后续添加测试）

#### 3.2.5 Job 4: Docker 镜像构建（已有，优化）

**名称**：`build-and-push-image`

**优化建议**：
1. 添加多阶段构建缓存
2. 添加安全扫描（Trivy）
3. 添加镜像大小检查

### 3.3 工作流文件结构

```
.github/workflows/
├── ci.yml                    # 主 CI 流水线（新增）
├── build-image.yml          # Docker 构建（已有，优化）
├── semantic-pull-request.yml # PR 标题验证（已有）
└── label-pr.yml             # PR 标签（已有）
```

### 3.4 并行执行策略

- **Job 1** 和 **Job 2** 可以并行执行（如果 Job 2 不依赖 Job 1 的缓存）
- 或者 **Job 2** 在 **Job 1** 成功后执行（推荐，节省资源）

### 3.5 失败处理策略

1. **通知机制**：
   - PR 评论自动通知
   - 可选：Slack/Discord 通知

2. **重试机制**：
   - 网络问题自动重试
   - 临时性错误重试

3. **阻塞策略**：
   - PR 合并前必须通过所有检查
   - 使用 GitHub 分支保护规则

---

## 4. 实施步骤

### 4.1 阶段一：Git Hooks 配置

#### 步骤 1.1：安装依赖

```bash
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional
```

#### 步骤 1.2：初始化 Husky

```bash
pnpm exec husky install
```

#### 步骤 1.3：配置 pre-commit Hook

创建 `.husky/pre-commit`：

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

#### 步骤 1.4：配置 commit-msg Hook

创建 `.husky/commit-msg`：

```bash
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm exec commitlint --edit $1
```

#### 步骤 1.5：配置 lint-staged

在 `package.json` 中添加：

```json
{
  "lint-staged": {
    "*.{js,ts,vue}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{ts,vue}": [
      "bash -c 'pnpm typecheck'"
    ]
  }
}
```

#### 步骤 1.6：创建 commitlint 配置

创建 `commitlint.config.js`：

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert',
        'breaking',
        'enhancement',
      ],
    ],
    'subject-case': [0],
  },
}
```

### 4.2 阶段二：CI 流水线配置

#### 步骤 2.1：创建主 CI 工作流

创建 `.github/workflows/ci.yml`（详见 5.1 节）

#### 步骤 2.2：优化现有工作流

优化 `.github/workflows/build-image.yml`（添加缓存和安全扫描）

#### 步骤 2.3：配置分支保护规则

在 GitHub 仓库设置中配置：
- 要求 PR 通过 CI 检查
- 要求代码审查
- 要求 PR 标题符合规范

### 4.3 阶段三：测试与验证

#### 步骤 3.1：本地测试 Git Hooks

```bash
# 测试 pre-commit
git add .
git commit -m "test: test pre-commit hook"

# 测试 commit-msg（应该失败）
git commit -m "invalid commit message"
```

#### 步骤 3.2：测试 CI 流水线

1. 创建测试分支
2. 推送代码触发 CI
3. 验证所有 Job 正常运行

### 4.4 阶段四：文档更新

更新 `README.md`，添加：
- Git Hooks 使用说明
- CI/CD 流程说明
- 贡献指南更新

---

## 5. 工具配置说明

### 5.1 主 CI 工作流配置

创建 `.github/workflows/ci.yml`：

```yaml
name: CI

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main
    tags:
      - 'v*'

env:
  PNPM_VERSION: 10.28.1
  NODE_VERSION: '24.13.0'

jobs:
  code-quality:
    name: Code Quality Check
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run ESLint
        run: pnpm lint

      - name: Run TypeScript type check
        run: pnpm typecheck

  build:
    name: Build Verification
    runs-on: ubuntu-latest
    needs: code-quality

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: ${{ env.PNPM_VERSION }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build production
        run: pnpm build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: .output
          retention-days: 7
```

### 5.2 package.json 更新

在 `package.json` 的 `scripts` 中添加：

```json
{
  "scripts": {
    "prepare": "husky install",
    "lint-staged": "lint-staged"
  }
}
```

### 5.3 .husky 目录结构

```
.husky/
├── _/
│   └── husky.sh
├── pre-commit
└── commit-msg
```

### 5.4 依赖包清单

**开发依赖**（需要安装）：

```json
{
  "devDependencies": {
    "husky": "^9.0.0",
    "lint-staged": "^15.0.0",
    "@commitlint/cli": "^19.0.0",
    "@commitlint/config-conventional": "^19.0.0"
  }
}
```

### 5.5 可选增强功能

#### 5.5.1 代码覆盖率

如果后续添加测试框架（如 Vitest），可以添加覆盖率检查：

```yaml
- name: Run tests with coverage
  run: pnpm test:coverage

- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v4
```

#### 5.5.2 安全扫描

添加依赖安全扫描：

```yaml
- name: Run security audit
  run: pnpm audit --audit-level=moderate
```

#### 5.5.3 性能测试

添加 Lighthouse CI 性能测试：

```yaml
- name: Run Lighthouse CI
  uses: treosh/lighthouse-ci-action@v11
```

---

## 6. 总结

### 6.1 方案优势

1. **早期发现问题**：通过 Git Hooks 在本地提交前发现问题
2. **自动化检查**：CI 流水线自动运行，无需手动触发
3. **代码质量保障**：多层检查确保代码质量
4. **标准化流程**：统一的提交规范和检查流程
5. **快速反馈**：本地检查快速，CI 检查全面

### 6.2 实施优先级

**高优先级**（立即实施）：
1. ✅ Git Hooks 配置（pre-commit, commit-msg）
2. ✅ 主 CI 流水线（代码质量检查 + 构建验证）

**中优先级**（后续优化）：
1. ⚠️ 测试框架集成
2. ⚠️ 代码覆盖率检查
3. ⚠️ 安全扫描

**低优先级**（可选）：
1. ⚪ 性能测试
2. ⚪ 自动化部署

### 6.3 维护建议

1. **定期更新依赖**：使用 Renovate（已配置）自动更新
2. **监控 CI 执行时间**：优化慢速 Job
3. **收集反馈**：根据团队反馈调整检查规则
4. **文档同步**：及时更新相关文档

---

## 附录

### A. 参考资源

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Husky 文档](https://typicode.github.io/husky/)
- [lint-staged 文档](https://github.com/okonet/lint-staged)
- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Commitlint 文档](https://commitlint.js.org/)

### B. 常见问题

**Q: Git Hooks 不生效？**
A: 确保已运行 `pnpm exec husky install`，并且 `.husky` 目录已提交到仓库。

**Q: CI 执行时间过长？**
A: 使用缓存机制，并行执行独立 Job，优化构建步骤。

**Q: 如何跳过 Git Hooks？**
A: 使用 `git commit --no-verify`（不推荐，仅在紧急情况下使用）。

---

**文档版本**：v1.0.0  
**最后更新**：2025-02-10  
**维护者**：项目团队
