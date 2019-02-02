---
showDate: true
date: "2016-02-20"
layout: post
tags: ['Tool', 'Terminal', 'Mac']
title: The new iterm2 is coming
---

Iterm2，Mac 下最好用的终端 App，今天提醒有新 beta 版本，于是下载体验了把。下面是体验感受：

### 一些变化

<!--more-->

1. 自从 Yosemite 后，OS X 也被拍扁了。iterm2 也顺应了潮流(似乎只有 tab bar 变平了)。

2. shell 的集成。  
这个特性很强大，包含了众多的功能点，包括：长时间任务执行完成后通知，一键通过 scp 上传文件到远程服务器上，或下载文件，最近输入的命令及目录自动切换 profile 等。要求所有 host 都必须安装 shell 集成。具体细节可以访问：[Shell Integration](https://iterm2.com/shell_integration.html)。

3. session 恢复
再也不用担心强制退出，崩溃或不小心关闭终端或 tab 了，取代了 tmux 的部分功能。

4. imgcat
用这个脚本可以在终端里看图了。
+ 首先，下载 [imgcat](https://iterm2.com/imgcat)，放到你的 ~/bin 目录下(确保 `bin` 目录已添加到 $PATH)。
+ `imgcat [filepath]` 便可看图了。

![imgcat](/images/imgcat.png)

5. tab 关闭后可以撤销关闭了。

6. 密码管理

7. 左侧的 tab bar, 方便管理更多 tab。
个人还是喜欢放在上方的

8. 改进一些其他细节

总的来说，变得更好用了。

### 与 Afred 的集成

官方介绍不再向下兼容老的 AppleScript 语法。所以导致 Alfred 的 iterm2 插件全部无法使用。而对此官方提供了新的 [Alfred 支持脚本](https://gist.githubusercontent.com/gnachman/4cbe6743baa7fe07536b/raw/466b24deb91b8c7dde396023f327326ca1fa3661/gistfile1.txt)。并给出修改方法 [https://iterm2.com/images/AlfredForiTerm2Version3.png](https://iterm2.com/images/AlfredForiTerm2Version3.png)。

试了试，只能唤起 iterm2。如果是执行多句命令，遇到阻塞命令并不会摊开新的 tab，这里有替代方案：[custom-iterm-applescripts-for-alfred](https://github.com/stuartcryan/custom-iterm-applescripts-for-alfred/blob/master/custom_iterm_script_iterm_2.9.applescript)，将 alfred terminal profile 的 custom 内容替换即可。

**第二个问题是 workflow 的 terminalfinder 插件不工作了**。

用 alfred workflow 的，肯定都装过这个。它可以让你在 finder 跟 terminal/iterm2 之间自由切换。修复办法是：到[这里](https://github.com/LeEnno/alfred-terminalfinder)下载新版。

iterm2 beta3 版本官方介绍: [iTerm2 Version 3 Now in Beta](https://iterm2.com/version3.html?src=4)。

### 最后晒上一张配色 >:p 

![iterm2 配色](/images/iterm2.png)

