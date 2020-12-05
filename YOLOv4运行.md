

# YOLOv4运行

##  一、下载darknet

```bash
git clone https://github.com/AlexeyAB/darknet.git
```

## 二、编译

### 1.根据需要修改Makefile文件

```bash
cd darknet
gedit Makefile
```

> GPU=0        --->  1                 //GPU=1 表示需要安装cuda ，这就是为了加速了，的确会快一些
> CUDNN=0       ----> 1          //CUDNN=1 表示需要安装cudnn
> CUDNN_HALF=0  ----> 1
> OPENCV=0    -----> 1           //OPENCV=1 需要opencv，这样就可以在运行直接弹出检测到的结果图
> AVX=0
> OPENMP=0
> LIBSO=0  ---->1
> ZED_CAMERA=0
> ZED_CAMERA_v2_8=0

### 2. 安装CUDA 10.0 和CUDNN7.3.0

​		参考后续笔记

### 3. make   !!!!!!

​		或make -j

- 最初没有修改Makefile文件，在make时没有错误，正常运行

- 后来让GPU=1，CUDNN=1,也没有报错，在测试的时候速度的确快了一些

- 当让OPENCV=1时，出现：

  ```shell
  /usr/bin/ld: 找不到  -lopencv_contrib
  /usr/bin/ld: 找不到  -lopencv_gpu
  /usr/bin/ld: 找不到  -lopencv_legacy
  /usr/bin/ld: 找不到  -lopencv_nonfree
  /usr/bin/ld: 找不到  -lopencv_ocl
  ```

  最终解决办法是

  - 最开始是想着找到这些库，然后在/usr/下搜了一下，发现只找到了opencv2.4.11下的这些包，然后就把这些包都复制到/usr/local/lib/下，再次make，发现没用。还是报和之前一样的错误，于是就又把那些包从目录下删除了。

  - 修改cmakelists指定opencv版本，发现没用

  - 可能是在Makefile文件下指定，然后去看了一下Makefile文件

    和opencv相关的就如下：

    ```shell
    ifeq ($(OPENCV), 1)
    COMMON+= -DOPENCV
    CFLAGS+= -DOPENCV
    LDFLAGS+= `pkg-config --libs opencv4 2> /dev/null || pkg-config --libs opencv`
    COMMON+= `pkg-config --cflags opencv4 2> /dev/null || pkg-config --cflags opencv`
    
    endif
    
    ifeq ($(OPENMP), 1)
        ifeq ($(OS),Darwin) #MAC
                CFLAGS+= -Xpreprocessor -fopenmp
            else
                    CFLAGS+= -fopenmp
            endif
    LDFLAGS+= -lgomp
    endif
    
    ```

    查询后，了解到 `2> /dev/null` 表示输出到黑洞，所以和这个应该是没什么关系喔。

    

    那先了解一下pkg-config的用法吧（看后续笔记）

    那看一下pkg-config --libs opencv这句话：

    ​           可以看出来这是输出了opencv的库的所有信息(由于当时还没修改bashrc，所以输出的是2.4.11版本的信息，现在修改后是输出3.2.0版本的)

    ```shell
    y@y-workstation:~$ pkg-config --libs opencv
    
    -L/usr/local/opencv2.4.11/share/OpenCV/3rdparty/lib -lopencv_calib3d -lopencv_contrib -lopencv_core -lopencv_features2d -lopencv_flann -lopencv_gpu -lopencv_highgui -lopencv_imgproc -lopencv_legacy -lopencv_ml -lopencv_nonfree -lopencv_objdetect -lopencv_ocl -lopencv_photo -lopencv_stitching -lopencv_superres -lopencv_ts -lopencv_video -lopencv_videostab -lrt -lpthread -lm -ldl
    ```

    然后是pkg-config --cflags opencv，这是输出opencv的头文件的位置

    ```shell
    y@y-workstation:~$ pkg-config --cflags opencv
    -I/usr/local/opencv2.4.11/include/opencv -I/usr/local/opencv2.4.11/include
    ```

  了解到有.pc文件，在每个版本的/build/unix-install/opencv.pc复制到/usr/lib/pkgconfig/目录下，并修改了文件名，分别为opencv2.pc, opencv3.pc, oepncv4.pc，分别表示opencv2.4.11，3.2.0，4.4.0版本。

  到这之后发现原来的pkg-config --libs opencv和pkg-config --cflags opencv都找的是2.4.11的，试着将Makefile修改一下（因为把opencv.pc都加上版本号了，所以想着修改一下，但其实后来发现，当把默认的换成3.2.0后，直接opencv.pc找的就是3.2.0的）：

  ```shell
  - ifeq ($(OPENCV), 1)
    COMMON+= -DOPENCV
    CFLAGS+= -DOPENCV
    LDFLAGS+= `pkg-config --libs opencv4 2> /dev/null || pkg-config --libs opencv3`
    COMMON+= `pkg-config --cflags opencv4 2> /dev/null || pkg-config --cflags opencv3`
  
    endif
  
    ifeq ($(OPENMP), 1)
        ifeq ($(OS),Darwin) #MAC
                CFLAGS+= -Xpreprocessor -fopenmp
            else
                    CFLAGS+= -fopenmp
            endif
    LDFLAGS+= -lgomp
    endif
  
  ```

  后来还在CmakeLists里指定了一下opencv 3

- 然后make成功了！！   撒花 :white_flower::white_flower:









# 多版本opencv管理 --pkg-config



```shell
y@y-workstation:~$ pkg-config --cflags opencv4 
-I/usr/local/opencv4.4.0/include/opencv4
y@y-workstation:~$ pkg-config --cflags opencv
-I/usr/local/opencv2.4.11/include/opencv -I/usr/local/opencv2.4.11/include
y@y-workstation:~$ pkg-config --cflags opencv3
-I/usr/local/include/opencv -I/usr/local/include
y@y-workstation:~$ pkg-config --cflags opencv
-I/usr/local/opencv2.4.11/include/opencv -I/usr/local/opencv2.4.11/include
y@y-workstation:~$ pkg-config --cflags opencv2
-I/usr/local/opencv2.4.11/include/opencv -I/usr/local/opencv2.4.11/include
```

[Ubuntu下多个版本OpenCV管理（Multiple Opencv version）](https://blog.csdn.net/baobei0112/article/details/80833223)





# pkg-config

https://www.cnblogs.com/rainsoul/p/10567390.html

https://www.cnblogs.com/sddai/p/10266624.html

pkg-config是一个linux下的命令，用于获得某一个库/模块的所有编译相关的信息。

例子：

  pkg-config opencv –libs –cflags

### pkgconfig有什么用：

​    大家应该都知道用第三方库，就少不了要使用到第三方的头文件和库文件。我们在编译、链接的时候，必须要指定这些头文件和库文件的位置。

​    对于一个比较大第三方库，其头文件和库文件的数量是比较多的。如果我们一个个手动地写，那将是相当麻烦的。所以，pkg-config就应运而生了。pkg-config能够把这些头文件和库文件的位置指出来，给编译器使用。如果你的系统装有gtk，可以尝试一下下面的命令$pkg-config --cflags gtk+-2.0。可以看到其输出是gtk的头文件的路径。

​    我们平常都是这样用pkg-config的。$gcc main.c `pkg-config --cflags --libs gtk+-2.0` -o main

​    上面的编译命令中，`pkg-config --cflags --libs gtk+-2.0`的作用就如前面所说的，把gtk的头文件路径和库文件列出来，让编译去获取。--cflags和--libs分别指定头文件和库文件。

​    Ps:命令中的`不是引号，而是数字1左边那个键位的那个符号。

 

​    其实，pkg-config同其他命令一样，有很多选项，不过我们一般只会用到--libs和--cflags选项。更多的选项可以在[这里](http://linux.die.net/man/1/pkg-config)查看。

 

### 配置环境变量：

​    看到这里，大家可能想试一下将pkg-config用于自己的库。下面就说一下，怎么写。

​    首先要明确一点，因为pkg-config也只是一个命令，所以不是你安装了一个第三方的库，pkg-config就能知道第三方库的头文件和库文件所在的位置。

>  pkg-config的信息从哪里来？
>
> 很简单，有2种路径：
> 第一种：取系统的/usr/lib下的所有*.pc文件。
> 第二种：PKG_CONFIG_PATH环境变量所指向的路径下的所有*.pc文件。
>
> 这些pc文件什么时候有的？都是在你安装某个库/模块的时候，添加的。比如你往系统安装opencv时，就会在/usr/lib/目录下，放一个opencv.pc。
>
> pkg-config命令是通过查询XXX.pc文件而知道这些的。我们所需要做的是，写一个属于自己的库的.pc文件。

​    但pkg-config又是如何找到所需的.pc文件呢？这就需要用到一个环境变量PKG_CONFIG_PATH了。这环境变量写明.pc文件的路径，pkg-config命令会读取这个环境变量的内容，这样就知道pc文件了。

​    对于Ubuntu系统，可以用root权限打开/etc/bash.bashrc文件。在最后输入下面的内容。

​    ![img](https://img-blog.csdn.net/20140501110459078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHVvdHVvNDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 

​    这样，pkg-config就会去/usr/local/lib/pkgconfig目录下，寻找.pc文件了。如果不是Ubuntu系统，那就没有/etc/bash.bashrc文件，可以参考我的一篇[博文](http://blog.csdn.net/luotuo44/article/details/8917764)，来找到一个合适的文件，修改这个环境变量。输入bash命令使得配置生效。

​    现在pkg-config能找到我们的.pc文件。但如果有多个.pc文件，那么pkg-config又怎么能正确找到我想要的那个呢？这就需要我们在使用pkg-config命令的时候去指定。比如$gcc main.c `pkg-config --cflags --libs gtk+-2.0` -o main就指定了要查找的.pc文件是gtk+-2.0.pc。又比如，有第三方库OpenCV，而且其对应的pc文件为opencv.pc，那么我们在使用的时候，就要这样写`pkg-config --cflags --libs opencv`。这样，pkg-config才会去找opencv.pc文件。

 

### pc文件书写规范：

​    好了，现在我们开始写自己的.pc文件。只需写5个内容即可：Name、Description、Version、Cflags、Libs。

​    比如简单的：

1.  Name: opencv
2.  Description:OpenCV pc file
3.  Version: 2.4
4.  Cflags:-I/usr/local/include
5.  Libs:-L/usr/local/lib –lxxx –lxxx

​     其中Name对应的内容要和这个pc文件的文件名一致。当然为了书写方便还会加入一些变量，比如前缀变量prefix。下面有一个更完整的pc文件的内容

​    ![img](https://img-blog.csdn.net/20140501110745640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHVvdHVvNDQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

​    其中，Cflags和Libs的写法，是使用了-I -L -l这些gcc的编译选项。







# 安装CUDA 10.0 和CUDNN7.3.0

> **相关命令：**https://blog.csdn.net/qq_34638161/article/details/80845366
>
> 查看cuda版本 ：    nvcc -V
>
> 查看位置  ：       which nvcc
>
> 查看NVIDIA动态使用情况：  watch -n 1 nvidia-smi  
>
> cuda 版本   ：   cat /usr/local/cuda/version.txt
>
> cudnn 版本  ：   cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
>
> NVIDIA 驱动版本  ：  cat /proc/driver/nvidia/version
>
> 查看环境变量  :      env
>
> LD_DEBUG=all cat
>
> 卸载cuda ：         sudo  /usr/local/cuda-8.0/bin/uninstall_cuda_8.0.pl
>
> 卸载NVIDIA Driver ：  sudo  /usr/bin/nvidia-uninstall
>
> **多版本CUDA切换:**
>
> sudo  rm  -rf  /usr/local/cuda          
>
> sudo  ln  -s   /usr/local/cuda-8.0  /usr/local/cuda  
>
> sudo  ln  -s   /usr/local/cuda-9.1  /usr/local/cuda  





  https://developer.nvidia.com/cuda-toolkit-archive （所有版本）

  https://developer.nvidia.com/cuda-10.0-download-archive（10.0）

### 1. 下载CUDNN

 这个下载之前需要注册  https://developer.nvidia.com/cudnn

### 2.  把解压后的cudnn和cuda的.run文件放在同一文件夹下

然后我在家目录下创建了CUDA文件夹，都放在这个文件夹下

### 3.  安装

```bash
（官网安装步骤）Installation Instructions:

1. Run `sudo sh cuda_10.0.130_410.48_linux.run`

2. Follow the command-line prompts
```

所以就先`sudo sh cuda_10.0.130_410.48_linux.run`，然后发现一直卡住不动，需要一直按回车（# 稍后会出现很多提示信息，可以长按空格键直接跳过 --- 后来看到这句话，没试），直到出现**安装选项**：

```Shell
Do you accept the previously read EULA?
accept/decline/quit: accept

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 410.48?  #这里出现的驱动版本是根据显卡自动匹配的
(y)es/(n)o/(q)uit: no           #因为之前已经安装了显卡驱动，所以这里输入 no

Install the CUDA 10.0 Toolkit?
(y)es/(n)o/(q)uit: yes

Enter Toolkit Location
 [ default is /usr/local/cuda-10.0 ]: 

Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit: no    # 根据看的那个教程（现在找不到了），说是除了装驱动的那个要no，其他的都yes了

Install the CUDA 10.0 Samples?
(y)es/(n)o/(q)uit: no  #这也是yes了
```

**接下来是安装cudnn：**

参考https://docs.nvidia.com/deeplearning/cudnn/install-guide/#installlinux-tar 2.3.1

```shell
#Before issuing the following commands, you'll need to replace x.x and v8.x.x.x with your specific CUDA version and cuDNN version and package date.
#Procedure

    Navigate to your <cudnnpath> directory containing the cuDNN Tar file.
  1.  Unzip the cuDNN package.
    $ tar -xzvf cudnn-x.x-linux-x64-v8.x.x.x.tgz

 2.  Copy the following files into the CUDA Toolkit directory, and change the file permissions.

    $ sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
    $ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    $ sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```



### 4. 添加环境变量

```shell
sudo gedit  .bashrc
```

在最后添加：

```shell
export PATH=/usr/local/cuda-10.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
```

### 5. 检测安装结果

```shell
#查询CUDA版本信息
y@y-workstation:~$ nvcc -V        
#输出：
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130
```

```shell
#检测是否能查询显卡信息
y@y-workstation:~$ cd /usr/local/cuda-10.0/samples/1_Utilities/deviceQuery
y@y-workstation:/usr/local/cuda-10.0/samples/1_Utilities/deviceQuery$ sudo make && ./deviceQuery
#输出：
[sudo] y 的密码： 
make: 对“all”无需做任何事。
./deviceQuery Starting...

 CUDA Device Query (Runtime API) version (CUDART static linking)

Detected 1 CUDA Capable device(s)

Device 0: "GeForce GTX 1650"
  CUDA Driver Version / Runtime Version          11.0 / 10.0
  CUDA Capability Major/Minor version number:    7.5
  Total amount of global memory:                 3903 MBytes (4093050880 bytes)
  (14) Multiprocessors, ( 64) CUDA Cores/MP:     896 CUDA Cores
  GPU Max Clock rate:                            1515 MHz (1.51 GHz)
  Memory Clock rate:                             6001 Mhz
  Memory Bus Width:                              128-bit
  L2 Cache Size:                                 1048576 bytes
  Maximum Texture Dimension Size (x,y,z)         1D=(131072), 2D=(131072, 65536), 3D=(16384, 16384, 16384)
  Maximum Layered 1D Texture Size, (num) layers  1D=(32768), 2048 layers
  Maximum Layered 2D Texture Size, (num) layers  2D=(32768, 32768), 2048 layers
  Total amount of constant memory:               65536 bytes
  Total amount of shared memory per block:       49152 bytes
  Total number of registers available per block: 65536
  Warp size:                                     32
  Maximum number of threads per multiprocessor:  1024
  Maximum number of threads per block:           1024
  Max dimension size of a thread block (x,y,z): (1024, 1024, 64)
  Max dimension size of a grid size    (x,y,z): (2147483647, 65535, 65535)
  Maximum memory pitch:                          2147483647 bytes
  Texture alignment:                             512 bytes
  Concurrent copy and kernel execution:          Yes with 3 copy engine(s)
  Run time limit on kernels:                     Yes
  Integrated GPU sharing Host Memory:            No
  Support host page-locked memory mapping:       Yes
  Alignment requirement for Surfaces:            Yes
  Device has ECC support:                        Disabled
  Device supports Unified Addressing (UVA):      Yes
  Device supports Compute Preemption:            Yes
  Supports Cooperative Kernel Launch:            Yes
  Supports MultiDevice Co-op Kernel Launch:      Yes
  Device PCI Domain ID / Bus ID / location ID:   0 / 1 / 0
  Compute Mode:
     < Default (multiple host threads can use ::cudaSetDevice() with device simultaneously) >

deviceQuery, CUDA Driver = CUDART, CUDA Driver Version = 11.0, CUDA Runtime Version = 10.0, NumDevs = 1
Result = PASS  #成功

```

```shell
#查询cudnn版本
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
#输出：
#define CUDNN_MAJOR 7
#define CUDNN_MINOR 3
#define CUDNN_PATCHLEVEL 0
--
#define CUDNN_VERSION (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

#include "driver_types.h"
```

安装完结，撒花 :white_flower: