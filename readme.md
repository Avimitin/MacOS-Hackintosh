---
title: 快速上手使用黑苹果
date: 2019-08-25
tags:
- hackintosh
categories: hackintosh
description: 黑苹果安装过程记录
cover: https://img.vim-cn.com/71/7d5576cb97bebf5e6d5beade2ec35ea4c25706.png
---

# 前言

本篇文章的目的是为了减少新手爬贴的麻烦，但是怕麻烦的人尽早放弃黑苹果。一旦决心折腾黑苹果了，就一定要耐心做下去。黑苹果的安装就是刚完成一个环节下一个环节就会有可能出问题的折腾历程。如果对自己的耐性没有信心还是尽早去淘宝叫人花50块钱给你装。

假如出现了文章内没有出现的问题怎么办呢，这里有很多黑苹果的论坛，多查多看帖子。

- [远景](http://bbs.pcbeta.com/forum.php?gid=86)(目前关闭注册，看看教程贴就好了，资源是没法下载的了)
- [Tonymacx86.com](https://www.tonymacx86.com/)(资源丰富，建议去注册一个账号）
- [Github](www.github.com)(善用GitHub的搜索功能，里面有很多很好用的教程仓库)

# 准备环节

## 硬件

首先要清楚的了解自己的设备型号，包括但不限于主板，CPU，显卡，声卡，网卡。将你的硬件型号写个备忘录记起来。假如是AMD显卡用户，恭喜你你不仅可以安装Mojave，因为AMD免驱，还可以减少很多配置步骤。而N卡用户，就直接放弃Mojave装High Sierra吧，Mojave只有开普勒架构的N卡才能免驱，而N卡的驱动只支持到High Sierra。

黑苹果安装的具体的硬件依赖看这：---> [🖥️](https://github.com/CrazyPegAsus/macOS-Mojave-Compatibility-hardware-list)

假如你还没有电脑，打算低价打造高性能苹果机，这里有[硬件购买指南](https://khronokernel-5.gitbook.io/anti-hackintosh-buyers-guide/)。文章会告诉你需要避免购买哪些不支持的硬件。

看不懂英文？这里有[中文购买指南](https://osx.cx/retail.html)，已经帮你设计好购买套餐了。

最后你还需要一台Windows设备和一个8G以上的U盘。

## 软件

> 黑苹果需要很多工具，所以这里的篇幅会很长，读起来并且下载完需要费一些时间。但是还请做好准备工作，不要等出问题再去查再去下载，无意义地浪费很多时间。

安装黑苹果你需要简单了解并使用：

- [GUID分区表](#GUID分区)
- [ESP分区](#ESP分区)
- [Clover](#Clover和Config.plist)
- config.plist
- [High Sierra 镜像](#High Sierra 镜像)
- [BIOS](#BIOS设置)

### GUID分区

> MBR 分区表的组织限制了最大可寻址存储空间为 2 TB。因此它只支持最多 4 个主分区，或者 3 个主分区和 1 个扩展分区组合。然而，随着时代的进步，更大的存储设备需要应用到计算机领域。因此,取代 MBR 分区的 GUID 分区表 (GPT) 在新电脑中被使用，GPT 可以与 MBR 共存，以便为旧系统提供某种有限形式的向后兼容性。

黑苹果**必须使用**GPT分区表，你需要将安装Windows的硬盘数据进行备份，然后使用分区工具对硬盘重新分区。微软官方有[MBR转GPT的教程](https://docs.microsoft.com/zh-cn/windows-server/storage/disk-management/change-an-mbr-disk-into-a-gpt-disk)。

### ESP分区

> 几乎所有 2012 年后生产的计算机都支持引导 UEFI，而且不再执行以前的 “Legacy” 标准。这些计算机需要一个 ESP 分区来引导。ESP 代表 EFI 系统分区，使用 FAT32/FAT16 文件系统。ESP 负责存储固件启动时使用的EFI 引导加载程序和其他实用程序。如果您不小心删除了这个分区，您的系统将无法再引导。为了安全起见，ESP 是默认隐藏的，因此它没有驱动器号。

黑苹果的安装和使用需要EFI分区进行引导，在对硬盘进行GPT分区的同时，创建新的空白分区并格式化为EFI分区格式。成功引导黑苹果安装并能使用磁盘工具**必须**要有大于**200M**的空间分为EFI分区。

使用[DiskGenius软件](http://www.diskgenius.cn/download.php)来划分新的EFI分区。

### Clover和Config.plist

> Clover是一个操作系统启动加载器(boot loader)，能够同时运行于支持EFI方式启动的新式电脑和不支持它的老式电脑上。一些操作系统可以支持以EFI方式启动，比如OS X, Windows 7/8/10 64-bit, Linux；
>
> EFI不仅存在于操作系统的启动过程中，它还会创建操作系统可访问的表和服务(tables and services)，操作系统的运行依赖于EFI正确的提供功能。CloverEFI和CloverGUI做了大量的工作来修正内部表，让运行OS X成为可能。

在黑苹果的安装中，Clover起着相当于控制中枢地位的作用，磁盘工具，黑苹果升级，双系统引导等等操作都需要Clover。而Config.plist文件则是Clover的配置文件，能否正确引导黑苹果安装都需要Config.plist文件。只有适合的config文件才能安装和引导黑苹果。所以编写正确的config文件是黑苹果安装中的最难点。

你可以选择根据自己的配置来自定义config.plist文件，但是如果你不想花时间学习config文件的编写，可以去Github或者谷歌使用`你的机型+clover`或者`你的机型+Hackintosh`的方式对config文件进行搜索(比如我是华硕H110M-F主板就去搜索`ASUS H110M-F Clover`)。底下是我搜集到的一些clover集合仓库，你可以看看有没有与你机型相同的配置文件。台式机用户除了主板，尽量还要找到相同批次的CPU。

- [Beipy/Mac-Hackintosh-Clover](https://github.com/Beipy/Mac-Hackintosh-Clover)
- [底噪|黑苹果合集](https://zhih.me/hackintosh/#/)
- [sqlsec/clover](https://github.com/sqlsec/clover)
- [daliansky/Hackintosh](https://github.com/sqlsec/clover)
- [Huangyz0918/Hackintosh-installer-University](https://github.com/huangyz0918/Hackintosh-Installer-University)

clover的config文件具体如何使用后面会讲，假如没法找到完全相同的配置，多准备一些相近配置的config，之后可以套用。

假如像我一样，用了很冷门的配置的(垃圾华硕Orz)，只能自己写config文件，可以看一下这两个教程学写config.plist文件。

- [Clover安装黑苹果之Config.plist必看](https://www.it72.com/thread-8335.htm)

- [安装黑苹果的config.plist](https://www.cnblogs.com/eleven24/p/7912663.html)

### High Sierra 镜像

镜像的准备相对来说简单一些，去使用包含Clover的镜像，方便刷写。

这里有一些下载站：

- [底噪|黑苹果合集](https://zhih.me/hackintosh/#/OS-images)
- [17G63版本百度云下载链接(提取码：beip)](https://pan.baidu.com/s/1qAvVbnMYiinVC1EpH7LCdA)

在这个百度云的链接里有一些工具也需要下载：

- `Clover_Configurator_5.1.5.0.dmg`:config.plist的配置软件。
- `balenaEtcher-Portable-X64.exe`:黑苹果镜像刷写工具。
- `AIDA64硬件查看.7z`:假如不清楚你的硬件型号用这个工具查看。

## BIOS设置

- 假如你将`High Sierra`安装到机械硬盘上，设置硬盘`AHCI`。
- 关闭安全启动`Secure Boot`使主板能引导至黑苹果安装。部分主板如华硕没有此选项，则选择安全启动至`其他系统`。
- 显示设置屏蔽核显。(华硕主板设置为`PCIE`优先，`iGPU`关闭）
- 关闭快速启动`FastBoot-Disabled`
- 网络堆叠关闭，VT-d关闭。
- `XHCI Hand-off`开启

## 刷写U盘

刷写系统很简单，打开刚刚下载的balenaEtcher软件，第一项选择你的镜像，第二项选择你的U盘，点击Flash。然后就刷好了。就这么简单。

![](https://img.vim-cn.com/d0/fd508140f68006b260678a89a4d92213012ecd.jpg)

## 修改Clover文件夹

刚刚刷写的U盘会被分成两个区：EFI和系统盘，为了正确引导我们要替换EFI里的Clover，也就是刚刚上面准备环节让你下载好的clover文件夹。重新插入U盘，打开EFI盘（假如打不开就用DiskGenius来复制。点击EFI区，在中间的菜单栏的浏览文件操作。）

![](https://img.vim-cn.com/9c/fd8ab68c437af209396b6ccc33d909008c4fc6.png)

把原来的Clover文件夹删除（或者剪切保存备份），然后把你下载的clover文件夹整个拷进去，退出U盘。

# 正式安装

## U盘启动

插入U盘，重启电脑，弹出BIOS信息时根据不同的信息提示点击`F2`或者`F10`，选择你的U盘并选择UEFI模式启动：`UEFI:Kingston-xxx`。

## 安装系统

> **提醒！先把整个步骤看一遍，大概了解安装要做什么以后再开始正式安装**

### 1.磁盘抹除和硬盘信息写入

假如你的Clover文件夹里有很多的`Config.plist`文件，则需要在Clover的`option->configs`选项中选择合适的config文件。

![](https://img.vim-cn.com/0c/b6e6876d5929d6d2cd901b9556224668d6aca0.png)

然后选择`return`，选择`install mac from High Sierra`(你的U盘安装启动项，可能名字不一样)，等进度条结束，进入恢复工具。

---

*假如进度条一直卡在最后一点，肯定是config文件的显卡设置有问题。重新启动电脑，重新前面的步骤，在进入启动项之前，选中启动项，然后按下`空格键`，进入启动`option页面`，在启动项的`option`页面下的`boot-args`勾选`Set Nvidia to VESA (nv_disable=1)`选项来关闭显卡。*

![](https://img.vim-cn.com/57/dde7aaea1b07630bc3d2d4227f0f5586fc7f99.png)

---

假如成功进入恢复选项以后，右上角改中文（可能会很卡，等一会。）然后进入磁盘工具，选择你要安装系统的分区，上方点击抹除，名称自定，格式选择`Mac OS 扩展(日志式)`。一旦出现报错，一定是准备步骤那里的ESP分区不足200M。

然后点击左上角关闭窗口，选择安装Mac OS。

等文件写入完成，电脑会自动重启。

### 2.系统安装

还是按下`F2`(或者F10)，选择U盘启动，进入Clover之后你会发现新的启动项，还是选择相同的config，显卡有问题的还是选择`nv_disable=1`，然后进入新的启动项(应该是叫`Boot macOS install from 你的硬盘名字`)，进入之后会正式安装，中间假如没有安装完就重启了，继续这一步，选择同样的启动项直到安装结束。安装结束后会再次重启，这个时候这个启动项就会消失了。

### 3. 系统配置

然后就到了激动人心的一步了，保持同样的配置：选择相同的config，显卡有问题的还是选择`nv_disable=1`。然后选择`Boot macOS from 你的硬盘名字`，假如成功了就可以看到自定义设置界面，设置完成后就可以看到High Sierra的桌面啦！

### 提示

一旦出现文章没有提到的问题，在启动项的`option`页面下的`boot-args`勾选Verbose，就可以看到后台的报错了，然后谷歌报错找解决方案。

# 配置系统

## 硬盘引导

之前安装系统引导系统都是用的U盘，其实Mac自己也有一个EFI分区引导，把U盘里的文件拷贝过去就可以了。

打开准备环节下载好的`Clover_Configurator_5.1.5.0.dmg`文件，密码是：`imac.hk`,把软件拖动到右边的Application文件夹就是安装好软件了。(PS：Mac的安装软件真是深得我心，软件全是打包好的，拖进应用程序文件夹就算安装了，不像Windows那样一堆乱七八糟的文件也不像Linux那样需要一堆依赖)

打开Clover Configurator，左边的菜单栏选择`Mount EFI`,然后在EFI分区那里选择`Mount Partition`。单击`Open Partition`打开EFI分区，然后把U盘EFI分区里的`Boot`和`Clover`文件夹拷进去就行了。

![](https://img.vim-cn.com/d9/d36627fc381b2e2e0685ae394d46fd3a2c153d.png)

然后你就可以拔掉U盘重启电脑了。**记得将装macOS的分区设置为第一启动项**。

## 驱动安装

假如你是推荐配置的话，大部分硬件免驱，基本装好系统就能正常使用了。恭喜你你的黑苹果折腾已经结束了。但是可能有的人和我一样，刚好就都不是推荐配置，进去电脑声卡网卡显卡全部残废。没关系，mac的驱动安装也不难，跟着步骤一步步走就行了

### 网卡

首先确定自己的网卡是不是真的无法识别，再确定`/System/Library/Extension`路径下没有任何的网卡驱动，有就删掉，反正也不能驱动。然后确认自己的网卡型号，在下面的仓库找一下有没有同型号驱动：

- [Rehabman-Repo](https://bitbucket.org/RehabMan/)
- [daliansky/Hackintosh/LinkList](https://github.com/daliansky/Hackintosh/blob/master/LinkList.md)
- [tonymacx86](https://www.tonymacx86.com/resources/categories/kexts.11/)

然后下载[Kext Wizard](https://mac.softpedia.com/get/Utilities/Kext-Wizard.shtml)安装驱动。点击``installation->Browse`，选择`网卡驱动.kext`,然后安装

![](https://img.vim-cn.com/1f/b07eeb6b69d3703580bb0a7022391716ec039f.png)

重启生效。

### 显卡驱动

点击左上角苹果标志->关于本机,在版本那里查看详细版本号(比如我就是17G8030)：

![](https://img.vim-cn.com/cc/7c8595da85f5a72b344fa9f57706b0de9d628a.png)

然后上[tonymacx86](https://www.tonymacx86.com/nvidia-drivers/)下载对应版本的N卡驱动,跟着提示安装完以后会提示你重启，这个时候**不要重启**！

打开`Config-Configurator`，挂载EFI分区，单击左下角导入图标，然后把硬盘EFI分区里的Config.plist文件导入，左边菜单栏选择Boot，在arguments区域勾选`nvda_drv=1`选项，然后在`Custom Flags`处填写`NvidiaWeb=1`，如图：

![](https://img.vim-cn.com/4c/60f780fc274e7414468feae6fc4a2a4ee02880.png)

然后选择`Graphics`菜单项，中间一排勾选：`Inject NVidia`,`NvidiaGeneric`

在SMBIOS菜单项点击右边的按钮，选择和你CPU相似的设备(**不要选Macbook Air！**)

![](https://img.vim-cn.com/2b/6e2d368a812be0f5e301b5b0b715e48b5025d6.png)

最后在System Parameters菜单项，将`Inject Kexts`选为Yes,勾选`Inject System ID`,`NvidiaWeb`。

![](https://img.vim-cn.com/3f/aedc85ee599f857916ff02cdb1d17062c1cfe7.png)

重启时不用再带上`nv_disable=1`启动项。假如显示器无信号，重新插拔连接显卡的线。

不确定自己的显卡是否打上驱动了？你可以跟着下面的步骤来测试。

- 运行象棋游戏(Chess)，如果可以移动棋子，那么祝贺你，你的显卡硬件加速已经打开。
- 检查屏幕保护程序是否工作。打开系统偏好设置，选择：桌面-屏幕保护，点屏幕保护程序标签，测试一个屏幕保护程序。如果此时看到的是一个空的屏幕，这就意味着显卡加速没有打开。
- 测试 Dashboard ，拖放一个 widget。当添加一个 widget 时，如果你看到一个像涟漪一样的效果(或很酷的效果)，那么恭喜你，硬件加速打开了。
- 打开 Front Row。如果可以运行，那么 QE/CI 已打开，否则 QE/CI 没有打开。
- 运行Grapher，如果能看到三维图像，那么 QE/CI 已打开，否则 QE/CI 没有打开。
- 如果可以用grab截屏，那么 QE/CI 已打开，否则 QE/CI 没有打开。

### 声卡驱动

检查`System/Library/Extension`下是否还有声卡驱动，有就全部删除。

下载[MultiBeast](https://www.tonymacx86.com/resources/multibeast-10-4-0-high-sierra.401/)，在QuickStart选项选择`UEFi Boot Mode`，在Driver选项选择合适的声卡驱动，然后选择Build选项卡，安装好了以后重启生效。

假如还有其他的设备问题也可以在这个软件里找驱动安装。但是一定要先删除再安装。

## 升级系统

假如你是用的我发的百度云的资源，可能你还用着(17G63)版本，打开APP Store之后会有更新提示。

登陆app store账号之后点击升级，等待系统更新下载，下载完之后点击重启。然后系统会自动安装，然后重启机器

**注意！**这个时候你的Clover会多出一个启动项，对，就是那个之前消失的启动项`install macOS from 你的硬盘名`。千万不要直接开机了，你要选择这个install启动项。安装结束会再次重启，选择Boot macOS开机。

# 引用

> [黑苹果安装教程]<https://zhih.me/hackintosh-install-guide/>
>
> [Mac-Hackintosh-Clover]<https://github.com/Beipy/Mac-Hackintosh-Clover/blob/master/about/README.md>