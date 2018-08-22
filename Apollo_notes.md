## 1 编译ESD CAN驱动源代码 
参考资料点[这里](https://github.com/ApolloAuto/apollo-kernel/blob/master/linux/ESDCAN-README.md)。

预编译的Apollo内核镜像不包含CAN卡驱动，安装预编译好的内核后可以按照下面的方式进行CAN卡驱动的编译：
1. 准备ESD CAN驱动的Linux代码包
2. 解压代码包，进入代码包的目录，进行以下操作：

```
cd src/; make -C /lib/modules/`uname -r`/build M=`pwd`
sudo make -C /lib/modules/`uname -r`/build M=`pwd` modules_install
```
3. esdcan-pcie402.ko就是新编译好的CAN卡驱动，安装在`/lib/modules/$(uname -r)/extra/`目录下
4. 重启IPC
5. 通过以下命令创建CAN设备节点

```
cd /dev; sudo mknod --mode=a+rw can0 c 52 0
```
6. 解压后的ESD CAN代码包中有测试程序，用其测试CAN卡驱动是否安装成功。这里注意，测试时可能会有如下提示

```
./cantest: error while loading shared libraries: libntcan.so.4: cannot open shared object file: No such file or directory
```
此时，将CAN卡代码包中的libntcan.so.4.0.1改名为libntcan.so.4，然后复制到`/usr/lib`目录下。或者将libntcan.so.4的路径加入环境变量中。

7. 测试。运行`sudo ./cantest`，出现以下信息

```
CAN Test Rev 2.12.6  -- (c) 1997-2015 esd electronic system design gmbh

Available CAN-Devices: 
Net   0: ID=CAN_PCIE402 (4 ports) Serial no.: GK001271
         Versions (hex): Lib=4.0.01 Drv=3.A.04 HW=1.0.16 FW=0.0.44 (0.0.00)
         Baudrate=7fffffff (Not set) Status=0000 Features=00000ffa
         Ctrl=esd Advanced CAN Core @ 80 MHz (Error Active / REC:0 / TEC:0)
         Transceiver=TI SN65HVD265
         TimestampFreq=80.000000 MHz Timestamp=0000002312FE7232

Syntax: cantest test-Nr [net id-1st id-last count
        txbuf rxbuf txtout rxtout baud testcount data0 data1 ...]
Test   0:  canSend()
Test  20:  canSendT()
Test  50:  canSend() with incrementing ids
Test   1:  canWrite()
Test  21:  canWriteT()
Test  51:  canWrite() with incrementing ids
Test   2:  canTake()
Test  12:  canTake() with time-measurement for 10000 can-frames
Test  22:  canTakeT()
Test  32:  canTake() in Object-Mode
Test  42:  canTakeT() in Object-Mode
Test   3:  canRead()
Test  13:  canRead() with time-measurement for 10000 can-frames
Test  23:  canReadT()
Test   4:  canReadEvent()
Test  64:  Retrieve bus statistics (every tx timeout)
Test  74:  Reset bus statistics
Test  84:  Retrieve bitrate details (every tx timeout)
Test   5:  canSendEvent()
Test   8:  Create auto RTR object
Test   9:  Wait for RTR reply
Test  19:  Wait for RTR reply without text-output
Test 100:  Object Scheduling test
Test  -2:  Overview without syntax help
Test  -3:  Overview without syntax help but with feature flags details
```
