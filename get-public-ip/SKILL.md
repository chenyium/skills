---
name: get-public-ip
description: >
  获取本机外网/公网 IP 地址及地理位置信息。当用户提到 public IP、external IP、
  外网 IP、公网 IP、WAN IP、出口 IP，或说"我的 IP 是什么""查一下出口 IP"
  "what's my IP"时使用。也适用于用户排查网络连通性、想确认外部世界看到的 IP 地址
  的场景，即使用户没有明确说"IP"，只要意图是确认出口地址就应触发。
version: 1.0.0
license: MIT
compatibility:
  - cursor
  - claude-code
  - opencode
metadata:
  category: networking
  tags:
    - ip
    - network
    - geolocation
  dependencies:
    - curl
---

# Get Public IP

## How it works

Use `curl` to query ipinfo.io 的 JSON 端点，一次性获取 IP 和地理位置信息：

```bash
curl -s --connect-timeout 5 ipinfo.io
```

返回 JSON 包含：ip、city、region、country、org（ISP）、timezone。

如果 ipinfo.io 不可用，fallback 到纯 IP 服务：

```bash
curl -s --connect-timeout 5 ifconfig.me \
  || curl -s --connect-timeout 5 api.ipify.org \
  || curl -s --connect-timeout 5 icanhazip.com
```

命令需要无限制网络访问（`required_permissions: ["all"]` 或 `["full_network"]`）。

## 输出格式

向用户展示时简洁明了，例如：

```
IP: 47.82.170.171
位置: Shanghai, CN
ISP: Alibaba Cloud
```

不需要展示全部字段，挑重点即可。如果用户需要更多细节再展开。
