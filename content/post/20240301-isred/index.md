---
title: 如何用python判断他是个红色
description: 遇到了一个需要在Excel表格里，判断背景色是否大概是红色的问题……
slug: 20240301-isred
date: 2024-03-01 11:00:00+0900
categories:
  - EXP
tags:
  - color
  - image
  - python
---

## 背景

在这之前，遇到的是一个处理奇形怪状的 Excel，并把结果以整理后的格式输出的问题。使用 Python 的 openpyxl，和 Python 那个基本不用过脑子的语法，（但强忍着对游标卡尺的恶心）走迷宫一样探索完了那个奇形怪状的表格。到这为止，基本都靠 ChatGPT 和 bing 搜索（别问为什么不用谷歌，凑积分罢了）就能解决问题。

然后就遇到了究极坑点。Excel 表格的上游，是通过单元格的背景颜色填充，来做一个维度的区分的……

而且这个上游部门，经常做一些突然就变了的事情，没有提前通知和商量其他部门的交接，就变了，给别的部门本来的工作方法带来了巨大的冲击。很多对接用的系统，都是已经异动到其他部门的前辈用你不熟悉的语言（VBA）写的，可维护性基本等于零。而且不去主动问，基本不会主动给你提供详细的情报，只能去猜。

## 红色

总之经历过了上述问题，答案最终变成了如何识别红色。

当然标准的红色rgb(255,0,0)已经是一个==就能判断的程度了。问题就是上游部门的任性程度，会不会在未来的某一天，突然决定用深红和浅红来再做进一步区分呢。

rgb值虽然简单易懂，但是红绿蓝三个数字的单纯组合，对我这种色彩感知仅有平均普通人水平的人来说，其实非常难以理解。我曾经想少点处理，直接用R和G、B的差值，R和GB平均值的差值，G和B的差值来粗暴判断，但是！RGB的增减不是线性变化（或者说不够均匀？），想尽可能判断更多细枝末节（比如R值偏低的时候，和GB之间的差值也要变化），表达式越来越迷乱，我在and和or的泥沼中爬向了据说更加现代直观的hsv色彩空间。

## 色彩空间的转换

用现成的库肯定是最好的嘛。我选择了python标准的colorsys。

首先colorsys除了某一个（忘记哪个了，反正我没用到）之外，取值的范围都在[0,1]之间，所以相应的，以0到255为取值范围的rgb，或者写成16进制格式的rgb，或者hsv等h值在0到360之间的，都需要做一步除法，来转换成colorsys所认同的格式。反之转换后的结果，也需要乘法来复原。

其次，通过openpyxl读取来的Excel单元格色彩值，是四位的0到255之间的值。例如rgb(255,0,0)，直接读取出来是“FFFF0000”其中第一位的FF，我也不知道是啥，大概是透明度之类的玩意，总之暂时用不太到。想要用colorsys做下一步处理，就得把这字符串的RGB的部分先取出来，再进行进制转化。


## hsv

哈哈，hsv，色相饱和度明度。确实很直观。本来以为只要判断h的范围就够了，然后发现在明度高，或饱和度低的时候，颜色会趋近于黑灰白，不过这总比rgb简略多了。最后我写的判断式是

```
return (hsv[0] <= 14/360 or 343/360 <= hsv[0]) and (hsv[1] > 0.33) and (hsv[2] > 0.5) and hsv[1]*hsv[2] > 0.36
```

`hsv[0]`，也就是h，在更广泛的领域里是0-360的范围里取值。在Python的colorsys里面，只有[0,1]的范围。那就简单变换。

其他就很简单了，s和v单独不低于阈值，且乘积不低于阈值。基本可以覆盖对于单独一个h值下的情况了。

最后为了直观，把遍历hsv色彩空间之后，根据我的函数判断是否为red的结果，直接输出在Excel里面。有兴趣的可以自己试试看。


```
from openpyxl import Workbook
from openpyxl.styles import PatternFill
from colorsys import rgb_to_hsv, hsv_to_rgb

def decimal_to_hex_string(decimal_value):
    decimal_value %= 256
    hex_value = hex(decimal_value)

    hex_string = str(hex_value)[2:].zfill(2)
    return hex_string


def hex_string_to_decimal(hex_string):
    decimal_value = int(hex_string, 16)
    return decimal_value


def is_rgb_string_red(rgb_string):
    # solid: rgb_string[0:2]
    if len(rgb_string) == 6:
        rgb_string = "ff"+rgb_string
    red = hex_string_to_decimal(rgb_string[2:4])
    green = hex_string_to_decimal(rgb_string[4:6])
    blue = hex_string_to_decimal(rgb_string[6:8])
    hsv = rgb_to_hsv(red/256, green/256, blue/256)
    # print(hsv[0], hsv[1], hsv[2])
    return (hsv[0] <= 14/360 or 343/360 <= hsv[0]) and (hsv[1] > 0.33) and (hsv[2] > 0.5) and hsv[1]*hsv[2] > 0.36

# color判断結果表作成
wb = Workbook()
ws = wb.active

# カウント
i = 0
for h in range(0, 100, 1):
    h = (h+90) % 100
    for s in range(30, 100, 5):
        for v in range(30, 100, 5):
            # 一行の列数
            width = int((100-30)/5)
            # hsvからrgbフォーマットに変換
            # colorsysの範囲は0~1
            rgb = hsv_to_rgb(h/100, s/100, v/100)
            rgb_string = decimal_to_hex_string(int(rgb[0]*256)) + decimal_to_hex_string(
                int(rgb[1]*256))+decimal_to_hex_string(int(rgb[2]*256))

            ws.cell(row=int(i/width)+1, column=i %
                    width+1).fill = PatternFill("solid", fgColor=rgb_string)
            result = is_rgb_string_red(rgb_string)
            if result:
                ws.cell(row=int(i/width)+1, column=i %
                        width+1).value = str(result)
            i += 1


wb.close()
wb.save('info/color.xlsx')
```
