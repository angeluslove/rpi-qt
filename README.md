
关于qt在树莓派pi4上的编译！

//1------------------------------------ubuntu换源
##中科大源
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse

//2-------------------------------------ubuntu安装编译必须组件
sudo apt-get install build-essential sshpass git python pkg-config re2c gperf bison flex  python ruby gcc-multilib g++-multilib
//3-------------------------------------创建 raspi文件夹
mkdir ~/raspi
//4-------------------------------------下载qt源文件并解压 下载过程就不说了
tar xvf  qt-everywhere-src-5.12.2.tar.xz
cd  qt-everywhere-src-5.12.2
//5-------------------------------------下载编译工具
 下载官方的交叉编译工具tools
https://pan.baidu.com/s/19QtlFMjT4D893jCty8nHxg
这个工具是以前下载的，因为git下载速度太慢了，所以不是最新的工具，大概是19年4月左右下载的，不过编译没问题
介意的可以去官方git下载 https://github.com/raspberrypi/tools
解压后修改名称为tools ,结构如这样~/raspi/tools

//-------------------------------------树莓派换源
sudo nano /etc/apt/sources.list
deb http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi
deb-src http://mirrors.aliyun.com/raspbian/raspbian/ buster main contrib non-free rpi

//-------------------------------------安装必要包pi
sudo apt-get install libboost1.58-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev
sudo apt-get update
sudo apt-get install libgles2-mesa-dev libgbm-dev
sudo apt-get install bluez libbluetooth-dev
//--------------------------------------同步

rsync -avz pi@192.168.5.116:/lib sysroot
rsync -avz pi@192.168.5.116:/usr/include sysroot/usr
rsync -avz pi@192.168.5.116:/usr/lib sysroot/usr
rsync -avz pi@192.168.5.116:/opt/vc sysroot/opt

//---------------------------------------下载脚本
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
chmod +x sysroot-relativelinks.py
./sysroot-relativelinks.py sysroot

//---------------------------------------编译
./configure -release -opengl es2 -device linux-rasp-pi3-vc4-g++ -device-option CROSS_COMPILE=~/raspi/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf-  -sysroot ~/raspi/sysroot -opensource -confirm-license -skip qtwayland -skip qtwebengine -skip qtgamepad -skip qt3d -skip qtlocation -skip qtscript -make libs -prefix /usr/local/qt5.12.6 -extprefix ~/raspi/qt5.12.6 -hostprefix ~/raspi/qt5.12.6-host -no-use-gold-linker -v 

//可选参数
-no-gbm   

//---------------------------------------部署到树莓派
//在树莓派上建立文件夹并修改权限（pi上执行）
sudo mkdir  /usr/local/Qt5.12.6
sudo chown -R pi:pi /usr/local/Qt5.12.6

//同步到树莓派上(ubuntu上执行)
rsync -avz qt5.12.6 pi@192.168.5.116:/usr/local

//设置引用配置（pi上执行）
echo /usr/local/qt5.12.6/lib | sudo tee /etc/ld.so.conf.d/qt5.conf
sudo ldconfig

//加入pi到渲染分组中
sudo gpasswd -a pi render

//结语，我是在win10的ubuntu子系统下进行的编译，你也可以使用独立主机进行
//树莓派使用的是pi4 2G 内存版本，其他版本应该差不多，但编译的是带硬件加速的，因为我需要加速显示流畅的界面
