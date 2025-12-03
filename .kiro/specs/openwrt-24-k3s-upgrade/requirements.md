# Requirements Document

## Introduction

本文档定义了将现有 k3s-on-openwrt 项目升级以支持 OpenWrt 24.10.0 和 k3s 1.30.11+k3s1 版本（x86_64 架构）的需求。该项目通过 Makefile 从官方 k3s 二进制文件生成 OpenWrt .opkg 包，使 k3s 能够在 OpenWrt 路由器系统上运行。

## Glossary

- **k3s**: 轻量级 Kubernetes 发行版
- **OpenWrt**: 面向嵌入式设备的 Linux 操作系统
- **opkg**: OpenWrt 包管理器使用的包格式
- **Build System**: 基于 Makefile 的构建系统，用于生成 opkg 包
- **Init Script**: OpenWrt 系统初始化脚本，位于 /etc/init.d/
- **Wrapper Script**: k3s-wrapper 脚本，用于启动和管理 k3s 进程
- **Control File**: opkg 包的控制文件，包含包元数据和依赖信息
- **cgroup**: Linux 控制组，k3s 运行所需的内核特性

## Requirements

### Requirement 1

**User Story:** 作为系统维护者，我希望项目支持最新的 k3s 1.30.11+k3s1 版本，以便使用最新的功能和安全更新。

#### Acceptance Criteria

1. WHEN 构建系统读取 VERSIONS 文件 THEN the Build System SHALL 包含 1.30.11+k3s1 作为可用版本
2. WHEN 用户执行 make 命令 THEN the Build System SHALL 默认使用 1.30.11+k3s1 版本
3. WHEN 构建过程下载 k3s 二进制文件 THEN the Build System SHALL 从 k3s-io/k3s GitHub releases 下载正确的 v1.30.11+k3s1 版本
4. WHEN 下载的二进制文件被验证 THEN the Build System SHALL 确认文件可执行且架构匹配 x86_64

### Requirement 2

**User Story:** 作为系统维护者，我希望生成的 opkg 包与 OpenWrt 24.10.0 兼容，以便在最新的 OpenWrt 系统上安装和运行。

#### Acceptance Criteria

1. WHEN 生成 control 文件 THEN the Build System SHALL 声明与 OpenWrt 24.10.0 兼容的依赖包版本
2. WHEN 包被安装到 OpenWrt 24.10.0 系统 THEN the Build System SHALL 确保所有依赖项在该版本中可用
3. WHEN 验证依赖关系 THEN the Build System SHALL 包含 iptables、iptables-mod-extra、kmod-ipt-extra、kmod-br-netfilter 和 ca-certificates
4. WHEN 检查内核模块需求 THEN the Build System SHALL 确保 OpenWrt 24.10.0 内核支持必需的 cgroup、namespace、vxlan 和 CFS 调度器特性

### Requirement 3

**User Story:** 作为系统维护者，我希望 init 脚本能够正确初始化 k3s 运行环境，以便 k3s 服务能够在 OpenWrt 24.10.0 上正常启动。

#### Acceptance Criteria

1. WHEN init 脚本执行 start 命令 THEN the Init Script SHALL 正确挂载 cgroup 文件系统
2. WHEN cgroup 已经挂载为旧格式 THEN the Init Script SHALL 先卸载后重新挂载为 tmpfs 格式
3. WHEN 启动 k3s 服务 THEN the Init Script SHALL 使用 start-stop-daemon 以后台模式启动 k3s-wrapper
4. WHEN 读取配置参数 THEN the Init Script SHALL 从 UCI 配置系统读取 k3s.globals.opts 和 k3s.globals.root
5. WHEN 停止 k3s 服务 THEN the Init Script SHALL 使用 PID 文件正确终止进程

### Requirement 4

**User Story:** 作为系统维护者，我希望 wrapper 脚本能够正确管理 k3s 进程和日志，以便监控和调试 k3s 运行状态。

#### Acceptance Criteria

1. WHEN wrapper 脚本启动 k3s THEN the Wrapper Script SHALL 将所有输出重定向到系统日志
2. WHEN k3s 进程产生输出 THEN the Wrapper Script SHALL 使用 logger 命令标记为 k3s 标签
3. WHEN 接收到终止信号 THEN the Wrapper Script SHALL 正确传递信号给 k3s 子进程
4. WHEN k3s 子进程终止 THEN the Wrapper Script SHALL 等待子进程完全退出

### Requirement 5

**User Story:** 作为开发者，我希望构建系统能够为 x86_64 架构生成正确的包，以便在 x86_64 平台的 OpenWrt 设备上部署。

#### Acceptance Criteria

1. WHEN 指定 ARCH=x86_64 THEN the Build System SHALL 下载 k3s 二进制文件时不添加架构后缀
2. WHEN 生成 control 文件 THEN the Build System SHALL 设置 Architecture 字段为 x86_64
3. WHEN 构建完成 THEN the Build System SHALL 生成名为 k3s_1.30.11+k3s1_x86_64.opk 的包文件
4. WHEN 打包二进制文件 THEN the Build System SHALL 确保 k3s 二进制文件具有可执行权限

### Requirement 6

**User Story:** 作为开发者，我希望能够轻松验证构建结果，以便确认升级后的包能够正常工作。

#### Acceptance Criteria

1. WHEN 执行 make 命令 THEN the Build System SHALL 在 build/ 目录下生成完整的包结构
2. WHEN 检查生成的包 THEN the Build System SHALL 包含 control.tar.gz、data.tar.gz 和 debian-binary 文件
3. WHEN 解压 data.tar.gz THEN the Build System SHALL 包含 /usr/bin/k3s、/usr/bin/k3s-wrapper 和 /etc/init.d/k3s 文件
4. WHEN 验证包元数据 THEN the Build System SHALL 在 control 文件中显示正确的版本号 1.30.11+k3s1

### Requirement 7

**User Story:** 作为系统维护者，我希望保持向后兼容性，以便现有的配置和脚本继续工作。

#### Acceptance Criteria

1. WHEN 升级到新版本 THEN the Build System SHALL 保持现有的文件结构和路径不变
2. WHEN 使用 UCI 配置 THEN the Init Script SHALL 继续支持相同的配置键名
3. WHEN 配置防火墙规则 THEN the Build System SHALL 继续使用 cni0 接口名称
4. WHEN 执行 make clean THEN the Build System SHALL 正确清理所有构建产物

### Requirement 8

**User Story:** 作为开发者，我希望更新文档以反映新的支持版本，以便用户了解当前支持的配置。

#### Acceptance Criteria

1. WHEN 用户查看 README.md THEN the Build System SHALL 显示支持 OpenWrt 24.10.0 和 k3s 1.30.11+k3s1
2. WHEN 用户查看 VERSIONS 文件 THEN the Build System SHALL 在第一行列出 1.30.11+k3s1
3. WHEN 用户查看构建说明 THEN the Build System SHALL 提供清晰的 x86_64 架构构建示例
4. WHEN 用户查看依赖说明 THEN the Build System SHALL 列出 OpenWrt 24.10.0 所需的内核配置要求
