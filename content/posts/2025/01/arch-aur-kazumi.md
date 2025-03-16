---
title: 为了一碟醋包了一盘饺子：构建和提交Kazumi AUR包
date: 2025-01-24 18:24:26
categories: [技术探索]
tags: [Arch,AUR]
image: https://file.101045700.xyz/2025/01/76ab1468eba28613d3177cea8f63994d.webp
---

最近追番的时候看到了一个名为Kazumi的程序，它是一个基于Flutter开发的，可自定义规则的番剧采集程序，支持流媒体在线观看，同时还支持弹幕和实时超分辨率。

Github仓库：https://github.com/Predidit/Kazumi

那听上去还是很有意思的，我属于看番剧喜欢顺带看点弹幕的那种，就像我明明知道不全，知道会删减，但依然还续着B站大会员也多少跟这有些关系。

总之我还是比较喜欢Kazumi的，它支持的平台也是不少的，iOS，安卓，Win，Linux，甚至新的鸿蒙Next都有支持。

不过我是一个ArchLinux+KDE用户，在使用它时就遇到了一些问题。

首先Kazumi官方提供的Linux安装方式是Deb（Debian）和tar.gz包，或者你也可以通过Flathub安装它。

但是很显然，Arch它不能用Deb安装，而我也不能用Flathub。

Kazumi Flathub的安装方式对我这个KDE桌面用户来说并不友好，居然还需要安装一整套GNOME组件？这不开玩笑嘛，我为了装个程序还得去装隔壁GNOME桌面，这肯定不行。

咱们有请万能的AUR来给我们看看，首先我们看到了也是有前人种树的：kazumi-bin，但是很可惜的是截止本文发布时，此包维护者已经很久没有更新过它了。

![](https://file.101045700.xyz/2025/01/de4c7618811b03e971e5092c703f530f.webp)

还是古早版本，好在Arch安装包是使用PKGBUILD管理的，过程不算复杂，我能很轻松复现它这个过程，更新为最新版本并且安装。

（为此我还去查了Arch Wiki看了半天软件包构建与AUR包提交资料）

我是解决了，但是其他用户呢？也许会有别的Arch用户也想装Kazumi，然后遇到我这个难受的情况。

所以作为老好人（bushi）的我决定提交下AUR包📦来帮助后人。

你现在想要在Arch使用我构建的这个包很简单，有yay的情况下直接输入下面命令然后Enter就行了。

```
yay -S kazumi
```

我为了琢磨这个包看了半天时间资料，毕竟我对Linux基础也不算特别好，这更是我第一次提交AUR包，配置公私密钥和构建花费了我俩小时的时间。

[我构建的AUR包](https://aur.archlinux.org/packages/kazumi)是以Kazumi的tar.gz为基底做的，它的PKGBUILD文件如下

```
# Maintainer: PaloMiku <palomiku@outlook.com>
pkgname=kazumi
pkgver=1.5.3
pkgrel=2
pkgdesc="基于自定义规则的番剧采集APP，支持流媒体在线观看，支持弹幕。"
arch=('x86_64')
url='https://github.com/Predidit/Kazumi'
license=('GPL3')
source_x86_64=("${pkgname}-${pkgver}.tar.gz::https://github.com/Predidit/Kazumi/releases/download/$pkgver/Kazumi_linux_${pkgver}_amd64.tar.gz" "icon.png")
sha256sums_x86_64=('2e06f4fa252a93324b74098ab336874780b62e5058d6a85e6ed97a54e3f0986c' 'SKIP')
depends=('libayatana-appindicator' 'xdg-user-dirs' 'webkit2gtk-4.1')

package() {
    install -d "${pkgdir}/opt/Kazumi" "${pkgdir}/usr/bin"

    bsdtar -xf "$srcdir/${pkgname}-${pkgver}.tar.gz" -C "${pkgdir}/opt/Kazumi"

    ln -s /opt/Kazumi/kazumi "${pkgdir}/usr/bin/kazumi"

    install -Dm644 /dev/stdin "${pkgdir}/usr/share/applications/kazumi.desktop" <<EOF
#!/usr/bin/env xdg-open
[Desktop Entry]
Name=Kazumi
Comment=Watch Animes online with danmaku support.
Comment[zh_CN]=一款好用的追番软件
Exec=kazumi
Icon=io.github.Predidit.Kazumi
Terminal=false
Type=Application
Categories=AudioVideo;Audio;Video;
EOF

    # 安装图标
    install -Dm644 "${srcdir}/icon.png" "${pkgdir}/usr/share/icons/hicolor/128x128/apps/io.github.Predidit.Kazumi.png"
}
```

它的实现方式上半段就是从Kazumi Github仓库下载tar.gz文件然后复制相关依赖文件到指定位置，后半段则是给应用程序列表增加Kazumi应用，这样可以让你不用终端`kazumi`启动它。

给Kazumi Github仓库也提交了一个[Pull requests](https://github.com/Predidit/Kazumi/pull/660)，希望能增加此安装方式，令人高兴的是被合并了。

![](https://file.101045700.xyz/2025/01/cfc871e26ce25d6ed2c5ecb43ba05109.webp)

希望能够帮到更多人。
