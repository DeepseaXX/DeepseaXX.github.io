---
title: 关于使用 ImageMagick 转换 pdf tif/tiff 至 png
description: 用 ImageMagick 转换 pdf tif/tiff 至 png
date: 2024-02-08 14:19:00+0900
slug: 20240208-imagemagick
categories:
  - EXP
tags:
  - ImageMagick
  - image
---

反正就是因为某个特殊原因，获得的文件都是用 pdf 或 tiff 格式，且大部分为矢量图。

## 为什么有这样的需求

我的需求也有很多情景，不同的需求。

- 含有多页的 PDF/TIF 可以拆分成单个图片。
  - CubePDF Utility 可以实现这样的需求，但是了解后发现只能通过 UI 操作，没有很好的 CLI 支持，不适用大量转换，每次都要多次操作。
- 对于不支持 tif 和 pdf 贴图（指直接复制粘贴）的 Excel，可以把整张图片直接粘贴入，并在上面添加文本框和标注。
  - PDF 原生支持标记，借助免费的 Sumatra 或者收费的 Acrobat 可以实现，但是考虑到其他协作人也要共同编辑，收费软件产生的花销原因排除，免费软件 Sumatra 平均操作两次就要无响应一次也可以排除。Excel 本身已经是很多公司付费软件的标配，大多数人也都熟悉基本操作，容易上手。
  - 图片上直接编辑只适用于临时使用的情况，情况有变之后需要编辑就很复杂。
  - Microsoft Whiteboard 可能适合我的想法，但是暂时还没成为被微软重视的项目，总感觉会在一个半成品的停留至死，在这种兼容性很差的平台投入工作总感觉非常危险。
- 从矢量图转换为位图，转换出来的文件要尽可能小。
  - 首先源文件为矢量图只是因为特殊需求，且在这个特定需求下矢量图的尺寸相当之小。其实输出是不是矢量图没啥关系，我也希望输出是不怎么占用空间的矢量图格式。但矢量图本身也有各种种类。另外就是，[magick 处理矢量图本身就不被推荐](https://www.imagemagick.org/Usage/formats/#vector)，推荐使用专门为了矢量图设计的工具。上文中也有其他推荐，暂时还没有了解。
  - 相比原图的风格，例如是否是有大片空白面积，是否黑白，是更多硬线条还是色彩丰富的风景画，jpg 和 png 的压缩方式分别适用于不同的场景，文件尺寸也有不同的表现。

## 命令

### tif to png

先上我最常用的，tif 格式转换为 png 格式的指令。

```
convert -density 144 -quality 100 -depth 8 -alpha remove -resize 50% input.tif out.png
```

这是把 tif 转换为 png 的一个最常用的参数。

- `-density 144`代表解像度。
- `-quality 100`代表压缩质量，因为这种单色图片比起压缩质量，更重要的是分辨率上的尺寸压缩，quality 本身对输出的影响很小。
- `-depth 8`就是我们说的 8bit 色深。
- `-alpha remove`指删除所有透明层，我常用到的 tiff 格式的矢量图片是透明背景，不删除透明层直接压缩成位图会变得很奇怪。
- `-resize 50%`尺寸缩放为原尺寸的 50%，因为原图一般为五位数乘四位数像素的大小，在不追求严格的场景下完全没必要，所以 50%甚至 30%都是可以的。

另外不知道 magick 内部是如何处理这些输入参数的，如果调换顺序，很有可能导致参数不起效果。我写的顺序姑且是保证所有参数都可以生效的。

### pdf to png

然后是 pdf 格式转换为 png 格式。

pdf 格式的输入就有很多种类型了，可能是 word 或 excel 文档的导出结果，有可能是海报图片等，也有可能是跟上述 tif 相同的黑白矢量图。所以一种设置无法覆盖所有情况，根据自己需求调整。

```
 convert -density 300 -quality 100 -depth 8 -alpha remove -resize 30% input.pdf out.png)
```

- `-density 300`代表解像度。相比 tif 格式的输入，pdf 内含的“图片尺寸”更小，所以用更高的解像度来保证转换足够保真。
- `-quality 100`同上
- `-depth 8`同上
- `-alpha remove`同上
- `-resize 30%`尺寸缩放为原尺寸的 30%，或自己修改为 50%，或者完全不进行 resize（删掉这一部分），像前文说的，pdf 的输入有各种各样的情况，尺寸也各有区别。

针对 pdf 的各种情景，推荐按照自己最常用的场景分别保存为不同的 bat 文件来一键执行。下一段放一下我自己（指导 ChatGPT）写的易用的批处理文件。

## 批处理

用刚才 pdf to png 的例子来说，在我的电脑里有这样一个结构的 convert（任意名）文件夹，内部保存了 pdf2png.bat（任意名）的批处理文件，该阶层下有 src 和 out 两个文件夹，（与 bat 内的 INPUT_DIR 和 OUTPUT_DIR 保持一致），src 存放输入文件，out 为输出位置。

```
convert/
│  pdf2png.bat
├─src/
│      example.pdf
└─out/
        example.png
```

然后这是 pdf2png.bat 的内容。

```
set INPUT_DIR=src
set OUTPUT_DIR=out
set INPUT_FORMAT=pdf
set OUTPUT_FORMAT=png

for /r "%INPUT_DIR%" %%i in (*.%INPUT_FORMAT%) do (
    convert -density 300 -quality 100 -depth 8 -alpha remove -resize 30%% "%%i" "%OUTPUT_DIR%\%%~ni.%OUTPUT_FORMAT%"
)
```

首先我没有使用`@echo off`，是因为部分情况下处理过程较慢，我希望试试看见现在的进度在哪，并知道他不是卡住了。

开头的四行`set 变量名=目录名`，谜底就在谜面上，分别对应了输入文件夹，输出文件夹，输入格式和输出格式。这样不用仔细去阅读命令内容就可以一眼看到这个 bat 文件大概的目的。

然后对输入文件夹内，后缀名为输入格式的文件进行循环。

循环的指令，便是对每一个指定的输入文件夹内的文件，输出对应格式的文件。除后缀外的其他部分与输入文件名相同。

同样的，复制内容，修改后缀名的变量，就可以分别为各种需求保存不同的 bat 文件了。你也可以修改 bat 文件名为对应的需求，比如`pdf2png.bat`和`pdf2png_hq.bat`就对应了压缩和不压缩图片尺寸的两种情况。当然这种做法有点蠢且粗暴，但我觉得还挺直观。
