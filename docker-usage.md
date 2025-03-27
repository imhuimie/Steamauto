# Steamauto Docker 使用指南

本文档提供了如何使用 Steamauto 的 Docker 镜像的详细说明，包括从 GitHub Container Registry 拉取镜像、配置和运行容器等内容。

## 目录

- [拉取 Docker 镜像](#拉取-docker-镜像)
- [运行方法一：使用 docker run 命令](#运行方法一使用-docker-run-命令)
- [运行方法二：使用 docker-compose（推荐）](#运行方法二使用-docker-compose推荐)
- [首次运行配置](#首次运行配置)
- [多平台支持说明](#多平台支持说明)
- [常见问题排查](#常见问题排查)

## 拉取 Docker 镜像

从 GitHub Container Registry 拉取最新的 Steamauto 镜像：

```bash
docker pull ghcr.io/用户名/steamauto:latest
```

将"用户名"替换为您的 GitHub 用户名。如果您想使用特定版本，可以将`:latest`替换为具体的版本标签，如`:v1.0.0`。

## 运行方法一：使用 docker run 命令

1. **创建必要的目录**

   ```bash
   mkdir -p config logs session
   ```

2. **运行容器**

   ```bash
   docker run -d \
     --name steamauto \
     -v $(pwd)/config:/app/config \
     -v $(pwd)/logs:/app/logs \
     -v $(pwd)/session:/app/session \
     --restart unless-stopped \
     -e TZ=Asia/Shanghai \
     ghcr.io/用户名/steamauto:latest
   ```

   这个命令会：
   - 以守护进程模式运行容器（`-d`）
   - 将容器命名为 "steamauto"（`--name steamauto`）
   - 挂载本地目录到容器中，用于持久化数据（`-v`）
   - 设置容器在崩溃时自动重启（`--restart unless-stopped`）
   - 设置时区为亚洲/上海（`-e TZ=Asia/Shanghai`）

## 运行方法二：使用 docker-compose（推荐）

1. **创建或修改 docker-compose.yml 文件**

   ```yaml
   version: '3'

   services:
     steamauto:
       image: ghcr.io/用户名/steamauto:latest
       container_name: steamauto
       volumes:
         - ./config:/app/config
         - ./logs:/app/logs
         - ./session:/app/session
       restart: unless-stopped
       environment:
         - TZ=Asia/Shanghai
   ```

   将"用户名"替换为您的 GitHub 用户名。

2. **创建必要的目录**

   ```bash
   mkdir -p config logs session
   ```

3. **启动容器**

   ```bash
   docker-compose up -d
   ```

## 首次运行配置

1. **查看容器日志**

   ```bash
   # 使用 docker 命令
   docker logs -f steamauto
   
   # 或使用 docker-compose
   docker-compose logs -f
   ```

2. **停止容器**

   程序会自动生成配置文件，此时会在`./config`目录下生成配置文件。需要停止容器，编辑配置文件：

   ```bash
   # 使用 docker 命令
   docker stop steamauto
   
   # 或使用 docker-compose
   docker-compose down
   ```

3. **编辑配置文件**

   编辑`./config/config.json5`和`./config/steam_account_info.json5`文件，填入您的 Steam 账号信息和其他配置。

4. **重新启动容器**

   ```bash
   # 使用 docker 命令
   docker start steamauto
   
   # 或使用 docker-compose
   docker-compose up -d
   ```

## 多平台支持说明

Steamauto 的 Docker 镜像支持多种 CPU 架构，包括：

- **linux/amd64**：适用于 Intel 和 AMD 处理器的系统
- **linux/arm64**：适用于 ARM64 架构的系统，如树莓派 4、Apple M 系列芯片的 Mac 电脑

由于我们配置了多平台构建，Docker 会自动选择适合您系统架构的镜像版本，您不需要手动指定架构。

## 常见问题排查

### 无法拉取镜像

如果无法拉取镜像，可能需要先登录到 GitHub Container Registry：

```bash
echo $GITHUB_TOKEN | docker login ghcr.io -u 用户名 --password-stdin
```

将 `$GITHUB_TOKEN` 替换为您的 GitHub 个人访问令牌，将 "用户名" 替换为您的 GitHub 用户名。

### 容器无法连接到 Steam

如果容器无法连接到 Steam，可以在 `config.json5` 中启用代理设置：

```json
"use_proxies": true,
"proxies": {
  "http": "http://host.docker.internal:7890",
  "https": "http://host.docker.internal:7890"
}
```

`host.docker.internal` 是 Docker 提供的特殊 DNS 名称，指向宿主机。如果您的代理运行在宿主机上的 7890 端口，这个配置将允许容器通过宿主机的代理连接到互联网。

### 查看容器状态

```bash
docker ps -a | grep steamauto
```

### 查看容器详细信息

```bash
docker inspect steamauto
```

### 查看容器资源使用情况

```bash
docker stats steamauto
```

### 进入容器内部

如果需要进入容器内部进行调试：

```bash
docker exec -it steamauto /bin/bash
```

### 更新到最新版本

当有新版本发布时，您可以按照以下步骤更新：

```bash
# 拉取最新镜像
docker pull ghcr.io/用户名/steamauto:latest

# 停止并删除旧容器（数据不会丢失，因为它们存储在挂载的卷中）
docker stop steamauto
docker rm steamauto

# 使用新镜像启动容器
docker run -d \
  --name steamauto \
  -v $(pwd)/config:/app/config \
  -v $(pwd)/logs:/app/logs \
  -v $(pwd)/session:/app/session \
  --restart unless-stopped \
  -e TZ=Asia/Shanghai \
  ghcr.io/用户名/steamauto:latest

# 或者，如果使用 docker-compose
docker-compose pull
docker-compose up -d
```

## 数据备份

所有重要数据都通过卷挂载持久化到宿主机。如果您需要备份数据，只需复制 `config`、`logs` 和 `session` 目录即可：

```bash
tar -czvf steamauto-backup.tar.gz config logs session
```

恢复备份：

```bash
tar -xzvf steamauto-backup.tar.gz
```

---

如有任何问题，请参考 [GitHub 仓库](https://github.com/jiajiaxd/Steamauto) 或提交 Issue。