---
title: KDE 中 Vscode 和 Cursor 的显示优化
date: 2024-10-30 12:00:00
tags:
    - KDE
    - Arch Linux
    - Vscode
cover: /img/my_vsc.png
---

由于electron的 [历史遗留问题](https://github.com/electron/electron/issues/43721)，KDE 中 Vscode 的窗口无法正常显示，比如没有阴影，双标题栏什么的，非常不好看。所以可以用 hack 一点的方法来解决。

## 配置调整

我们希望使用自定义标题栏搭配系统阴影，所以下面我们隐藏双标题栏中的系统标题栏并且强制显示系统标题栏的阴影。

### Vscode 配置

![my_vsc](/img/my_vsc.png)

#### 调整配置到自定义标题栏

打开 Vscode，按 `Ctrl + ,` 打开设置，在搜索框中输入 `window`，点击 `Edit in settings.json`。

按照如下内容修改配置文件：

```json
{
    "window.titleBarStyle": "custom",
    "window.autoDetectColorScheme": true,
    "window.customTitleBarVisibility": "auto",
    "window.zoomLevel": 0.4,
}
```

这样 Vscode 的标题栏就会变成自定义的，并且有一个系统绘制的阴影。

![vscode_custom_titlebar](/img/vsc_custom_titlebar.png)

下面我们隐藏双标题栏中的系统标题栏并且显示系统标题栏的阴影。

#### 修改窗口装饰

然后打开 KDE 的颜色和主题（就是配置应用外观那个），在 `窗口装饰元素` 中编辑，添加 `特定窗口优先规则` 匹配窗口标题 `(Visual)` 勾选 `隐藏窗口标题栏`，把标题栏隐藏掉。

如果你使用 [Klassy](https://github.com/paulmcauley/klassy) （我用的这个） 可以点开 `Window-Specific Overrides` 添加 `Visual Studio Code` 的规则，勾选 `Hide window titlebar`。Border size 设置为 tiny，然后切换到 Window Tab 修改圆角 `5` 。

#### 配置外观修正

此时你会发现 Vscode 没有阴影，只是没有系统标题栏了。

打开 Vscode，点按 `Alt + F3` 选择 `特殊操作` ，选择 `配置特殊窗口设置`，在外观和修正中添加 `无标题栏和边框`，设置为 `强制` 且 `否`。确定应用。

这样 Vscode 的阴影就正常显示了且保留 Vscode 自己的标题栏。

![vscode_shadow](/img/vsc_shadow.png)

不过要小心的是，如果你的窗口规则匹配到了其他应用，可能会导致其他应用的显示出现问题。

### Cursor 配置

Cursor 虽然脱胎于 Vscode，但是由于 Cursor 适配了全局菜单栏，所以在 Custom 下绘制的界面应用上面的方法会始终显示两层菜单栏并且无法拖动。所以我们只能用 `window.titleBarStyle` 设置为 `native` 来使用系统原生标题栏。

![cursor_native_titlebar](/img/cursor_native_titlebar.png)

在文件，首选项，设置中使用如下配置即可，不需要任何窗口规则或装饰，不然没办法拖动窗口。

```json
"window.customTitleBarVisibility": "never"  // 因为自定义的标题栏是不可拖动的，所以设置为 never
"window.zoomLevel": 0.4  // 缩放比例
"window.titleBarStyle": "native" // 使用系统原生标题栏
```

#### 去掉 Cursor 的双标题栏

利用一些 hack 的小技巧，可以去掉 Cursor 的双标题栏。[参考资料](https://github.com/getcursor/cursor/issues/837#issuecomment-2326443145)。

我从 [https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=cursor-appimage](https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=cursor-appimage) 下载了 PKGBUILD 文件，并做了一些修改。这是 10/30 的最新版本，**如果在你的时间线已经修复了这个问题，请忽略。** 

```shell
# Maintainer: Menghuan1918 <menghuan2003 at outlook dot com>
# Contributor: TimeTrap <zhaoyuanpan at gmail dot com>
# Contributor: Jingu <xiuluo dot android at gmail dot com>
# Contributor: Usama <eruzzamma at gmail dot com>
pkgname=cursor-appimage-fix
_pkgname=cursor
pkgver=0.42.4
pkgrel=1
pkgdesc="Write, edit, and chat about your code with GPT. (AppImage)"
arch=('x86_64')
url="https://cursor.so"
license=('custom')
options=('!strip' '!debug')
depends=('hicolor-icon-theme' 'zlib' 'fuse2')

# Use curl to get the filename and extract the version
# pkgver=$(curl -s -o /dev/null -D - -r 0-0 https://download.cursor.sh/linux/appImage/x64 | grep -o -E 'filename=.*$' | sed -e 's/.*cursor-\(.*\)\(.*\)\.AppImage.*/\1\.\2/')

source=("${_pkgname}-${pkgver}.AppImage::https://downloader.cursor.sh/linux/appImage/x64")
sha256sums=('SKIP')  // 我不知道
_install_path="/opt/appimages"

prepare() {
	chmod a+x "${_pkgname}-${pkgver}.AppImage"
	"./${_pkgname}-${pkgver}.AppImage" --appimage-extract >/dev/null

	# Apply the fix by replacing all occurrences of ",minHeight" with ",frame:false,minHeight"
    local target_file="${srcdir}/squashfs-root/resources/app/out/vs/code/electron-main/main.js"
    sed -i 's/,minHeight/,frame:false,minHeight/g' "$target_file"

	# Modify the original desktop file
	sed 's/AppRun/\/opt\/appimages\/Cursor.AppImage/g' -i "${srcdir}/squashfs-root/cursor.desktop"
	sed 's/Exec=\/opt\/appimages\/Cursor.AppImage/Exec=\/opt\/appimages\/cursor.AppImage/g' -i "${srcdir}/squashfs-root/cursor.desktop"
	sed 's/StartupWMClass=Cursor/StartupWMClass=cursor/g' -i "${srcdir}/squashfs-root/cursor.desktop"

	# Create a copy of the desktop file for Wayland
	cp "${srcdir}/squashfs-root/cursor.desktop" "${srcdir}/squashfs-root/cursor-wayland.desktop"
	sed -i 's/^Name=Cursor/Name=Cursor (Wayland)/' "${srcdir}/squashfs-root/cursor-wayland.desktop"
	sed -i 's|Exec=/opt/appimages/cursor.AppImage|Exec=/opt/appimages/cursor.AppImage --enable-features=UseOzonePlatform --enable-features=WaylandWindowDecorations --ozone-platform=wayland --disable-features=WaylandFractionalScaleV1|' "${srcdir}/squashfs-root/cursor-wayland.desktop"
}

package() {
	install -Dm755 "${srcdir}/${_pkgname}-${pkgver}.AppImage" "${pkgdir}/${_install_path}/${_pkgname}.AppImage"

	# Install icons
	for _icons in 32x32 64x64 128x128 256x256 512x512; do
		install -Dm645 "${srcdir}/squashfs-root/usr/share/icons/hicolor/${_icons}/apps/cursor.png" "${pkgdir}/usr/share/icons/hicolor/${_icons}/apps/cursor.png"
	done

	# Install the desktop files
	install -Dm755 "${srcdir}/squashfs-root/cursor.desktop" "${pkgdir}/usr/share/applications/${_pkgname}.desktop"
	install -Dm755 "${srcdir}/squashfs-root/cursor-wayland.desktop" "${pkgdir}/usr/share/applications/${_pkgname}-wayland.desktop"
}
```

使用 `makepkg -si` 安装即可。


#### Cursor Wayland 下字体渲染模糊

为了在 Wayland 下获得更好的显示效果，需要修改 Cursor 的启动配置。

1. 首先从系统复制默认的桌面配置文件：
```bash
cp /usr/share/applications/cursor-wayland.desktop ~/.local/share/applications/
```

2. 编辑复制后的配置文件，添加必要的 Wayland 支持参数：
```shell
[Desktop Entry]
Name=Cursor (Wayland)
Exec=/opt/appimages/cursor.AppImage --enable-features=UseOzonePlatform,WaylandWindowDecorations --ozone-platform=wayland --disable-features=WaylandFractionalScaleV1 --enable-wayland-ime --no-sandbox %U
Type=Application
Terminal=false
Categories=Development;IDE;
```

这些参数的作用：
- `UseOzonePlatform` 和 `WaylandWindowDecorations`：启用 Wayland 原生窗口装饰
- `ozone-platform=wayland`：使用 Wayland 显示后端
- `disable-features=WaylandFractionalScaleV1`：修复高分屏缩放问题
- `enable-wayland-ime`：启用 Wayland 输入法支持

## 参考资料

- [Visual Studio Code has no window borders and shadow on Wayland](https://www.reddit.com/r/gnome/comments/1fnison/visual_studio_code_has_no_window_borders_and/)

- [Fix Wayland blurry fonts](https://github.com/microsoft/vscode/issues/203303)
