# CI/CD 快速实施指南

本文档提供快速实施 CI/CD 流水线的步骤说明。

## 📋 前置要求

- Node.js 24.13.0 或更高版本
- pnpm 10.28.1
- Git 已安装并配置

## 🚀 快速开始

### 步骤 1: 安装依赖

```bash
pnpm install
```

这将自动安装以下新依赖：
- `husky` - Git Hooks 管理
- `lint-staged` - 暂存文件检查
- `@commitlint/cli` - 提交信息检查
- `@commitlint/config-conventional` - 提交信息规范配置

### 步骤 2: 初始化 Husky

```bash
pnpm exec husky install
```

或者直接运行：

```bash
pnpm prepare
```

### 步骤 3: 验证 Git Hooks

#### 测试 pre-commit Hook

1. 创建一个测试文件或修改现有文件
2. 暂存文件：
   ```bash
   git add .
   ```
3. 尝试提交（使用符合规范的提交信息）：
   ```bash
   git commit -m "test: test pre-commit hook"
   ```
4. 如果代码有 ESLint 错误或 TypeScript 类型错误，提交将被阻止

#### 测试 commit-msg Hook

尝试使用不符合规范的提交信息：

```bash
git commit -m "invalid commit message"
```

应该会看到错误提示，要求使用符合 Conventional Commits 规范的提交信息。

### 步骤 4: 验证 CI 流水线

1. 推送代码到 GitHub：
   ```bash
   git push origin main
   ```

2. 或者创建 Pull Request

3. 在 GitHub Actions 页面查看 CI 执行结果：
   - 访问：`https://github.com/your-username/zhuyi-blog/actions`

## ✅ 验证清单

- [ ] 依赖安装成功
- [ ] Husky 初始化成功
- [ ] pre-commit Hook 正常工作（提交时自动运行 ESLint 和类型检查）
- [ ] commit-msg Hook 正常工作（验证提交信息格式）
- [ ] CI 流水线在 GitHub Actions 中正常运行

## 📝 提交信息格式

所有提交信息必须遵循 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

### 格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 允许的类型

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
- `breaking`: 破坏性更改
- `enhancement`: 功能增强

### 示例

✅ **正确**：
```
feat: add user authentication
fix(api): resolve email sending issue
docs: update README with setup instructions
```

❌ **错误**：
```
added new feature
fixed bug
update docs
```

## 🔧 常见问题

### Q: Git Hooks 不执行？

**A:** 确保：
1. 已运行 `pnpm prepare` 或 `pnpm exec husky install`
2. `.husky` 目录已提交到仓库
3. Git Hooks 文件有执行权限（在 Linux/Mac 上）

### Q: 如何跳过 Git Hooks？

**A:** 使用 `--no-verify` 标志（不推荐）：

```bash
git commit --no-verify -m "紧急修复"
```

### Q: lint-staged 检查太慢？

**A:** 可以调整 `package.json` 中的 `lint-staged` 配置，只检查必要的文件类型。

### Q: CI 流水线失败？

**A:** 检查：
1. GitHub Actions 日志中的错误信息
2. 确保所有依赖都已正确安装
3. 确保 Node.js 和 pnpm 版本匹配

## 📚 更多信息

详细的设计方案请参考：[CI-CD-PIPELINE-DESIGN.md](./CI-CD-PIPELINE-DESIGN.md)

## 🆘 需要帮助？

如果遇到问题，请：
1. 查看详细设计文档
2. 检查 GitHub Actions 日志
3. 提交 Issue 到项目仓库
