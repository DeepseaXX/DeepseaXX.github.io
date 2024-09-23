---
title: 在 ESXi 上装 Windows11
description:
slug: 20240814-esxi-windows
date: 2024-08-14 12:00:00+0900
categories:
  - EXP
tags:
  - ESXi
  - Windows 11
---

由于用 Debian 怎么都折腾不好 LLOnebot 和 Koishi 的联动，转投 Win11。
意外地感觉自己肯定记不住。

## 镜像下载

官网或者[UUP dump](https://uupdump.net/)下载 ISO 镜像。后者可以定制，但是需要自己电脑跑挺久的。我自己勾选了运行组件清理、集成.NET3.5 和固实压缩，总共花费了 30 分钟左右。

创建完成后在脚本同目录出现了.iso 文件，把他上载进 ESXi 的数据存储中。

## 创建新的虚拟机

选择了 ESXi 8.0 版本，分配了 4 核心，内存 8GB，硬盘 128GB。（需要注意[ Windows 11 的最低要求](https://support.microsoft.com/zh-cn/windows/windows-11%E7%B3%BB%E7%BB%9F%E8%A6%81%E6%B1%82-86c11283-ea52-4782-9efd-7674389a7ba3)，至少后期可以修改。）

需要加载 CD/DVD 驱动器，选择刚才上载的镜像文件。

然后打开电源，打开控制台进行操作就可以了。

## 绕过 TPM 检测

装到一半提示“这台电脑无法运行 Windows 11”，然后才想起来有 TPM2.0 这档子事。在 ESXi 环境中有两种方法，使用虚拟的 vTPM，或者绕过检测。
参考（照抄）了这篇[文章](https://www.dinghui.org/vmware-esxi-windows-11.html)。

省流版：在出现“现在安装”的界面，按下 `Shift+F10` 呼出命令行，输入

`REG ADD HKLM\SYSTEM\Setup\LabConfig /v BypassTPMCheck /t REG_DWORD /d 1`

ESXi 网页控制台默认不开启复制粘贴功能，需要参考[这篇文章](https://blog.exsvc.cn/article/esxi-enable-clipboard-copy-paste.html)。
以及右侧 Shift 没用，需要注意。

## Windows11 初始化

正常流程，点点点就行了。

## 后记

想到再补充。
