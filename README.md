# k3s on OpenWrt
Makefile to generate OpenWrt .opkg packages from official k3s binaries.

## Supported Versions
- **OpenWrt**: 24.10.0 and later
- **k3s**: 1.30.11+k3s1 (default), and other versions listed in VERSIONS file
- **Architectures**: x86_64, arm64, armhf, aarch64_cortex-a72

## Requirements

### Kernel Configuration
This requires a custom kernel with support for various features. For OpenWrt 24.10.0, ensure the following kernel options are enabled:

**Control Groups (cgroups):**
- CONFIG_CGROUPS
- CONFIG_CGROUP_CPUACCT
- CONFIG_CGROUP_DEVICE
- CONFIG_CGROUP_FREEZER
- CONFIG_CGROUP_SCHED

**Namespaces:**
- CONFIG_NAMESPACES
- CONFIG_NET_NS
- CONFIG_PID_NS
- CONFIG_IPC_NS
- CONFIG_UTS_NS

**Networking:**
- CONFIG_VXLAN
- CONFIG_BRIDGE_NETFILTER

**Scheduler:**
- CONFIG_CFS_BANDWIDTH (CFS scheduler)

See here for reference openwrt config: https://github.com/5pi-home/openwrt/blob/master/config

### Runtime Dependencies
The generated package depends on:
- iptables
- iptables-mod-extra
- kmod-ipt-extra
- kmod-br-netfilter
- ca-certificates
- containerd

## Usage

### Firewall
To allow the k3s' flannel bridge to access the internet, configure a interface
for cni0 in uci:

/etc/config/network:
```
config interface 'k8s'
	option proto 'none'
	option ifname 'cni0'
```

/etc/config/firewall
```
config zone
        option name 'k8s'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        option network 'k8s'
```

## Building

### Build Default Version (x86_64)
Run `make` to build the default version (1.30.11+k3s1) for x86_64:
```bash
make
```

### Build Specific Version and Architecture
You can override ARCH and VERSION:
```bash
# Build for x86_64 with specific version
make VERSION=1.30.11+k3s1 ARCH=x86_64

# Build for ARM64
make ARCH=arm64

# Build for ARMhf
make ARCH=armhf
```

### Build All Versions and Architectures
```bash
make build-all
```

See ARCHS and VERSIONS files for available architectures and versions.

### Output
The build process will:
1. Download the k3s binary from GitHub releases
2. Package it with init scripts and wrapper
3. Generate an .opk file in `build/k3s_${VERSION}_${ARCH}.opk`

### Clean Build Artifacts
```bash
make clean
```
