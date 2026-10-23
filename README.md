# Gonic for LazyCat

将 [Gonic](https://github.com/sentriz/gonic) 打包为 LazyCat LPK v2。Gonic 是一个自托管音乐流媒体服务器，实现了 Subsonic API，可配合 Symfonium、DSUB、Ultrasonic、Amperfy 等客户端使用。

## 镜像来源

- 上游镜像：`docker.io/sentriz/gonic:v0.23.0`（amd64 / arm64 / armv6 / armv7）
- 运行镜像：`docker.1ms.run/sentriz/gonic:v0.23.0`（Docker Hub 默认加速），manifest 内固定 digest
- 容器内监听 `80`

上游每个 release 打 `vX.Y.Z` tag，Action 跟踪该版本族自动更新。

## 音乐库挂载（参考 lark-lzcapp）

媒体通过共享主目录挂载，方式与 [lark](https://github.com/lazycat-contrib/lark-lzcapp) 一致：

```yaml
ext_config:
  enable_media_access: true
  enable_document_access: true
  permissions:
    - PERM_MEDIA_MOUNT_SHARED
```

容器内路径：

| 容器路径 | 懒猫路径 | 内容 |
| --- | --- | --- |
| `/music` | `/lzcapp/run/mnt/home/{{ .U.music_dir }}` | 音乐库（只读） |
| `/podcasts` | `/lzcapp/run/mnt/home/{{ .U.podcast_dir }}` | 播客节目（只读） |
| `/playlists` | `/lzcapp/run/mnt/home/{{ .U.playlists_dir }}` | m3u 播放列表 |
| `/data` | `/lzcapp/var/data` | SQLite 数据库、设置与用户 |
| `/cache` | `/lzcapp/var/cache` | 转码缓存与封面 |

安装向导中的三个目录参数指的是共享主目录（home）下的**文件夹名**，默认分别为 `Music`、`Podcasts`、`Playlists`。`package.yml` 声明了 `document.read` 与 `document.private` 权限以支持挂载。

音乐与播客以只读方式挂载，Gonic 不会修改媒体文件；数据库和缓存放在应用数据目录。

## 扫描策略

```yaml
- GONIC_SCAN_AT_START_ENABLED=true   # 启动时扫描一次
- GONIC_SCAN_WATCHER_ENABLED=true    # 监听文件变化增量更新
- GONIC_SCAN_INTERVAL=10            # 每 10 分钟兜底检查
```

## 账号

上游在数据库迁移里硬编码创建 `admin` / `admin` 账号（`db/migrations.go`），没有提供环境变量来预置密码，因此登录页无法自动填充。首次登录后请在管理界面中修改密码。

## 本地构建

```bash
lzc-cli project build -o dist/application.lpk
lzc-cli lpk info dist/application.lpk
```

## 自动发布

`.github/workflows/lazycat.yml` 使用 `ca-x/lazycat-github-action` 跟踪上游版本 tag，创建版本化 GitHub Release Asset 并只发布到喵喵私有商店。官方商店已关闭。

## 许可

GPL-3.0（上游）