# Design Document

## Overview

本设计文档描述了将 k3s-on-openwrt 项目升级以支持 OpenWrt 24.10.0 和 k3s 1.30.11+k3s1 的技术方案。该项目是一个基于 Makefile 的构建系统，用于从官方 k3s 二进制文件生成 OpenWrt opkg 包。

升级的核心目标是：
- 添加对 k3s 1.30.11+k3s1 版本的支持
- 确保与 OpenWrt 24.10.0 的兼容性
- 验证并更新依赖关系
- 保持向后兼容性
- 更新相关文档

## Architecture

### 系统组件架构

```
┌─────────────────────────────────────────────────────────┐
│                    Build System (Makefile)               │
├─────────────────────────────────────────────────────────┤
│  - Version Management (VERSIONS file)                    │
│  - Architecture Support (ARCHS file)                     │
│  - Package Generation (control, data, debian-binary)     │
│  - Binary Download (from k3s GitHub releases)            │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│              Generated .opk Package                      │
├─────────────────────────────────────────────────────────┤
│  ├── control.tar.gz (metadata, dependencies)            │
│  ├── data.tar.gz (files)                                │
│  │   ├── /usr/bin/k3s (binary)                          │
│  │   ├── /usr/bin/k3s-wrapper (wrapper script)          │
│  │   └── /etc/init.d/k3s (init script)                  │
│  └── debian-binary (format version)                     │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│           OpenWrt 24.10.0 Runtime Environment            │
├─────────────────────────────────────────────────────────┤
│  - Init System (procd/rc.common)                        │
│  - UCI Configuration System                              │
│  - Kernel Features (cgroup, namespaces, vxlan)          │
│  - Network Stack (iptables, br-netfilter)               │
└─────────────────────────────────────────────────────────┘
```

### 构建流程

```
[VERSIONS file] ──→ [Makefile] ──→ [Download k3s binary]
                         ↓
                    [Copy files/]
                         ↓
                [Generate control file]
                         ↓
                [Create tar archives]
                         ↓
                [Package .opk file]
```

## Components and Interfaces

### 1. Build System (Makefile)

**职责：**
- 管理构建过程
- 下载 k3s 二进制文件
- 生成 opkg 包结构
- 创建控制文件和元数据

**关键变量：**
- `VERSION`: k3s 版本（默认从 VERSIONS 文件第一行读取）
- `ARCH`: 目标架构（默认 x86_64）
- `PVERSION`: 包版本号
- `suffix`: 架构后缀（x86_64 为空）

**关键目标：**
- `all`: 构建单个包
- `build-all`: 构建所有架构和版本的组合
- `clean`: 清理构建产物

### 2. Version Configuration (VERSIONS file)

**职责：**
- 定义支持的 k3s 版本列表
- 第一行为默认版本

**格式：**
```
1.30.11+k3s1
1.27.4+k3s1
...
```

### 3. Architecture Configuration (ARCHS file)

**职责：**
- 定义支持的目标架构

**当前支持：**
- x86_64
- arm64
- armhf
- aarch64_cortex-a72

### 4. Init Script (/etc/init.d/k3s)

**职责：**
- 初始化 k3s 运行环境
- 挂载 cgroup 文件系统
- 启动和停止 k3s 服务
- 从 UCI 读取配置

**关键函数：**
- `ensure_cgroup_mount()`: 确保 cgroup 正确挂载
- `start()`: 启动 k3s 服务
- `stop()`: 停止 k3s 服务

**UCI 配置接口：**
- `k3s.globals.opts`: k3s 启动选项
- `k3s.globals.root`: k3s 数据目录

### 5. Wrapper Script (/usr/bin/k3s-wrapper)

**职责：**
- 启动 k3s 进程
- 重定向输出到系统日志
- 处理信号传递
- 管理子进程生命周期

**信号处理：**
- 捕获终止信号
- 传递给 k3s 子进程
- 等待子进程退出

### 6. Control File (Package Metadata)

**职责：**
- 定义包元数据
- 声明依赖关系
- 指定架构和版本

**关键字段：**
- `Package`: k3s
- `Version`: ${VERSION}-${PVERSION}
- `Architecture`: ${ARCH}
- `Depends`: 运行时依赖列表

## Data Models

### Package Structure

```
k3s_${VERSION}_${ARCH}.opk
├── debian-binary          # "2.0"
├── control.tar.gz
│   └── control           # Package metadata
└── data.tar.gz
    ├── usr/
    │   └── bin/
    │       ├── k3s       # k3s binary
    │       └── k3s-wrapper
    └── etc/
        └── init.d/
            └── k3s       # Init script
```

### Version String Format

k3s 版本遵循格式：`MAJOR.MINOR.PATCH+k3sRELEASE`

示例：`1.30.11+k3s1`
- MAJOR: 1
- MINOR: 30
- PATCH: 11
- k3s RELEASE: 1

### Download URL Pattern

```
https://github.com/k3s-io/k3s/releases/download/v${VERSION}/k3s${suffix}
```

对于 x86_64：
```
https://github.com/k3s-io/k3s/releases/download/v1.30.11+k3s1/k3s
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system-essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Version file consistency
*For any* build execution, the VERSION variable should equal the first line of the VERSIONS file when no explicit VERSION is provided
**Validates: Requirements 1.2**

### Property 2: Binary download URL correctness
*For any* supported k3s version and architecture, the constructed download URL should point to a valid k3s binary on GitHub releases
**Validates: Requirements 1.3**

### Property 3: Package structure completeness
*For any* successful build, the generated .opk file should contain exactly three files: debian-binary, control.tar.gz, and data.tar.gz
**Validates: Requirements 6.2**

### Property 4: Control file dependency declaration
*For any* generated control file, it should declare all required dependencies: iptables, iptables-mod-extra, kmod-ipt-extra, kmod-br-netfilter, and ca-certificates
**Validates: Requirements 2.3**

### Property 5: Data archive file presence
*For any* generated data.tar.gz, it should contain all three required files: /usr/bin/k3s, /usr/bin/k3s-wrapper, and /etc/init.d/k3s
**Validates: Requirements 6.3**

### Property 6: Binary executable permission
*For any* k3s binary included in the package, it should have executable permissions set
**Validates: Requirements 5.4**

### Property 7: Architecture suffix handling
*For any* x86_64 architecture build, the download URL suffix should be empty (no architecture suffix appended)
**Validates: Requirements 5.1**

### Property 8: Package naming convention
*For any* build with VERSION and ARCH specified, the output filename should follow the pattern k3s_${VERSION}_${ARCH}.opk
**Validates: Requirements 5.3**

### Property 9: Init script cgroup mount idempotence
*For any* system state, running the ensure_cgroup_mount function multiple times should result in the same final cgroup mount configuration
**Validates: Requirements 3.1, 3.2**

### Property 10: Wrapper script signal propagation
*For any* termination signal received by the wrapper script, the signal should be propagated to the k3s child process
**Validates: Requirements 4.3**

### Property 11: UCI configuration preservation
*For any* upgrade, the init script should continue to read from the same UCI configuration keys (k3s.globals.opts and k3s.globals.root)
**Validates: Requirements 7.2**

### Property 12: Build artifact cleanup
*For any* state of the build directory, running make clean should remove all build artifacts and leave no residual files
**Validates: Requirements 7.4**

## Error Handling

### Build-Time Errors

1. **Binary Download Failure**
   - Cause: Network issues, invalid version, missing release
   - Handling: curl 使用 `-f` 标志，失败时返回非零退出码
   - Recovery: 用户需要检查网络连接和版本号

2. **File Permission Errors**
   - Cause: 构建目录权限不足
   - Handling: Makefile 使用 `mkdir -p` 确保目录存在
   - Recovery: 用户需要检查文件系统权限

3. **Missing Dependencies**
   - Cause: 构建工具缺失（tar, curl, find）
   - Handling: Shell 命令失败时 make 会停止
   - Recovery: 用户需要安装必需的构建工具

### Runtime Errors

1. **cgroup Mount Failure**
   - Cause: 内核不支持 cgroup，权限不足
   - Handling: Init 脚本检查挂载状态，尝试挂载
   - Recovery: 用户需要配置内核支持或以 root 运行

2. **UCI Configuration Missing**
   - Cause: k3s UCI 配置未初始化
   - Handling: `uci_get` 返回空值，传递给 k3s
   - Recovery: 用户需要创建 UCI 配置或使用默认值

3. **k3s Process Crash**
   - Cause: k3s 内部错误，资源不足
   - Handling: Wrapper 脚本将错误日志发送到 syslog
   - Recovery: 用户需要检查系统日志，调整资源或配置

4. **PID File Conflicts**
   - Cause: 旧的 PID 文件残留
   - Handling: start-stop-daemon 会检查进程是否实际运行
   - Recovery: 手动清理 /var/run/k3s.pid

## CI/CD Strategy - CircleCI Integration

### Overview

为了支持在 GitHub 中自动构建当前分支的 k3s opkg 包，我们需要改造现有的 CircleCI 配置。当前配置仅在 tag 推送时触发构建，我们需要扩展以支持分支构建和测试。

### Current CircleCI Configuration Analysis

**现有配置问题：**
1. 仅在 tag 推送时触发（`filters.tags.only`）
2. 忽略所有分支构建（`filters.branches.ignore: /.*/`）
3. 使用过时的 Go 1.11 镜像
4. 缺少构建验证和测试步骤
5. 没有分支构建的 artifact 保存

### Improved CircleCI Architecture

```
┌─────────────────────────────────────────────────────────┐
│                   GitHub Events                          │
├─────────────────────────────────────────────────────────┤
│  - Push to any branch                                    │
│  - Pull Request                                          │
│  - Tag creation                                          │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                CircleCI Workflows                        │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │  build-and-test │  │  release        │              │
│  │  (all branches) │  │  (tags only)    │              │
│  └─────────────────┘  └─────────────────┘              │
└─────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────┐
│                   Build Outputs                          │
├─────────────────────────────────────────────────────────┤
│  - Branch builds: CircleCI Artifacts                     │
│  - Tag builds: GitHub Releases                           │
└─────────────────────────────────────────────────────────┘
```

### New CircleCI Configuration Design

#### Job 1: build-and-test (所有分支)

**目的：** 验证构建过程，生成 artifact 供测试和下载

**步骤：**
1. **环境准备**
   - 使用现代化的 Docker 镜像（Ubuntu 22.04 或 Alpine）
   - 安装必需工具：curl, tar, gzip, make

2. **构建验证**
   - 执行 `make VERSION=1.30.11+k3s1 ARCH=x86_64`
   - 验证构建成功完成
   - 检查生成的 .opk 文件存在

3. **包结构验证**
   - 解压 .opk 文件
   - 验证包含 control.tar.gz, data.tar.gz, debian-binary
   - 检查 control 文件内容
   - 验证 data.tar.gz 包含所需文件

4. **脚本语法检查**
   - 使用 `sh -n` 检查 init 脚本语法
   - 检查 wrapper 脚本语法

5. **Artifact 保存**
   - 保存生成的 .opk 文件
   - 保存构建日志
   - 保存包结构验证报告

**触发条件：**
- 所有分支的 push 事件
- Pull Request

#### Job 2: release (仅 tags)

**目的：** 发布正式版本到 GitHub Releases

**步骤：**
1. **多架构构建**
   - 执行 `make build-all`
   - 为所有支持的架构构建包

2. **发布到 GitHub**
   - 使用 ghr 工具上传到 GitHub Releases
   - 使用 tag 名称作为 release 版本

**触发条件：**
- Tag 推送（格式：v*.*.* 或数字版本）

### Configuration Parameters

**环境变量：**
- `GITHUB_TOKEN`: GitHub API token（用于 release）
- `DEFAULT_VERSION`: 默认 k3s 版本（1.30.11+k3s1）
- `DEFAULT_ARCH`: 默认架构（x86_64）

**Docker 镜像选择：**
- 推荐：`cimg/base:2024.01` 或 `ubuntu:22.04`
- 原因：包含现代工具链，支持最新的构建需求

### Build Verification Strategy

**自动化验证检查：**

1. **文件存在性检查**
   ```bash
   test -f build/k3s_${VERSION}_${ARCH}.opk
   ```

2. **包结构验证**
   ```bash
   tar -tzf build/k3s_${VERSION}_${ARCH}.opk | grep -q "debian-binary"
   tar -tzf build/k3s_${VERSION}_${ARCH}.opk | grep -q "control.tar.gz"
   tar -tzf build/k3s_${VERSION}_${ARCH}.opk | grep -q "data.tar.gz"
   ```

3. **Control 文件验证**
   ```bash
   # 解压并检查 control 文件
   tar -xzf control.tar.gz
   grep -q "Package: k3s" control
   grep -q "Version: ${VERSION}" control
   grep -q "Architecture: ${ARCH}" control
   ```

4. **Data 文件验证**
   ```bash
   # 检查必需文件存在
   tar -tzf data.tar.gz | grep -q "usr/bin/k3s"
   tar -tzf data.tar.gz | grep -q "usr/bin/k3s-wrapper"
   tar -tzf data.tar.gz | grep -q "etc/init.d/k3s"
   ```

5. **脚本语法验证**
   ```bash
   sh -n files/etc/init.d/k3s
   sh -n files/usr/bin/k3s-wrapper
   ```

### Artifact Management

**分支构建 Artifacts：**
- 路径：`build/*.opk`
- 保留时间：30 天
- 访问方式：CircleCI Artifacts 页面

**Tag 构建 Releases：**
- 位置：GitHub Releases
- 命名：使用 tag 名称
- 包含：所有架构的 .opk 文件

### Error Handling

**构建失败处理：**
1. **下载失败**
   - 检查 k3s 版本是否存在
   - 验证网络连接
   - 提供清晰的错误信息

2. **打包失败**
   - 检查文件权限
   - 验证目录结构
   - 保存中间产物用于调试

3. **验证失败**
   - 输出详细的验证日志
   - 标记失败的具体检查项
   - 阻止 artifact 上传

### Performance Optimization

**缓存策略：**
1. **依赖缓存**
   - 缓存下载的 k3s 二进制文件
   - 使用版本号作为缓存键

2. **构建缓存**
   - 缓存 build 目录（清理后）
   - 加速重复构建

**并行构建：**
- 对于 build-all，可以并行构建不同架构
- 使用 CircleCI 的并行执行功能

### Notification Strategy

**构建状态通知：**
- GitHub Checks API 集成
- PR 中显示构建状态
- 失败时提供详细日志链接

### Testing the CI Configuration

**本地测试：**
```bash
# 使用 CircleCI CLI 本地测试
circleci local execute --job build-and-test
```

**验证清单：**
- [ ] 分支推送触发构建
- [ ] PR 触发构建
- [ ] 构建成功生成 .opk 文件
- [ ] Artifacts 正确保存
- [ ] Tag 推送触发 release
- [ ] Release 包含所有架构
- [ ] 构建失败时正确报错

## Implementation Notes

### OpenWrt 24.10.0 Compatibility Considerations

1. **Kernel Requirements**
   - OpenWrt 24.10.0 默认内核版本：6.6.x
   - 需要启用的内核选项：
     - CONFIG_CGROUPS
     - CONFIG_CGROUP_CPUACCT
     - CONFIG_CGROUP_DEVICE
     - CONFIG_CGROUP_FREEZER
     - CONFIG_CGROUP_SCHED
     - CONFIG_NAMESPACES
     - CONFIG_NET_NS
     - CONFIG_PID_NS
     - CONFIG_IPC_NS
     - CONFIG_UTS_NS
     - CONFIG_VXLAN
     - CONFIG_BRIDGE_NETFILTER

2. **Package Dependencies**
   - 所有依赖包在 OpenWrt 24.10.0 中均可用
   - 无需修改依赖列表

3. **Init System**
   - OpenWrt 24.10.0 继续使用 procd 和 rc.common
   - 现有 init 脚本格式兼容

### k3s 1.30.11+k3s1 Specific Changes

1. **Binary Size**
   - k3s 1.30.x 二进制文件约 60-70MB
   - 确保构建环境有足够磁盘空间

2. **Runtime Requirements**
   - 最低内存要求：512MB（推荐 1GB+）
   - 最低 CPU：1 核心（推荐 2 核心+）

3. **New Features**
   - 支持 Kubernetes 1.30 API
   - 改进的嵌入式 etcd 性能
   - 增强的网络策略支持

### Backward Compatibility

1. **Configuration Migration**
   - UCI 配置格式保持不变
   - 现有配置文件无需修改

2. **Upgrade Path**
   - 可以直接从旧版本升级
   - 建议备份 k3s 数据目录

3. **Rollback Strategy**
   - 保留旧版本包以便回滚
   - 数据目录兼容性需要测试

## Security Considerations

1. **Binary Verification**
   - 当前实现未验证下载的二进制文件签名
   - 建议：添加 SHA256 校验和验证

2. **Network Security**
   - k3s 默认使用 TLS 加密通信
   - 确保防火墙规则正确配置

3. **Privilege Requirements**
   - k3s 需要 root 权限运行
   - cgroup 操作需要特权访问

## Performance Considerations

1. **Build Performance**
   - 下载 60-70MB 二进制文件需要时间
   - 考虑使用本地缓存加速重复构建

2. **Runtime Performance**
   - k3s 在嵌入式设备上资源消耗较高
   - 建议监控内存和 CPU 使用

3. **Storage Requirements**
   - 包文件：约 60-70MB
   - 运行时数据：取决于工作负载（建议预留 1GB+）

## Future Enhancements

1. **Binary Verification**
   - 添加 SHA256 校验和验证
   - 支持 GPG 签名验证

2. **Multi-Version Support**
   - 支持同时安装多个 k3s 版本
   - 版本切换机制

3. **Configuration Management**
   - 提供默认 UCI 配置模板
   - 配置验证工具

4. **Monitoring Integration**
   - 集成 Prometheus metrics
   - 健康检查端点

5. **Automated Testing**
   - CI/CD 集成测试
   - 自动化安装和运行测试
