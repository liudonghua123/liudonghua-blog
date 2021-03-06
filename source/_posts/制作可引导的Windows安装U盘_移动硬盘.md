---
title: 制作可引导的Windows安装U盘/移动硬盘
tags:
  - EFI
  - GPT
  - MBR
  - PBR
id: 235
categories:
  - 学习
date: 2014-10-19 16:32:45
---

<audio controls autoplay>
    <source src="/resources/2014/10/愿得一人心.mp3">
</video>
通常可引导的Windows安装U盘/移动硬盘 要求
1. 所在U盘活硬盘的MBR（主引导记录）是Windows NT 5.x/6.x MBR；
2. 所在分区的PBR（分区引导记录）是对于Windows XP是NTLDR，对于Vista、Win7、Win8+是BOOTMGR；
3. 所在分区要是活动的主分区；

<!--more-->

只要满足以上三点基本都可以引导安装系统，最好是在一个空的分区上格式化后操作（一般在Windows上格式化U盘后，MBR、PBR、活动的分区都已经自动设置了，不过还是可以用[BOOTICE](http://bbs.ipauly.com/viewforum.php?f=2)检测一下）
后面就是将ISO挂载后拷贝所有文件至U盘根目录下即可（以防万一最好在用BOOTICE检测一下）
[![bootice_mbr](/resources/2014/10/bootice_mbr.png)](/resources/2014/10/bootice_mbr.png)
[![bootice_pbr](/resources/2014/10/bootice_pbr.png)
](/resources/2014/10/bootice_pbr.png)[![bootice_active_partition](/resources/2014/10/bootice_active_partition.png)](/resources/2014/10/bootice_pbr.png)[
](/resources/2014/10/bootice_active_partition.png)

[![rufus-v1.3.4-UEFI](/resources/2014/10/rufus-v1.3.4-UEFI.png)](/resources/2014/10/rufus-v1.3.4-UEFI.png)
当然以上步骤只能安装用来引导安装系统在MBR硬盘分区上，如果我安装到GPT硬盘分区上就会提示
["windows cannot be installed on this disk. The selected disk is of the GPT partition style."](http://answers.microsoft.com/en-us/windows/forum/windows_7-windows_install/windows-cannot-be-installed-on-this-disk-the/8fa72a3e-10c5-47da-a040-1e0db62af309)

关于MBR、GPT、(U)EFI的关系其实很复杂，简答的说就是
Plain Ordinary BIOS -> MBR (只能有最多4个主分区，需要更多分区就只能通过扩展分区（占用一个主分区）下划分多个逻辑分区实现)
(U)EFI based BIOS    -> GPT (支持很多主分区)

所以需要制作(U)EFI的可引导安装U盘，这里参考了
[How To Make UEFI Bootable USB Flash Drive to Install Windows 8](http://www.nextofwindows.com/how-to-make-uefi-bootable-usb-flash-drive-to-install-windows-8/)
[Installing Windows on a GPT disk](https://social.technet.microsoft.com/Forums/en-US/05798532-d8c0-4f9e-b12f-58d3a0ee4c07/installing-windows-on-a-gpt-disk?forum=w7itproinstall)
[Installing Windows 7 on UEFI based computer](http://blogs.technet.com/b/askcore/archive/2011/05/31/installing-windows-7-on-uefi-based-computer.aspx)

不过我直接使用rufus制作的U盘在开机UEFI引导项下没有，最后只能通过diskpart先把U盘转换成GPT分区，然后在用rufus(其实这步应该可选的，直接拷贝系统文件到U盘应该也可以的了)。
大概用到的命令是

```shell
diskpart
sel disk 1 #根据U盘实际序号
clean
convert gpt
exit
```

最后要说明一点的是安装Win7及以上版本的Windows系统，如果是MBR分区硬盘，如果你自己没分区，系统就会自动创建一个300M左右的隐藏分区，所以在进行选择分区前shift+F10调出cmd执行diskpart自己分区然后再安装；如果是GPT分区，必须要有两个额外的分区，即使你自己分好区了，没有这两个分区，系统也会帮你创建，为了把这两个分区放到前面位置，可以在diskpart中手动创建
大概用到的命令是

```shell
diskpart
select disk 0 #根据硬盘实际序号
clean
convert gpt
create partition efi size=100
format fs=fat32 quick label=system
create partition msr size=300
create partition primary size=102400 #设置系统分区大小，M单位
format fs=NTFS quick label=win8
assign letter=c
exit
```

附上创建分区命令的类型参数，MBR类型中一般会用到PRIMARY、EXTENDED、LOGICAL；GPT类型中一般会用到EFI、MSR、PRIMARY

```shell
DISKPART> help create partition

EFI         - Create an EFI system partition.
EXTENDED    - Create an extended partition.
LOGICAL     - Create a logical drive.
MSR         - Create a Microsoft Reserved partition.
PRIMARY     - Create a primary partition.
```

另外从Win8开始开机如果有多系统开机系统选择界面默认是Metro风格的，都差不多启动完系统才让选择其他系统，非常不好用，如果要恢复Win7风格可以使用如下命令

```shell
bcdedit /set {default} bootmenupolicy legacy
```

如果要恢复则使用

```shell
bcdedit /set {default} bootmenupolicy standard
```

检测硬盘是MBR还是GPT分区格式，使用diskpart的list disk命令，Gpt列如果有*说明是GPT，否则是MBR
[![diskpart_mbr_gpt](/resources/2014/10/diskpart_mbr_gpt.png)](/resources/2014/10/diskpart_mbr_gpt.png)