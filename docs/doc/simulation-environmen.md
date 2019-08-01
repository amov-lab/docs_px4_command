# 仿真平台环境搭建

### 目录

-  第一章 概述（描述我们的这个仿真环境，演示结果描述）
-  第二章 搭建过程
    - 第一节 硬件准备
          1、台式电脑一台
          2、显示器两台
    - 第二节 软件配置
          1、Ubuntu16.04操作系统
          2、PX4环境安装（自动安装gazebo9）
          3、ROS（kinetic版本）的安装（自动安装gazebo7）
          4、mavlink与mavros的安装
          5、QGroundControl下载
    - 第三节 仿真过程
          1、注册阿木社区的gitlab账户
          2、下载源码并建立工作区间
          3、更新环境变量
          4、启动仿真
    - 第四节 仿真源码说明
          1、sitl_gazebo_iris.sh正常offboard飞行
          2、sitl_gazebo_square.sh飞正方形
          3、sitl_gazebo_formation.sh多机编队仿真

### 第一章 概述

PX4提供了一种全自主飞行控制方式，offboard模式。而阿木社区具有一套较完整，可靠的系统体系。阿木社区的学员们在购买我们的飞机回去之后发现对我们的系统体系还不是很了解，导致操作不当还是会有很多炸机现象，为了让学员们更加了解系统体系的种种情况，于是乎出现了仿真系统平台下对阿木系统体系的模拟。有了这一套仿真系统，学员可以更加理解阿木系统体系中的逻辑，熟悉这一套系统体系，结合仿真减少实际飞行中炸机的发生。
本篇文章中，会讲解如何用模拟器来按照我们的系统体系控制无人机飞行。首先，先从搭建环境开始讲。

### 第二章 搭建环境

本套仿真平台在Ubuntu16.04（16.04.6）LTS，ROS-Kinetic（v1.12.14），Firmware（1.8.2），QGroundControl-v3.3.2，交叉编译工具链为gcc-arm-none-eabi-5_4-2016q2，mavros以及mavlink的二进制安装下，测试通过。

#### 第一节、硬件准备
- 1、台式电脑一台
- 2、显示器两台

笔者刚开学习的时候，用的是虚拟机vmware，但是发现用虚拟机的话，不能进行仿真，查阅大量资料之后发现，多数由于Java原因，也有显卡问题。我们都知道，vm里面是虚拟显卡，而且安装Ubuntu本身自带的nouveau显卡性能本身就不行，满足不了仿真的需求。
笔者的第一个台式电脑是学校标配的电脑，显卡记得好像是630还是730来着，在种种环境安装配置好之后，正常的mavros，gazebo仿真没有很多问题，但是如果要做视觉导航部分的仿真，比较吃力。
笔者现在电脑的配置为GTX2060，AMD Ryzen 5 3600 6-Core Processor × 12，16G的内存。目前笔者还没有尝试视觉导航部分的仿真，但这么高的配置，应是没有什么问题。
再次建议各位，尽量用一台比较高配的台式电脑，显示器数量个人看个人情况。笔者建议显卡至少GTX1060，内存大小16G，CPU6核12线程。

#### 第二节、软件配置

__1、Ubuntu16.04操作系统__

用UltraISO制作U盘启动盘，步骤如下：
- 1）、打开UltraISO软件，打开文件，选择要安装的Ubuntu版本，笔者安装的是Ubuntu-16.04.6，偶数版本的都是长期支持版的系统镜像，至少到2021年之前是一直被维护的。
- 2）、接着我们点击启动->写入磁盘映像，进入下面的界面，在制作Ubuntu的U盘启动盘的时候，要选择RAW。笔者之前也尝试过其他的写入方式，有时候会成功，有时候会失败。但自从选择了RAW之后，装的好几次都是成功了的，所以笔者建议用RAW写入方式。
- 3）、之后就是把U盘格式化，然后点击写入即可，等待......（长短取决于电脑性能），待完成之后U盘启动盘就制作成功了。
   
安装Ubuntu系统
- 插上U盘，开机启动选项中设置U盘启动即可，一般不同的电脑进入BIOS的方式不同。之后的安装很简单，百度上有很多教程，按照教程安装即可完成安装。笔者直接安装的是Ubuntu系统，所以没有什么分区设置。

Ubuntu系统的一点设置
- 笔者在装完Ubuntu后发现分辨率很低，看的很难受。查看了一下，分辨率只有640x480，然而我的屏幕是1920x1080的。然后通过改xrandr和cvt都无效。最后笔者找到解决方案，修改grub默认的分辨率，具体过程如下：
sudo gedit /etc/default/grub
找到: #GRUB_GDXMODE=640x480
改为: GRUB_GDXMODE=1920x1080
然后更新一下grub: sudo update-grub
最后重启电脑即可
- 笔者还建议在安装完系统之后，只留下现在所使用的版本的内核，删除其余多余的内核，并且禁用内核的更新，否则过段时间，系统默认启动更新后的内核。
- 安装显卡驱动，Ubuntu默认的显卡驱动是nouveau，你需要安装与你显卡相匹配的驱动程序。以NVIDIA驱动为例，首先是查看自己显卡，发现是设备ID为1f08，通过 https://devicehunt.com/view/type/pci/vendor/10DE/device/1F08 搜索发现该驱动是GTX2060，然后我们到 https://www.nvidia.com/Download/index.aspx?lang=cn 下载相应的驱动安装程序。安装的过程你可以参考这篇文档 https://zhuanlan.zhihu.com/p/31575356

      amov@amov:~$ lspci | grep VGA
      0a:00.0 VGA compatible controller: NVIDIA Corporation Device 1f08 (rev a1)


__2、PX4环境安装__
参考官方文档
https://dev.px4.io/master/en/setup/dev_env_linux_ubuntu.html
笔者在安装完Ubuntu系统的第一件事情就是用户组的添加

      sudo usermod -a -G dialout $USER

然后按照官网教程，~/下新建一个文件，重命名为ubuntu_sim.sh。在官网打开ubuntu_sim.sh脚本，然后全部复制拷贝到新建的脚本中，接着给脚本可执行权限。最后执行这个脚本。

      sudo gedit ubuntu_sim.sh
      chmod +x ubuntu_sim.sh
      sudo ./ubuntu_sim.sh
这个安装的快慢与你的网速有关。这个脚本本身是没有安装交叉编译工具链的。交叉编译工具链需要手动安装，接下来是手动安装交叉编译工具链：
通过 https://bigsearcher.com/mirrors/gcc/releases/ 下载你所需要的gcc版本。
下载之后解压并放到/opt/之下，如下所示为笔者gcc的路径

      amov@amov:/opt/gcc-arm-none-eabi-5_4-2016q2/bin$ pwd
      /opt/gcc-arm-none-eabi-5_4-2016q2/bin
然后打开/etc/profile文件，如下

      amov@amov:/opt/gcc-arm-none-eabi-5_4-2016q2/bin$ sudo gedit /etc/profile
在最下面添加一行

      export PATH=$PATH:/opt/gcc-arm-none-eabi-5_4-2016q2/bin
路径就是gcc存放的路径。接着source一下刚才修改的/etc/profile

      source /etc/profile
测试安装gcc是否成功，输入

      arm-none-eabi-gcc --version
若出现如下类似，说明安装成功

      amov@amov:~$ arm-none-eabi-gcc --version
      arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.4.1 20160609 (release) [ARM/embedded-5-branch revision 237715]
      Copyright (C) 2015 Free Software Foundation, Inc.
      This is free software; see the source for copying conditions.  There is NO
      warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
若输出是：

      arm-none-eabi-gcc --version
      arm-none-eabi-gcc: No such file or directory
需要安装32位支持库（ https://px4.osdrone.net/1_Getting-Started/adcanced_linux.html）

      sudo apt-get install libc6:i386 libgcc1:i386 libstdc++5:i386 libstdc++6:i386

现在PX4环境配置已经完成，之前在运行ubuntu_sim.sh脚本中下载过Firmware，建议重新下载一个PX4固件。

      amov@amov:~/Desktop/px4-src/src-1.8.2$ ls
      amov@amov:~/Desktop/px4-src/src-1.8.2$ git clone https://github.com/PX4/Firmware.git
      Cloning into 'Firmware'...
      remote: Enumerating objects: 278734, done.
下载完之后，我们进入到Firmware中，还需要更新子模块

      amov@amov:~/Desktop/px4-src/src-1.8.2$ cd Firmware/
      amov@amov:~/Desktop/px4-src/src-1.8.2/Firmware$ git submodule update --init --recursive
漫长等待之后，你就可以编译源码了，先试试最基本的能力。
首先是编译源代码

      amov@amov:~/Desktop/px4-src/src-1.8.2/Firmware$ make px4_fmu-v5_default 
若编译成功的话，再执行编译最基本的gazebo仿真

      amov@amov:~/Desktop/px4-src/src-1.8.2/Firmware$ make px4_sitl_default gazebo

到此为止，说明你的PX4环境配置已经搭建完成了。下来我们会配置与Ubuntu16.04系统对应的ROS Kinetic版本。

__3、ROS-Kinetic安装__

ROS-Kinetic的安装参考 http://wiki.ros.org/kinetic/Installation/Ubuntu 
需要注意的一点是，一般笔者在安装ROS时候，选择镜像是中科大的源或者是清华的源，其他就是按照官网提示一步步安装即可。

安装ROS（大概有700~800MB）完成之后，查看是否安装成功，如下表示安装ROS完成。

      amov@amov:~$ roscore 
      ... logging to /home/amov/.ros/log/d98e04fe-b1ca-11e9-bf5f-e0d55ee7d1ba/roslaunch-amov-23391.log
      Checking log directory for disk usage. This may take awhile.
      Press Ctrl-C to interrupt
      Done checking log file disk usage. Usage is <1GB.

      started roslaunch server http://amov:39279/
      ros_comm version 1.12.14


      SUMMARY
      ========

      PARAMETERS
       * /rosdistro: kinetic
       * /rosversion: 1.12.14

      NODES

      auto-starting new master
      process[master]: started with pid [23401]
      ROS_MASTER_URI=http://amov:11311/

      setting /run_id to d98e04fe-b1ca-11e9-bf5f-e0d55ee7d1ba
      process[rosout-1]: started with pid [23414]
      started core service [/rosout]

__4、mavlink与mavros安装__

mavlink与mavros的安装参考 https://github.com/mavlink/mavros/blob/master/mavros/README.md#installation 

按照教程安装应该没有什么问题的。

__5、下载QGroundControl__

笔者的qgc版本是v3.3.2，是通过Qt5.11.0编译生成的。建议直接下载可执行程序，可参考开发者手册 https://docs.qgroundcontrol.com/en/getting_started/download_and_install.html 


#### 第三节、仿真过程

上节中，我们已经搭建好PX4仿真的环境了，而本节旨在下载阿木社区的源码，并且建立新的工作空间到个人工作路径下，然后配置仿真所使用的固件版本的选择以及环境配置，最后进行仿真操作。先从如何下载阿木社区源码说起

__1、注册阿木社区的gitlab账户__

上网登录 http://gitlab.amovauto.com 阿木社区的gitlab。未注册过的，请先注册。登录之后，等待管理员授予权限，可进行相应的可读，可下载权限，进行下一步操作。

__2、下载源码并建立工作区间__

详细的建立工作空间请查看阿木社区gitlab上的项目 __px4_commander__ 。或者如下链接：http://gitlab.amovauto.com/amovlab/px4_command 

建立好工作空间之后，笔者的工作空间如下：

      amov@amov:~/AMOV_WorkSpace$ cd px4_ws/
      amov@amov:~/AMOV_WorkSpace/px4_ws$ ls
      build  devel  src
      amov@amov:~/AMOV_WorkSpace/px4_ws$ cd devel/
      amov@amov:~/AMOV_WorkSpace/px4_ws/devel$ ls
      cmake.lock  lib               local_setup.zsh  _setup_util.py
      env.sh      local_setup.bash  setup.bash       setup.zsh
      include     local_setup.sh    setup.sh         share
      amov@amov:~/AMOV_WorkSpace/px4_ws/devel$ 
打开.bashrc 文件

      amov@amov:~/AMOV_WorkSpace/px4_ws/devel$ sudo gedit ~/.bashrc 
需要在.bashrc 文件最后添加一行如下：

      source ~/AMOV_WorkSpace/px4_ws/devel/setup.bash

__3、添加环境变量__
.bashrc 文件添加如下：

    source ~/Desktop/px4-src/src-1.8.2/Firmware/Tools/setup_gazebo.bash ~/Desktop/px4-src/src-1.8.2/Firmware/ ~/Desktop/px4-src/src-1.8.2/Firmware/build/px4_sitl_default
    export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/Desktop/px4-src/src-1.8.2/Firmware
    export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/Desktop/px4-src/src-1.8.2/Firmware/Tools/sitl_gazebo


__4、启动仿真__

进入工作区间仿真部分目录下，可以看到有6个脚本文件

      amov@amov:~/AMOV_WorkSpace/px4_ws/src/px4_command/sh/sh_for_simulation$ ls
      sitl_gazebo_formation.sh       sitl_gazebo_square.sh
      sitl_gazebo_iris.sh            sitl_jMAVSim_pos_controller.sh
      sitl_gazebo_pos_controller.sh  sitl_test.sh

启动sitl_gazebo_iris.sh脚本,执行如下

      amov@amov:~/AMOV_WorkSpace/px4_ws/src/px4_command/sh/sh_for_simulation$ ./sitl_gazebo_iris.sh  
即可进入仿真界面。

#### 第四节、仿真脚本说明

__1、sitl_gazebo_iris.sh正常offboard飞行__

正常启动sitl_gazebo_iris.sh腳本，基本操作流程和实体飞机操作流程一致。
先起飞3m,如下图:
![iris_takeoff](https://i.loli.net/2019/08/01/5d4293c857c3095302.png)
接着,我们在Move_Body坐标系下,x,y,z分别为1,1,0.飞行轨迹如下图:
![iris_MoveBody_110](https://i.loli.net/2019/08/01/5d4294fb6fe2f91490.png)
最后我们执行一下land模式,如下图:
![iris_land](https://i.loli.net/2019/08/01/5d429671d1af767648.png)

存在Bug描述：
（1）、若起飞之后飞机降落至地面，无法进行再次起飞。（和实体飞机一致现象）
（2）、在ENU坐标系下，若使用速度控制，进行起飞2M，飞机一致向上飞，不会停止，在gazebo中，飞至26M，切换至悬停模式，无法成功相应，飞至30M，切换至land，正常降落。
（3）、经过多次测试，move节点中，按键4hold模式无响应，在两种坐标系下的速度控制中，飞机一直向上飞。
（4）、在passivity控制率下，正常设置起飞3M，飞机纯粹油门量最大向上直飞，到达53M左右之后，有姿态角的迅速降落，直至炸机。
（5）、在NE控制率下，正常设置起飞3M，飞机纯粹油门量最大向上直飞，一直飞。


__2、sitl_gazebo_square.sh飞正方形__
正常启动sitl_gazebo_square.sh脚本。确定并初始化px4_pos_controller节点。然后在set_mode节点中切换至offboard模式。检查square节点中，按键１执行飞正方形。最后在qgc中解锁飞机，飞机正常按照Point点进行飞行。
在飞机飞正方形的时候,有5个point点的设置,飞行过程部分截图如下:
point1:
![square_point1.png](https://i.loli.net/2019/08/01/5d42997d165f597762.png)
point2:
![square_point2.png](https://i.loli.net/2019/08/01/5d42997cc54df85520.png)
point4:
![square_point4.png](https://i.loli.net/2019/08/01/5d4299960207886771.png)
point5
![square_point5.png](https://i.loli.net/2019/08/01/5d429c53633cf52748.png)

__3、sitl_gazebo_formation.sh多机编队仿真__
正常启动sitl_gazebo_formation.sh，在启动正常的情况下（qgc可以连接上三个飞机），此时确认formation_control节点并初始化，按照ENU坐标系下，设置坐标点，三架飞机同步执行动作。如下图:
![formation_start.png](https://i.loli.net/2019/08/01/5d429dfd8f27054497.png)

存在Bug描述：
（１）、启动脚本失败（已将时间由２改为４，成功启动概率增大）
（２）、确认初始化formation_control节点之后，打印信息有问题。UAV2显示未连接，解锁状态无响应，飞行模式无显示
（３）、飞机解锁之后，设置好第一个坐标点，飞机起飞，相互位置会有所调换，然后悬停至稳定
（４）、使用land模式之后，有的飞机会直接失控，有的会缓缓降落。
（５）、飞机执行land落地之后飞行模式在pos与RTL之间频繁切换
