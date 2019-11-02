# Android9.0内核源码编译及烧入pixel 3

作者：谢琳 时间：2019年8月19日 邮箱：shallin123@163.com

## 一、Android源码下载

### 1、git的安装与配置

```
sudo apt-get install git
git config --global user.email “test@test.com”
git config --globa user.name “test”
```

其中[test@test.com](mailto:test@test.com)为你自己的邮箱.例如927002786@708.com

### 2、repo的下载与安装

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

### 3、建立源码文件夹

我们需要为项目在本地创建对应的仓库 即创建一个文件夹

```
mkdir android9
cd android9
```

### 4、初始化仓库

```
repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-9.0.0_r21
repo sync
```

其中android-9.0.0_r17是Android对应的版本表示号，每个版本对应的版本标识号参考[安卓版本标识号](https://source.android.com/source/build-numbers.html#source-code-tags-and-builds)

> （1）如果执行该命令的过程中,如果提示无法连接到 gerrit.googlesource.com，
>
> 那么我们只需要编辑 ~/bin/repo文件，找到REPO_URL这一行,然后将其内容修改为: `REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'`
>
> （2）repo sync出现“fatal: '../platform/abi/cpp.git' does not appear to be a git repository”的解决方案
>
> a、在存放android系统源代码的目录（也就是执行repo sync命令的目录）下，有个.repo的隐藏目录，用ls -a可以查看的到，进入该目录：cd .repo；
>
> b、打开.repo目录下的manifest.xml文件（命令vim manifest.xml）并找到fetch属性，在我的文件中显示fetch=".."，将fetch修改为 fetch="git://Android.git.linaro.org/"( 或则改为 git://git.omapzoom.org )，保存并退出；
>
> c、继续repo sync就可以下载了。

## 二、Android源码的编译与镜像烧入

### 1、Android源码编译

```
source build/envsetup.sh
lunch aosp_blueline-userdebug
make -j4（用四个线程去编译，一般线程数为CPU内核的2倍）
```

### 2、Android编译镜像烧入

a、[解锁bootloader手机](https://android.gadgethacks.com/how-to/unlock-bootloader-your-google-pixel-pixel-xl-0174627/)

b、打开usb调试，进入BootLoader模式

`adb reboot bootloader`

c、烧入编译生成的所有镜像

`fastboot flashall -w`

## 三、内核的下载

```
repo init -u https://https://aosp.tuna.tsinghua.edu.cn/kernel/manifest -b android-msm-crosshatch-4.9-pie-qpr2
```

## 四、内核源码的修改

由于当编译生成的内核烧入手机由于ko文件签名不对、ko文件版本号(version magic)和内核不匹配、内核编译开启了CONFIG_MODVERSIONS选项会导致生成的内核镜像在手机中运行时候导致ko文件加载不成功，所以要进行以下操作
### 1、关闭内核编译选项模块签名和模块对内核函数符号crc检验选项
修改b1c1_defconfig文件，具体如下(左边是修改后的)
![image](https://github.com/shallin123/Android9.0-pixel-3/blob/master/1.png)

另外修改了b1c1_defconfig后，对build/build.sh编译脚本做如下修改，不然编译会报错。
![image](https://github.com/shallin123/Android9.0-pixel-3/blob/master/2.png)
### 2、 关闭内核版本检测（versionmagic）检测
修改内核源码目录下的kernel/module.c,如下：
![image](https://github.com/shallin123/Android9.0-pixel-3/blob/master/3.png)
## 五、内核编译

```
build/build.sh
```

## 六、内核烧入

1、将内核源码out/android-msm-crosshatch-4.9/dist目录下的Image.lz4-dtb拷贝到Android9系统源码的

device/google/crosshatch-kernel目录下

2、编译系统源码生成boot.image在系统目录下执行以下命令（执行前先make clean，清除之前编译的内容）

```
source build/envsetup.sh
lunch aosp_blueline-userdebug
make bootimage
```
3、将生成的boot.image加载进内核
```
cd /android9.0.0_r21/out/target/product/blueline
fastboot flash boot boot.img`
```
## 七、检测内核是否烧入成功

打开adb shell输入`cat  /proc/version`

![image](https://github.com/shallin123/Android9.0-pixel-3/blob/master/4.png)

结果如上图，其中显示的用户名为自己计算机的用户名。

