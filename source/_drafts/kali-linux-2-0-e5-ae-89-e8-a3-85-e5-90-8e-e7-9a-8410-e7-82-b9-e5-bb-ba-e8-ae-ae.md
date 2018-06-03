---
layout: post
title: kali linux 2.0 安装后的10点配置建议
categories:
  - 安全
url: 122.html
id: 122
comments: false
date: 2016-01-12 22:18:48
tags:
---

自Kali 2.0发布以来，我们发现在安装kali之后经常会重复一些配置操作，对此，我们想要把它们分享出来，希望对大家也能够有所帮助。我们整理了一些遇到的常见问题的回答。下面是其中最常见的10个：

*   ### 激活或禁用智能侧边栏
    

有的人喜欢它，有的人不喜欢它。我们谈一下如何改变侧边栏的行为。进入【优化工具】，选择【扩展】，可以看到【Dash to dock】处于开启的状态，点击右侧的设置按钮，在【Position and size】tab页可以看到【Intelligent autohide】，只要将该选项禁用即可改变侧边栏的智能隐藏行为，附视频链接：[Disabling the Kali Linux 2.0 Sidebar Autohide feature](https://vimeo.com/136136854) from [Offensive Security](https://vimeo.com/offsec) on [Vimeo](https://vimeo.com).

*   ### 将你的SSH公钥添加到Kali 2.0
    

Kali Linux 2.0 延续了Debian的SSH配置选项， 从Jessie版本之后默认不允许root用户在没有key的情况下登录。

> root@kali:~# grep Root /etc/ssh/sshd_config PermitRootLogin without-password

比较不提倡的一种方法是将PermitRootLogin参数设置为“yes”然后重启SSH服务，可以允许root用户远程登录。如果要实现更加安全的远程root访问，可以将你的公钥添加到/root/.ssh/authorized_keys 文件中。

*   ### 需要时安装NVIDIA驱动
    

如果你有一块NVIDIA显卡，你应该按照这些步骤来为Kali 2.0安装[NVIDIA](http://docs.kali.org/general-use/install-nvidia-drivers-on-kali-linux)驱动。

*   ### 需要时安装VMware或者VirualBox Guest Tools
    

安装虚拟机客户工具的方法并没有很大变化，在最新的[Vmware](http://docs.kali.org/general-use/install-vmware-tools-kali-guest)（Workstation和Fusion）以及[VirtualBox](http://docs.kali.org/general-use/kali-linux-virtual-box-guest)中都能够正常工作。

*   ### 禁用Gnome桌面的锁屏特性
    

我们没有在正式版中禁用这个特性但是会在接下来的开发中以及将来的ISO发行版中这样做。下面是禁用Gnome锁屏特性的最快方法：进入【设置】-》【电源】，将【省电】中的空白屏幕设置为【从不】（never），然后回到【设置】主界面，进入【隐私】，将【锁屏】设为关。 附视频链接：[Disable the Screen Lock Gnome feature in Kali 2.0](https://vimeo.com/136158400)  from [Offensive Security](https://vimeo.com/offsec) on [Vimeo](https://vimeo.com).

*   ### 不要在你安装的kali 2.0中添加额外的源
    

如果因为某些原因，你在安装kali过程中，当被问到“使用网络镜像”时选择了否，可能会导致你的sources.list文件中丢失一些条目。如果是这种情况，检查一下[官方的源列表](http://docs.kali.org/general-use/kali-linux-sources-list-repositories)来确定哪些条目应该在那个文件中。即使很多非官方的说明让你这样做，你也应该避免在sources.list文件中添加多余的源。不要添加kali-dev, kali-rolling 或者任何其他的kali源除非你有特别的理由，一般情况下你是没有的。如果你不得不添加额外的源，在/etc/apt/sources.list.d/目录下添加一个新的源文件。

*   ### 如果以root用户运行让你觉得不爽，添加一个非root用户
    

我们看到很多用户使用Kali非常谨慎因为主要的系统用户是root。这经常会令我们感到困惑，因为添加一个非root用户到Kali中是一件微不足道的事情并且通过几条类似下面的简单命令很容易就能够做到这一点：

> root@kali:~# useradd -m muts -G sudo -s /bin/bash root@kali:~# passwd muts Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully root@kali:~#

*   ### 避免安装Flash Player
    

只要记住不要这样做。

*   ### 保持kali系统更新
    

我们每天4次同步来自debian的更新。这使得Kali的安全更新一直保持最新状态。你应该通过经常执行下面的命令来保持你的系统处在最新状态：

> apt-get update apt-get dist-upgrade

*   ### 避免在FSH定义的目录中手动安装工具
    

你有几种不同的方式来使用Kali——或者作为一个用后可仍的渗透系统或者作为一个长期使用的系统。用后可仍的方式是指安装kali作为一次性或者短期用途之后就扔掉。长期使用则是指想要每天使用Kali。这两种方式都是有效的但是需要区别对待。如果你计划每天使用Kali， 你应该避免在FSH定义的目录中手动安装程序因为这可能会与现有的包管理工具冲突。 参考：[Kali Linux 2.0 Top 10 Post Install Tips](https://www.offensive-security.com/kali-linux/top-10-post-install-tips/)