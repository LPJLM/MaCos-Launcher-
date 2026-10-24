# MaCos Launcher

一个功能完整的 Minecraft Java 版启动器，参考 **HMCL** 与 **PCL2** 的架构实现。

![MaCos Launcher](docs/poster.png)

![version](https://img.shields.io/badge/version-1.7.1-FF8EC4)
![platform](https://img.shields.io/badge/platform-Windows%2010%2F11-66F0FF)
![license](https://img.shields.io/badge/license-MIT-3EBD5B)
![electron](https://img.shields.io/badge/Electron-30-47848F)

---

## 特性

### 实例管理
- 多实例并存，每个实例独立的 mods / saves / config / 资源包 / 光影
- 集成 PCL2 式版本隔离：`versions/<实例ID>/` 即游戏目录，与 `--gameDir` 完全一致
- 复制、重命名、删除、扫描导入已有版本
- 列表按 **运行中 → 最近游玩 → 最近创建 → 名称** 排序

### 账户
- **离线账户**：Java 标准离线 UUID（MD5 + RFC4122）
- **微软正版**：OAuth 设备码/回环授权，自动续期
- **第三方登录**：authlib-injector（LittleSkin 等 Yggdrasil 皮肤站）
- **从其他启动器导入**：官方启动器、HMCL、PCL2、Prism、MultiMC、BakaXL
- 侧边栏快捷切换，多账户并存

### 加载器
| 加载器 | 版本数 | 说明 |
|---|---|---|
| 原版 | 全部 | Mojang 官方清单 |
| Fabric | 全部 | `meta.fabricmc.net`，`inheritsFrom` 自动合并 |
| Forge | 118 | 安装器优先 BMCLAPI 镜像，失败回退官方 |
| NeoForge | 259 | 元数据与安装器走 BMCLAPI |
| Quilt | 全部 | 后端就绪 |

### 内容下载
- **模组 / 资源包 / 光影 / 数据包 / 整合包** 五类统一搜索（Modrinth API v2）
- 安装时可**选择具体版本**（PCL2 式），不是只能装最新版
- 自动递归安装必需依赖，带依赖环检测
- 整合包：`.mrpack` / CurseForge `.zip` / MultiMC `.zip` / `.minecraft` 目录结构，导入即建实例

### 界面
- 12 套配色主题（8 基础 + 4 二次元），CSS 变量驱动，**实时切换**
- 亚克力毛玻璃 + 独立**透明度滑块**（20%–100%）
- 自定义界面圆角（0–32px）、窗口外圆角
- 二次元模式：角色立绘、樱花飘落、星光、猫耳、开机问候
- 支持替换成自己的角色图片

### 插件系统
- zip 包**拖进窗口即装**，格式与 API 见 [docs/PLUGIN.md](docs/PLUGIN.md)
- `vm` 沙箱运行，拿不到 `require` / `process` / `fs` / `electron`
- 可注册配色主题、品牌信息（左上角）、外链入口，订阅游戏事件
- **实时启用/禁用**，无需重启

### 其他
- 下载支持**暂停/继续**，暂停时进行中的文件会跑完，不留残片
- 日志面板支持**复制 / 导出报告**（含 Java 环境、崩溃报告）
- 自动生成 `options.txt`（按内存调视距）
- 多源下载：Mojang 官方 / BMCLAPI 镜像，支持**自动测速择优**
- 内存安全钳制：`-Xmx` 不超过系统提交额度，避免 JVM 原生内存崩溃

---

## 快速开始

### 直接使用

到 [Releases](../../releases) 下载：

| 文件 | 说明 |
|---|---|
| `MaCos Launcher-x.x.x-x64.exe` | 安装包，可选安装目录、创建快捷方式 |
| `MaCos Launcher-x.x.x-portable.exe` | 免安装单文件，双击即用 |

> 首次启动会自动创建离线账户，**零配置**即可开始玩。

### 从源码构建

```powershell
git clone <repo>
cd maCos-launcher

# 国内网络建议先设镜像
$env:ELECTRON_MIRROR = "https://npmmirror.com/mirrors/electron/"
npm install

npm run dev        # 开发模式
npm run build:win  # 打包
```

产物在 `release/`。详细说明见 [BUILD.md](BUILD.md)。

---

## 目录结构

```
maCos-launcher/
├── electron/              主进程
│   ├── main.ts            窗口 + 全部 IPC
│   ├── preload.ts         contextBridge 暴露的 API
│   ├── auth/              离线 / 微软 / authlib / 其他启动器导入
│   ├── core/              版本解析、下载器、Java 探测、Forge、zip、日志
│   ├── mods/              Modrinth 搜索安装、整合包导入
│   └── plugins/           插件沙箱
├── src/                   渲染进程
│   ├── components/        界面
│   ├── store/             Zustand 状态
│   ├── lib/               主题、颜文字
│   └── types/
├── docs/
│   ├── PLUGIN.md          插件开发文档
│   ├── poster.png         宣传图
│   └── cover-*.png        视频封面
├── examples/              示例插件
├── scripts/afterPack.cjs  打包时写入图标与版本信息
└── build/                 应用图标
```

---

## 数据位置

| 用途 | 路径 |
|---|---|
| 启动器配置与账户 | `%APPDATA%\MaCos Launcher\launcher.json` |
| 实例索引 | `%APPDATA%\MaCos Launcher\instances.json` |
| 插件 | `%APPDATA%\MaCos Launcher\plugins\<id>\` |
| 游戏本体 | 设置里指定的 `.minecraft` 目录 |

> v1.7.1 从 MC Launcher 更名而来，启动时会自动把 `%APPDATA%\MC Launcher` 的内容迁移过来。

---

## 插件

```json
{
  "id": "com.example.neon-theme",
  "name": "赛博霓虹",
  "version": "1.0.0",
  "apiVersion": 1
}
```

```js
api.registerTheme({ id: 'neon', name: '赛博霓虹', primary: '#FF00AA', dark: '#AA0077' })

api.on('game:launch', (p) => {
  const n = (api.storage.get('launches') || 0) + 1
  api.storage.set('launches', n)
  if (n === 10) api.notify('成就达成', '你已经启动游戏 10 次啦！')
})
```

打包成 zip 拖进启动器即可。完整 API 与安全说明见 [docs/PLUGIN.md](docs/PLUGIN.md)，示例见 `examples/`。

---

## 已知限制

- **微软登录**需要自己的 Azure 应用 ID。打包时填入 `electron/auth/microsoft.ts` 的 `BUILTIN_CLIENT_ID`，终端用户即可零配置登录（微软要求每个启动器使用自己的应用 ID）
- Forge 安装器内的 `processors`（1.13+ 二进制补丁）未执行，依赖版本 JSON 已正确写入
- 插件沙箱基于 `vm`，**不是完整安全沙箱**，请只安装可信来源的插件

---

## 技术栈

Electron 30 · React 18 · TypeScript 5 · Vite 5 · TailwindCSS 3 · Zustand · Framer Motion

数据来源：Mojang 官方清单 / Modrinth API v2 / BMCLAPI 镜像 / Fabric Meta

---

## 许可

[MIT](LICENSE) —— 可自由使用、修改、分发、商用。

Minecraft 是 Mojang Studios 的商标，本项目与其无隶属关系。

---
