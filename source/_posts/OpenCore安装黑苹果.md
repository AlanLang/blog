---
title: OpenCore安装黑苹果
tags: 瞎折腾
categories: 瞎折腾
abbrlink: 11242
date: 2020-06-11 09:49:23
---
[EFI文件](https://github.com/AlanLang/ASRock-Z390-Phantom-Gaming-ITXac-OpenCore-Hackintosh)
## 配置清单
|硬件配置|选型|
|---|---|
|CPU|i7 9700k|
|主板|华擎 Z390 Phantom Gaming-ITX AC|
|显卡|蓝宝石RX560 + 显卡延长线|
|内存条|芝奇幻光戟16G * 1|
|硬盘|WD/西部数据 SN550系列500G SSD|
|电源|全汉MS600 铜牌全模组+定制线|
|无线网卡|博通DW1560 (可以直接替换主板上的)|
|散热|乔思伯240水冷|
|机箱|定制的A4机箱|

<!-- more -->

## BIOS设置
禁用如下：

|英文|中文|
|:----|:---|
| Fast Boot | 快速启动 |
| CFG Lock (MSR 0xE2 write protection) | CFG 锁 (MSR 0xE2 写入保护) |
| vt-d | vt-d |

启用如下:

| 英文 | 中文 |
| :------------------- | :------------------------------------------------------- |
| Above 4G decoding | 大于 4G 地址空间解码 |
| CSM | 兼容性支持模块 |
|IGPU|IGPU多监视器|
|XHCI Hand-off|XHCI Hand-off|

注意 关于CSM看网上教程都说要关闭它，但是我关闭了之后有一定概率引导失败，所以就把它打开了

## config.plist
### ACPI
#### ACPI—–Add
根据ACPI目录下所用的SSDT填写
#### ACPI—–Block
不用填
#### ACPI—–Patch
一些热补丁
#### ACPI—–Quirks
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
|FadtEnableReset| Boolean| false |一些旧的主板需要对FADT进行标记来激活电脑的开机和关机功能，这里我们不许要启动它（如果你遇到关机变重启，可以打开试试，我们之后也会在nvram中将这个问题修复）|
|NormalizeHeaders| Boolean| false |清理ACPI头，一些主板的ACPI表需要打开这个修复启动。但如果补丁点亮系统，请试试NO|
|RebaseRegions| Boolean| false |换硬件、升级BIOS等对硬件的操作会对ACPI表产生影响，一般不需要打开，若发现卡PCI Configuraion Begin，请尝试打开|
|ResetHwSig| Boolean| false |休眠相关项，台式机不需要|
|ResetLogoStatus| Boolean| false |重置登录状态|

### Booter
内存相关选项设置。
#### Booter--MmioWhitelist
默认的第一项是为Haswell芯片提供的内存寻址修复，如果此类芯片碰到内存相关问题，请开启它(enable选择yes)。
默认第二项是开机卡PCI Configuration这里。ACPI、PCI device同时释放到内存时发生0x1000内存地址被占用而卡在PCI Configration.如果碰到此类问题，请开启它。

#### Boot—Quirks
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| AvoidRuntimeDefrag | Boolean | true | 修复UEFI运行时服务，例如日期，时间，NVRAM，电源控制等|
| DevirtualiseMmio | Boolean | false | 减少被盗的内存占用空间，扩展值的选项|
| DisableSingleUser | Boolean | false | 禁止单用户模式|
| DisableVariableWrite | Boolean | false | 非原生NVRAM主板需要模拟nvram.plist进而写入variable值，因此要禁止此项来防止其他程序对nvram进行写入, 如果你的主板支持原生nvram，请选择NO|
| DiscardHibernateMap | Boolean | false | 重用原始的休眠内存映射 |
| EnableSafeModeSlide | Boolean | true | 允许在安全模式下使用Slide 值|
| EnableWriteUnprotector | boolean | true | 在执行期间从CR0寄存器中删除写保护|
| ForceExitBootServices| boolean | false | 这个选项是让那些非常老旧的主板也能使用内存寻址|
| ProtectMemoryRegions | boolean | false | 官方对此项目的解释与AvoidRuntimeDefrag类似，除非你明白这是什么，不然选择NO，其实我也不明白。|
| ProtectSecureBoot | boolean | fasle | 保护uefi安全启动被写入|
| ProtectUefiServices | boolean | fasle | 保护UEFI服务不被固件覆盖，主要与VM，Icelake和较新的Coffeelake系统有关， 一般Z490的主板需要|
| ProvideCustomSlide | boolean | true | 如果Slide 值存在冲突，则此选项将强制macOS使用伪随机值。接收调试消息的人需要Only N/256 slide values are usable!|
| RebuildAppleMemoryMap | boolean | false | 生成与macOS兼容的内存映射|
| SetupVirtualMap | boolean | true | 将SetVirtualAddresses调用修复为虚拟地址，在Skylake和更高版本上不需要|
| SignalAppleOS | boolean | false | 促使硬件始终启动macOS，主要是对带有dGPU的MacBook Pro有利，因为启动Windows不允许使用iGPU|
| SyncRuntimePermissions | boolean | true |修正硬件在注入内存时无法注入权限的问题|

### DeviceProperties
此项是用来注入你的设备的，主要是显卡和声卡两部分。
#### DeviceProperties--Add
根据实际情况填写声卡和显卡的信息
#### DeviceProperties--Block
这里是禁用一些设备的，我们按默认就行了，不需要任何修改。

### Kernel
这里是内核相关选项。
#### Kernel--Add
根据Kexts目录下的文件进行填写
#### Kernel--Block
不需要
#### Kernel--Emulate
此选项帮助Ivy Bridge 和一些不受支持的CPU加载电源管理的，所有选项按默认即可。
#### Kernel--Patch
保持默认
#### Kernel--Quirks
内核相关的快捷选项

| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| AppleCpuPmCfgLock | boolean | false | 仅当无法在BIOS中禁用CFG-Lock时才需要 |
| AppleXcpmCfgLock | boolean | false | 仅当无法在BIOS中禁用CFG-Lock时才需要+1 | 
| AppleXcpmExtraMsrs | boolean | false | 禁用奔腾和许多Xeon等不受支持的CPU所需的多个MSR访问。|
| AppleXcpmForceBoost | boolean | false | 强制使用最大性能，仅建议在持续负载的设备或媒体计算机上启用。|
| CustomSMBIOSGuid | boolean | false | 对UpdateSMBIOSMode自定义模式执行GUID修补。通常与戴尔笔记本电脑有关|
| DisableIoMapper | boolean | false | 禁用vt-d，我们在BIOS里已经禁用vt-d了，这里我们选择NO就行了。|
| DisableRtcChecksum | boolean | false | 越过两条rtc检查(0x58及0x59) |
| DummyPowerManagement | boolean | false | 替代NullCpuPowerManagement.kext，如果你使用此补丁，请删除并选择yes。我们一般选择no。|
| ExternalDiskIcons | boolean | false | 修复苹果系统把内部硬盘识别为外置硬盘时（黄色图标的硬盘）开启，我们一般选择NO。|
| IncreasePciBarSize | boolean | false | 解决卡PCI configuration，如果碰到请选择yes, 我们选择no。|
| LapicKernelPanic | boolean | false | 适用于HP笔记本的内核奔溃选项|
| PanicNoKextDump | boolean | false | 防止kext出错打报告而让我们看不到真正的panic原因，这个随便选，我选择NO。|
| PowerTimeoutKernelPanic | boolean | false | 10.15系统中存在一些设备自身的电源管理无法让系统进入睡眠而超时，导致内核奔溃，如果有这个问题请选择YES。|
| ThirdPartyDrives | boolean | false | 开启Sata类SSD的trim功能，我没有sata类的ssd，我选择NO。
| XhciPortLimit | boolean | true | 解除15个端口限制|
### Misc
开机引导类的设置。
#### Misc--BlessOverride
这个选项是帮助我们寻找一些不寻常的EFI位置的，除非你有这种情况，不然我们不需要填写任何东西。
#### Misc--Boot
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| ConsoleAttributes | Number | 0 | 开机选择界面的颜色 |
| HibernateMode | String | None | 检测休眠模式。|
| HideAuxiliary | Boolean | false | 在开机选择画面隐藏一些辅助项目，比如recovery盘，clean NVRAM等。一般我们选择NO。|
| HideSelf | Boolean | YES | 隐藏自身的EFI引导盘选项|
| PickerAttributes| Number | 0 | 用于设置自定义选择器属性，此处将不介绍 | 
| PickerAudioAssist | Boolean | No | 用于在选择器中启用VoiceOver之类的支持 |
| PickerMode | String | Builtin | 设置OpenCore使用内置的选择器 |
| PollAppleHotKeys | Boolean | true | 允许您在引导过程中使用Apple的热键 |
| ShowPicker | Boolean | true | 是否显示开机启动盘选项 |
| TakeoffDelay | Number | 0 | 开机热键延时 |
| Timeout | Number | 5 | 倒计时进入指定硬盘 |
#### Misc--Debug
是否开启debug模式，这里我们暂时不需要，全部忽略过。
#### Misc--Entries
这里是帮助我们添加一些你希望的引导路径，不用修改。
#### Misc--Security
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| AllowNvramReset | Boolean | false | 是否在开机引导项中加入重置nvram缓存功能的选项 |
| AllowSetDefault | Boolean | false | 选择yes后即可在开机选择系统页面中通过Ctrl+enter键设置默认启动盘|
| AuthRestart | Boolean | false | filevault相关项，选择NO。|
| BootProtect | String | None ||
| ExposeSensitiveData | Number | 6 | 显示更多的调试信息 |
| HaltLevel | Number | 2,147,483,648 | 按默认设置即可 |
| ScanPolicy | Number | 11,470,595 | |
| Vault | String | Optional | 黑苹果的vault加密方式，我们不需要这个功能，填Optional |
#### Misc--Tools
这里是加入一些开机时候的工具的。
### NVRAM
#### NVRAM--Add
4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14

| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| DefaultBackgroundColor | Data | 00000000 | 默认开机背景色为黑色 |
| UIScale | Data | 01 | 01为普通的UI显示模式，02为开启HIDPI的UI显示模式|
7C436110-AB2A-4BBB-A880-FE41995C9F82

| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| boot-args | String | keepsyms=1 alcid=1 agdpmod=pikera | slide=1表示从第一组内存开始连续注入；darkwake=0代表一键唤醒机器并偏好设置中节能选项的小憩功能。如果你要用小憩功能请填8； -v是跑代码，在没装好稳定的黑果前我建议加上，方便定位错误，弄完后再删除-v |
| csr-active-config | Data | 00000000 | |
| prev-lang:kbd | Data | 7A682D48 616E733A 323532 | |
#### NVRAM--Block
禁用一些nvram变量，我们这里按默认设置不必理会。
#### NVRAM--LegacyEnable
如果你的主板不支持原生NVRAM，请一定要选择YES! 如果你的主板支持原生nvram的，填no。
#### NVRAM-LegacyOverwrite
对模拟nvram用户来说，将nvram.plist写入硬件，我认为不管是原生nvram还是模拟nvram，都选择no。
#### NVRAM-LegacySchema
nvram的变量设置
#### NVRAM-WriteFlash
如果你的主板bios因为nvram导入垃圾内容，请关闭它，一般都是选择no。
### PlatformInfo
机型信息
#### PlatformInfo--Automatic
这里意思是是否自动填写系统信息。因为后面的很多选项都好繁琐，我们只要认真填几个选项就行了，这里我选YES，不重要的信息让它自动填。
#### PlatformInfo--Generic
填写三码信息
#### PlatformInfo--UpdateDataHub
yes
#### PlatformInfo--UpdateNVRAM
yess
#### PlatformInfo--UpdateSMBIOS
yes
#### PlatformInfo--UpdateSMBIOSMode
Create
### UEFI
UEFI相关的设置。
#### UEFI--APFS
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| EnableJumpstart | Boolean | false | 从APFS容器中加载内置APFS驱动 |
| HideVerbose | Boolean | false | 是否隐藏啰嗦模式，一般我们需要看日志的时候才开启|
| JumpstartHotPlug | Boolean | false | 是否加载APFS格式的热插设备 |
| MinDate | Number | 0 | 加载最低发行的APFS格式 |
| MinVersion | Number | 0 | 加载最低版本的APFS格式。填0代表从HIGH SIERRA开始加载。|
#### UEFI--Audio
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| AudioSupport | Boolean | False | 是否开启黑苹果的开机提示音支持。如果你选择YES，后面的内容你必须认真看，不支持DP类的数字音频。|
| AudioCodec | Number | 0 | 填写音频声卡in节点 |
| AudioDevice | String  | PciRoot(0x0)/Pci(0x1b,0x0) | 填写你声卡的路径 |
| AudioOut | Number | 0 | 音频声卡out节点 |
| MinimumVolume | Number | 20 | 声音音量 |
| PlayChime | Boolean | false | 如果要使用开机duang声音 |
| VolumeAmplifier | Number | 0 | 按照默认设置 |
#### UEFI--ConnectDrivers
是否加载补丁，我们选择YES
#### UEFI--Drivers
根据用的驱动填写
#### UEFI--Input
默认即可，无需修改
#### UEFI--Output
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| ClearScreenOnModeSwitch | Boolean | false | 消除开机时从图形模式转换到文本时出现残影的问题 |
| ConsoleMode | String | | 这里填主机的输出方式,留空即可|
| DirectGopCacheMode | String | | 此项请留空！ |
| DirectGopRendering | Boolean | false | 是否使用内置显卡直接渲染开机画面，建议选择no|
| IgnoreTextInGraphics | Boolean | false | 修复在不使用-v跑马模式时候，开机日志导致的苹果logo显示不正确的问题|
| ProvideConsoleGop | Boolean | True | 调用显卡gop|
| ReconnectOnResChange | Boolean | false | 一些固件在 GOP 分辨率改变后会重新连接显示器才能输出|
| ReplaceTabWithSpace | Boolean | false | 一些固件在UEFI Shell下TAB功能键不生效。开启这个会用空格键代替。|
| Resolution | String | Max | 开机分辨率 |
| SanitiseClearScreen | Boolean | false | 修复4k及以上显示器的输出问题|
| TextRenderer | String | BuiltinGraphics | OC开机代码字体渲染方式，我这里填BuiltinGraphics|
#### UEFI--ProtocolOverrides
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| AppleAudio | Boolean | false | 是否有开机DUANG的声音 |
| AppleBootPolicy | Boolean | false | 虚拟机的mac需要用的 |
| AppleDebugLog | Boolean | false | 重新安装苹果错误日志界面|
| AppleEvent | Boolean | false | 虚拟机并具有vault的mac需要用的|
| AppleImageConversion | Boolean | false | 重建apple图标|
| AppleKeyMap | Boolean | false | 重建苹果功能键|
| AppleRtcRam | Boolean | false | 重装applertc协议|
| AppleSmcIo | Boolean | false | 代替之前的VirtualSMC.efi|
| AppleUserInterfaceTheme | Boolean | false | 重新安装 Apple User Interface Theme 协议|
| DataHub | Boolean | false | 重建datahub|
| DeviceProperties | Boolean | false | 虚拟机或者老款的电脑需要选择YES才能注入device property|
| FirmwareVolume | Boolean | false | VAULT相关项|
| HashServices | Boolean | false | VAULT相关项|
| OSInfo | Boolean | false | 通知主板以及一些程序关于MAC引导的信息|
| UnicodeCollation | Boolean | false | 旧的主板需要|
#### UEFI--Quirks
| 名称 | 类型 | 值 | 解释|
| :--- | :---- | :----- | :----- |
| ExitBootServicesDelay | Number | 0 | 旧主板需要给予主板退出时间|
| IgnoreInvalidFlexRatio | Boolean | false | 如果你没有在bios中解锁MSR0x194，一定要选YES|
| ReleaseUsbOwnership | Boolean | false | 自动释放USB所有权的功能 |
| RequestBootVarFallback | Boolean | false | 一些固件会主动扫描系统启动盘的位置而阻止OC扫描，如果碰到这样的问题选择YES，一般这个BUG在华硕的主板中比较常见。|
| RequestBootVarRouting | Boolean | true | 增加”启动磁盘” 的可靠性|
| UnblockFsConnect | Boolean | false | 惠普笔记本可能会让OC无法扫描到启动项，一般选择NO|
#### UEFI--ReservedMemory
空

## 挂载EFI分区
```
diskutil list //使用diskutil list 查看磁盘分区情况
diskutil mount disk1s1 //使用diskutil mount disk0s1 挂载disk1的EFI分区。
```

## Thank
* [使用OpenCore引导黑苹果 by XJN](https://blog.xjn819.com/?p=543)
* [精读OpenCore by 黑果小兵](https://blog.daliansky.net/OpenCore-BootLoader.html)