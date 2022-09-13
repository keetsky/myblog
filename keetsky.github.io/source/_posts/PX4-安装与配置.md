---
title: PX4 安装与配置
date: 2018-02-09 20:02:19 
comments: true
english_title: px4 install  #对于中文标题需要个英文字体代替中文显示在URL上
categories:  #分类 
- ubuntu 
tags:	     #两个标签
- px4

---
[官网教程](https://dev.px4.io/)

### ubuntu 系统

#### 权限设置
把用户添加到用户组　"dialout"

``` python
$ sudo usermod -a -G dialout $USER
```

然后注销后，重新登录，因为重新登录后所做的改变才会有效。
 
#### 安装-基于NuttX的硬件
Ubuntu配备了一系列代理管理，这会严重干扰任何机器人相关的串口（或usb串口），卸载掉它也不会有什么影响:

``` bash
sudo apt-get remove modemmanager
```

更新包列表和安装下面的依赖包。务必安装指定的版本的包.

```
sudo apt-get install python-serial openocd \
    flex bison libncurses5-dev autoconf texinfo build-essential \
    libftdi-dev libtool zlib1g-dev \
    python-empy  -y
```
在添加arm-none-eabi工具链之前，请确保删除残余，如果需要变更工具链也需要删除先前版本。

```
sudo apt-get remove gcc-arm-none-eabi gdb-arm-none-eabi binutils-arm-none-eabi gcc-arm-embedded
sudo add-apt-repository --remove ppa:team-gcc-arm-embedded/ppa
```

如果gcc-arm-none-eabi版本导致PX4/Firmware编译错误，

##### 安装arm-none-eabi工具链
要求gcc版本>=4.9 cmake 版本大于3.2，Ubuntu系统gcc为4.8,cmake 为3.2,所以需要重新安装新的版本
gcc-arm-none-eabi GCC 5.4版本

```
cd ~
wget https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q2-update/+download/gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2
tar -jxf gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2
exportline="export PATH=$HOME/gcc-arm-none-eabi-5_4-2016q2/bin:\$PATH"     #或直接在.bashrc末尾写入 export PATH=home/keetsky/gcc-arm-none-eabi-5_4-2016q2/bin:\$PATH
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
```
gcc-arm-none-eabi GCC 4.9 版本

```
 cd ~
wget https://launchpad.net/gcc-arm-embedded/4.9/4.9-2015-q3-update/+download/gcc-arm-none-eabi-4_9-2015q3-20150921-linux.tar.bz2
tar -jxf gcc-arm-none-eabi-4_9-2015q3-20150921-linux.tar.bz2
exportline="export PATH=$HOME/gcc-arm-none-eabi-4_9-2015q3/bin:\$PATH"
if grep -Fxq "$exportline" ~/.profile; then echo nothing to do ; else echo $exportline >> ~/.profile; fi
. ~/.profile
```

也可通过源库安装，不过安装的是4.9版本

```
sudo add-apt-repository ppa:team-gcc-arm-embedded/ppa
sudo apt-get update
sudo apt-get install gcc-arm-embedded
```

安装32位支持库（如果已经是运行在32位，那么可能会失败，并且此步骤可以跳过）：

```
sudo dpkg --add-architecture i386
sudo apt-get update
sudo apt-get install libc6:i386 libgcc1:i386 libstdc++5:i386 libstdc++6:i386
sudo apt-get install gcc-4.6-base:i386
```
arm-none-dabi-gcc测试 

```
arm-none-eabi-gcc --version   #查看输出对否有如下输出
-----------------------------------------------
arm-none-eabi-gcc (GNU Tools for ARM Embedded Processors) 5.4.1 20160609 (release) [ARM/embedded-5-branch revision 237715]
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
或

```
$arm       #再按下tab键是否有如下输出

-------------------------------
arm2hpdl                  arm-none-eabi-gcc-5.4.1   arm-none-eabi-ld.bfd
arm-none-eabi-addr2line   arm-none-eabi-gcc-ar      arm-none-eabi-nm
arm-none-eabi-ar          arm-none-eabi-gcc-nm      arm-none-eabi-objcopy
arm-none-eabi-as          arm-none-eabi-gcc-ranlib  arm-none-eabi-objdump
arm-none-eabi-c++         arm-none-eabi-gcov        arm-none-eabi-ranlib
arm-none-eabi-c++filt     arm-none-eabi-gcov-tool   arm-none-eabi-readelf
arm-none-eabi-cpp         arm-none-eabi-gdb         arm-none-eabi-size
arm-none-eabi-elfedit     arm-none-eabi-gdb-py      arm-none-eabi-strings
arm-none-eabi-g++         arm-none-eabi-gprof       arm-none-eabi-strip
arm-none-eabi-gcc         arm-none-eabi-ld  
```

cmake 安装：
>  [官网下载](https://cmake.org/) 

```
$cd cmake-3.9.0
$./configure
$make
$make install //在/usr/local/bin可以看到cmake可执行程序，添加cmake到PATH环境变量中
$cmake --version //查看版本为3.9.0
```

### 编译px４ 

```
cd ~
mkdir -p ~/src
cd ~/src
git clone https://github.com/PX4/Firmware.git
cd Firmware
git submodule update --init --recursive

make px4fmu-v2_default      #也可执行v3版本:make px4fmu-v3_default 
```

编译的二进制程序就会通过USB上传到飞控硬件:

```
make px4fmu-v2_default upload
```

### 错误分析

- error:ld returned 1 exit status
> 不能直接对源代码进行编译，直接编译可能会出现内存溢出的错误error:ld returned 1 exit status，原因是arm-none-eabi 4.7.4版本不对，需要重新安装arm-none-eabi直接编译失败的结果显示如下:

```
collect2.exe:error:ld returned 1 exit status
make[3]: *** [src/firmware/nuttx/firmware_muttx] Error 1
make[2]: *** [src/firmware/nuttx/CMakeFiles/firmware_muttx.dir/all] Error 2
make[1]: *** [all] Error 2
make: *** [px4fmu-v2_default] Error 2
```

-  提示cmake 错误
> 如果出现了类似错误，是由于cmake版本过低造成的，使用apt-get安装的cmake版本为3.2.2，而在3.3版本以下就会报这个错误，如果出现无法进入文件夹的错误也是这个问题：
```
CMake Error at platforms/nuttx/NuttX/CMakeLists.txt:113 (add_dependencies):
  add_dependencies Cannot add target-level dependencies to INTERFACE library
  target "nuttx_build".
```

- 内存溢出
>解决方案：移除不是必要的模块. 配置在这里. 为了移除一个模块, 可以直接注释掉它:
对于FMUv2(Pixhawk1)或者FMUv3(Pixhawk 2)硬件，找到Firmware/cmake/configs/nuttx_px4fmu-v2_default.cmake
修改为注释掉不要的driver 如`#drivers/trone`  或者使用`v3`版本 `make px4fmu-v3_default` 

```
..regin 'falsh ' overflowby 34234 bytes
collect2.exe:error:ld returned 1 exit status
make[3]: *** [src/firmware/nuttx/firmware_muttx] Error 1
make[2]: *** [src/firmware/nuttx/CMakeFiles/firmware_muttx.dir/all] Error 2
make[1]: *** [all] Error 2
make: *** [px4fmu-v2_default] Error 2
```

