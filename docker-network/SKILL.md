---
name: docker-network
description: 修复 Docker 镜像拉取网络故障。当用户提到 docker pull 超时、拉不到镜像、镜像下载失败、docker 下载很慢、TLS handshake timeout、connection reset by peer、dial tcp i/o timeout、registry 连接失败、构建时拉基础镜像卡住等任何与 Docker 镜像获取相关的网络问题时，必须使用此 skill。即使用户没有贴出具体报错，只是说"docker 拉不下来""镜像拉不动""docker 网不行"，也应主动调用此 skill 给出方案。
---

# Docker 镜像拉取网络修复

## 触发场景

执行 `docker pull` 或构建拉基础镜像时出现：

- `connection reset by peer`
- `TLS handshake timeout`
- `dial tcp: i/o timeout`
- 长时间卡在 `Pulling from ...` 不动

## 方案零：镜像源前缀（一次性，零配置，首选）

不改任何系统配置，直接用镜像源前缀拉：

```bash
docker pull docker.1ms.run/library/alpine
docker pull docker.xuanyuan.me/library/nginx:latest
```

Dockerfile 里同样可用：

```dockerfile
FROM docker.1ms.run/library/python:3.12-slim
```

> 适用场景：一次性、临时、没有 sudo 权限、不想重启 docker。若要长期使用，用方案一持久化。

## 方案一：配置镜像加速（长期，无代理环境）

编辑 `/etc/docker/daemon.json`（不存在则新建）：

```json
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}
```

重启 daemon：

```bash
sudo systemctl restart docker
```

配置后 `docker pull alpine` 会自动走加速源，无需加前缀。

> 镜像源会失效。若上述源不通，联网搜索 "docker mirror 可用" 获取最新源。

## 方案二：给 daemon 配代理（本地有 HTTP 代理，如 clash / v2ray）

`docker pull` 由 daemon 执行，**客户端环境变量无效**（`HTTP_PROXY=... docker pull` 不生效），必须配置 daemon 本身。将 `7890` 改为实际代理端口：

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo tee /etc/systemd/system/docker.service.d/proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890"
Environment="HTTPS_PROXY=http://127.0.0.1:7890"
Environment="NO_PROXY=localhost,127.0.0.1"
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 决策指引

| 场景 | 选择 |
|---|---|
| 临时拉一次、不想动系统配置 | 方案零 |
| 长期使用、国内网络、无代理 | 方案一 |
| 本地有 HTTP 代理（clash/v2ray 等） | 方案二 |

## 验证

```bash
docker pull alpine
docker info | grep -A 5 "Registry Mirrors"
docker info | grep -iE "proxy"
```
