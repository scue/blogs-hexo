---
layout: post
title: "关键时刻，拿什么来拯救你的Mac？"
description: "记一次由于Macbook Pro自动升级开机失败，导致无法在线重装，蛮也无法使用TimeMachine恢复系统的惊心动魄之事。"
category: 技术
tags: [MacOS]
---

很久以前，我是十分信任MacOS是不会出现问题的，我们平时都可以随时将电脑合上就带走，去到另一个地方再打开又可以继续工作了。出于这样子的信任，我的电脑一般都是打开着「自动更新」的，如下。

![-w668](https://ws1.sinaimg.cn/large/6e22ca27gy1fqjyux1u0wj21140ty0yt)

其实很长一段时间都没有什么问题，直到最近一次自动更新了之后无法正常重启了，下边是整个事件的回放。

# 事件回放

出现这一次问题，案例id（100507377606）最初原因是勾选了 “自动升级”导致重启过程中更新软件失败，导致的MacOS无法开机（已尝试重启两次，中途有把log发送给apple）。

## 尝试使用TimeMachine恢复

使用TimeMachine恢复整个系统是失败的（尝试不同日期，一天前/一周前/一个月前均失败），第二个客服小姐姐提到，让我等待你们的服务器网络恢复，有可能是TimeMachine中途有需要下载软件或更新导致不能正常安装。

## 在线网络安装MacOS

尝试在线重新安装os（Reinstall MacOS），不是提示重要更新无法下载，就是最后还有两分钟剩余时候提示遇到一个错误（An Error Occurred ...），包括格式化过最顶层的SSD。周末我尝试了很多遍在线安装，也更换了几次网络环境。我怀疑是苹果升级服务器网络有问题。

## U盘启动安装

幸好第三次技术支持的时候，技术支持客服小哥哥提到了**可以制作一个u盘启动**，随后我使用U盘启动成功地安装好了系统，再使用TimeMacthine将的文件、系统设置、用户资料给恢复了（**没有恢复Applications**，出于考虑网络的原因，我更关注我的资料和文件，毕竟我有一些项目资料还没有来得及push至git repository）。在随后的时间里，我通过App Store和Brew Cask将我需要的软件给安装好了，直到现在已经把我的所有重要资料和文件都恢复好了。

整个过程我觉得问题可能出现在Mac软件更新的时候网络异常导致的，不排除我们国内网络的原因引起的，但是遇到这样子的问题的确让人很烦恼。以至于我现在一点也不也想看到“您有重要的更新可用，提高了稳定性...”之类的通知，现在我都把「自动更新」给关闭了，简直太危险了。

# 关键时刻，如何拯救你的Mac

可以看到，由于网络的原因，导致MacOS无法在线完成安装，使用U盘来安装成了关键的拯救方式。

我的思路是使用U盘安装好了操作系统，再使用TimeMachine来恢复自己重要的资料。（有了操作系统的相应文件之后，应用程序可以使用Brew和App store来完成安装）

## 制作U盘启动

这个步骤需要你使用另一台Mac来完成，幸好我还有另一台Mac Air在家里呆着，下边是详细的步骤。

1. 第一步，先下载 macOS High Sierra，https://itunes.apple.com/cn/app/macos-high-sierra/id1246284741?mt=12&l=zh-cn&ls=1

跳转至App Store，点击下载即可

2. 第二步，下载完成之后会弹出安装提示，我们不需要安装直接退出即可

3. 第三步，寻找一个U盘，至少是8GB的空间，接着调用一行命令：

```sh
sudo /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume --applicationpath /Applications/Install\ macOS\ High\ Sierra.app
```

这里的`/Volumes/MyVolume`需要替换成你的U盘的具体挂载路径。

参考官网链接：
https://support.apple.com/en-hk/HT201372#download

## 如何使用U盘启动

可以参考官网：

https://support.apple.com/zh-cn/HT202796

即，开机的时候按住Alt键，然后就会出现安装MacOS的提示，点击它（不需要连接网络），然后就可以直接进入到熟悉的安装系统的界面了。

## TimeMachine恢复重要资料

待安装好了MacOS之后，重启了会有一个提示是否需要从TimeMachine或从其他机器上恢复的提示，此时连接你的TimeMachine硬盘，会有如下的操作界面。

![](https://ws1.sinaimg.cn/large/6e22ca27gy1fqkbbf7khij20l50d175p)

我的建议尽量不要勾选“应用程序”（Applications）出于网络方面的原因考虑。

PS：你一般的配置都会在你的Home目录下，你的Home目录是可以完成恢复的，所以剩下的只需要自己去Brew Cask或App Store把应用程序安装回来即可。

# 后记

我希望您的电脑不要出现我类似的问题，万一出现了的话，建议再重新看看这一篇文章。整个过程我觉得可能就是Apple服务器在国内不稳定或是升级包可能异常导致的。从此以后，我建议还是不要勾选「自动更新」MacOS了，系统更不更新不重要，重要的是我能依靠它把我的工作做好。谢谢阅读。






