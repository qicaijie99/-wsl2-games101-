# 在WSL Ubuntu 中搭建games101环境配置指南


```markdown
# 在WSL Ubuntu 中搭建games101环境配置指南

本指南提供在Windows系统中配置WSL Ubuntu环境的完整流程，包含C++编译工具链、Eigen数学库和OpenCV计算机视觉库的安装。

## 系统要求
- 推荐 Windows 11
- 至少 8GB RAM（OpenCV编译推荐）
- 20GB 可用磁盘空间
- 稳定网络连接
- 确保已启用windows虚拟化功能

## 1. 安装WSL和Ubuntu

```bash
# 以管理员身份打开CMD/PowerShell
wsl --install              # 安装WSL基础组件，多数电脑默认就会安装ubuntu
wsl --set-default-version 2  # 设置为WSL 2版本

# 查看可用Linux发行版
wsl --list --online

# 安装Ubuntu LTS版本
wsl --install -d Ubuntu-22.04

# 启动Ubuntu并创建用户账户
wsl -d Ubuntu-22.04
```

## 2. 安装基础开发工具

```bash
# 更新软件源 重要！
sudo apt-get update && sudo apt-get upgrade -y

# 安装G++编译器和构建工具
sudo apt-get install build-essential -y
g++ --version  # 用于验证安装 

# 安装CMake
sudo apt-get install cmake -y
cmake --version  # 验证安装 (3.22+)

# 安装Eigen3数学库
sudo apt-get install libeigen3-dev -y
```

## 3. 安装OpenCV依赖项

```bash
# 安装核心编译依赖
sudo apt-get install -y build-essential

# 安装GUI和图像处理依赖
sudo apt-get install -y libgtk2.0-dev libcanberra-gtk-module

# 安装视频编解码依赖
sudo apt-get install -y libavcodec-dev libavformat-dev

# 安装图像格式支持
sudo apt-get install -y libjpeg-dev libtiff5-dev libswscale-dev

# 安装开发工具
sudo apt-get install -y pkg-config unzip

# 安装附加依赖
sudo apt-get install -y libpng-dev libopenexr-dev libgstreamer-plugins-base1.0-dev
```

## 4. 编译安装OpenCV

```bash

# 下载OpenCV源码
<a href="https://github.com/opencv/opencv/releases/tag/4.12.0" target="_blank">OpenCV 4.12.0 Release</a>

# 解压源码（将压缩包下载到本地后，需要复制到当前Ubuntu里的位置再进行解压）
sudo unzip opencv.zip

# 创建构建目录
cd opencv-xxx
mkdir build && cd build

# 配置CMake
sudo cmake \
  -D CMAKE_BUILD_TYPE=RELEASE \
  -D CMAKE_INSTALL_PREFIX=/usr/local \
  -D OPENCV_GENERATE_PKGCONFIG=ON \
  -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib-4.9.0/modules \
  -D WITH_GTK=ON \
  -D WITH_FFMPEG=ON \
  -D BUILD_EXAMPLES=OFF ..

# 编译安装（根据CPU核心数调整-j参数，例如主包使用的9950x为16核32线程）
sudo make -j32
sudo make install
```

## 5. 配置系统环境

```bash
# 添加库路径
sudo sh -c "echo '/usr/local/lib' >> /etc/ld.so.conf.d/opencv.conf"
sudo ldconfig  # 更新库缓存

# 设置pkg-config路径
echo 'export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig' | sudo tee -a /etc/bash.bashrc
source /etc/bash.bashrc  # 立即生效

# 验证配置
pkg-config --cflags opencv4
```

## 6. 验证安装

```bash
#运行作业1
 /home/作业1$ mkdir build
 /home/作业1$ cd build/
 /home/作业1/build$ cmake ..
 /home/作业1/build$ make
 /home/作业1/build$ ./XXX
```
## 常见问题解决

### 1. 依赖安装失败
```bash
# 更新软件源后重试
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install -f  # 修复损坏的依赖
```

### 2. OpenCV编译错误
- 确保安装所有依赖项
- 清除build目录重新编译：`cd build && sudo make clean`
- 减少编译线程：`sudo make -j4`

### 3. GUI显示问题
```bash
# 在Windows端安装X服务器（推荐VcXsrv）
# 在WSL中设置显示环境变量
export DISPLAY=$(awk '/nameserver / {print $2}' /etc/resolv.conf):0
```

### 4. 环境变量未生效
```bash
source ~/.bashrc
exec bash  # 重新加载shell
```

## 性能优化建议

```ini
# 创建或修改 %USERPROFILE%\.wslconfig
[wsl2]
memory=8GB   # 分配8GB内存
processors=4 # 分配4个CPU核心
swap=4GB     # 分配4GB交换空间
```

## 开发工具推荐
1. **VS Code**：安装Remote - WSL扩展进行开发
2. **CLion**：支持WSL环境的专业C++ IDE
3. **Xming/VcXsrv**：Windows端X服务器用于GUI显示

> **编译时间说明**：完整编译OpenCV可能需要30-90分钟，取决于硬件配置。建议在系统空闲时执行。

---

