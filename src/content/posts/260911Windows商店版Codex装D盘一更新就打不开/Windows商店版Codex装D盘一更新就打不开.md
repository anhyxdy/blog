---
title: Windows商店版 Codex 装 D 盘，一更新就打不开
published: 2026-09-11
description: '记录 Windows 商店版 Codex 安装到 D 盘后，更新导致只有后台进程没有窗口时的排查与修复方法。'
image: ''
tags: [Codex, Windows, 踩坑记录]
category: '随想记录'
draft: false
lang: 'zh_CN'
---

# Windows商店版 Codex 装 D 盘，一更新就打不开

今天被 Windows 更新搞崩 Codex，折腾好久，写个笔记存一下。

我习惯把商店软件全塞 D 盘，不想占用 C 盘空间，所以 OpenAI Codex（商店版 ChatGPT 桌面端）也装 D 盘。

之前就发现一个怪事：只要 Windows 更新，或者 Codex 自己更新完，双击图标就完全没反应。

打开任务管理器一看，ChatGPT 进程明明在后台挂着，但是窗口死活弹不出来。

重启、清缓存、修复应用全都试过，好了一次下次更新又复发。

研究半天终于搞懂根因：

MSIX 商店应用挪到 D 盘之后，文件会被系统打上 EFS 加密标记。Codex 启动的时候，会自动解压内置的 `cua_node` 运行库，放到 C 盘用户缓存目录。

但是 Codex 自带的解压工具没办法读取 D 盘带 EFS 加密的源文件，解压到一半中断，生成的 `.staging-*` 文件夹里面文件残缺。

进程能拉起，但是缺少运行库，界面渲染不出来。只要包版本一变，它就会重新执行解压，旧修复直接失效。

## 我的解决办法：PowerShell 脚本手动拷贝文件

思路很简单，放弃 Codex 自带解压，用 `xcopy` 带上 `/G` 参数。这个参数可以解密 EFS 加密文件，直接把安装包里完整的 `cua_node` 复制到那个损坏的 staging 目录，补齐文件。

完整脚本：

```powershell
# 1. 获取 Appx 商店版 Codex 包信息
$p = Get-AppxPackage OpenAI.Codex

# 2. 定位出问题的 cua_node 运行时目录
$r = "$env:LOCALAPPDATA\OpenAI\Codex\runtimes\cua_node"

# 3. 找到最新生成的 .staging-* 临时部署文件夹（损坏的目录就在这里）
$s = Get-ChildItem $r -Directory -Force |
    Where-Object Name -like ".staging-*" |
    Sort-Object LastWriteTime -Descending |
    Select-Object -First 1
$id = $s.Name -replace '^\.staging-([^-]+)-.*$', '$1'
$d = "$r\$id"

# 4. 强制杀掉所有残留 ChatGPT/Codex 后台进程
Get-Process ChatGPT -ErrorAction SilentlyContinue | Stop-Process -Force

# 5. 创建目标目录
New-Item -ItemType Directory -Path $d -Force | Out-Null

# 6. 从 App 安装目录，把完整原版 cua_node 文件强行拷贝到损坏的 staging 目录，补齐残缺文件
xcopy "$($p.InstallLocation)\app\resources\cua_node\*" "$d\" /E /I /H /Y /G
# /E 递归复制子目录，/H 复制隐藏/系统文件，/G 处理 EFS 加密文件（Windows 商店包加密）

# 7. 关闭 Sparkle 自动更新（防止再次触发 runtime 部署，再次弄坏文件）
$env:CODEX_SPARKLE_ENABLED = "false"

# 8. 在 Appx 沙箱容器内启动 Codex（普通 Start-Process 无法正常启动商店 MSIX 包）
Invoke-CommandInDesktopPackage `
    -PackageFamilyName $p.PackageFamilyName `
    -AppId App `
    -Command "$($p.InstallLocation)\app\ChatGPT.exe"
```

![PowerShell 执行修复脚本并成功复制文件](./Snipaste_2026-09-09_11-29-13.png)

## 使用要点

1. 先在任务管理器确认所有 ChatGPT 进程全部关掉。
2. 打开 PowerShell 7，粘贴脚本运行。
3. 看到输出复制了几千个文件，代表拷贝成功，脚本会自动拉起 Codex。

## 脚本改动了哪些地方？

源文件夹是 D 盘里面的 Codex 本体目录，脚本只是读取，不会修改 D 盘任何程序文件。

唯一写入修改的路径：

`C:\Users\hyx\AppData\Local\OpenAI\Codex\runtimes\cua_node\.staging-xxxx`

也就是 C 盘的缓存运行库，几百 MB。

这个缓存文件夹可以手动删掉，删完 Codex 又会尝试自动解压，bug 复现。

## 可选方案对比

### 1. 脚本修复

最快，不用移动大文件。缺点是 Windows 或 Codex 更新之后，bug 可能再次出现，需要重新运行脚本。适合临时救急。

### 2. 应用移动

先把 Codex 挪回 C 盘，打开确认正常，再移回 D 盘。可以重置 EFS 标记，复发概率低。但是移动按钮灰色的时候没法用。

### 3. 直接放在 C 盘

根治这个 EFS 加密 bug，后续更新基本不会炸。代价就是占用 C 盘约 1.78 GB 空间。

## 兜底方案

如果文件拷贝成功，但是依旧只有进程没有窗口，大概率是 Electron GPU 渲染问题，可以尝试下面的命令启动：

```powershell
$p = Get-AppxPackage OpenAI.Codex
Invoke-CommandInDesktopPackage `
    -PackageFamilyName $p.PackageFamilyName `
    -AppId App `
    -Command "$($p.InstallLocation)\app\ChatGPT.exe --disable-gpu --disable-gpu-compositing"
```
