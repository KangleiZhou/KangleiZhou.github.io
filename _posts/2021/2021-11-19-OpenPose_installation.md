```
layout: post
category: DL
title: Ubuntu 18.04 OpenPose 安装踩坑之旅
tagline: by Kanglei Zhou
author: Kanglei Zhou
tags: 
  - 姿态估计
    published: true
```



# 安装`nvidia`显卡驱动

最简单的办法，添加驱动源并更新，然后在软件和更新的附加驱动里面就可以找到驱动了！我的推荐的是`460`版本，非常新，以下图只做演示，不一定代表我的实际情况。

![系统设置](https://files.mdnice.com/user/3650/82c4cb86-9569-473e-9795-5576dcf1a267.png)

点上它后点击apply changes，等待五分钟左右驱动安装结束，然后重启电脑。

重启后在指令行敲`nvidia-smi`或者`nvidia-settings`的命令，就可以看到显卡信息了，同时也能看到推荐的`cuda`版本。

![nvidia-smi](https://files.mdnice.com/user/3650/954c502c-32e5-41b0-9c5e-d224db025f26.png)

![nvidia-settings](https://files.mdnice.com/user/3650/119c61b8-b288-4786-93f7-4db1e570cb12.png)

# 安装`cuda10.1`

先给出下载地址：

- `cuda`历史版本下载地址：https://developer.nvidia.com/cuda-toolkit-archive

- `cudnn`下载地址：https://developer.nvidia.com/rdp/cudnn-archive

先下载`cuda10.1*.run`文件

![cuda10.1下载](https://files.mdnice.com/user/3650/9255ee94-8a6b-401d-96bd-e5a1ec14be43.jpg)

再安装好依赖

```bash
sudo apt-get install freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libgl1-mesa-glx libglu1-mesa libglu1-mesa-dev
```

然后进入命令行，输入

```bash
sudo sh cuda_10.1.105_418.39_linux.run
```

安装时，会让你先读文章，你就直接按空格键就OK

最后，配置环境变量

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

然后安装`cudnn`（注意要和`cuda`版本相符合）。下载`cudnn-10.1-linux-x64-v7.6.5.32.tgz`（点击cudnn Library for Linux）

![cudnn](https://files.mdnice.com/user/3650/19fdb064-913b-49a2-86b9-da0789ec6901.png)

执行命令

```bash
# 也就是把cudnn解压后把cudnn.h和libcudnn*放到cuda安装目录里面去
sudo cp cuda/include/cudnn.h /usr/local/cuda/include/
sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64/
sudo chmod a+r /usr/local/cuda/include/cudnn.h
sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
```

# 安装`Opencv`

安装依赖

```bash
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
```

下载

```bash
sudo apt-get install libopencv-dev
```

# 安装`protobuf`

```bash
sudo apt-get install autoconf automake libtool curl make g++ unzip build-essential -y
git clone https://github.com/google/protobuf.git
cd protobuf/
git submodule update --init --recursive 
./autogen.sh 
./configure
make 
make check 
sudo make install 
sudo ldconfig
```

# 安装`cmake`

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.15.6/cmake-3.15.6.tar.gz 
tar -zxvf cmake-3.15.6.tar.gz 
cd cmake-3.15.6 
./bootstrap 
make -j`proc` 
sudo make install -j`nproc`
```

# 安装`openpose`

## 下载`openpose`

```bash
git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose
```

##　下载模型

```bash
cd openpose
cd models
./getModels.sh
```

## 安装`caffe`

手动克隆`caffe`和`Pybind11`到3rdparty文件夹

```bash
cd openpose/3rdparty/
git clone https://github.com/BVLC/caffe.git
git clone https://github.com/pybind/pybind11
```

检查`caffe`版本：

```bash
cd caffe/
git checkout f019d0dfe86f49d1140961f8c7dec22130c83154
```

`OpenPose`贡献人员还没解决因最新版`Caffe`增加Layer导致不兼容的问题，所以需要使用commit为f019d0dfe86f49d1140961f8c7dec22130c83154的Caffe。

安装相关依赖

```bash
sudo apt-get --assume-yes install build-essential
sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
```

### 修改`Makefile.config`

进入 `caffe `，将 Makefile.config.example 文件复制一份并更名为 Makefile.config ，也可以在 caffe 目录下直接调用以下命令完成复制操作 ：

```bash
sudo cp Makefile.config.example Makefile.config
```

复制一份的原因是编译 caffe 时需要的是 Makefile.config 文件，而Makefile.config.example 只是caffe 给出的配置文件例子，不能用来编译 caffe。

修改 Makefile.config 文件，在 caffe 目录下打开该文件：

```bash
sudo gedit Makefile.config
```

在文件中替换一下几个地方： 

```bash
...
将
#USE_CUDNN := 1
修改成： 
USE_CUDNN := 1
...
 
...
#如果此处是OpenCV2，则不用修改
将
#OPENCV_VERSION := 3 
修改为： 
OPENCV_VERSION := 3
...
 
...
将
#WITH_PYTHON_LAYER := 1 
修改为 
WITH_PYTHON_LAYER := 1
...
 
...
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib 
修改为： 
INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial
LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial       
...
 
...
将
CUDA_ARCH := -gencode arch=compute_20,code=sm_20 \
        -gencode arch=compute_20,code=sm_21 \
        -gencode arch=compute_30,code=sm_30 \
        -gencode arch=compute_35,code=sm_35 \
        -gencode arch=compute_50,code=sm_50 \
        -gencode arch=compute_52,code=sm_52 \
        -gencode arch=compute_60,code=sm_60 \
        -gencode arch=compute_61,code=sm_61 \
        -gencode arch=compute_61,code=compute_61
修改为
CUDA_ARCH := -gencode arch=compute_30,code=sm_30 \
        -gencode arch=compute_35,code=sm_35 \
        -gencode arch=compute_50,code=sm_50 \
        -gencode arch=compute_52,code=sm_52 \
        -gencode arch=compute_60,code=sm_60 \
        -gencode arch=compute_61,code=sm_61 \
        -gencode arch=compute_61,code=compute_61
...
```

### 修改 caffe 目录下的 Makefile 文件

```bash
...
将：
NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)
替换为：
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
...
 
...
将：
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5
改为：
LIBRARIES += glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial
...
```

### 修改 `/usr/local/cuda/include/host_config.h` 文件 

```bash
将
#error-- unsupported GNU version! gcc versions later than 4.9 are not supported!
改为
//#error-- unsupported GNU version! gcc versions later than 4.9 are not supported!
```

### 编译

```bash
#jn  就是cpu的核心数，j8也就是八核
sudo make all -j8
测试，上面成功了也可以不用测试，测试挺费时间的。
sudo make runtest -j8
```

### 复制文件

在`openpose/3rdparty/caffe/`目录下，终端操作：

```bash
protoc src/caffe/proto/caffe.proto --cpp_out=.
mkdir include/caffe/proto
mv src/caffe/proto/caffe.pb.h include/caffe/proto/
```

## 利用Cmake Gui 生成build文件

- 创建build目录

```bash
cd openpose
mkdir build
```

- 打开cmake软件，填写openpose源码目录以及build

- 点击Configure按钮, 选择Unix Makefile和use default native compling，点击finish按钮 

![cmake](https://files.mdnice.com/user/3650/b2f9e07e-1b01-48f9-87f0-72d5b1d900aa.png)

- 点击Generate按钮

- 过程中无报错，且出现configuring done，中间会出现一些红色的可配置项。

- 接着配置caffe编译路径 

- 如上图所示caffe编译后的目录项填写完成

- 最后点击Generate按钮

## 编译安装

```bash
cd openpose/build/
make -j`nproc`
sudo make install
```

# Python API

OpenPose 的 Python API，需要在 CMake GUI 中设置BUILD_PYTHON.

修改python文件：

```bash
cd openpose/examples/tutorial_api_python
vim 01_body_from_image.py
```

修改第38行`params["model_folder"] = "../../../models/"`为

```bash
params["model_folder"] = "../../models/"

# If you run `make install` (default path is `/usr/local/python` for Ubuntu), you can also access the OpenPose/python module from there. This will install OpenPose and the python library at your desired installation path. Ensure that this is in your python path in order to use it.
sys.path.append('/usr/local/python')
```

修改第33行

```bash
parser.add_argument("--image_path", default="../../../examples/media/COCO_val2014_000000000192.jpg", help="Process an image. Read all standard formats (jpg, png, bmp, etc.).")
```

为

```bash
parser.add_argument("--image_path", default="../../examples/media/COCO_val2014_000000000192.jpg", help="Process an image. Read all standard formats (jpg, png, bmp, etc.).")
```

此外，使用`body_25`模型可能出错，换成其它模型，如`COCO`成功。

![kun](https://files.mdnice.com/user/3650/95a019e1-2afc-4da2-938b-a49bb4a65575.png)

在此，贴出我的修改代码：

```bash
# From Python
# It requires OpenCV installed for Python
import sys
import cv2
import os
from sys import platform
import argparse

try:
    # Import Openpose (Windows/Ubuntu/OSX)
    dir_path = os.path.dirname(os.path.realpath(__file__))
    try:
        # Windows Import
        if platform == "win32":
            # Change these variables to point to the correct folder (Release/x64 etc.)
            sys.path.append(dir_path + '/../../python/openpose/Release');
            os.environ['PATH']  = os.environ['PATH'] + ';' + dir_path + '/../../x64/Release;' +  dir_path + '/../../bin;'
            import pyopenpose as op
        else:
            # Change these variables to point to the correct folder (Release/x64 etc.)
            # sys.path.append('../../python');
            # If you run `make install` (default path is `/usr/local/python` for Ubuntu), you can also access the OpenPose/python module from there. This will install OpenPose and the python library at your desired installation path. Ensure that this is in your python path in order to use it.
            sys.path.append('/usr/local/python')
            from openpose import pyopenpose as op
    except ImportError as e:
        print('Error: OpenPose library could not be found. Did you enable `BUILD_PYTHON` in CMake and have this Python script in the right folder?')
        raise e

    # Flags
    parser = argparse.ArgumentParser()
    parser.add_argument("--image_path", default="/home/zhoukanglei/local/openpose/examples/media/COCO_val2014_000000000192.jpg", help="Process an image. Read all standard formats (jpg, png, bmp, etc.).")
    args = parser.parse_known_args()

    print(args)

    # Custom Params (refer to include/openpose/flags.hpp for more parameters)
    params = dict()
    params["model_folder"] = "/home/zhoukanglei/local/openpose/models/"
    params["model_pose"] = "COCO"

    # Add others in path?
    for i in range(0, len(args[1])):
        curr_item = args[1][i]
        if i != len(args[1])-1: next_item = args[1][i+1]
        else: next_item = "1"
        if "--" in curr_item and "--" in next_item:
            key = curr_item.replace('-','')
            if key not in params:  params[key] = "1"
        elif "--" in curr_item and "--" not in next_item:
            key = curr_item.replace('-','')
            if key not in params: params[key] = next_item

    # Construct it from system arguments
    # op.init_argv(args[1])
    # oppython = op.OpenposePython()

    # Starting OpenPose
    opWrapper = op.WrapperPython()
    opWrapper.configure(params)
    opWrapper.start()

    # Process Image
    datum = op.Datum()
    imageToProcess = cv2.imread(args[0].image_path)
    datum.cvInputData = imageToProcess
    opWrapper.emplaceAndPop(op.VectorDatum([datum]))

    # Display Image
    print("Body keypoints: \n" + str(datum.poseKeypoints))
    cv2.imshow("OpenPose 1.7.0 - Tutorial Python API", datum.cvOutputData)
    cv2.waitKey(0)
except Exception as e:
    print(e)
    sys.exit(-1)
```

更多应用，欢迎留言讨论。
