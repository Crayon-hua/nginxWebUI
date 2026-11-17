# Whaee Fork：listen backlog

基于 [cym1102/nginxWebUI](https://github.com/cym1102/nginxWebUI) 的 fork，在反向代理（server）中增加官方 `listen ... backlog=N` 配置。

## 镜像

推送到 master / 打 tag / 手动 `workflow_dispatch` 后构建：

```text
ghcr.io/crayon-hua/nginxwebui:latest
ghcr.io/crayon-hua/nginxwebui:4.4.4
```

用法与原版一致（`--net=host` + 数据卷）：

```bash
docker run -itd \
  -v /home/nginxWebUI:/home/nginxWebUI \
  -e BOOT_OPTIONS="--server.port=8080" \
  --net=host \
  --restart=always \
  --name nginx-webui \
  ghcr.io/crayon-hua/nginxwebui:latest
```

从原版切换时，保留原 `/home/nginxWebUI` 数据目录即可；sqlite 会自动多一列 `backlog`（默认 0）。

## UI 怎么配

反向代理 → 编辑站点 → **listen backlog** 填 `4096` → 保存 → 启用配置 → 校验 / 替换 / 重载。

生成结果示例：

```nginx
listen 443 ssl backlog=4096;
listen 80 backlog=4096;
```

留空或 `0`：不输出 backlog（Linux 仍为 nginx 默认 511）。

同一 `address:port` 只需在某一个 server 上配置一次（nginx 官方约束）。

## 可选：同步推阿里云

在仓库 Settings → Secrets and variables：

| 类型 | 名称 | 示例 |
|------|------|------|
| Variable | `ALIYUN_IMAGE` | `registry.cn-hangzhou.aliyuncs.com/你的命名空间/nginxwebui` |
| Variable | `ALIYUN_REGISTRY` | `registry.cn-hangzhou.aliyuncs.com`（可省略） |
| Secret | `ALIYUN_REGISTRY_USER` | 阿里云镜像仓库用户 |
| Secret | `ALIYUN_REGISTRY_PASSWORD` | 阿里云镜像仓库密码 |

配置后 CI 会把同版本镜像再打到 `ALIYUN_IMAGE:latest` / `:版本号`。

## 相对上游的改动

- `Server.backlog` 字段 + 表单 / i18n
- `ConfService` 生成 http/tcp listen 与 http→https 跳转 listen 时追加 `backlog=N`
- GitHub Actions 多架构镜像发布（amd64/arm64）
