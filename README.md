# hints 的 Arch Linux 软件包

版本：0.1.1-1。PKGBUILD 为单文件配方，下载固定的上游 0.1.1 标签并校验 SHA-256。

## 构建、安装

在本目录以普通用户运行（需要 base-devel）：

```sh
makepkg -si
```

也可以安装随附的已构建包：

```sh
sudo pacman -U hints-0.1.1-1-any.pkg.tar.zst
```

已构建包使用本机 Python 3.14；其他 Python 小版本的系统请从 PKGBUILD 重新构建。

## 启用

包提供用户服务、uinput 模块加载配置以及授予当前本地活动会话权限的 udev 规则。
安装后重新登录或重启最直接。若当前会话需立即使用：

```sh
sudo modprobe uinput
sudo udevadm trigger --subsystem-match=misc --sysname-match=uinput
systemctl --user import-environment DISPLAY WAYLAND_DISPLAY XDG_SESSION_TYPE XDG_CURRENT_DESKTOP
systemctl --user enable --now hintsd.service
hints
```

在桌面设置中将快捷键绑定到 `/usr/bin/hints`，按 Escape 关闭提示。
X11 需要 `libwnck3` 以及合成器（否则透明区域可能显示为黑色）；支持 layer-shell 的 Wayland 合成器需要 `gtk-layer-shell`，截图后端可能需要 `grim`。

GNOME Wayland 还需要上游随附的扩展，可在普通用户会话中安装：

```sh
mkdir -p ~/.local/share/gnome-shell/extensions
ln -s "$(python -c 'from importlib.resources import files; print(files("hints") / "extensions/gnome/hints@realh.co.uk")')" ~/.local/share/gnome-shell/extensions/hints@realh.co.uk
```

注销并重新登录后运行 `gnome-extensions enable hints@realh.co.uk`。扩展能否工作取决于 GNOME 版本及上游支持情况，本次未验证 GNOME Wayland 的完整交互。

某些应用需要另行启用辅助功能。可参考包内 `/usr/share/doc/hints/README.md`。
上游 `sudo ... hints --setup` 会修改全局环境、将用户加入 input 组并询问是否重启；本包已管理服务和 uinput 权限，通常无需为这两项再运行该向导。

## 打包调整

- 移除 setuptools 自定义安装钩子的注册，避免构建时向构建用户的 HOME 写入服务。
- 将 PyGObject 的 `==3.50.0` 限制放宽为 `>=3.50.0`，依赖由 pacman 管理。
- 将用户服务安装到 `/usr/lib/systemd/user`，使用 `/usr/bin/hintsd`，由用户自行启用。
- udev 和模块加载配置均为 pacman 管理的文件，不自动修改用户组或全局环境。
- 保留上游 GNOME 扩展和 KWin 脚本资源。

## 本机测试记录

2026-10-12，Arch Linux x86_64，Python 3.14.8、PyGObject 3.58.1、OpenCV 5.0.0。

通过的检查：

- makepkg 实际构建、源文件校验及 check()。
- pacman 实际安装；`pacman -Qkk hints`：131 个文件，0 个变更。
- `hints --version` 输出 0.1.1；`hints --help` 正常。
- systemd 用户服务文件和 udev 规则验证。
- 当前桌面会话中启动 hintsd，运行 26 秒后仍为 active，随后停止，未设置开机自启。
- Xvfb + Openbox 中导入运行模块、创建真实 uinput 设备、OpenCV 截图及识别（9 个目标）、显示真实提示窗口、已安装 CLI 与守护进程的 socket 通信。

范围与限制：隔离会话的 AT-SPI 注册服务未能激活，本次提示窗口测试使用 OpenCV 回退；未验证 AT-SPI 控件识别、真实鼠标点击/拖动或各 Wayland 合成器。Xvfb 测试未启用合成器，因此截图的透明区域呈黑色。不是干净 chroot 构建。

namcap 的结果保存在 `test-results/namcap.log`：它将明确指定 Gtk/Gdk 3.0 的 GI 导入误判为 gtk4，并将包内类/函数误判为缺失模块；实际模块导入及提示窗口测试已通过。libwnck3 / gtk-layer-shell 为按桌面选择的可选依赖，glib2 由 GTK 依赖满足。

测试脚本、构建日志和截图位于 `test-results/`。

## 补充：Hyprland 当前桌面测试

2026-10-12 已在实际 Hyprland 0.56.2 Wayland 会话验证提示显示、输入提示字母点击测试按钮、重复唤起及 Escape 退出，均通过。已补装 grim，并向 systemd 用户环境导入 HYPRLAND_INSTANCE_SIGNATURE 等当前会话变量。
详细记录和截图见 `test-results/hyprland/`。尚未验证拖动、多屏、缩放；截图存在额外目标和提示偏移，不能据此保证小控件定位精度。
