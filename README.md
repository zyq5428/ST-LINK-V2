# ST-LINK-V2
依据参考资料制作ST-LINK-V2，并实践一些Allegro技巧。

## 初衷

* 自己做一些ST-LINK便于后期自己调试STM32芯片
* 扩展一些UART，I2C，GPIO接口出来，其他时候作为一个STM32F103开发板用
* 重新熟悉下Allegro软件，并实践一些新版本的技巧
* 自己根据需要，亲自做里面用到的所有器件的封装和3D

## 产品效果图

* 正面

![TOP.png](./assets/TOP.png)

* 背面

![BOTTOM.png](./assets/BOTTOM.png)

* 为了方便以后可能用加热台自己贴片，所以尽量将元器件布置在同一面
* 为了最大化降低成本，采用双层板设计，但依旧尽力保证设计规范和美观

## 回板焊接

![焊接成品.jpg](./assets/焊接成品.jpg)

## 原理介绍

借鉴于：https://oshwhub.com/CYIIOT/ST_LINK-V2_1

### 简介

* ST-Link的硬件，官方推出了三大版本：V1、V2和V3。在官方《TN1235 ST-LINK衍生产品概述》中有详细的说明。
* ST-Link/V2：支持STM32和STM8调试，不带虚拟串口，TB上卖的大多是这种，目前手头还有好几个这个版本的ST-Link。后面会使用这个版本进行烧录。
* ST-Link/V2-1：支持STM32调试，带虚拟串口和虚拟U盘下载，目前ST官方的Nucleo系列评估板上面板载的ST-Link就是这个版本。
* 潘多拉开发板上的ST-link V2-1 出厂主控使用的是FLASH 容量为64K的STM32F103C8T6，但是ST-link V2-1最新的固件已经超过了64K，芯片容量不足。 因此本项目主控选择的是FLASH容量为128KB的STM32F103CBT6（商品编号：C8304），这是C8T6的大容量版本，可以直接PIN to PIN 替换。

### COM指示灯

* 在ST官方的ST-Link V2.1图纸中有一个名为COM的指示灯,这是一个红绿双色的LED指示灯，就是下图所示的这个LED。

![COM指示灯.png](./assets/COM指示灯.png)

* 那么这个指示灯有什么作用呢，在官方的TN1235技术手册中有专门的介绍，截取相关部分如下：

![COM_LED.png](./assets/COM_LED.png)

* 自ST-LINK/V2起，所有ST-LINK板均带有一个标有“COM”的LED（在外壳或PCB上）。无论连接类型如何，此LED都会显示ST-LINK状态：
    * LED呈红色闪烁：正在与PC进行第一个USB枚举。
    * LED为红色：PC与ST-LINK之间的通信已建立（枚举结束）。
    * LED呈绿色和红色交替闪烁：正在目标和PC之间交换数据。
    * LED为绿色：上一次通信已成功。
    * LED为橙色：与目标的ST-LINK通信失败

## 烧录固件

### 固件版本

* 由于官方没有将ST-Link里面源码公开，同时也没有直接给出ST-Link固件（读保护），但是目前（2020年7月）网上已有流传多个版本的固件。
    * 版本一：STLink V2.J16.S4版本固件：标准V2版本，支持SWD和SWIM接口，这个版本的固件是。
    * 版本二：STLink V2.J28.M18版本固件：是用于ST-LINK/V2-1、ST-LINK/V2-A、ST-LINK/V2-B板(具有STM32调试接口、大容量存储接口、虚拟COM端口)的版本。
    * 本项目制作ST-Link V2-1 必须使用V2.J28.M18这个版本作为烧录的固件版本。

### 烧录工具

* 第一次烧录固件可使用这两个工具：
    * STM32CubeProg
    * STM32 ST-LINK Utility
* 本次烧录使用STM32 ST-LINK Utility，STM32 ST-LINK Utility的功能比STM32CubeProg要稍微简单一些，其主要功能也是编程（下载）。烧录截图如下，操作很简单，大概看一下就知道了:

![烧录固件.png](./assets/烧录固件.png)

### 升级ST-LINK

* 固件更新有三种方法：
    * 官方固件升级应用程序ST-LinkUpgrade
    * 使用STM32CubeProg或者STM32 ST-LINK utility自带的升级工具升级
    * 使用Keil MDK-ARM 内置的升级工具进行升级，当ST-link 的版本低于MDK内置的版本时，会提示进行升级。Keil MDK-ARM v5.31内置固件升级版本与STM32CubeProg V2.4.0 内置固件升级版本是一样的。
* 本次升级使用ST-LinkUpgrade，升级截图如下：

![升级固件.png](./assets/升级固件.png)

* 至此，ST-LINK的制作就完成了。

## 注意事项

* ST-LINK 固件升级工具不知从哪个版本开始，不支持跨版本更新固件。
* ST-LINK 最近的几个版本的固件已加入了读保护，SWD口是锁上的，所以不能通过SWD口读写固件。
    * 如果想改为其他的，我们可以用STM32 ST-LINK utility将写保护的级别修改，然后全部擦除，就可以重新烧录了。
    
![写保护级别.png](./assets/写保护级别.png)

* 如果想改成DAP-LINK 之类的固件，不能通过SWD口烧录固件，但是可以通过ISP的方式擦除STM32的固件信息，然后就可以使用SWD口正常读写。
    * USB转串口连接线烧写（又称ISP烧写，且使用的串口必须是串口1）:
        * Step1:将BOOT0设置为1，BOOT1设置为0，然后按下复位键，这样才能从系统存储器启动BootLoader;
        * Step2:最后在BootLoader的帮助下，通过串口下载程序到Flash中;
        * Step3:程序下载完成后，必须要将BOOT0设置为GND，手动复位，这样，STM32才可以从Flash中启动。




