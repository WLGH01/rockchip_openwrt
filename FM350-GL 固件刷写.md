# FM350-GL 固件刷写

# 精简方法

## 备份

插上模块，然后使用ADB工具备份分区

- 确保在备份过程中没有出现错误。
- 备份后，请检查得到的文件是否不为空且没有被FF填满（特别是`nv*`分区）。

```shell
adb pull /dev/mtd0 C:\FM350\mtd0
adb pull /dev/mtd C:\FM350
```

## 准备固件

在HEX编辑器中打开备份的mtd0（preloader）文件，然后转到地址`0x40100`

- 在线HEX编辑器：https://hexed.it/

如果值为`3C 10 14 89`，则可以刷入该组合好的固件

## 固件升级

可以直接跳转到原版方法里的固件升级中，安装步骤升级

# 原版方法

方法用于将FM350调制解调器固化为USB。

注意！仅适用于有经验的用户和实验者。 您对自己的调制解调器所做的任何事情都是自担风险。

如果您格式化调制解调器，将删除所有分区和调制解调器的所有数据，包括IMEI等。 尚未找到失败的固件升级或格式化后的恢复方法。 尝试在受保护的分区中写入调制解调器时会出现Security deny for [...]错误。

调制解调器可以使用不早于v6.2124的SP Flash Tool。 SP Flash Tool v5不起作用，因为调制解调器使用了新的下载代理（DA）协议。 同样，没有办法使用高于v6.2124的版本，因为它们实现了更新的协议。

https://spflashtools.com/windows/sp-flash-tool-v6-2124

https://mtkdriver.com/mtk-driver-v5-2307

所有的固件测试都是在USB中进行的。目前还不清楚SP Flash Tool是否会在PCIe中工作。

建议您熟悉相关主题。https://4pda.to/forum/index.php?showtopic=469340

目前还不清楚任何固件是否适用于任何调制解调器。



## 备份

请务必备份所有工作调制解调器的分区。当找到恢复方法时，这可能会有所帮助。

到目前为止，备份只能通过ADB完成。 尝试使用SP Flash Tool读取时，无法下载一些受保护的分区，例如nvdata。

通过ADB进行备份的命令

```shell
adb pull /dev/mtd0 C:\FM350\mtd0
adb pull /dev/mtd C:\FM350
```

确保在备份过程中没有出现错误。

备份后，请检查得到的文件是否不为空且没有被FF填满。特别是nv*分区。

## 准备固件

为了使用来自Microsoft网站的固件，首先需要准备它，以便SP Flash Tool能够"识别"它。

1、从Microsoft网站下载固件https://www.catalog.update.microsoft.com/Search.aspx?q=3500+fibocom
2、解压下载的文件，例如使用7-Zip，直到找到FwPackage.flz文件
3、解压FwPackage.flz

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30099233.png)

4、接下来，需要从Scatter.xml文件中收集所需的固件文件。
打开FwPackageInfo.xml文件并查找Subsysid="default"。
这样我们就知道从哪里以及需要哪些文件了。

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30099229.png)

将文件复制到包含Scatter.xml的文件夹中：
- 从文件夹FM350.E09复制所有内容
- 从文件夹FM350.E09_preloader复制所有内容
- 从文件夹81600.0000.00.29.22.06复制所有内容

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30099232.png)

一些文件的名称与Scatter.xml中的不同。需要将它们重命名。具体的文件名可以在Scatter.xml中查看。

\- FM350.E09_loader_ext-verified_00.08.img => loader_ext-verified.img
\- FM350.E09_preloader_35001CF8_00.08.bin => preloader_k6880v1_mdot2_datacard.bin

在固件中可能有几个preloader文件，它们有不同的数字：

\- FM350.E41_preloader_**8A3A103C**_00.08.bin
\- FM350.E41_preloader_**35001CF8**_00.08.bin
\- FM350.E41_preloader_**8914103C**_00.08.bin

为了理解哪一个要刷入，可以查看来自您自己备份的preloader（mtd0）文件。 在HEX编辑器中打开它，然后转到地址0x40100。

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30339302.png)

在固件中的FM350.E41_preloader_8914103C_00.08.bin文件，需要跳转到地址0x100。

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30339301.png)

我们看到一组3C 10 14 89 => 翻转 => 8914103C - 这就是我们需要的。 我们取FM350.E41_preloader_8914103C_00.08.bin文件并重命名为preloader_k6880v1_mdot2_datacard.bin

这些数字还存储在NVDATA分区中。我不知道这是否会影响任何东西... 如果你找不到你自己的，要么寻找另一个固件，要么就放弃并简单地刷入那个其中有00 00 00 00的文件。

\- OP_OTA_006.048.img => OP_OTA.img
\- OEM_OTA_5000.0000.054.img => OEM_OTA.img

\- 文件DEV_OTA.img不在所有固件中都有。目前还不清楚它负责什么。在刷写时最好取消勾选这个分区。分区mcf3。但是我设法使用了来自Acer dongle固件的DEV_OTA.img。在Acer的固件中，这个分区由文件reserved.txt组成。已附上。

在固件中通常有带有后缀-verified的文件。但在Scatter.xml中，所有文件名都是没有这个后缀的。
据我所知，带有后缀的文件在末尾包含一些额外的数据，很可能是一种签名。
而且，SP Flash Tool首先寻找带有后缀的文件，如果找不到，就会使用没有后缀的文件。这在SP Flash Tool的日志中是可见的。所以不需要重命名这样的文件。



## 固件升级

首先安装必要的驱动程序。

SP Flash Tool支持3种固件升级模式：

1. **Format All + Download**
   这是一种危险模式，可能会变砖！在这种模式下，将完全格式化所有内存，然后用新的分区进行划分，并在其中写入固件文件。这将导致调制解调器的所有信息（如IMEI等）丢失。
2. **Firmware Upgrade**
   这是一种危险模式，可能会变砖！在这种模式下，重要的分区将被保存到计算机（？），然后完全格式化所有内存，接着用新的分区进行划分，并在其中写入固件文件，之后重要的分区将被恢复。
3. **Download Only**
   相对安全的模式。在这种模式下，并不是格式化所有内存，而是只格式化那些将要被刷写的分区。而重要的分区，如nvdata，将不会被影响。



由于目前还未找到恢复方法，建议使用Download Only模式！

刷写过程：
- 断开调制解调器与计算机的连接
- 启动SP Flash Tool

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30099441.png)

- 选择准备好的固件的download_agent文件夹中的flash.xml文件
- 选择准备好的固件的download_agent文件夹中的auth_sv5.auth文件
- 设置为Download Only模式！

![Прикрепленное изображение](https://ds-blobs-3.cdn.devapps.ru/30099460.png)

- 如果您对DEV_OTA.img文件不确定，最好在刷写时取消勾选此分区。分区mcf3。
- 点击Download按钮
- 将调制解调器连接到计算机

刷写过程应该开始。

如果在更新过程中出现错误，很可能是选定的固件不适合您调制解调器的分区布局，或者固件组装不正确。
只有通过格式化才能更新到具有不同分区布局的固件。而这可能导致变砖的风险。

如果出现错误，请查看SP Flash Tool的日志，特别是host.log文件。在这个文件中记录了刷写过程中与调制解调器的所有通信。
