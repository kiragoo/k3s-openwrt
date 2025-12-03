# 触发 CircleCI 构建指南

## 🎯 快速触发方法

### 方法 1：空提交触发（最简单）

```bash
# 创建一个空提交
git commit --allow-empty -m "Trigger CircleCI build"

# 推送到远程分支
git push origin <your-branch-name>
```

### 方法 2：修改文件触发

```bash
# 添加一个小改动（比如在 README 末尾加个空行）
echo "" >> README.md

# 提交并推送
git add README.md
git commit -m "Trigger CircleCI build"
git push origin <your-branch-name>
```

## 🔍 检查 CircleCI 状态

### 1. 在 CircleCI 网站检查

访问：https://app.circleci.com/pipelines/github/<your-username>/k3s-openwrt

您应该能看到：
- ✅ 绿色：构建成功
- 🔴 红色：构建失败
- 🟡 黄色：构建进行中
- ⚪ 灰色：等待中

### 2. 在 GitHub 检查

1. 访问您的 GitHub 仓库
2. 点击最新的 commit
3. 查看 commit 旁边的状态图标
4. 点击 "Details" 查看 CircleCI 构建详情

### 3. 检查 GitHub Webhook

如果构建没有自动触发：

1. 访问：https://github.com/<your-username>/k3s-openwrt/settings/hooks
2. 查找 CircleCI webhook
3. 点击 webhook 查看最近的交付记录
4. 如果没有 webhook，需要在 CircleCI 重新设置项目

## 🛠️ 首次设置 CircleCI

如果这是第一次使用 CircleCI：

### 步骤 1：连接 GitHub 账号
1. 访问 https://circleci.com/
2. 点击 "Sign Up" 或 "Log In"
3. 选择 "Sign up with GitHub"
4. 授权 CircleCI 访问您的 GitHub 账号

### 步骤 2：添加项目
1. 登录后，点击左侧的 "Projects"
2. 找到 `k3s-openwrt` 仓库
3. 点击 "Set Up Project"
4. 选择 "Use Existing Config"（因为我们已经有 .circleci/config.yml）
5. 选择您的分支
6. 点击 "Set Up Project"

### 步骤 3：验证构建
1. CircleCI 会自动触发第一次构建
2. 等待构建完成（大约 5-10 分钟）
3. 检查构建日志确认一切正常

## 📊 预期的构建流程

当 CircleCI 触发后，您会看到以下步骤：

```
1. Spin up environment (启动环境)
2. Checkout code (检出代码)
3. Install dependencies (安装依赖)
4. Build k3s package (构建 k3s 包)
   - 下载 k3s 1.30.11+k3s1 二进制文件
   - 打包成 .opk 文件
5. Verify package structure (验证包结构)
   - 检查文件存在性
   - 验证 control 文件
   - 验证 data 归档
6. Verify script syntax (验证脚本语法)
7. Store artifacts (保存构建产物)
```

## 🐛 常见问题排查

### 问题 1：CircleCI 没有触发

**可能原因：**
- CircleCI 项目未设置
- GitHub webhook 未配置
- 分支名称不匹配

**解决方法：**
1. 检查 CircleCI 项目是否已添加
2. 检查 GitHub webhook 设置
3. 确认推送到正确的分支

### 问题 2：构建失败

**可能原因：**
- 依赖安装失败
- k3s 二进制下载失败
- 包结构验证失败

**解决方法：**
1. 查看 CircleCI 构建日志
2. 检查具体失败的步骤
3. 根据错误信息调整配置

### 问题 3：找不到 artifacts

**解决方法：**
1. 等待构建完全完成
2. 在 CircleCI 构建页面点击 "Artifacts" 标签
3. 下载 `k3s_1.30.11+k3s1_x86_64.opk` 文件

## 📦 下载构建产物

构建成功后，可以在以下位置找到 .opk 文件：

1. **CircleCI Artifacts**
   - 访问构建页面
   - 点击 "Artifacts" 标签
   - 下载 `k3s_1.30.11+k3s1_x86_64.opk`

2. **本地构建**（如果在 Linux 环境）
   ```bash
   make VERSION=1.30.11+k3s1 ARCH=x86_64
   # 文件位置：build/k3s_1.30.11+k3s1_x86_64.opk
   ```

## 🏷️ 创建 Release

要触发 release workflow 并发布到 GitHub Releases：

```bash
# 创建并推送 tag
git tag v1.30.11-1
git push origin v1.30.11-1
```

这会触发 `release-workflow`，构建所有架构并发布到 GitHub Releases。

## 📞 需要帮助？

如果遇到问题：
1. 检查 CircleCI 构建日志
2. 查看 GitHub commit 状态
3. 验证 webhook 配置
4. 确认 CircleCI 项目设置

---

**当前配置支持：**
- ✅ 所有分支自动构建和测试
- ✅ Tag 推送自动发布到 GitHub Releases
- ✅ 构建产物自动保存为 artifacts
- ✅ 完整的包结构验证
- ✅ 脚本语法检查
