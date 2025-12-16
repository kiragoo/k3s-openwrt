# k3s containerd 配置模板保障指南

## 问题说明

k3s 在每次启动时（`/etc/init.d/k3s start`）会从模板文件 `/opt/k3s/data/agent/etc/containerd/config.toml.tmpl` 重新生成 containerd 的配置文件 `/opt/k3s/data/agent/etc/containerd/config.toml`。

如果直接修改生成的 `config.toml`，修改内容在下次启动时会被覆盖。

## 配置方式

### 1. 镜像仓库配置（通过模板文件）

k3s 使用模板文件 `/opt/k3s/data/agent/etc/containerd/config.toml.tmpl` 来配置镜像仓库的 mirrors。

**模板文件位置：** `/opt/k3s/data/agent/etc/containerd/config.toml.tmpl`

**当前配置（使用阿里云镜像）：**
```toml
version = 2

[plugins."io.containerd.grpc.v1.cri".registry.mirrors]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
    endpoint = ["https://registry.aliyuncs.com"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry-1.docker.io"]
    endpoint = ["https://registry.aliyuncs.com"]
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry.aliyuncs.com"]
    endpoint = ["https://registry.aliyuncs.com"]
```

### 2. Pause 镜像配置（通过 k3s 启动参数）

Pause 镜像通过 k3s 的 `--pause-image` 参数设置，在 UCI 配置中：

```bash
k3s.globals.opts='--config /opt/k3s/conf/config.yaml --system-default-registry registry.aliyuncs.com --pause-image registry.aliyuncs.com/google_containers/pause:3.6'
```

### 3. 系统默认镜像仓库（通过 k3s 启动参数）

通过 `--system-default-registry` 参数设置，在 UCI 配置中：

```bash
k3s.globals.opts='--system-default-registry registry.aliyuncs.com'
```

## 配置保障方案

### 方案 1：使用 registries.yaml（k3s 官方推荐）

k3s 支持通过 `/etc/rancher/k3s/registries.yaml` 配置文件来设置镜像仓库，这是 k3s 官方推荐的方式。

1. **创建配置文件：**
   ```bash
   mkdir -p /etc/rancher/k3s
   cat > /etc/rancher/k3s/registries.yaml << 'EOF'
   mirrors:
     docker.io:
       endpoint:
         - "https://registry.aliyuncs.com"
     registry-1.docker.io:
       endpoint:
         - "https://registry.aliyuncs.com"
     registry.aliyuncs.com:
       endpoint:
         - "https://registry.aliyuncs.com"
   EOF
   ```

2. **重启 k3s 服务：**
   ```bash
   /etc/init.d/k3s restart
   ```

3. **验证配置：**
   ```bash
   cat /opt/k3s/data/agent/etc/containerd/config.toml
   ```

**优点：**
- k3s 官方推荐方式
- 配置格式清晰
- 自动转换为 containerd 配置

### 方案 2：修改模板文件（备选方案）

1. **编辑模板文件：**
   ```bash
   vi /opt/k3s/data/agent/etc/containerd/config.toml.tmpl
   ```

2. **添加或修改镜像仓库配置：**
   ```toml
   version = 2

   [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
       endpoint = ["https://registry.aliyuncs.com"]
     [plugins."io.containerd.grpc.v1.cri".registry.mirrors."registry-1.docker.io"]
       endpoint = ["https://registry.aliyuncs.com"]
   ```

3. **重启 k3s 服务：**
   ```bash
   /etc/init.d/k3s restart
   ```

4. **验证配置：**
   ```bash
   cat /opt/k3s/data/agent/etc/containerd/config.toml
   ```

### 方案 2：通过 UCI 配置（推荐用于 pause 镜像和默认仓库）

1. **编辑 UCI 配置：**
   ```bash
   uci set k3s.globals.opts='--config /opt/k3s/conf/config.yaml --system-default-registry registry.aliyuncs.com --pause-image registry.aliyuncs.com/google_containers/pause:3.6'
   uci commit k3s
   ```

2. **重启 k3s 服务：**
   ```bash
   /etc/init.d/k3s restart
   ```

### 方案 3：通过 k3s config.yaml（推荐用于系统级配置）

编辑 `/opt/k3s/conf/config.yaml`：
```yaml
data-dir: /opt/k3s/data
disable:
  - traefik
  - servicelb
node-ip: 192.168.2.2
system-default-registry: registry.aliyuncs.com
```

## 当前远端配置总结

### 模板文件配置
- **文件路径：** `/opt/k3s/data/agent/etc/containerd/config.toml.tmpl`
- **配置内容：** 镜像仓库 mirrors 指向 `registry.aliyuncs.com`

### UCI 配置
- **Pause 镜像：** `registry.aliyuncs.com/google_containers/pause:3.6`
- **系统默认仓库：** `registry.aliyuncs.com`
- **数据目录：** `/opt/k3s/data`

### k3s config.yaml
- **系统默认仓库：** `registry.aliyuncs.com`
- **数据目录：** `/opt/k3s/data`
- **节点 IP：** `192.168.2.2`

## 验证配置是否生效

### 1. 检查生成的 config.toml
```bash
cat /opt/k3s/data/agent/etc/containerd/config.toml
```

### 2. 检查镜像仓库配置
```bash
grep -A 10 'registry.mirrors' /opt/k3s/data/agent/etc/containerd/config.toml
```

### 3. 检查 pause 镜像
```bash
/usr/bin/k3s ctr images ls | grep pause
```

### 4. 检查集群状态
```bash
/usr/bin/k3s kubectl get pods -A
```

## 注意事项

1. **模板文件优先级：** k3s 会合并模板文件中的配置到生成的 `config.toml`，但主要使用模板中的 `registry.mirrors` 部分。

2. **其他配置：** containerd 的其他配置（如 sandbox_image、runtimes 等）由 k3s 自动生成，通常不需要在模板中配置。

3. **配置持久化：** 
   - 模板文件 `/opt/k3s/data/agent/etc/containerd/config.toml.tmpl` 需要手动保护，不会被 k3s 覆盖
   - UCI 配置 `/etc/config/k3s` 需要手动保护
   - k3s config.yaml `/opt/k3s/conf/config.yaml` 需要手动保护

4. **配置备份：** 建议在修改配置前备份：
   ```bash
   cp /opt/k3s/data/agent/etc/containerd/config.toml.tmpl /opt/k3s/data/agent/etc/containerd/config.toml.tmpl.bak
   cp /etc/config/k3s /etc/config/k3s.bak
   cp /opt/k3s/conf/config.yaml /opt/k3s/conf/config.yaml.bak
   ```

## 故障排查

如果配置不生效：

1. **检查模板文件是否存在：**
   ```bash
   ls -l /opt/k3s/data/agent/etc/containerd/config.toml.tmpl
   ```

2. **检查文件权限：**
   ```bash
   ls -l /opt/k3s/data/agent/etc/containerd/
   ```

3. **查看 k3s 启动日志：**
   ```bash
   logread | grep k3s | tail -50
   ```

4. **检查生成的配置文件：**
   ```bash
   cat /opt/k3s/data/agent/etc/containerd/config.toml
   ```

5. **验证配置格式：**
   ```bash
   # 检查 TOML 格式是否正确
   cat /opt/k3s/data/agent/etc/containerd/config.toml.tmpl
   ```

