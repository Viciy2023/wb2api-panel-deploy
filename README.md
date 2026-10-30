# wb2api-panel-deploy

自动构建 [misakano7545/workbuddy2api-panel](https://github.com/misakano7545/workbuddy2api-panel) 的 Docker 镜像，供飞牛 NAS 拉取部署。

本仓库**不存源码**，只有 GitHub Actions 工作流和构建指纹 `last-build.txt`。所有源码在构建时从上游 clone。

## 产出镜像

| 通道 | 镜像 |
| --- | --- |
| Docker Hub（飞牛实际拉取） | `<DOCKERHUB_USERNAME>/wb2api-panel:latest` / `:sha-<12位>` |
| GHCR（备用） | `ghcr.io/viciy2023/wb2api-panel-deploy:latest` / `:sha-<12位>` |

- 平台：`linux/amd64`（飞牛 x86_64）
- 上游 SHA 没变时不重复构建（比对 `last-build.txt` 里的 `<sha>v1`）

## 触发方式

- 定时：每 4 小时（UTC 00/04/08/12/16/20）
- 手动：Actions → `Sync & Build wb2api-panel` → Run workflow

## 需要在仓库配置的 Secrets

| Secret | 用途 |
| --- | --- |
| `DOCKERHUB_USERNAME` | Docker Hub 用户名 |
| `DOCKERHUB_TOKEN` | Docker Hub Access Token（Read & Write） |

未配置时工作流照常运行，只推送 GHCR，不会失败。

## FNOS 部署

目录：`/vol1/1000/Docker/wb2api-panel/`

```bash
mkdir -p /vol1/1000/Docker/wb2api-panel/{auths,data}
cd /vol1/1000/Docker/wb2api-panel
cp <上游 config.example.json> config.json   # 设好 api_key
docker compose up -d
```

`docker-compose.yml`：

```yaml
services:
  wb2api-panel:
    image: ccy2026/wb2api-panel:latest
    container_name: wb2api-panel
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
      - WB2A_LISTEN=0.0.0.0:7863
    ports:
      - "7865:7863"
    volumes:
      - ./auths:/app/auths
      - ./data:/app/data
      - ./config.json:/app/config.json
```

容器以 uid 10001 运行，挂载目录需 `chown -R 10001:10001 auths data config.json`。

## 与旧部署的关系

旧容器 `workbuddy2api-panel`（7864，镜像 `ccy2026/workbuddy2api-panel`）来自已停更的
`linguo2625469/workbuddy2api-panel`，**保持原样运行**，本仓库不改动它。

两套容器共用同一批 CodeBuddy 账号会因 token 轮换互相踢下线（12153），
新容器请使用独立账号，或停用旧容器的定时任务后再迁移账号。
