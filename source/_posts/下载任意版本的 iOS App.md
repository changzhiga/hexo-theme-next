---
title: 下载任意版本的 iOS App
date: 2018-08-14 02:15:37
tags: iOS
categories: Tech
description: 在 iTunes 上下载任意版本的 iOS App
---

> 工具：iTunes 12 、 Fiddler for .NET4 、Charles

# 1. 获取 XML

* 打开 Fiddler，选择菜单栏 > Tools > Fiddler Options，在 HTTPS 选项卡中勾选 Decrypt HTTPS traffic，弹出窗口点 Yes，新弹出安装证书窗口选择“是”。

    需要注意的是，整个过程都不要关闭 Fiddler，如果出现安装证书失败或打开 iTunes 无法加载页面的情况，可以参考下面的解决办法：

    > Fiddler 菜单栏 > Tools > Fiddler Options > HTTPS 选项卡；
    > 选择下方的 Export Root Certificate to Desktop；
    > 桌面上会出现一个 FiddlerRoot.cer 文件，右键安装证书；
    > 安装证书的位置选择第二个，并点击“浏览”，选择“信任的根证书存储”。

* 打开 iTunes（如之前已打开，需要关闭 iTunes 后重新打开），搜索想下载的 App（以下载 QQ 5.9.1 版为例）。

* 点击下载，等右上角出现箭头后删除下载（选中下载按两次 Delete 键）。

* 返回 Fiddler 将还在下载的项目删除。

* 在该删除的下载项上方找到域名为 `p32-buy.itunes.apple.com`、url开头为 `/WebObjects/MZBuy.woa` 的请求，切换右侧至 `Inspectors` 选项卡，并点击中间的黄色块 `Response is encoded and may require decoding before inspection. Click here to transform.`

* 保存该请求 `右键请求 - Save - Response - Response Body`。需要注意一点，如果没有点击黄色方块，将会保存一个乱码文件。

* 打开保存的 .xml 文件（系统默认一般是 IE 打开），向下翻动找到 `softwareVersiIxternalIdentifiers` 并伴随着一大串 `xxxxxxxxxx` 的项目。此处为该 App 自第一个版本起每个版本在 App Store 中的版本 ID，从后向前即为最新到最老。

## 2. 下载旧版本 App（以微信为例）

* 找到 App 的目标版本 ID，复制备用。

* iTunes 中如果已经存在App的安装包要将其删除。

* 点击右上角搜索框，点 `iTuens Store` 选项卡，输入`微信`，并搜索。

* 搜索完成后，运行 Charles，然后切换回 iTunes，点击微信下方的下载按钮，微信开始下载。 注意：微信有下载进度时，立刻选中该项目，按键盘上的 Delete 键2次，删除当前下载。

* 打开 Charles 窗口，在 `Sequence` 选项卡，`Host` 一栏，可以找到类似 `p数字-buy.itunes.apple.com` 的字段，点击右键，勾选 `Enable SSL Proxyiŋ` 和 `Breakpoints` 两个选项。

* 切换回 iTunes 窗口，发现微信的状态仍然处于『正在下载』。不要紧，重新搜索『微信』。

* 重新搜索后，微信状态变为可下载。点击『下载』按钮，这时 Charles 会主动跳出，询问编辑请求。然后需要在 Charles 界面点击 `Edit Request - XML Text`，将 `appExtVrsId` 字段的键值改为`816092740`（目标版本ID）,这个键值对应的就是微信 6.3.10 版本的 BuildID。最后选择 `Excute` 按钮。

* Charles 会再次弹出收到请求的包，不用管，直接点击 `Excute` 按钮，iTunes 会开始下载旧版本的微信。 注意：iTunes 下载旧版本过程中请勿关闭Charles。

* 等待下载结束，关闭 Charles，在 iTunes 资料库即可看到旧版本的微信。备份重要聊天记录后，卸载 iOS 上的新版微信，再使用 iTunes 重新安装旧版即可。

* 安装完成后，打开 iOS 设备的 `设置 - iTunes Store 与 App Store`，关闭`更新`选项卡，以防止微信自动更新。直到微信发布新版本修复推送。

    **注意：不要试图共享微信安装包，每个 iTunes Store 下载的 App 均有签名，他人无法安装。篡改 installd 除外。**