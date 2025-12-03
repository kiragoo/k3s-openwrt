# GitHub Actions CI/CD 指南

## 📋 概述

本项目使用 GitHub Actions 实现自动化构建和发布流程，完全替代 CircleCI。

## 🔄 工作流说明

### 1. Build and Test Workflow (构建和测试)

**文件**: `.github/workflows/build-and-test.yml`

**触发条件**:
- ✅ 推送到任何分支
- ✅ 创建 Pull Request
- ❌ 不在 tag 推送时触发

**执行步骤**:
1. 检出代码
2. 安装依赖 (curl, tar, gzip, make)
3. 构建 k3s 1.30.11+k3s1 x86_64 包
4. 验证包结构
   - 检查 .opk 文件存在
   - 验证 control.tar.gz, data.tar.gz, debian-binary
   - 检查 control 文件内容
   - 验证包含所需文件
5. 验证脚本语法
   - init 脚本语法检查
   - wrapper 脚本语法检查
6. 上传构建产物
   - 单个 .opk 文件
   - 完整 build 目录

**产物保留**: 30 天

### 2. Release Workflow (发布)

**文件**: `.github/workflows/release.yml`

**触发条件**:
- ✅ 推送 tag (格式: v*.*.* 或数字版本)
- 例如: v1.30.11-1, 1.30.11+k3s1

**执行步骤**:
1. 检出代码
2. 安装依赖
3. 构建所有架构
   - x86_64
   - arm64
   - armhf
   - aarch64_cortex-a72
4. 创建 GitHub Release
   - 自动生成 release notes
   - 上传所有 .opk 文件
5. 保存构建产物

**产物保留**: 90 天

## 🚀 使用方法

### 方法 1: 自动触发（推荐）

#### 触发构建和测试:
```bash
# 推送到任何分支即可自动触发
git add .
git commit -m "Your commit message"
git push origin your-branch-name
```

#### 触发发布:
```bash
# 创建并推送 tag
git tag v1.30.11-1
git push origin v1.30.11-1
```

### 方法 2: 手动触发

可以在 GitHub Actions 页面手动触发工作流（需要先配置 workflow_dispatch）。

## 📊 查看构建状态

### 在 GitHub 仓库中查看:

1. **Actions 标签页**
   - 访问: `https://github.com/YOUR_USERNAME/k3s-openwrt/actions`
   - 查看所有工作流运行历史
   - 点击具体运行查看详细日志

2. **Commit 状态**
   - 在 commit 列表中，每个 commit 旁边会显示状态图标
   - ✅ 绿色勾: 构建成功
   - ❌ 红色叉: 构建失败
   - 🟡 黄色圆: 构建进行中

3. **Pull Request 检查**
   - PR 页面会显示所有检查状态
   - 必须通过所有检查才能合并

## 📦 下载构建产物

### 从 Actions 页面下载:

1. 访问 Actions 标签页
2. 点击具体的工作流运行
3. 滚动到页面底部的 "Artifacts" 部分
4. 下载:
   - `k3s-openwrt-package`: 单个 .opk 文件
   - `build-directory`: 完整构建目录

### 从 Releases 页面下载:

1. 访问: `https://github.com/YOUR_USERNAME/k3s-openwrt/releases`
2. 找到对应的 release 版本
3. 在 "Assets" 部分下载所需架构的 .opk 文件

## 🔧 配置说明

### 权限要求

Release workflow 需要写入权限来创建 release:
```yaml
permissions:
  contents: write
```

这个权限已经在 `.github/workflows/release.yml` 中配置。

### Secrets 配置

GitHub Actions 自动提供 `GITHUB_TOKEN`，无需额外配置。

如果需要访问其他服务，可以在仓库设置中添加 secrets:
1. 访问: Settings → Secrets and variables → Actions
2. 点击 "New repository secret"
3. 添加所需的 secret

## 📈 工作流对比

### GitHub Actions vs CircleCI

| 特性 | GitHub Actions | CircleCI |
|------|----------------|----------|
| 集成方式 | 原生集成 | 需要外部配置 |
| 配置文件 | `.github/workflows/*.yml` | `.circleci/config.yml` |
| 触发方式 | 自动（推送即触发） | 需要 webhook |
| 产物存储 | GitHub Artifacts | CircleCI Artifacts |
| Release | 直接创建 GitHub Release | 需要 ghr 工具 |
| 免费额度 | 公开仓库无限 | 有限制 |
| 设置难度 | ⭐ 简单 | ⭐⭐ 中等 |

## 🎯 工作流示例

### 示例 1: 开发分支构建

```bash
# 1. 创建功能分支
git checkout -b feature/new-feature

# 2. 进行修改
echo "some changes" >> README.md

# 3. 提交并推送
git add .
git commit -m "Add new feature"
git push origin feature/new-feature

# 4. GitHub Actions 自动触发构建
# 5. 在 Actions 页面查看构建状态
# 6. 构建成功后下载 artifacts
```

### 示例 2: 创建 Release

```bash
# 1. 确保主分支是最新的
git checkout main
git pull origin main

# 2. 创建 tag
git tag v1.30.11-1

# 3. 推送 tag
git push origin v1.30.11-1

# 4. GitHub Actions 自动:
#    - 构建所有架构
#    - 创建 GitHub Release
#    - 上传所有 .opk 文件

# 5. 访问 Releases 页面查看发布
```

### 示例 3: Pull Request 工作流

```bash
# 1. Fork 仓库或创建分支
git checkout -b fix/bug-fix

# 2. 进行修改并推送
git add .
git commit -m "Fix bug"
git push origin fix/bug-fix

# 3. 在 GitHub 创建 Pull Request

# 4. GitHub Actions 自动运行检查
# 5. 所有检查通过后才能合并
```

## 🐛 故障排查

### 问题 1: 工作流没有触发

**检查项**:
- ✅ 确认 `.github/workflows/` 目录存在
- ✅ 确认 YAML 文件语法正确
- ✅ 确认推送到了正确的分支
- ✅ 检查 Actions 是否被禁用

**解决方法**:
1. 访问 Settings → Actions → General
2. 确保 "Actions permissions" 设置为 "Allow all actions"

### 问题 2: 构建失败

**常见原因**:
- 依赖安装失败
- k3s 二进制下载失败
- 包验证失败

**解决方法**:
1. 点击失败的工作流运行
2. 展开失败的步骤查看详细日志
3. 根据错误信息修复问题
4. 重新推送代码触发构建

### 问题 3: Release 创建失败

**可能原因**:
- Tag 格式不正确
- 权限不足
- 同名 release 已存在

**解决方法**:
1. 检查 tag 格式是否符合 `v*.*.*` 或数字版本
2. 确认 workflow 文件中有 `contents: write` 权限
3. 删除已存在的 release 或使用新的 tag

### 问题 4: 找不到 Artifacts

**解决方法**:
1. 确认工作流运行完成
2. 在工作流运行页面滚动到底部
3. 查看 "Artifacts" 部分
4. 如果没有，检查 upload-artifact 步骤是否成功

## 📝 自定义配置

### 修改构建的 k3s 版本

编辑 `.github/workflows/build-and-test.yml`:
```yaml
- name: Build k3s package
  run: |
    make VERSION=1.31.0+k3s1 ARCH=x86_64  # 修改这里
```

### 添加更多架构

编辑 `.github/workflows/release.yml`:
```yaml
- name: Build all architectures
  run: |
    for arch in x86_64 arm64 armhf aarch64_cortex-a72 mips; do  # 添加新架构
      echo "Building for $arch..."
      make VERSION=1.30.11+k3s1 ARCH=$arch
    done
```

### 修改 Artifacts 保留时间

```yaml
- name: Upload build artifacts
  uses: actions/upload-artifact@v4
  with:
    name: k3s-openwrt-package
    path: build/k3s_1.30.11+k3s1_x86_64.opk
    retention-days: 60  # 修改保留天数
```

## 🔐 安全最佳实践

1. **不要在代码中硬编码敏感信息**
   - 使用 GitHub Secrets 存储 tokens 和密码

2. **限制工作流权限**
   - 只授予必需的权限
   - 使用 `permissions` 字段明确声明

3. **审查第三方 Actions**
   - 使用官方或知名的 Actions
   - 固定 Actions 版本（使用 @v4 而不是 @main）

4. **保护主分支**
   - 要求 PR 通过所有检查
   - 要求代码审查

## 📚 相关资源

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Workflow 语法](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Actions Marketplace](https://github.com/marketplace?type=actions)

## ✅ 验证清单

部署后验证:
- [ ] 推送代码触发 build-and-test workflow
- [ ] 工作流运行成功
- [ ] Artifacts 正确上传
- [ ] 创建 PR 触发检查
- [ ] 推送 tag 触发 release workflow
- [ ] GitHub Release 创建成功
- [ ] 所有架构的 .opk 文件都已上传

---

**当前配置特性**:
- ✅ 自动构建和测试（所有分支）
- ✅ 自动发布（tag 推送）
- ✅ 完整的包验证
- ✅ 脚本语法检查
- ✅ 多架构支持
- ✅ Artifacts 自动保存
- ✅ GitHub Release 自动创建
- ✅ 无需外部服务配置
