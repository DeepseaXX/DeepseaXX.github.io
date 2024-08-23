---
title: 在Bookmarklet中实现复制时
description: 纯总结
slug: 20240823-bookmarklet-copy
date: 2024-08-56 16:30:00+0900
categories:
  - EXP
tags:
  - Javascript
  - Bookmarklet
---

以前我抄了一段代码是这样的 ↓

因为是Bookmarklet的要求，代码要写在一行里。懒得改了就凑合看看吧。


```
javascript: (function () { let path = "C:/BoxDrive/Box/"; const dotButton = document.querySelectorAll( ".ItemListBreadcrumb > button")[0]; if (dotButton) { dotButton.click(); path += [ ...document.querySelectorAll( "a[data-resin-target='openfolder'].menu-item"), ] .map((e) => e.innerText) .filter( (v) => v !== decodeURI( "すべてのファイル")) .join("/"); if (!path.endsWith("/")) path += "/"; } path += [...document.querySelectorAll(".ItemListBreadcrumb-listItem")] .map((v) => v.innerText) .filter( (v) => v !== "" && v !== decodeURI( "すべてのファイル")) .join("/"); !(function (a) { var b = document.createElement("textarea"), c = document.getSelection(); (b.textContent = a), document.body.appendChild(b), c.removeAllRanges(), b.select(), document.execCommand("copy"), c.removeAllRanges(), document.body.removeChild(b); })(path); document.body.click(); })();
```

前面的部分跟这次的主题无关，反正就是从网页的某个元素里获得每一个内容物，并依次添加到字符串中，再继续从另一个元素中获得每个内容添加到字符串里，缝合成一个可以本地访问的绝对路径。

然后下一步，把这个绝对路径复制到剪贴板，这段代码的实现方式是`function (a)`内部，在网页上创建一个元素，通过`document.execCommand("copy")`复制这个元素的内容，再删掉这个元素。

这个方法在我和大部分人的电脑上还是可行的，直到有人说，在他的电脑上 Edge 执行不了，Chrome 可以执行。

我开始疑惑，不知道为什么直觉告诉我可能是复制这部分实现的问题。

毕竟这个实现方法虽然可用，但是还是有点扭曲，虽然扭曲，但因为可以用就没管它。

然后我注意到 VSCode 插件告诉我这个写法现在不推荐了，那推荐的是什么呢，查了一下↓。

```
navigator.clipboard.writeText(path);
```

一行就行，不需要创建一个临时的元素曲线救国了。

然后我就立刻替换进去，让那个哥试试。果然解决了。

嘻嘻。