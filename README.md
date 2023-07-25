### 概述

这是和[FlashLightOS](https://gitee.com/redstoner-thirty-five/flash-light-os "固件地址")固件所搭配的Xtern Ripper V1.0的硬件平台。也是该固件的官方首发平台。该硬件平台一共有三块PCB,分别是功率板,MCU板和侧按PCB。

### 工程结构

该工程一共有四个文件夹，分别对应四套PCB。文件格式为Altium Designer各个PCB作用如下：

+ PCB-DiffTorch-PowerStage：这个文件夹包含完成降压恒流DCDC变换的功率级，电池功率计以及线性和PWM调光的电路,以及辅助电源的底板PCB。
+ PCB-DiffTorch-MCU：这个文件夹包含MCU相关的外围电路，以及按键去抖和配置EEPROM,包括typec console口和DFU功能的相关电路。
+ PCB-DiffTorch-SideKey：这个文件夹包含这套驱动配套使用的LED侧按按键板，通过细排线连接到驱动上层的子板。侧按板直径12mm，适配东东海D8B外壳。
+ PCB-DiffTorch-BattTypecCharger：这个文件夹包含给东东海D8B外壳配套的Typec充电节所准备的三锂电串联直充板子。此驱动并不依赖于此PCB。

### 焊接调试注意事项

这个工程包含120+元器件，三块PCB，而且有非常小且高引脚密度的QFN和SON-8芯片（最小封装2x2mm），大量的SOT323和0402原件，因此对焊接者的手工有非常高的要求。焊接工具方面需要的准备如下：

+ 加热台
+ 趁手的大功率恒温刀头烙铁（例如JBC245）
+ 高质量的焊锡
+ 高性能的恒温旋风热风枪
+ BGA焊接用的那种高品质助焊剂
+ 63/37配方的有铅中温锡膏
+ 高质量的镊子
+ 吸锡带（用来处理typec座子和QFN连锡）
+ 万用表

#### MCU上层子板的调试焊接

首先应该焊接的是MCU子板，这块板子会比较难焊接，我建议的焊接顺序如下：

1. 使用有铅锡膏和加热台贴装正面的QFN-32封装的HT32F52352单片机
2. 在加热台上面趁热用烙铁给PCB所有焊盘上锡。然后风枪焊接PCB正面所有的芯片（一共2个SOT32*，一个SOT23-3的LDO，一个MSOP-10的USBMUX）
3. 焊接PCB正面的两个二极管
4. 焊接PCB正面所有的0402和0603封装的电阻电容
5. 使用锡膏+风枪贴装背面的UDFN-8封装的FM24C512D EEPROM（此芯片体积极小，容易连锡，先贴装会比较好处理一些）
6. 用烙铁给背面所有贴片焊盘上锡，贴装背面的MSOP-10封装的CH340E
7. 贴装背面所有的0402封装的电阻电容
8. 贴装背面的3225晶振,二极管和SOT323的2N7002KW

全部贴装完毕之后，就需要利用万用表的二极管档进行按照下图指示的位置（红表笔接如图红框位置的焊盘，黑表笔接黑框位置）进行测试，此时万用表的读数应该在0.4-0.7V之间。如果是0或者很低，则说明有芯片方向反了或者HT32F52352焊接出问题连锡短路，需要处理干净。确保没有短路之后就可以准备开始贴装typec座子。这个座子因为是立贴的，贴装会较为困难，具体步骤如下：

![测试点](img/4.jpg)

1. 用镊子在PCB的Type-c座子上均匀的涂抹薄薄的一层锡膏。
2. 将座子放上去，风枪开320度，风速5-6级，围着座子均匀的打圈圈直到锡膏融化。
3. 熔化后趁热将座子拿下，用烙铁和吸锡带去除掉座子引脚和PCB焊盘上连一起的焊锡。
4. 在PCB上薄薄涂抹一层焊油，将座子放回去
5. 风枪330度风速5-6级围着座子均匀打圈直到焊锡熔化，座子会微微下沉和PCB贴合。
6. 撤走风枪等待冷却。
7. 用烙铁+焊锡将四个固定脚焊接到PCB上。

焊接完毕的成品的正面和背面如下所示：

![成品](img/1.jpg)

焊接完毕后就可以把板子丢进洗板水里面清理干净，下一步则是连接电脑刷机。你需要准备一条typec线（A to C和C to C都支持）然后去合泰半导体官网下载[HT32 Flash Programmer](https://www.holtek.com.cn/documents/10179/6393521/HT32_Flash_Programmer_v109d.exe "HT32 Programmer")然后用镊子短接正面如图所示的两个触点，短接之后插上USB线。此时你应该听到USB设备插入的声音。如果没有声音或者提示未知的USB设备，那你就需要检查你MCU、typec座子、正面那个MSOP-10的USB MUX以及电源部分的SOT23封装的LDO是不是虚焊了。

![正面触点](img/5.jpg)

然后下一步就是打开HT32 Flash Programmer软件，将portname调为USB，接着点击右下角的connect。

![编程中](img/6.jpg)

如果焊接没有问题，软件会提示connected，然后你就需要在Code页面选择随工程一起附带的[debug版本固件](/PCB-DiffTorch-MCU/Firmware/FlashLightOS-Debug-SBT90G2.hex "debug固件")。下面的flash option里面勾选上Erase、Program、Verify，然后点击Programming。此时程序会自动给您的驱动板刷入固件。当程序提示operation success之后，你就可以将小板拔下并重新连接。此时你应该听到设备接入的声音。然后你要干的就是去设备管理器里面看看有没有COM和LPT的子项出来。如果固件正常运行，则会有相应的子项，在我这台电脑上，分配的端口为COM7。

![编程成功](img/7.jpg)
![设备管理器里面的子项](img/8.jpg)

接着就可以打开putty软件，配置好参数然后连接驱动，输入ver然后回车。如果固件正确运行应有如下输出：

![编程成功](img/9.jpg)

如果有输出则说明固件已经成功启动。此时你需要把刷固件的步骤重复一次，为子板刷上随工程一起附带的[Production版本固件](/PCB-DiffTorch-MCU/Firmware/FlashLightOS-Production-SBT90G2.hex "production固件")这一次的话Code页面需要选择附带的production版本固件。刷写完毕后MCU上层板的焊接就完成了。

#### 下层功率板的焊接

----------------------------------------------------------------------------------------------------------------------------------
© redstoner_35 @ 35's Embedded Systems Inc.  2023