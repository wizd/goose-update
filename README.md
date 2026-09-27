# goose-update.vcorp.ai

只读静态站。Coolify 终止 HTTPS，容器里的 Caddy 只监听 80，没有上传入口。

在 Coolify 里把这个仓库单独建成一个应用，Build Pack 选 Docker Compose，Base Directory 用仓库根目录。

- 域名填 `https://goose-update.vcorp.ai`。容器监听 80，不要在 Compose 里写 `ports:`，否则会绕过 Coolify 的反向代理。
- 存储只保留这一条目录挂载：宿主机 `/data/goose-updates` → 容器 `/srv`，只读。部署前这个目录必须已经存在。
- `Caddyfile` 打进镜像，不要再做文件挂载。Coolify 克隆仓库的位置不在宿主机上；文件不存在时 Docker 会建成目录，容器就起不来。

先把域名解析到这台机器。在宿主机上创建目录和只能写入该目录的系统用户：

```bash
sudo useradd --system --create-home --shell /bin/bash goose-update
sudo mkdir -p /data/goose-updates
sudo chown goose-update:goose-update /data/goose-updates
sudo chmod 755 /data/goose-updates
```

把该用户的 SSH 公钥配好。私钥放进 GitHub Actions Secret `GOOSE_UPDATE_SSH_KEY`，主机地址放进 `GOOSE_UPDATE_HOST`。这两个 Secret 没配时，构建仍会发布 GitHub Release，只是跳过上传。
