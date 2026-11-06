---
title: "把小米平板 5 折腾成三系统：Android 17、Windows 11 25H2 和 Fedora 43"
published: 2026-11-07
description: "一台已经刷过 Windows 双系统的 512GB 小米平板 5，重新分区后装成 Android 17、Windows 11 25H2 和 Fedora 43 三系统。记录分区表、DBKP 磁吸补丁、几个文档里没写对的地方，以及刷机前该留的后路。"
image: "./cover.webp"
tags: ["小米平板5", "Android", "Windows", "Fedora", "刷机"]
category: "技术实践"
draft: false
author: "阿杰"
comment: true
licenseName: "CC BY-NC-SA 4.0"
licenseUrl: "https://creativecommons.org/licenses/by-nc-sa/4.0/"
---

> [!WARNING]
> 刷机有风险。变砖、丢数据、保修失效、硬件损坏，都是可能的结果。本文只是记录我自己的操作，不是教程，也不提供任何保证。照着做，后果自负，成砖别来找我。

## 这台平板不是“原厂未拆”状态

小米平板 5，codename `nabu`，SM8150，512GB UFS。它之前已经刷过 Android + Windows 双系统，所以 GPT 不是原厂布局：ESP 只有 334MB，Windows 分区 232GB，Android userdata 232GB，整块盘基本被占满。

这也是我一开始最担心的地方。网上能找到的 nabu 三系统教程，大多默认你是 256GB 原厂机，从 `userdata` 后面切一块出来。直接照抄那些 `10.9GB / 90GB / 150GB` 的数字，在这台 512GB、已经改过分区的机器上大概率会重叠或者把 Windows 分区切坏。

最后我用的布局是：

```text
分区号  名称       起始 LBA     结束 LBA     大小        用途
31      userdata   2654272      44597311     160 GiB     Android 数据
32      linux      44597312     65568831     80 GiB      Fedora rootfs
33      esp        65568832     65830975     1 GiB       rEFInd / UKI / Windows Boot
34      win        65830976     124380927    223.35 GiB  Windows 11
```

这块盘是 4096 字节逻辑扇区，`parted` 里看到的 `s` 就是 4096 字节，不是常见的 512。LBA 数字看起来很大，别拿 512 字节扇区的习惯去算。

ESP 我给了 1GB。原来的 334MB 其实也能塞下 Windows Boot、rEFInd 和一个 64MB 左右的 UKI，但三系统以后还要留余量，索性一次给够。

## 版本和物料

这次用的东西都比较新，先把版本钉死：

| 部件 | 版本 |
|---|---|
| Android | DerpFest 17 Official，Android 17，build `CP2A.260605.016` |
| Root | KernelSU Next `v3.3.0-spoofed` |
| Windows | Windows 11 25H2 中文专业版，build `26200.9818` |
| Windows 驱动 | MiPad5 BSP `2608.03` |
| Linux | Fedora 43 KDE，kernel `6.17.0-sm8150-1-4` |
| 恢复环境 | V4 Modded TWRP `2026-10-15` |
| 引导 | 2608.03 nSB UEFI + rEFInd + Fedora UKI |

所有大文件都校验过：DerpFest 包的 SHA256 是 `baed533c...da05`，Windows ESD 是 `6198089a...8356`，Fedora rootfs 是 `51b4088f...4822`，BSP 驱动是 `39a4eba9...dd39`。校验这一步没什么技术含量，但省得刷到一半才发现包是坏的。

Android 安装包里还有一个细节：`autoinstaller.conf` 里写着 `ANDROID_VER="16"`，标题也是 A16。但实际发布页、系统属性、build 号都是 Android 17。这个配置文件是旧模板没改干净，别被它吓到。

## 刷 Android 17

DerpFest 的 Auto-Installer 本质上是一组 fastboot 命令。我没有直接双击 bat，而是把 zip 解开，核对了 `autoinstaller.conf` 里的 SHA1 列表，再按它同样的顺序刷：

```cmd
fastboot set_active a
fastboot flash boot_a images\ksu-n_boot.img
fastboot flash boot_b images\ksu-n_boot.img
fastboot flash dtbo_a images\dtbo.img
fastboot flash dtbo_b images\dtbo.img
fastboot flash vendor_boot_a images\vendor_boot.img
fastboot flash vendor_boot_b images\vendor_boot.img
fastboot flash vbmeta_a images\vbmeta.img
fastboot flash vbmeta_b images\vbmeta.img
fastboot flash vbmeta_system_a images\vbmeta_system.img
fastboot flash vbmeta_system_b images\vbmeta_system.img
fastboot flash super images\super.img
fastboot erase metadata
fastboot flash userdata images\userdata.img
fastboot erase userdata
```

脚本里还有一行 `fastboot erase frp`，我跳过了。FRP 分区我提前做了只读备份，没有动它。不是说这行一定会出事，而是这种不可逆分区，能不动就不动。

`super.img` 有 5.48GB，fastboot 会分成多段 sparse 写入，耗时大概两分半。刷完第一次开机等了十几分钟，中间还自动重启了一次，最后进了 Android 17。KernelSU Next 的安装包是 `KernelSU_Next_v3.3.0.apk`，装完打开显示 Working，但它的包名是一个随机字符串 `vctsrt.cntgtj.uqfwgg`，不是常见的 `me.weishu.kernelsu`——这是 ROM 预装的 KSU-N 管理器用了伪装包名，不是装错了。

## 重新分区：别抄 256GB 的数字

刷完 Android 后，我进 TWRP 重新分区。原来的 31/32/33 分别是 ESP、Windows、userdata，和很多教程里的“31=userdata”完全不是一回事。

第一次执行 `parted rm 33` 时，它报了：

```text
Partition(s) 33 on /dev/block/sda have been written, but we have been unable to inform the kernel of the change, probably because it/they are in use.
```

翻译过来就是“GPT 改了，但内核还在用旧分区”。这不是硬盘坏了，是 TWRP 还挂着 userdata/ESP。我的处理是直接重启回 TWRP，让内核重新读一遍 GPT，然后从空闲区重新建分区。

这里有个小插曲：重启后 `parted print` 里 31、32、33 都已经没了，说明上一次其实已经写进磁盘，只是内核没刷新。于是我没有再删一遍，而是直接从 `2654272s` 开始建四个新分区。

```text
parted /dev/block/sda --script mkpart userdata ext4 2654272s 44597311s
parted /dev/block/sda --script mkpart linux ext4 44597312s 65568831s
parted /dev/block/sda --script mkpart esp fat32 65568832s 65830975s
parted /dev/block/sda --script mkpart win ntfs 65830976s 124380927s
parted /dev/block/sda --script set 33 esp on
```

`parted` 会提示“not properly aligned for best performance”，因为起始 LBA 不是 1MB 对齐。这个警告可以忽略，原来的 ESP 也不是 1MB 对齐的；只要不重叠、不越界，功能上没问题。

然后是格式化：

```text
mkfs.fat -F32 -s1 /dev/block/sda33 -n ESPNABU
mkfs.ntfs -f /dev/block/sda34 -L WINNABU
make_f2fs -f /dev/block/sda31
fixgpt
```

`fixgpt` 的输出很吓人，会把 `sda` 到 `sdf` 几个 LUN 都扫一遍，还会问要不要改 backup GPT 的位置。它最后会自己写完。只要最后 `parted print` 里四个分区名和大小都对，就不用管中间那些 `Aborting change!`。

第一次重启 Android 时，系统停在 DerpFest Recovery，报 `enablefilecrypto_failed`。这不是变砖，是 userdata 的加密初始化没完成。点 `Factory data reset`，让它重新格式化一次 userdata，再重启就进系统了。数据本来就是空的，这一步没有额外损失。

## Windows 11 25H2

Windows 这一步反而比较顺。TWRP 里执行 `adb shell msc`，把整块 UFS 暴露给 PC，然后给 `WINNABU` 分配 `X:`，给 `ESPNABU` 分配 `Y:`。

中文版 Windows 11 25H2 ESD 里只有两个索引：

```text
索引 1：Windows 11 家庭中文版
索引 2：Windows 11 专业版
```

网上很多教程写 `dism /apply-image /index:6`，那是英文 ESD 的索引。中文 ESD 照抄 6 会直接 `Error 87`。我先用 `dism /Get-ImageInfo` 确认索引 2 是专业版，再 apply：

```cmd
dism /Apply-Image /ImageFile:C:\nabu\win\install.esd /Index:2 /ApplyDir:X:\
```

29GB 的镜像通过 USB 写到 TWRP 的 mass storage，花了大概二十分钟。之后用 BSP 里的 `DriverUpdater.exe` 离线注入 2608.03 驱动，再跑 `bcdboot X:\Windows /s Y: /f UEFI`。`bcdboot` 输出“成功创建启动文件”就算过。

第一次启动 Windows 要用 **SB** UEFI 镜像：

```cmd
fastboot boot MiPad5.UEFI-v2608.03.img
```

OOBE 中间它自动重启回 Android 一次，这是正常的。再 `fastboot boot` 一次 SB 镜像，继续把 OOBE 走完。到桌面后，以管理员运行我提前放到 `C:\windows-postinstall.cmd` 的脚本，做四件事：

- `powercfg /h off`
- `manage-bde -off C:`
- 备份并删除 ESP 里的 `SiPolicy.p7b`
- `bcdedit /store Y:\EFI\Microsoft\BOOT\BCD /set "{default}" testsigning on`

重启后改用 **nSB** UEFI：

```cmd
fastboot boot MiPad5.UEFI-v2608.03_nSB.img
```

能进 Windows 桌面，就说明 testsigning 和 SiPolicy 都对了。右下角会出现 `Test Mode` 水印，这是 nSB 的预期结果，不是没激活。

设备管理器里有一个 `ACPI\QCOM057E` 的黄色感叹号。我查了 BSP 的 `nabu.xml`，这行是：

```xml
<!--DriverPackageFile Path="...PMIC\ExternalIC" Name="qcpmicEIC8150.inf" ID="qcpmicEIC8150"/-->
```

官方自己把它注释掉了。也就是说这个“未知设备”是故意不装的，不是缺了关键驱动。别看到黄叹号就随便找个 Qualcomm 驱动糊上去。

## Fedora 43

Fedora 这边只刷 rootfs，不刷 ESP 镜像。`esp-43.0.img.zst` 会覆盖整个 ESP，Windows 的 `EFI/Microsoft` 就没了。正确做法是用 `efi-files-43.0.zip`，只往 ESP 里加 rEFInd 和 UKI。

先把 `.zst` 解压：

```bash
zstd -d fedora-43.0-nabu-rootfs-kde.img.zst
```

得到 6.69GiB 的 ext4 镜像，然后进 fastboot：

```cmd
fastboot flash linux fedora-43.0-nabu-rootfs-kde.img
```

只写 `linux` 分区，不碰 `esp`。刷完后，在 Windows 里把 `efi-files-43.0.zip` 解压到 `Y:\`，保留 `EFI/Microsoft`，删掉可选的 `EFI/Android`。关键文件对一下：

- `Y:\EFI\BOOT\bootaa64.efi`：256,960 字节，rEFInd
- `Y:\EFI\fedora\fedora-6.17.0-sm8150-1-4.efi`：约 64.5MB，Fedora UKI
- `Y:\EFI\Microsoft\Boot\bootmgfw.efi`：Windows 的引导还在

Fedora UKI 的 cmdline 里写的是 `root=PARTLABEL=linux`，所以 GPT 分区名必须逐字是 `linux`，不能写成 `Linux` 或者 `fedora`。

第一次进 Fedora 会黑屏一会儿，终端里闪几行 `usr`、`data` 之类的初始化信息，然后进登录页。默认账号 `user`，密码 `fedora`。进去后我核对了：

```text
uname -r          → 6.17.0-sm8150-1-4
findmnt -n -o FSTYPE / → ext4
df -h /           → 79G，已用 4.8G
findmnt /boot/efi → /dev/sda33，vfat
ls /boot/efi/EFI  → Boot fedora Microsoft
```

到这里三系统其实已经能用了，但还差最后一步：磁吸盖冷启动。

## 磁吸双启动：文档里最容易把人带沟里的一节

我一开始按交接文档用 WOA Helper 1.8.7.2 的 DUALBOOT KERNEL PATCHER。后来拆开看，它下载的 `nabu.fd` 是 `erdilS/Port-Windows-11-Xiaomi-Pad-5` 的 `Files` release 里的旧文件，内嵌构建日期是 2026 年 1 月；而驱动和 UEFI 是 2608.03。官方反复强调 UEFI 和驱动不能混版本，这个东西正好踩在红线上。

所以我没有用 WOA Helper 的自动下载，而是手动从 2608.03 的 nSB 镜像里取 FD：

1. `magiskboot unpack MiPad5.UEFI-v2608.03_nSB.img`，得到 `kernel`。
2. 这个 `kernel` 前面有 112 字节的包装头，去掉后得到 3,145,728 字节的 raw FD，也就是 `MiPad5.UEFI-v2608.03_nSB.fd`。
3. `magiskboot unpack ksu-n_boot.img`，得到 Android 的 kernel 和 ramdisk。
4. 用 `DualBootKernelPatcher-x86_64.exe` 把 FD、`DualBoot.Sm8150.cfg`、`ShellCode.Nabu.bin` 打到 kernel 上。
5. 替换 kernel，`magiskboot repack`，得到 `df17_dbkp_boot.img`。

我没有直接刷，而是先：

```cmd
fastboot boot df17_dbkp_boot.img
```

合上盖子，机器进了 UEFI/rEFInd；开盖重启，进了 Android。说明补丁和 2608.03 FD 都对。确认之后才写入两个槽：

```cmd
fastboot flash boot_a df17_dbkp_boot.img
fastboot flash boot_b df17_dbkp_boot.img
```

这里必须说清楚方向：**官方 DBKP 的逻辑是合盖 → UEFI，开盖 → Android**。我看到的交接文档写的是“合盖 Android、开盖 rEFInd”，正好反了。如果你按文档去理解，测试时会以为补丁失败。

另外，DBKP 官方自己写了“约 50% 用户打完后 Windows 无法启动”。所以刷之前一定保留未打补丁的 `df17_boot_a.img` / `df17_boot_b.img`。万一 Windows 起不来，刷回这两个文件就能回到 Android + fastboot 的状态，再改用 WOA Helper 的软件切换或音量键方案。不要用改造前 MIUI 的 `normal_boot.img` 回滚当前 Android，那个 boot 和 DerpFest 不匹配。

## 几个只有实际刷过才会遇到的小问题

**KSU 显示 Working，但 adb shell 里没有 `su`。** 这台机器上就是这样。管理器里 root 是正常的，app 授权也能用，但 `adb shell su` 会报 not found。它不影响日常使用，也不代表 KSU 没装好，别因为这个去重刷 boot。

**笔一开始不能用。** Android 的设置里有一个“启用触控笔”的开关，打开之后一代笔就正常了。底层其实早就有 `NVTCapacitivePen` 这个 input device，但开关没开的时候收不到事件。Fedora 下笔是正常的，所以不是笔坏了。

**Windows 的黄叹号不一定是缺驱动。** 前面说的 `QCOM057E` 就是官方主动不装的 PMIC EIC。设备管理器里留一个未知设备，比乱装一个不匹配的驱动安全。

**`fixgpt` 的输出很长。** 它会扫多个 LUN，还会问 GPT 备份区的位置。看起来像要出事，其实最后写完了就行。真正要看的是最后的 `parted print`。

## 最后的结果

现在这台平板的冷启动逻辑是：

- 开盖 → Android 17
- 合盖 → UEFI → rEFInd → Fedora / Windows

三个系统都实际进过桌面：

| 系统 | 结果 |
|---|---|
| Android 17 | DerpFest 17，KSU-N 3.3.0 Working，Wi‑Fi/声音/触控/笔正常 |
| Windows 11 25H2 | 26200.9818，2608.03 驱动，Test Mode 水印，除官方保留的 PMIC EIC 外无异常 |
| Fedora 43 | kernel 6.17.0-sm8150-1-4，根分区 79G，EFI 已挂载，Wi‑Fi/声音/触控/笔正常 |

留下的长期注意事项：

- **Fedora 下不要充电**。这不是省电建议，是这台机器的硬件限制。
- Windows 睡眠后如果出现白线/花屏，立刻重启，别继续用。
- nSB 下 Windows 不能通过 Windows Update 升 26H2，建议暂停大版本更新。
- Android OTA 或重刷 DerpFest 会覆盖 boot，KSU root 和 DBKP 磁吸补丁都会失效，需要重新打。
- 不要刷 `esp-43.0.img.zst`，不要跑 Fedora/Arch 指南里的 `installer_bootmanager.zip`，不要跑 `postinstall extract-efi`。

## 写在最后

这次最花时间的不是某一条 fastboot 命令，而是核对“文档里写的”和“机器上实际的”之间的差距：分区号、扇区大小、ESD 索引、DBKP 用的 FD 版本、磁吸方向，几乎每一项都和某份教程有出入。

如果让我给一句建议：先把 GPT、boot、ESP、FRP、persist、modem 备份好，再动手。刷机本身不难，难的是出错之后还有没有回头路。至于成砖——本文不负责，也负不了这个责。
