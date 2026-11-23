# 作家助手 · Arch Linux 打包

`authorwrite-bin` 将阅文官方 Linux 客户端重新打包为 Arch Linux 软件包，支持 `x86_64` 和 `aarch64`。这是第三方打包项目。客户端和账号服务由阅文提供。[作家助手官网](https://zuojia.write.qq.com/)

## 安装

### 使用预构建包

运行 `uname -m` 查看架构，从 [Releases](https://github.com/HeShian/authorwite/releases/latest) 下载后缀与架构匹配的 `.pkg.tar.zst`。在下载目录执行以下命令。ARM64 用户将文件名里的 `x86_64` 改为 `aarch64`。

```bash
sudo pacman -U ./authorwrite-bin-5.24.0-4-x86_64.pkg.tar.zst
```

将同一 Release 的 `SHA256SUMS` 下载到同一目录，再运行以下命令验证下载完整性。

```bash
sha256sum --ignore-missing -c SHA256SUMS
```

### 从打包文件构建

```bash
sudo pacman -S --needed base-devel git
git clone https://github.com/HeShian/authorwite.git
cd authorwite
makepkg -si
```

`makepkg` 通过阅文的下载接口获取当前架构的官方 `.deb`，并校验 `PKGBUILD` 中固定的 SHA-256。官网换版后，如果接口开始返回不同版本，构建会提示版本不匹配。此时可以使用对应 Release 的预构建包，或将校验值匹配的原版 `.deb` 放在 `PKGBUILD` 同一目录后重新构建。

## 使用

从应用菜单选择「作家助手」，或在终端运行 `authorwrite`。Waybar 托盘菜单提供「新建便签」「打开作家助手」和「退出」。卸载软件包可以运行 `sudo pacman -R authorwrite-bin`。

## Linux 兼容性

- 通过桌面入口显示窗口。在 niri 上再次启动时，会将已有窗口切到前台。
- 向支持 StatusNotifierItem 的托盘注册菜单。
- 绕过客户端的 HTTP 缓存，避免登录页一直显示「加载中」。
- 修正内嵌登录页面在 Linux 上的渲染进程崩溃，避免登录区域显示灰块。

`5.24.0-4` 已在 x86_64 的 niri、Fuzzel 和 Waybar 会话验证应用启动、登录表单显示与托盘菜单。ARM64 安装包已构建并检查，尚未在 ARM 设备上运行验证。

## 许可证

打包文件使用 [0BSD 许可证](LICENSE)。作家助手客户端为阅文的专有软件，适用上游条款。
