# ART-Pi智能物联网小车

## 简介

这个扩展板兼容支持ART-Pi开发板和ESP32-DevkitC开发板！
首先它是一台小车的标准配置：引出了，四驱电机+超声波+舵机+两路巡线，以及OV2640摄像头接口！

再讲它的扩展功能：扩展E53标准接口，接口包括SPI，IIC，UART，以及7个IO口！

另外，接入了太阳能充电电路！

畅想一下：标准的ART-PI小车，E53我如果接入温湿度模块，空气检测模块，那它就是一辆物联网小车；

如果接入有害气体检测模块，那它就是一辆有害气体检测小车，可以进入无人环境检测有害气体；

如果接入红外装置，它还能是一辆搜救小车，帮助消防员进入恶劣环境探测是否有人体存在；

它还有太阳能充电模块，接上几块太阳能板，它就是永不断电的太阳能车车了；

当然你也可以不接入小车底盘，它还是一块ART-PI物联网开发板，接入不同的E53模块实现不同的应用开发场景；

发挥你的想象力吧！实现它的无限可能......

首个E53标准外接板已完成，接入：红外+人体+温湿度+光线+RGB灯。

红外可以用来控制小车，也可以接制RGB灯，其它感应器用来实现物联网案例！

硬件开源地址：https://oshwhub.com/520world/zhi-neng-xiao-che

## 硬件说明

车底盘和电机为淘宝购买，主板支持四电机，所以两电机或四电机的单层底盘都可以，我使用的是两电机底盘，电机为T1直流电机。

ART-Pi与扩展板之间是排针连接，扩展板与小车底盘间用铜柱连接！


## 软件开发

### 1.首先安装好RT-Thread Studio,按照如下文章写入示例程序,
https://blog.csdn.net/toopoo
https://blog.csdn.net/toopoo/article/details/113487496

这里有个坑,下载程序的时候发现我的STLink驱动有问题,在设备中显示问号!
网上找了一圈驱动,结果还是要ST官网下载的驱动是可以用的,官网填下Email地址,发到你Email里下载的!

下载完成,按下user按键,蓝色的灯切换开关状态!

rt_pin_mode()	设置引脚模式
rt_pin_write()	设置引脚电平
rt_pin_read()	读取引脚电平
rt_pin_attach_irq()	绑定引脚中断回调函数
rt_pin_irq_enable()	使能引脚中断
rt_pin_detach_irq()	脱离引脚中断回调函数
GET_PIN(port, pin)	获取引脚编号（RT-Thread 提供的引脚编号需要和芯片的引脚号区分开来）

### 2.线程:
https://blog.csdn.net/toopoo/article/details/113604115
下载例程thread,运行成功,红蓝灯闪

### 3.照猫画虎造轮子,新手第一跑!
轮子跑起来首先只用到GPIO的输出,所以基于blink_led来写:
新建RT-Thread项目--name:wheel--基于开发板Art-PI--示例工程--blink_led 
双击 RT-Thread Settings--点亮ulog日志--双击进入配置,使能ISR日志(我也不知道这个干嘛的)--保存
打开 applications--main.c  改代码:现在买的小车底盘只有两个电机,所以使用电机1和电机3输出
电机1,3端口配置:
	电机1----PB12
	电机1----PB13
	电机3----PB0
	电机3----PB1
define MOTO_1_1_PIN GET_PIN(B, 12)
define MOTO_1_2_PIN GET_PIN(B, 13)
define MOTO_2_1_PIN GET_PIN(B, 0)
define MOTO_2_2_PIN GET_PIN(B, 1)

main函数修改
rt_pin_mode(MOTO_1_1_PIN, PIN_MODE_OUTPUT);
rt_pin_mode(MOTO_1_2_PIN, PIN_MODE_OUTPUT);
rt_pin_mode(MOTO_2_1_PIN, PIN_MODE_OUTPUT);
rt_pin_mode(MOTO_2_2_PIN, PIN_MODE_OUTPUT);

主循环
rt_thread_mdelay(1000);
rt_pin_write(MOTO_1_1_PIN, PIN_HIGH);
rt_pin_write(MOTO_1_2_PIN, PIN_LOW);
rt_pin_write(MOTO_2_1_PIN, PIN_HIGH);
rt_pin_write(MOTO_2_2_PIN, PIN_LOW);            //前进5秒
rt_thread_mdelay(5000);
rt_pin_write(MOTO_1_1_PIN, PIN_LOW);
rt_pin_write(MOTO_1_2_PIN, PIN_LOW);
rt_pin_write(MOTO_2_1_PIN, PIN_LOW);
rt_pin_write(MOTO_2_2_PIN, PIN_LOW);            //停止1秒
rt_thread_mdelay(1000);
rt_pin_write(MOTO_1_1_PIN, PIN_LOW);
rt_pin_write(MOTO_1_2_PIN, PIN_HIGH);
rt_pin_write(MOTO_2_1_PIN, PIN_LOW);
rt_pin_write(MOTO_2_2_PIN, PIN_HIGH);            //后退5秒
rt_thread_mdelay(5000);
rt_pin_write(MOTO_1_1_PIN, PIN_LOW);
rt_pin_write(MOTO_1_2_PIN, PIN_LOW);
rt_pin_write(MOTO_2_1_PIN, PIN_LOW);
rt_pin_write(MOTO_2_2_PIN, PIN_LOW);            //停止1秒


构建和下载程序,接上电机.测试一下,轮子前后转动正常,通过!

### 4.新手第二跑:读取sht30温湿度
新建RT-Thread项目--name:sht30--基于开发板Art-PI--示例工程--基于Wifi
（基于Wifi示例的目的是让程序自带Wifi功能后面我要用到，我试过自己加Wifi功能有问题！）
双击 RT-Thread Settings--点亮ulog日志--双击进入配置,使能ISR日志(我也不知道这个干嘛的)--保存
参考大佬的文章,首先添加sht3x软件包,使能I2C组件
双击 RT-Thread Settings--软件包中心,立即添加--外设--sht3x添加
双击sht3x配置--version我选了latest--硬件:硬件上使能I2C1，并且配置引脚，这里保持默认配置不改动scl-22,sda-23--组件,设备驱动程序,看到已经默认使能了I2C
保存,rtconfig.h中查看已经引入了sht3x包,但是sda,scl引脚编号有问题,这里显示是22-PB6,23-PB7,
I2C   sht30+bh1750		SDA-PH12 SCL-PH11
参考:https://blog.csdn.net/qq_42913442/article/details/110912956
rtt-studio打开libraries/drivers/include/drv_gpio.c 找PH11、PH12对应的引脚号,跟文章不同这里并没有看到定义

我用的应该是I2C3 123-PH11-SCL 124-PH12-SDA
修改硬件上的配置,使能I2C3--保存
再查看rtconfig.h,这次对了,
/* Notice: PH12 --> 124; PH11 --> 123 */

define BSP_I2C3_SCL_PIN 123
define BSP_I2C3_SDA_PIN 124

先不写代码,构建,下载后用终端测试一下:这里我用putty 115200(打开RT-Thread自带的终端有问题,老是断开)
list_device命令先看一下I2C
输入help命令发现有新增命令sht3x

sht3x终端命令如下:
sht3x probe <i2c_dev_name> <pu/pd>  --挂载SHT3x设备，需要指定i2c设备名称和上下拉方式，默认下拉
sht3x read --阅读SHT3x温湿度
sht3x status --读取查看状态寄存器值
sht3x reset --软件复位SHT3x
sht3x heater <on/off> --开/关heater
msh >sht3x
msh >sht3x probe i2c3
msh >sht3x heater on
msh >sht3x status
msh >sht3x read
读取温湿度正常,调用sht3x成功!!下面写代码:
applications下新建文件app_sht3x.c
参考大佬文章

复制大佬的代码:https://blog.csdn.net/qq_38113006/article/details/105349975
复制大佬的代码:https://blog.csdn.net/mculover666/article/details/104153715?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-2&spm=1001.2101.3001.4242
最终代码见:app_sht3x.c

下载后发现线程直接是启动状态的.成功!

### 5.连接MQTT服务器：
新建mqtt.c抄代码：https://blog.csdn.net/qq_38113006/article/details/105935328
https://blog.csdn.net/zz100861122/article/details/111902798?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-2&spm=1001.2101.3001.4242
cJson使用：https://blog.csdn.net/qq_27508477/article/details/107323765

双击 RT-Thread Settings--+Add--pahomqtt和cJson，由于调试出错，一直以为pahomqtt不行，最后用了kawaii

~~!!!!!!!!!!!!!调试中怎么会有这种错误，调试选项-download选项卡里面外部flash算法莫名没有了，一直都不下载我都不知道！~~

用kawaii mqtt里面的test.c代码改，很容易就成功了，MQTT OK了，下面开始做线程间消息队列！

### 6.线程间消息传递

## 运行

### 编译&下载

编译完成后，将开发板的 ST-Link USB 口与 PC 机连接，然后将固件下载至开发板。

### 运行效果



## 注意事项

1. AP6212 正常运行依赖 WIFI 固件，如果固件丢失，参考[WIFI固件下载手册](https://github.com/RT-Thread-Studio/sdk-bsp-stm32h750-realthread-artpi/blob/master/documents/UM5003-RT-Thread%20ART-Pi%20BT_WIFI%20%E6%A8%A1%E5%9D%97%E5%9B%BA%E4%BB%B6%E4%B8%8B%E8%BD%BD%E6%89%8B%E5%86%8C.md)。

