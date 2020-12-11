# Linux客户端

## 打包机

Linux(x86_64) ubuntu16备注这里不能安装版本太高的gcc: 

10.0.0.55 root 123YES!!

这个打包机主要用于打包Linux下的x86的deb以及rpm的包


**注意**
1. 如果打包机掉线了需要运行一下```/media/xinda/新加卷/jenkins_qt_linux_remote```下的run_slave.sh执行一下启动


Windows(x86_64):
10.0.0.69 Administrator 123YES!!

打Windows的x86的安装包，Windows打包采用的是Qt5.7.1


## 国产化机器

通过视频切换器进行相关机器的切换

**UOS(MIPS64EL龙芯)**

xinda 123456

这个打包需要启动的时候需要进入boost界面然后选择boot from file，才能启动uos

后面如果需要支持银河麒麟加龙芯应该也需要同样的低版本Qt和低版本的gcc上进行相关编译工作


**UOS(ARM64鲲鹏920)**

huawei 123456

> 这个系统主要用于用于打包ARM64的UOS打包机，同时也需要支持打出来的arm64的包需要在uos的麒麟990的笔记本上运行，因此注意打包的时候需要将wayland相应的插件打包到安装包中

**银河麒麟(ARM64鲲鹏920)**

sancog sancog123

> 银河麒麟的ARM64机器主要是为了解决在glibc2.23版本下无法运行glibc2.28版本的安装包，会出现跨libc兼容问题，因此在这个机器上使用使用自己编译出来的qt5.6.2(包括webengine模块)来重新编译youduqt

**UOS(ARM64麒麟990)**

账号密码：

huawei 123456

要专门适配的问题是因为此芯片下的桌面是基于wayland与xcb的有不同，因此需要专门适配一下，这里注意一定要把qt相关plugins下的wayland相关的so库全部打包到安装中，同时记得查看程序是否以xcb兼容模式启动，不然导致窗口位置无法移动等严重问题 因此一定要加上词句，如果应用是需要在wayland上进行运行的


```bash
export QT_QPA_PLATFORM='xcb' # 表示以xcb的兼容模式进行启动
```

# 编译环境以及打包

## 概述
编译环境的搭建主要分成几个点：
1. jgapi依赖库的编译
2. jgapi的编译
3. 数据库加解密库的编译
4. crashdump依赖库的编译
5. youdu工程的编译

本文演示环境deepinV20 ，Qt5.14.2
1. git clone https://repo.cindacode.com/qt-client/qt-linux.git 拉取代码，推荐工具gitkraken
2. 切换到master分支 git checkout master
3. 下载Qt5.14.2(注意是x86环境) 下载地址 http://download.qt.io/archive/qt/5.14/5.14.2/ 也可以直接到25上拿 \\10.0.0.25\公司数据盘\_software\windows\youduqt环境搭建
4. 安装qt5.14.2 ![picture 5](images/91785f77e348d1d282ae3d1bb8813285ec269f1ce3865663be266230ba89ae32.png)  
记得勾选上webengine模块
5. 全新环境安装g++ sudo apt-get install g++,sudo apt-get install libgl1-mesa-dev,sudo apt-get install gdb
6. 编译数据库加密库 QtCipherSqlitePlugin release版本，编译完成后，cp libsqlitecipher.so /home/xinda/Qt5.14.2/5.14.2/gcc_64/plugins/sqldrivers/，移动到qt的插件sqldrivers目录下

依赖库环境编译：
![picture 7](images/e41df97119875dce40325973c5f8cd43140075d2d6e39f11707201e0d6d75e58.png)  
1. zlib-1.2.11 /configure --prefix=/home/xinda/Desktop/qt-linux-dep-source/deps/zlib ，make -j8，make install

2. openssl-1.0.1g ./config --prefix=/home/xinda/Desktop/qt-linux-dep-source/deps/openssl shared zlib

3. curl7.53 注意在编译检查的时候一定要看清楚是否支持https协议
```bash
./configure --prefix=/home/xinda/Desktop/qt-linux-dep-source/deps/curl --with-ssl=/home/xinda/Desktop/qt-linux-dep-source/deps/openssl (换成删除编译出来的openssl路径)
```
![picture 6](images/d6fb8bba9f607c5dfee20c58d8dae44cb27f9c35ba1991a44e2404c0a7ac3094.png)  
4. nopoll-0.4.6 在某些系统上出现 需要先安装 sudo apt-get install libtool，同时需要安装libssl-dev，nopoll会依赖openssl

5. protolbuf-2.6.1 
```bash
./configure --prefix=/home/xinda/Desktop/qt-linux-dep-source/deps/protobuf --with-zlib 
make -j4 && make install 
```
6. qBreakpad 直接导入qtcreator编译release版本即可
　

### Qt环境
1. Windows采用的是Qt5.7.2版本(jgapi的asio升级到高版本会有障碍以及后续产品可能需要支持xp系统等)
2. Linux(x86)采用的版本是Qt5.14.2 (之前升级上来主要要兼容音视频的QtBrowser)
3. 国产化的机器主要采用UOS进行打包，UOS的鲲鹏以及UOS的龙芯，因此采用的版本是设备上内置的的Qt5.11.3进行打包
4. 在处理银河麒麟机器上主要是使用Qt5.6.2(自己编译的)

### GCC版本
1. gcc版本也同样在编译构建的时候一定要选用gcc编译器，不要选中clang（MACOS除外）等其他编译器这样会造成一些未知错误，因此需要慎重。
2. Windows则主要不要用MSVC配对的Qt5.7.1
3. MACOS可以自行选择，当前选择的是Qt5.15(也可以选择Qt5.14.2都没有关系)


### 打包工具
#### Linux
在linux无论什么架构上都是采用[linuxdeployqt](https://github.com/probonopd/linuxdeployqt)
因此需要在国产化架构上进行打包需要升级到linuxdeployqt release版本的编译出来然后将linuxdeployqt移动到/usr/bin目录下

注意事项:

在编译linuxdeployqt的时候需要避免高版本的gcc编译linuxdeployqt，如果是高版本在编译的时候会报错，这时候需要屏蔽掉相应的代码：
```cpp
// openSUSE Leap 15.0 uses glibc 2.26 and is used on OBS
    /*if (strverscmp (glcv, "2.27") >= 0) {  //注释版本检查
      qInfo() << "ERROR: The host system is too new.";
      qInfo() << "Please run on a system with a glibc version no newer than what comes with the oldest";
      qInfo() << "currently still-supported mainstream distribution (xenial), which is glibc 2.23.";
      qInfo() << "This is so that the resulting bundle will work on most still-supported Linux distributions.";
      qInfo() << "For more information, please see";
      qInfo() << "https://github.com/probonopd/linuxdeployqt/issues/340";
      return 1;
    }*/
```
还有如果必要，需要安装：
```cpp
sudo apt install patchelf
```
参考：

https://blog.csdn.net/weixin_44713381/article/details/107894339
https://cloud.tencent.com/developer/article/1720779

**提醒**

1. 一定要使用linuxdeployqt打包工具，防止自己书写脚本将youduqt可执行文件ldd的依赖全部挪到相应目录，这样多余的系统so文件会造成程序启动时的crash，因此需要注意要使用打包工具进行打包。


#### Windows
在Windows上需要用到的是windeployqt这个官方提供的工具进行打包，需要注意一些第三方依赖需要自己移植的相应目录下

1. windows则需要注意 msvcr120.dll，msvcp120.dll需要打包

#### MACOS
1. macos的打包工具同样是使用macdeployqt

## Linux打包概述

### 

## Windows打包概述

## Mac概述



# 打包

# 程序结构

## 分支情况
![picture 4](images/2c6b071655cccdf59746cfbdf9cbf0430c3dc13b7a68cccbef14d350f08a73c6.png)  
- master 分支主要为维护最新的代码分支
- dev 分支为之前的规范，现在已经废弃
- longson_build 之前中标麒麟下mips的打包分支，现在可以删除
- branch_master_netdisk_feature 为网盘的功能分支
- branch_master_package_uos_store 为打包uos下的的打包，主要是uos下打包目录下不允许有usr目录，而debian的系统则需要，因此专门维护这条分支用户uos下x86_64的打包
- release_master_203.11 版本为203.11的特性包分支
- branch_master_202.9_release 为202.9的补丁包
- branch_master_fix_windows_package 主要为用于Windows的打包，主要处理了一下quazip这个库zlib依赖问题
- branch_master_feature_slide_btn 主要处理收缩侧边栏，效果不太好，故没有合入主分支


# 注意事项

# 问题