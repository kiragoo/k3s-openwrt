# k3s 容器镜像清单

**生成时间：** 2025-12-10  
**集群节点：** openwrt (192.168.2.2)  
**k3s 版本：** v1.30.11+k3s1  
**集群状态：** 正常运行（所有系统 Pod 处于 Running 状态）

## 快速摘要

- **唯一镜像数量：** 14 个
- **总镜像大小：** ~108.2 MiB
- **当前使用的镜像：** 6 个（CoreDNS、Metrics Server、Local Path Provisioner、Flannel、Pause）
- **主要镜像源：** `registry.aliyuncs.com`（阿里云镜像）
- **网络插件：** Flannel v0.22.3

## 镜像清单

### 1. 核心系统镜像

#### Pause 容器镜像
- **镜像名称：** `rancher/mirrored-pause:3.6`
- **实际使用：** `registry.aliyuncs.com/google_containers/pause:3.6`
- **大小：** 294.7 KiB
- **用途：** Pod 沙箱容器（所有 Pod 的基础容器）
- **平台：** linux/amd64, linux/arm/v7, linux/arm64, linux/ppc64le, linux/s390x, windows/amd64

#### CoreDNS 镜像
- **镜像名称：** `docker.mirrors.ustc.edu.cn/rancher/mirrored-coredns-coredns:1.12.0`
- **备用镜像：** `registry.aliyuncs.com/google_containers/coredns:1.12.0`
- **大小：** 20.0 MiB
- **用途：** Kubernetes DNS 服务
- **平台：** linux/amd64, linux/arm/v7, linux/arm64, linux/ppc64le, linux/riscv64, linux/s390x

#### Metrics Server 镜像
- **镜像名称：** `docker.mirrors.ustc.edu.cn/rancher/mirrored-metrics-server:v0.7.2`
- **备用镜像：** `registry.aliyuncs.com/google_containers/metrics-server:v0.7.2`
- **大小：** 18.6 MiB
- **用途：** Kubernetes 资源指标收集服务
- **平台：** linux/amd64, linux/arm/v7, linux/arm64/v8, linux/ppc64le, linux/s390x

#### Local Path Provisioner 镜像
- **镜像名称：** `docker.mirrors.ustc.edu.cn/rancher/local-path-provisioner:v0.0.31`
- **备用镜像：** `registry.aliyuncs.com/rancher/local-path-provisioner:v0.0.31`
- **大小：** 19.8 MiB
- **用途：** 本地存储动态供应器
- **平台：** linux/amd64, linux/arm/v7, linux/arm64, linux/riscv64

### 2. 网络插件镜像（Flannel）

#### Flannel 主镜像
- **镜像名称：** `docker.io/flannel/flannel:v0.22.3`
- **大小：** 25.8 MiB
- **用途：** Flannel CNI 网络插件主程序
- **平台：** linux/amd64, linux/arm/v6, linux/arm64/v8, linux/mips64le, linux/ppc64le, linux/s390x

#### Flannel CNI 插件镜像
- **镜像名称：** `docker.io/flannel/flannel-cni-plugin:v1.2.0`
- **大小：** 3.7 MiB
- **用途：** Flannel CNI 网络配置插件
- **平台：** linux/amd64, linux/arm/v6, linux/arm64/v8, linux/mips64le, linux/ppc64le, linux/s390x

## 当前 Pod 使用的镜像

| 命名空间 | Pod 名称 | 使用的镜像 | 状态 |
|---------|---------|-----------|------|
| kube-flannel | kube-flannel-ds-fb57p | `docker.io/flannel/flannel:v0.22.3`<br>`docker.io/flannel/flannel-cni-plugin:v1.2.0` | Running |
| kube-system | coredns-66c6684677-8vqn2 | `registry.aliyuncs.com/rancher/mirrored-coredns-coredns:1.12.0` | Running |
| kube-system | local-path-provisioner-6fb74cc587-mqhfh | `registry.aliyuncs.com/rancher/local-path-provisioner:v0.0.31` | Running |
| kube-system | metrics-server-6f79575d7-qzxq8 | `registry.aliyuncs.com/rancher/mirrored-metrics-server:v0.7.2` | Running |

## 镜像仓库配置

### 当前使用的镜像源
- **默认镜像仓库：** `registry.aliyuncs.com`（阿里云镜像）
- **备用镜像仓库：** `docker.mirrors.ustc.edu.cn`（中科大镜像）

### 镜像拉取配置
- **系统默认仓库：** `registry.aliyuncs.com`
- **Pause 镜像：** `registry.aliyuncs.com/google_containers/pause:3.6`
- **代理配置：** HTTP 代理 `192.168.2.1:7890`（仅用于外部镜像拉取）

## 完整镜像列表（去重后）

| 镜像名称 | 大小 | 用途 | 来源 |
|---------|------|------|------|
| `docker.io/flannel/flannel-cni-plugin:v1.2.0` | 3.7 MiB | Flannel CNI 插件 | Docker Hub |
| `docker.io/flannel/flannel:v0.22.3` | 25.8 MiB | Flannel 网络插件 | Docker Hub |
| `docker.io/rancher/mirrored-pause:3.6` | 294.7 KiB | Pause 容器（备用） | Docker Hub |
| `docker.mirrors.ustc.edu.cn/rancher/local-path-provisioner:v0.0.31` | 19.8 MiB | 本地存储供应器（备用） | 中科大镜像 |
| `docker.mirrors.ustc.edu.cn/rancher/mirrored-coredns-coredns:1.12.0` | 20.0 MiB | CoreDNS（备用） | 中科大镜像 |
| `docker.mirrors.ustc.edu.cn/rancher/mirrored-metrics-server:v0.7.2` | 18.6 MiB | Metrics Server（备用） | 中科大镜像 |
| `rancher/mirrored-pause:3.6` | 294.7 KiB | Pause 容器（备用） | Rancher |
| `registry.aliyuncs.com/google_containers/coredns:1.12.0` | 20.0 MiB | CoreDNS（备用） | 阿里云镜像 |
| `registry.aliyuncs.com/google_containers/metrics-server:v0.7.2` | 18.6 MiB | Metrics Server（备用） | 阿里云镜像 |
| `registry.aliyuncs.com/google_containers/pause:3.6` | 294.7 KiB | Pause 容器（当前使用） | 阿里云镜像 |
| `registry.aliyuncs.com/rancher/local-path-provisioner:v0.0.31` | 19.8 MiB | 本地存储供应器（当前使用） | 阿里云镜像 |
| `registry.aliyuncs.com/rancher/mirrored-coredns-coredns:1.12.0` | 20.0 MiB | CoreDNS（当前使用） | 阿里云镜像 |
| `registry.aliyuncs.com/rancher/mirrored-metrics-server:v0.7.2` | 18.6 MiB | Metrics Server（当前使用） | 阿里云镜像 |
| `registry.k8s.io/pause:3.8` | 304.0 KiB | Pause 容器（K8s 官方） | Kubernetes 官方 |

**说明：** 表中标记为"当前使用"的镜像是实际运行中的 Pod 使用的镜像。

## 镜像统计

### 按组件分类

| 组件 | 唯一镜像数 | 单个镜像大小 | 说明 |
|------|----------|------------|------|
| Pause 容器 | 4 个标签 | 294.7-304.0 KiB | 所有 Pod 的基础容器 |
| CoreDNS | 3 个标签 | 20.0 MiB | DNS 服务 |
| Metrics Server | 3 个标签 | 18.6 MiB | 资源指标服务 |
| Local Path Provisioner | 2 个标签 | 19.8 MiB | 本地存储供应器 |
| Flannel 主程序 | 1 个镜像 | 25.8 MiB | 网络插件 |
| Flannel CNI 插件 | 1 个镜像 | 3.7 MiB | CNI 配置插件 |
| **总计** | **14 个唯一镜像** | **~108.2 MiB** | 实际磁盘占用可能更小（层共享） |

### 镜像来源

| 镜像仓库 | 镜像数量 | 说明 |
|---------|---------|------|
| `docker.io` | 4 | Flannel 相关镜像 |
| `registry.aliyuncs.com` | 6 | 阿里云镜像（主要使用） |
| `docker.mirrors.ustc.edu.cn` | 3 | 中科大镜像（备用） |
| `registry.k8s.io` | 2 | Kubernetes 官方镜像 |

## 镜像拉取命令参考

如果需要手动拉取这些镜像，可以使用以下命令：

```bash
# Pause 镜像
k3s ctr images pull registry.aliyuncs.com/google_containers/pause:3.6

# CoreDNS
k3s ctr images pull registry.aliyuncs.com/google_containers/coredns:1.12.0

# Metrics Server
k3s ctr images pull registry.aliyuncs.com/google_containers/metrics-server:v0.7.2

# Local Path Provisioner
k3s ctr images pull registry.aliyuncs.com/rancher/local-path-provisioner:v0.0.31

# Flannel
k3s ctr images pull docker.io/flannel/flannel:v0.22.3
k3s ctr images pull docker.io/flannel/flannel-cni-plugin:v1.2.0
```

## 注意事项

1. **镜像标签重复：** 部分镜像存在多个标签指向同一个镜像（如不同镜像仓库的相同镜像）
2. **SHA256 引用：** 部分镜像通过 SHA256 摘要引用，这些是镜像的不可变引用
3. **平台支持：** 所有镜像都支持 linux/amd64 平台（当前节点架构）
4. **镜像大小：** 实际磁盘占用可能因层共享而小于总大小

## 镜像清理建议

可以安全删除的镜像标签（保留当前使用的镜像即可）：

**当前实际使用的镜像（保留）：**
- `registry.aliyuncs.com/google_containers/pause:3.6` - Pause 容器
- `registry.aliyuncs.com/rancher/mirrored-coredns-coredns:1.12.0` - CoreDNS
- `registry.aliyuncs.com/rancher/mirrored-metrics-server:v0.7.2` - Metrics Server
- `registry.aliyuncs.com/rancher/local-path-provisioner:v0.0.31` - Local Path Provisioner
- `docker.io/flannel/flannel:v0.22.3` - Flannel 主程序
- `docker.io/flannel/flannel-cni-plugin:v1.2.0` - Flannel CNI 插件

**可以删除的重复镜像标签：**
- `docker.io/rancher/mirrored-pause:3.6`
- `rancher/mirrored-pause:3.6`
- `registry.k8s.io/pause:3.8`
- `docker.mirrors.ustc.edu.cn/rancher/mirrored-coredns-coredns:1.12.0`
- `registry.aliyuncs.com/google_containers/coredns:1.12.0`
- `docker.mirrors.ustc.edu.cn/rancher/mirrored-metrics-server:v0.7.2`
- `registry.aliyuncs.com/google_containers/metrics-server:v0.7.2`
- `docker.mirrors.ustc.edu.cn/rancher/local-path-provisioner:v0.0.31`

**清理命令示例：**
```bash
# 删除重复的 pause 镜像
k3s ctr images rm docker.io/rancher/mirrored-pause:3.6
k3s ctr images rm rancher/mirrored-pause:3.6
k3s ctr images rm registry.k8s.io/pause:3.8

# 删除重复的 coredns 镜像
k3s ctr images rm docker.mirrors.ustc.edu.cn/rancher/mirrored-coredns-coredns:1.12.0
k3s ctr images rm registry.aliyuncs.com/google_containers/coredns:1.12.0

# 删除重复的 metrics-server 镜像
k3s ctr images rm docker.mirrors.ustc.edu.cn/rancher/mirrored-metrics-server:v0.7.2
k3s ctr images rm registry.aliyuncs.com/google_containers/metrics-server:v0.7.2

# 删除重复的 local-path-provisioner 镜像
k3s ctr images rm docker.mirrors.ustc.edu.cn/rancher/local-path-provisioner:v0.0.31
```

**注意：** 清理前请确保集群运行正常，建议在非生产环境先测试。

