---
title: Emby 怎么设置弹幕插件
description: 自己折腾了一顿自己也忘了，记录一下。
slug: 20250714-emby-danmaku
date: 2025-07-14 22:27:00+0900
categories:
  - EXP
tags:
  - Emby
---

# 写在前面

感谢

- [Shurelol/Emby.CustomCssJS: Easy to manage your Custom JavaScript and Css to modify Emby](https://github.com/Shurelol/Emby.CustomCssJS/tree/main)
- [chen3861229/dd-danmaku: Emby danmaku extension](https://github.com/chen3861229/dd-danmaku)


## 步骤
### 1. 安装 CustomCssJS

   #### 1.1 下载 `CustomCssJS.js` 和 `Emby.CustomCssJS.dll`
   `https://github.com/Shurelol/Emby.CustomCssJS/releases`
   #### 1.2 修改后端（服务端）
   复制`src\Emby.CustomCssJS.dll`到`programdata\plugins`
   #### 1.3 修改桌面客户端
   复制`src\CustomCssJS.js`到`electronapp\plugins`

### 2. 安装 dd-danmaku 到服务端

- 找到`服务端`的 CustomCssJS Provider 插件设置
- 添加`自定义 JavaScript`，把`ede.js`的内容粘贴进去，并设置为开启

`https://github.com/chen3861229/dd-danmaku/blob/main/ede.js`

### 3. 在客户端启动启动

- 在用户首选项的 `Custom Css and JavaScript` 中 `自定义JavaScript` 启动 `ede.js`

### 4. 查看是否正常运行

随便点进一个视频，正常的话右下角会出现对话气泡样式的图标，进行设置。

以下是本人的弹幕样式设置。

```
{
  "danmakuChConvert": 0,
  "danmakuSwitch": true,
  "danmakuFilterLevel": 1,
  "danmakuHeightPercent": 100,
  "danmakuFontSizeRate": 1.4,
  "danmakuFontOpacity": 0.7,
  "danmakuBaseSpeed": 1.1,
  "danmakuTimelineOffset": 0,
  "danmakuFontWeight": 600,
  "danmakuFontStyle": 0,
  "danmakuFontFamily": "sans-serif",
  "danmakuDanmuList": 0,
  "danmakuTypeFilter": [],
  "danmakuSourceFilter": [],
  "danmakuShowSource": [],
  "danmakuAutoFilterCount": 0,
  "danmakuMergeSimilarEnable": false,
  "danmakuMergeSimilarPercent": 100,
  "danmakuMergeSimilarTime": 10,
  "danmakuFilterKeywords": "",
  "danmakuFilterKeywordsEnable": true,
  "danmakuEngine": "canvas",
  "danmakuOsdTitleEnable": true,
  "danmakuOsdLineChartEnable": true,
  "danmakuOsdLineChartSkipFilter": false,
  "danmakuOsdLineChartTime": 10,
  "danmakuOsdHeaderClockEnable": false,
  "danmakuTimeoutCallbackUnit": 1,
  "danmakuTimeoutCallbackValue": 90,
  "danmakuBangumiEnable": false,
  "danmakuBangumiToken": "",
  "danmakuBangumiPostPercent": 95,
  "danmakuConsoleLogEnable": false,
  "danmakuUseFetchPluginXml": false,
  "danmakuDebugShowDanmakuWrapper": false,
  "danmakuDebugShowDanmakuCtrWrapper": false,
  "danmakuDebugReverseDanmu": false,
  "danmakuDebugRandomDanmuColor": false,
  "danmakuDebugForceDanmuWhite": false,
  "danmakuDebugGenerateLarge": false,
  "danmakuDebugDialogHyalinize": false,
  "danmakuDebugDialogWindow": false,
  "danmakuDebugDialogRight": false,
  "danmakuDebugTabIframeEnable": false,
  "danmakuDebugH5VideoAdapterEnable": false,
  "danmakuQuickDebugOn": false,
  "danmakuCustomeCorsProxyUrl": "https://ddplay-api.7o7o.cc/cors/",
  "danmakuCustomeDanmakuUrl": "https://danmaku.7o7o.cc/danmaku.min.js",
  "danmakuCustomeGetCommentUrl": "${dandanplayApi.prefix}/comment/${episodeId}?withRelated=true&chConvert=${chConvert}",
  "danmakuCustomeGetExtcommentUrl": "${dandanplayApi.prefix}/extcomment?url=${encodeURI(url)}",
  "danmakuCustomePosterImgUrl": "https://img.dandanplay.net/anime/${animeId}.jpg"
}
```


## 貌似是已知问题

无法正常识别多季的弹幕。例如药屋少女的呢喃第二季，会被自动识别为"药屋少女的呢喃 2"，然后匹配到第一季的第二集。