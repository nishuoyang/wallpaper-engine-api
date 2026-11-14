# wallpaper-engine-api

> v0.5.1 · MIT License · Windows
> 独立仓库：<https://github.com/nishuoyang/wallpaper-engine-api>

把本机 Wallpaper Engine 的已安装壁纸与当前桌面壁纸，以**只读** HTTP 接口暴露给 [dsh-wallpaper-bg](https://github.com/nishuoyang/dsh-wallpaper-bg) 的「WE 壁纸库」来源。

这是**独立手动服务**形态：需要本机 Node.js、双击 `.bat` 启动，可选开机自启。如果只想一条命令装好、随 DSH 启停，请改用插件形态的 [dsh-wallpaper-engine-api](https://github.com/nishuoyang/dsh-wallpaper-engine-api)。

## 目录结构

```
wallpaper-engine-api/
├── server.js              # 只读代理服务（HTTP，仅绑定 127.0.0.1:8088）
├── test-bridge.js         # 联调脚本：模拟插件 Host 侧 weApi 调用
├── package.json           # 依赖 wallpaper-engine-api（上游 npm 包）
├── we-api.config          # 本机路径配置（首次运行由向导生成，不入库）
├── 启动服务.bat            # 启动（带 /setup 向导，可重写配置）
├── 启动服务-静默.vbs        # 无窗口启动，供开机自启调用
├── 重启服务(管理员).bat     # 结束后台进程并重新启动
├── 设置开机自启.bat         # 写入 HKCU Run
└── 取消开机自启.bat         # 移除 HKCU Run 与 StartupApproved 记录
```

## 快速开始

```bat
:: 1) 安装依赖（仅首次）
npm install

:: 2) 启动（首次会引导填写 WE 安装目录 / 创意工坊目录）
启动服务.bat

:: 3) 验证
curl http://127.0.0.1:8088/health
```

要求：Windows + 已通过 Steam 安装 Wallpaper Engine + Node.js 20 或更高版本（需在 PATH 中）。

配置写在 `we-api.config`：

```ini
WE_INSTALL_PATH=E:\Steam\steamapps\common\wallpaper_engine
WE_WORKSHOP_PATH=E:\Steam\steamapps\workshop\content\431960
```

## 接口（全部为只读 GET）

| 接口 | 说明 |
| --- | --- |
| `GET /health`、`GET /` | 服务状态，含 `webShim` / `monitorSelect` 能力标记与 `weRunning` |
| `GET /api/wallpapers` | 已安装壁纸列表（别名 `/wallpapers`、`/list`） |
| `GET /api/current[?monitor=]` | 当前桌面壁纸，`monitor` 可指定 `MonitorN`，缺省 `auto` |
| `GET /files/<id>/<rel>` | Web 类壁纸目录内的文件，供插件 iframe 渲染（支持 ETag / Range） |

服务只调用上游 `wallpaper-engine-api` 的 `listWallpapers()` 与 `wallpaper().current()`，不触碰 `load` / `stop` / `openProfile` 等写入接口，不会改动桌面壁纸。

## 开发与验证

```bat
node server.js          :: 前台运行，日志直接打到控制台
node test-bridge.js     :: 联调：核对 /health、壁纸列表与当前壁纸
```

服务默认监听 `127.0.0.1:8088`；端口被占用时先运行 `重启服务(管理员).bat`。

## 相关仓库

- [dsh-wallpaper-bg](https://github.com/nishuoyang/dsh-wallpaper-bg) — DSH 壁纸背景插件（消费方）
- [dsh-wallpaper-engine-api](https://github.com/nishuoyang/dsh-wallpaper-engine-api) — 同能力的 DSH 插件形态（推荐）
- [cabraviva/wallpaper-engine-api](https://github.com/cabraviva/wallpaper-engine-api) — 上游 npm 包，负责与 WE 本体交互

## 许可证

MIT
