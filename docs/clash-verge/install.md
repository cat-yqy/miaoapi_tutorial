# 安装与首次启动

Clash Verge Rev 只通过 GitHub Release 发布正式版。第三方下载站、网盘“优化版”和捆绑包不要用，避免被替换内核或塞入额外程序。

## 安装前检查

1. 确认系统和架构：
   - Windows 10/11：x64 或 arm64；
   - macOS 12+：Intel 或 Apple Silicon；
   - Linux：x64 或 arm64。
2. 确认旧版本已退出。不要一边运行旧版一边覆盖安装。
3. 如果已有配置，先备份（设置里的 WebDAV 或手动复制配置目录）。
4. 准备订阅链接或本地 YAML 配置。

!!! warning "Windows 7 已不支持"
    当前版本不再支持 Windows 7。请升级系统，或改用仍受支持的 Linux 桌面发行版。

## Windows

### 方式一：GitHub Release

打开 [最新 Release](https://github.com/clash-verge-rev/clash-verge-rev/releases/latest)，根据架构选择：

| 文件 | 用途 |
| --- | --- |
| `Clash.Verge_x.x.x_x64-setup.exe` | 大多数 Intel/AMD 电脑 |
| `Clash.Verge_x.x.x_arm64-setup.exe` | Windows on ARM |
| `..._fixed_webview2-setup.exe` | 系统缺少 WebView2、面板打不开时使用 |

安装时如果 SmartScreen 提示未知发布者，先确认文件来自官方 Release，再选择“更多信息 → 仍要运行”。

### 方式二：WinGet

```powershell
winget install ClashVergeRev.ClashVergeRev
```

### 方式三：Scoop

Scoop 是社区维护分发，适合需要便携/自定义目录的用户：

```powershell
scoop bucket add extras
scoop install extras/clash-verge-rev
```

### WebView2

Clash Verge Rev 的界面依赖 Microsoft Edge WebView2。如果程序能启动但没有界面，或点击托盘没反应：

1. 安装/修复 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)；
2. 或直接使用带 `fixed_webview2` 的安装包。

### 服务模式

首次启动通常会提示安装服务。Windows 会弹 UAC，请允许。

服务模式的作用是让内核以服务子进程运行，从而在不以管理员身份运行整个 GUI 的情况下使用 TUN。安装失败时：

- 检查 360、金山毒霸等安全软件是否拦截；
- 把 Clash Verge Rev 加入白名单；
- 或以管理员身份运行一次，再重试安装服务。

## macOS

### 下载

在 [最新 Release](https://github.com/clash-verge-rev/clash-verge-rev/releases/latest) 选择：

- Apple Silicon（M 系列）：`aarch64.dmg`；
- Intel：`x64.dmg`。

打开 DMG，把 **Clash Verge** 拖进 `/Applications`。

### 首次打开

1. 从“应用程序”启动 Clash Verge；
2. 如果系统提示“无法验证开发者”，到 **系统设置 → 隐私与安全性** 里选择仍要打开；
3. 首次启动会请求安装服务/网络权限，输入管理员密码；
4. 允许通知（可选）。

!!! warning "不要同时运行两个版本"
    旧版还在运行时覆盖安装，可能导致新版本损坏或无法启动。先完全退出旧版，再安装。

### macOS 的 TUN 与 DNS

开启 TUN 时，Clash Verge Rev 会把系统 DNS 临时改为 `223.6.6.6`，以保证 TUN 下的 DNS 被 Mihomo 接管；关闭 TUN 后会恢复原来的 DNS。

这是正常行为。v2.5.8 修复了“关闭 TUN/退出后 DNS 仍停留在旧值”的问题；如果你在旧版本遇到无法解析域名，升级后再试。

## Linux

### Debian / Ubuntu

下载 `.deb` 后：

```bash
sudo apt install -y ./Clash.Verge_x.x.x-_xxx.deb
```

### Fedora / RHEL

```bash
sudo dnf install ./Clash.Verge_x.x.x-_xxx.rpm
# 或
sudo yum localinstall ./Clash.Verge_x.x.x-_xxx.rpm
```

### Arch Linux

```bash
yay -S clash-verge-rev-bin        # 正式版
yay -S clash-verge-rev-autobuild-bin  # 测试版
```

Linux 需要图形桌面环境。无 GUI 的服务器不适合运行 Clash Verge Rev；那种场景应该直接用 Mihomo 内核或其他无头方案。

如果遇到 Wayland/GTK/WebKit 相关问题，先查看[官方 Linux FAQ](https://clash-verge-rev.github.io/faq/linux.html)。

## 首次启动会看到什么

刚安装时通常没有配置：

1. 选择语言和主题；
2. 同意/拒绝服务模式安装（建议安装）；
3. 内核启动，但订阅列表为空；
4. 进入“订阅”页导入配置；
5. 回到“代理”页选择节点；
6. 打开系统代理或 TUN。

如果内核启动失败，先看“日志”页和设置里的内核状态，不要急着重装。

## 更新

### 应用更新

设置里可以检查更新。也可以到 GitHub Release 下载新版本覆盖安装。

Windows 如果应用内更新后仍显示旧版本，通常是安装目录/权限问题；用新版安装包手动覆盖。

### 内核更新

设置里可以切换/更新 Mihomo 内核。日常建议稳定版；Alpha 只用于测试新特性。

### Geo 数据

GeoIP/GeoSite 数据可以自动更新。规则依赖 Geo 数据时，数据过期会导致分流异常。

## 卸载

### Windows

使用安装目录的卸载程序，或在“设置 → 应用”里卸载。若安装了服务模式，先卸载服务，再删除应用。

### macOS

退出应用，把 `/Applications/Clash Verge.app` 移到废纸篓。服务模式/配置文件可能需要单独清理；参考官方卸载页。

### Linux

用包管理器卸载：

```bash
sudo apt remove clash-verge-rev
# 或
sudo dnf remove clash-verge-rev
```

配置目录不会自动删除；确认不需要后再手动清理。

## 安装检查清单

- [ ] 从官方 GitHub Release 下载
- [ ] 选对架构（x64 / arm64）
- [ ] 旧版本已完全退出
- [ ] Windows 已确认 WebView2
- [ ] macOS 已拖入 Applications
- [ ] 服务模式安装成功
- [ ] 内核能正常启动
- [ ] 已导入至少一个配置

## 官方参考

- [下载与安装](https://clash-verge-rev.github.io/install.html)
- [卸载软件](https://clash-verge-rev.github.io/uninstall.html)
- [Windows FAQ](https://clash-verge-rev.github.io/faq/windows.html)
- [macOS FAQ](https://clash-verge-rev.github.io/faq/macos.html)
- [Linux FAQ](https://clash-verge-rev.github.io/faq/linux.html)
