---
title: 关于使用 ImageMagick 处理 apng png gif
description: 使用 ImageMagick 进行 apng png gif格式的各种转换和处理
date: 2024-02-07 11:53:00+0900
slug: 20240207-imagemagick
categories:
  - EXP
tags:
  - ImageMagick
---

从某网站上偷来的表情包默认是 apng 格式，想在 QQ 里用的话默认支持非常差，一方面 iPhone 发送 apng 图片会直接变成静态，另一方面即使是支持 apng 的设备，这些图片循环次数并不是无限。所以怎么都需要修改。

很多不知名小公司做的软件，付费或广告，且功能不全，总感觉没法按照自己的想法处理。直接用 ImageMagick 这个开源工具来处理，而且涉及到各种参数微调的时候，个人感觉输入文本命令比 GUI 来得直观可控得多。

本文发布在 qiita/hatenablog。现修改后转载在自己的 blog（并方便存档）。

## 静态 PNG 图片

去除 png 图片的透明背景（转换为白色）

```
magick mogrify -format png -alpha remove *.png
```

注：mogrify 为覆盖（但后缀名不同仍然无法覆盖），convert 为输出新文件

## 动态 PNG 图片转换为 GIF

一键指令

```
 magick.exe convert -format gif -set dispose Previous -layers coalesce -loop 0 apng:origin.png test.gif
```

拆解：

- -formart gif：转换为 gif 格式
- -set dispose Previous 重置 dispose 选项，并设置为“Previoius”模式，详细看这里[ImageMagick （legacy） – 命令行选项](https://legacy.imagemagick.org/script/command-line-options.php?#dispose)
- -layers coalesce 让 gif 变成类似胶片的模式，虽然没看懂但是……加了这个之后突然就好了，说明在这里 [ImageMagick （legacy） – 命令行选项](https://legacy.imagemagick.org/script/command-line-options.php?#layers)
- -loop 0 ：修改循环次数为无限
- apng: ：动态 png 强制以 apng 格式读取，否则将认作静态 png 处理

## 批量处理

本来想用 shell，后来发现想要做的循环和判断几乎可以在一行里解决。
由于忘了当时我用的是 powershell 还是 wsl，后面再补充保存为批处理文件的方法。
那么先上一行流。

```
 ls ./apng |awk -F ".png" '{print $1}' |xargs -I {} magick.exe convert -format gif -set dispose Previous -layers coalesce -loop 0 apng:./apng/{}.png ./gif/{}.gif
```

拆解

- ls ./apng 打印目录下文件
- awk -F ".png" '{print $1}'：以.png 为分隔符切割前一步输出，并打印第一个区块（这里顺便起到了 grep 的作用，但是总感觉可以用 ls 解决……）
- xargs -I {}：将前文输出内容多次使用 例如后面 ./apng/{}.png ./gif/{}.gif，就使用了文件名两次，来分别制定原文件名和输出文件名。

### 想要保存下来一键执行

微软辛辛苦苦开发了 powershell，那当然是想用 powershell 对应的 ps1 来执行了。但是 ps1 默认需要右键执行，分享给小白难免会有“这怎么用”的问题。

于是在不考虑执行效率和优雅程度，同时为了利用 powershell 一些方便的指令，单纯为了强行把他变成大家都熟悉的.bat 格式，可以这么写指令。

```
powershell -command "$a = Get-Clipboard;$a='"'+$a.replace('/','\')+'"'; explorer($a)"
```

引号内部替换为想执行的 powershell 指令即可。

这里的示例是在资源管理器中打开粘贴板中的文件目录。

因为某些奇怪的原因现在用得非常多。

## 参考

[ImageMagick – Command-line Tools: Mogrify](https://imagemagick.org/script/mogrify.php)

[提取与优化 Line 贴图的正确姿势 - 哔哩哔哩](https://www.bilibili.com/read/cv362796)

[Can I use ImageMagick to convert Animated PNG (APNG) into animated WEBP format? · Discussion #3846 · ImageMagick/ImageMagick](https://github.com/ImageMagick/ImageMagick/discussions/3846)

[ImageMagick – Image Formats](https://imagemagick.org/script/formats.php)

[为什么有的 GIF 图片只会播放一遍，而有的会重复播放？关于 gif 你想知道的一切！ - 轻狂书生 han - 博客园](https://www.cnblogs.com/qkshhan/p/16202931.html)

[Animation Basics -- IM v6 Examples](https://legacy.imagemagick.org/Usage/anim_basics/#previous)
