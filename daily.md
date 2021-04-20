## vim Tips
### 1 move
ctr+o / ctr+i : 跳转到上/下一次
ctr+]         : 进入函数


## 2020/10/09
替换windows路径
```sh
sed -i 's|\\|/|g' ./result.txt
sed -i 's|D:/Datasets/guangtie/LessImages/dense|/home/ld/Documents/modelFiles/123/mvs|g' ./result.txt
```

## 2020/10/11

### 1 使用OpenMVS贴纹理
#### 1.1 文件准备
the dir which has result.out(camera's info), should have a result.txt which demostrate where is the images
attention: result.txt should be the imagelist file;
```sh
[nash5 mvs]# head -n 5 result.txt 
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0146.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0132.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0133.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0134.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0141.JPG
[nash5 mvs]# head -n 10 result.out
# Bundle file v0.3
63 66674
4318.92 0 0
-0.999016 0.0442186 0.003457
0.0197872 0.514087 -0.85751
-0.0396951 -0.856598 -0.514456
-0.285562 3.57279 -1.96389
4318.92 0 0
-0.999732 0.00834863 0.0216004
-0.0141811 0.5167 -0.856049
```
#### 1.2 使用OpenMVS转换文件格式然后贴纹理
```sh
/home/ld/prjs/OpenMVS/openMVS_build/bin/InterfaceVisualSFM -i result.out -o new
/home/ld/prjs/OpenMVS/openMVS_build/bin/TextureMesh --mesh-file ./result_dense_mesh_refined.ply -i new.mvs -o textured
```

### 2 使用MVE和texRecon贴纹理
#### 2.0 MVE和texRecon安装
[https://www.gcc.tu-darmstadt.de/home/proj/texrecon/index.en.jsp](https://www.gcc.tu-darmstadt.de/home/proj/texrecon/index.en.jsp)
在使用texRecon的时候, 注意如下:
texrecon/elibs/mve/libs/mve/camera.cc +125的
<font color=#FF0000> void CameraInfo::fill_calibration </font>函数

其ax乘的flen是单位化以后的flen， 单位化的flen的方式见[https://github.com/simonfuhrmann/mve/wiki/Math-Cookbook](https://github.com/simonfuhrmann/mve/wiki/Math-Cookbook): 
```cpp
void
CameraInfo::fill_calibration (float* mat, float width, float height) const
{
    float dim_aspect = width / height;
    float image_aspect = dim_aspect * this->paspect;
    float ax, ay;
    if (image_aspect < 1.0f) /* Portrait. */
    {
        ax = this->flen * height / this->paspect;
        ay = this->flen * height;
    }
    else /* Landscape. */
    {
        ax = this->flen * width;
        ay = this->flen * width * this->paspect;
    }

    mat[0] = this->flen; mat[1] =       0.0f; mat[2] =  width * this->ppoint[0];
    mat[3] =       0.0f; mat[4] = this->flen; mat[5] = height * this->ppoint[1];
    mat[6] =       0.0f; mat[7] =       0.0f; mat[8] =                     1.0f;

    //mat[0] =   ax; mat[1] = 0.0f; mat[2] = width * this->ppoint[0];
    //mat[3] = 0.0f; mat[4] =   ay; mat[5] = height * this->ppoint[1];
    //mat[6] = 0.0f; mat[7] = 0.0f; mat[8] = 1.0f;
}
```

#### 2.1 对相机数据进行预处理然后转化格式
- 1 你会发现相机的平移和旋转矩阵的后两行都添加了负号(使用meshlab打开看相机的朝向和result.out的rot[2][2] (代表相机朝向的是z正方向还是负方向))， 按照需要改过来并且转换为MVE支持的一种格式(详细见2.2),该bundle file的详细格式可以参考：[https://www.cs.cornell.edu/~snavely/bundler/bundler-v0.4-manual.html](https://www.cs.cornell.edu/~snavely/bundler/bundler-v0.4-manual.html)
```sh
### ori camfile from colmap:
[nash5 123]# head -n 10 result.out 
# Bundle file v0.3
63 66674                        # imageNum pointsNum
4318.92 0 0                     # focal_len disortionCoe1 distortionCoe2
-0.999732 0.00834863 0.0216004  # rot[0][0:2]
-0.0141811 0.5167 -0.856049     # rot[1][0:2]
-0.0183078 -0.856126 -0.516443  # rot[2][0:2]
1.99173 4.31377 -2.39484        # translation[0:2]
4318.92 0 0
-0.999016 0.0442186 0.003457
0.0197872 0.514087 -0.85751

### 转换后（针对于一个数据）
[nash5 123]# cd scene1006/
[nash5 scene1006]# cat DJI_0132.CAM 
1.99173 -4.31377 2.39484 -0.999732 0.00834863 0.0216004 .0141811 -.5167 .856049 .0183078 .856126 .516443
4318.92 0 0 1 0.5 0.5
```
#### 2.2 文件格式准备:
首先需要有4个文件：
```sh
result_dense_mesh_refined.ply # 需要贴图的mesh
result.imagelist              # 贴图的图片的位置（views）
result.out                    # 每个图片（view）的相机参数
images/*.JPG                  # 图片
[nash5 123]# head -n 5 result.imagelist
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0146.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0132.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0133.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0134.JPG
/home/ld/Documents/modelFiles/123/mvs/images/DJI_0141.JPG
```
- 1 转化后的数据(转化要求详见: texRecon -h或mve在github上的math部分)例子：使用脚本将result.out作为输入, 为每一个图片生成一个.CAM文件：(可以参照：[https://github.com/xychen5/fft/blob/master/colmapToTexrecon/turnYZ_in_camera_rot_and_translation.sh](https://github.com/xychen5/fft/blob/master/colmapToTexrecon/turnYZ_in_camera_rot_and_translation.sh))
```sh
### 转换前（使用texrecon -h获得它需要的文件格式）
[nash5 123]# head -n 10 result.out 
# Bundle file v0.3
63 66674                        # imageNum pointsNum
4318.92 0 0                     # focal_len disortionCoe1 distortionCoe2
-0.999732 0.00834863 0.0216004  # rot[0][0:2]
-0.0141811 0.5167 -0.856049     # rot[1][0:2]
-0.0183078 -0.856126 -0.516443  # rot[2][0:2]
1.99173 4.31377 -2.39484        # translation[0:2]
4318.92 0 0
-0.999016 0.0442186 0.003457
0.0197872 0.514087 -0.85751
### 转换后（针对于一个数据）
[nash5 123]# cd scene1006/
[nash5 scene1006]# cat DJI_0132.CAM 
1.99173 -4.31377 2.39484 -0.999732 0.00834863 0.0216004 .0141811 -.5167 .856049 .0183078 .856126 .516443
4318.92 0 0 1 0.5 0.5
```
- 2 经过step1获得如下一个文件夹：（对应的每一个图片都有一个相机文件）
```sh
[nash5 reRun]# ls scene1006/
 DJI_0132.CAM   DJI_0145.CAM   DJI_0188.CAM      'DJI_0303(1).CAM'   DJI_0327.CAM   DJI_0334.CAM   DJI_0341.CAM   DJI_0348.CAM   DJI_0367.CAM
 DJI_0132.JPG   DJI_0145.JPG   DJI_0188.JPG      'DJI_0303(1).JPG'   DJI_0327.JPG   DJI_0334.JPG   DJI_0341.JPG   DJI_0348.JPG   DJI_0367.JPG
```
- 3 将该文件夹作为texRecon的输入即可：(耗时较长，几十分钟到几个小时，看数据和机器)
```sh
/home/ld/prjs/recon3d/texRecon/texrecon/cmake-build-debug/apps/texrecon/texrecon scene1006 result_dense_mesh_refined.ply textured2 --skip_geometric_visibility_test
```

## 2020/10/13
margin probality

## 2020/10/19
- 3 注释补齐插件 for vim
- 2 BP算法详细博文
- 1 导出osgb然后查看

bug:       关注读取图片后，m_image_data结构体的值: 在出入函数前后，本来被修改的图片的宽高数据在出函数后却是0。
fix:          因为真实的处理逻辑确实如此，主要是因为在快出函数的时候有一个函数又将宽高数据更改成0了。
suggest: debug的时候一定每一个流程都要认真看，尤其是经过了函数，务必进入，然后就是少用debugger（只定位问题），直接看代码效率高。

## 2020/10/24
pac __SuiteSparse__. Needed for solving large sparse linear systems. Optional; strongly recomended for large scale bundle adjustment
pac __TBB__ is a C++11 template library for parallel programming that optionally can be used as an alternative to OpenMP. Optional

CGAL的依赖很多，QT在E:/360Downloads里面有

## 2020/10/25

### 安装GMP和MFPR
- 1 安装Cygwin
```log
Setup Environment
Install Cygwin, add the following packages to the default installation
gcc-core
gcc-g++
libgcc
m4
make #不知为何安装后在Cygwin根目录下搜不到make程序
cmake
bash
Add the following Environment Variable to the User PATH: C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\VC\Tools\MSVC\14.15.26726\bin\Hostx64\x64
This is so you can use the lib command. Your lib.exe may be located elsewhere.
```
安装好Cygwin以及一些依赖以后，在其根目录(方便说明记为： CygwinRoot="D:\CygwinRooto")下的bin/minnty.exe是其终端入口，然后每次打开该终端，进入的是：$CygwinRoot/home/$(userName)， 运行"cd /"后就可以理解了； 

- 2 下载并安装make
  - 2.1 从如下网址下载make的源码，https://ftp.gnu.org/gnu/make/，然后解压
  - 2.2 打开Cygwin64 Terminal命令行，进入源码根目录，然后运行：configure && ./build.sh
  - 2.3 编译得到了make.exe后将其移动到Cygwin的bin目录下

- 3 编译gmp
运行两个： ./configure 和 make install
```sh
./configure --prefix=/home/chenxy/mylibs/newTry/gmp-6.2.0/build/static --enable-static --disable-shared
configure: summary of build options:

  Version:           GNU MP 6.2.0
  Host type:         skylake-pc-cygwin
  ABI:               64
  Install prefix:    /home/chenxy/mylibs/newTry/gmp-6.2.0/build/static
  Compiler:          gcc
  Static libraries:  yes
  Shared libraries:  no
```
编译结果（默认生成的是static的数据）：
```log
@nash-5 ~/mylibs/gmp-6.2.0
$ ls /usr/local/include/
gmp.h

@nash-5 ~/mylibs/gmp-6.2.0
$ ls /usr/local/lib/
libgmp.a  libgmp.la  pkgconfig
```
生成动态连接库（注意： 动态连接库和静态连接库的.h文件不同，所以注意分成2个文件夹，至少对于gmp是如此）：
```sh
./configure --prefix=/home/chenxy/mylibs/gmp-6.2.0/build/shared --enable-shared --disable-static
```
- 4 编译mfpr（需要gmp的依赖，而且是动态连接库）
进入mfpr的根目录：
运行./configure：
```log
checking for gmp.h... no
configure: error: gmp.h can't be found, or is unusable.
```
运行./configure --help
```sh
···
  --with-gmp-include=DIR  GMP include directory
  --with-gmp-lib=DIR      GMP lib directory
···
```
所以：
```sh
./configure --prefix=/home/chenxy/mylibs/newTry/mpfr-4.1.0/build/static \
--enable-static --disable-shared \
--with-gmp-include=/home/chenxy/mylibs/newTry/gmp-6.2.0/build/static/include \
--with-gmp-lib=/home/chenxy/mylibs/newTry/gmp-6.2.0/build/staic/lib

make install
```
```sh
./configure --prefix=/home/chenxy/mylibs/mpfr-4.1.0/build/static \
--with-gmp-include=/home/chenxy/mylibs/gmp-6.2.0/build/static/include \
--with-gmp-lib=/home/chenxy/mylibs/gmp-6.2.0/build/static/lib \
--enable-static --disable-shared
```

### cmake查看debug信息辅助cmake配置
以boost错误为例：
在（cmake-gui中）配置了变量：Boost_INCLUDE_DIR=Z:/BASE_ENV/forOpenMVS/boost_1_74_0后依然出错
```log
CMake Error at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:165 (message):
  Could NOT find Boost (missing: Boost_INCLUDE_DIR iostreams program_options
  system serialization)
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:458 (_FPHSA_FAILURE_MESSAGE)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2177 (find_package_handle_standard_args)
  CMakeLists.txt:122 (FIND_PACKAGE)
```
则提供解决bug的思路：
- 1 首先，到cmake的安装目录里查看文件： _CMake/share/cmake-3.18/Modules/FindBoost.cmake_， 其定义了Boost的各个参数分别代表什么意思： 根据每个变量对应的实际意义重新配置一下即可；例如boost，主要配置两个参数： 
>  - Boost_INCLUDE_DIR: 含有boost头文件的目录
>  - Boost_LIBRARYDIR: 偏好的含有boost库的库目录

- 2 若配置完仍然有问题，可以根据 _CMake/share/cmake-3.18/Modules/FindBoost.cmake_ 文件里规定的debug参数： Boost_DEBUG=ON(在cmake-gui中则是添加一个entery，然后其bool值为true即可，直接在CMakeLists.txt里，则是添加： set(Boost_DEBUG ON) 就会打开调试信息)，根据调试信息再去对比生成的库的文件名称和搜索目录等信息，如下给出样例的调试信息：
```log
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1491 ] _boost_TEST_VERSIONS = "1.73.0;1.73;1.72.0;1.72;1.71.0;1.71;1.70.0;1.70;1.69.0;1.69;1.68.0;1.68;1.67.0;1.67;1.66.0;1.66;1.65.1;1.65.0;1.65;1.64.0;1.64;1.63.0;1.63;1.62.0;1.62;1.61.0;1.61;1.60.0;1.60;1.59.0;1.59;1.58.0;1.58;1.57.0;1.57;1.56.0;1.56;1.55.0;1.55;1.54.0;1.54;1.53.0;1.53;1.52.0;1.52;1.51.0;1.51;1.50.0;1.50;1.49.0;1.49;1.48.0;1.48;1.47.0;1.47;1.46.1;1.46.0;1.46;1.45.0;1.45;1.44.0;1.44;1.43.0;1.43;1.42.0;1.42;1.41.0;1.41;1.40.0;1.40;1.39.0;1.39;1.38.0;1.38;1.37.0;1.37;1.36.1;1.36.0;1.36;1.35.1;1.35.0;1.35;1.34.1;1.34.0;1.34;1.33.1;1.33.0;1.33"
<被作者省略>
boost_1_74_0/../lib64-msvc-14.0;Z:/BASE_ENV/forOpenMVS/boost/lib64-msvc-14.0;PATHS;C:/boost/lib;C:/boost;/sw/local/lib"
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1871 ] _boost_LIBRARY_SEARCH_DIRS_DEBUG = "Z:/BASE_ENV/forOpenMVS/boost_1_74_0/lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/../lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/stage/lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/../lib64-msvc-14.1;Z:/BASE_ENV/forOpenMVS/local/lib"
<被作者省略>
CMake Warning at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1187 (message):
  New Boost version may have incorrect or missing dependencies and imported
  targets
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1311 (_Boost_COMPONENT_DEPENDENCIES)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1919 (_Boost_MISSING_DEPENDENCIES)
  CMakeLists.txt:122 (FIND_PACKAGE)


CMake Warning at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1187 (message):
  New Boost version may have incorrect or missing dependencies and imported
  targets
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1311 (_Boost_COMPONENT_DEPENDENCIES)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1919 (_Boost_MISSING_DEPENDENCIES)
  CMakeLists.txt:122 (FIND_PACKAGE)


CMake Warning at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1187 (message):
  New Boost version may have incorrect or missing dependencies and imported
  targets
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1311 (_Boost_COMPONENT_DEPENDENCIES)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1919 (_Boost_MISSING_DEPENDENCIES)
  CMakeLists.txt:122 (FIND_PACKAGE)


CMake Warning at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1187 (message):
  New Boost version may have incorrect or missing dependencies and imported
  targets
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1311 (_Boost_COMPONENT_DEPENDENCIES)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1919 (_Boost_MISSING_DEPENDENCIES)
  CMakeLists.txt:122 (FIND_PACKAGE)


CMake Warning at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1187 (message):
  New Boost version may have incorrect or missing dependencies and imported
  targets
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1311 (_Boost_COMPONENT_DEPENDENCIES)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1919 (_Boost_MISSING_DEPENDENCIES)
  CMakeLists.txt:122 (FIND_PACKAGE)


[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2056 ] Searching for IOSTREAMS_LIBRARY_RELEASE: boost_iostreams-vc141-mt-x64-1_74;boost_iostreams-vc141-mt-x64;boost_iostreams-vc141-mt;boost_iostreams-vc140-mt-x64-1_74;boost_iostreams-vc140-mt-x64;boost_iostreams-vc140-mt;boost_iostreams-mt-x64-1_74;boost_iostreams-mt-x64;boost_iostreams-mt;boost_iostreams-mt;boost_iostreams
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2111 ] Searching for IOSTREAMS_LIBRARY_DEBUG: boost_iostreams-vc141-mt-gd-x64-1_74;boost_iostreams-vc141-mt-gd-x64;boost_iostreams-vc141-mt-gd;boost_iostreams-vc140-mt-gd-x64-1_74;boost_iostreams-vc140-mt-gd-x64;boost_iostreams-vc140-mt-gd;boost_iostreams-mt-gd-x64-1_74;boost_iostreams-mt-gd-x64;boost_iostreams-mt-gd;boost_iostreams-mt;boost_iostreams
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2056 ] Searching for PROGRAM_OPTIONS_LIBRARY_RELEASE: boost_program_options-vc141-mt-x64-1_74;boost_program_options-vc141-mt-x64;boost_program_options-vc141-mt;boost_program_options-vc140-mt-x64-1_74;boost_program_options-vc140-mt-x64;boost_program_options-vc140-mt;boost_program_options-mt-x64-1_74;boost_program_options-mt-x64;boost_program_options-mt;boost_program_options-mt;boost_program_options
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2111 ] Searching for PROGRAM_OPTIONS_LIBRARY_DEBUG: boost_program_options-vc141-mt-gd-x64-1_74;boost_program_options-vc141-mt-gd-x64;boost_program_options-vc141-mt-gd;boost_program_options-vc140-mt-gd-x64-1_74;boost_program_options-vc140-mt-gd-x64;boost_program_options-vc140-mt-gd;boost_program_options-mt-gd-x64-1_74;boost_program_options-mt-gd-x64;boost_program_options-mt-gd;boost_program_options-mt;boost_program_options
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2056 ] Searching for SYSTEM_LIBRARY_RELEASE: boost_system-vc141-mt-x64-1_74;boost_system-vc141-mt-x64;boost_system-vc141-mt;boost_system-vc140-mt-x64-1_74;boost_system-vc140-mt-x64;boost_system-vc140-mt;boost_system-mt-x64-1_74;boost_system-mt-x64;boost_system-mt;boost_system-mt;boost_system
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2111 ] Searching for SYSTEM_LIBRARY_DEBUG: boost_system-vc141-mt-gd-x64-1_74;boost_system-vc141-mt-gd-x64;boost_system-vc141-mt-gd;boost_system-vc140-mt-gd-x64-1_74;boost_system-vc140-mt-gd-x64;boost_system-vc140-mt-gd;boost_system-mt-gd-x64-1_74;boost_system-mt-gd-x64;boost_system-mt-gd;boost_system-mt;boost_system
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2056 ] Searching for SERIALIZATION_LIBRARY_RELEASE: boost_serialization-vc141-mt-x64-1_74;boost_serialization-vc141-mt-x64;boost_serialization-vc141-mt;boost_serialization-vc140-mt-x64-1_74;boost_serialization-vc140-mt-x64;boost_serialization-vc140-mt;boost_serialization-mt-x64-1_74;boost_serialization-mt-x64;boost_serialization-mt;boost_serialization-mt;boost_serialization
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2111 ] Searching for SERIALIZATION_LIBRARY_DEBUG: boost_serialization-vc141-mt-gd-x64-1_74;boost_serialization-vc141-mt-gd-x64;boost_serialization-vc141-mt-gd;boost_serialization-vc140-mt-gd-x64-1_74;boost_serialization-vc140-mt-gd-x64;boost_serialization-vc140-mt-gd;boost_serialization-mt-gd-x64-1_74;boost_serialization-mt-gd-x64;boost_serialization-mt-gd;boost_serialization-mt;boost_serialization
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2056 ] Searching for REGEX_LIBRARY_RELEASE: boost_regex-vc141-mt-x64-1_74;boost_regex-vc141-mt-x64;boost_regex-vc141-mt;boost_regex-vc140-mt-x64-1_74;boost_regex-vc140-mt-x64;boost_regex-vc140-mt;boost_regex-mt-x64-1_74;boost_regex-mt-x64;boost_regex-mt;boost_regex-mt;boost_regex
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2111 ] Searching for REGEX_LIBRARY_DEBUG: boost_regex-vc141-mt-gd-x64-1_74;boost_regex-vc141-mt-gd-x64;boost_regex-vc141-mt-gd;boost_regex-vc140-mt-gd-x64-1_74;boost_regex-vc140-mt-gd-x64;boost_regex-vc140-mt-gd;boost_regex-mt-gd-x64-1_74;boost_regex-mt-gd-x64;boost_regex-mt-gd;boost_regex-mt;boost_regex
CMake Error at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:165 (message):
  Could NOT find Boost (missing: iostreams program_options system
  serialization) (found version "1.74.0")
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:458 (_FPHSA_FAILURE_MESSAGE)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2177 (find_package_handle_standard_args)
  CMakeLists.txt:122 (FIND_PACKAGE)
```



## 2020/10/26 - 2020/10/28
### vs2019编译boost
进入boost解压的根目录，执行：
```sh
# 生成b2.exe
./bootstrap.bat

# 编译指定的工具集的库的版本(可能出现没有cl和cstddef的问题，见后面的注意)
.\b2.exe  --toolset=msvc-14.2  `
--address-model=64 `
--architecture=x86 `
--threading=multi `
--with-iostreams --with-program_options --with-system --with-serialization `
stage --stagedir="F:\BASE_ENV\forOpenMVS\boost_1_74_0\forOpenMVS"

# 可能需要使用boost追加编译zlib的库(github下源码)，如下：
.\b2.exe  --toolset=msvc-14.2  `
--address-model=64 `
--architecture=x86 `
--threading=multi `
--with-iostreams -sZLIB_SOURCE="F:\BASE_ENV\forOpenMVS\zlib-1.2.11" -sZLIB_INCLUDE="F:\BASE_ENV\forOpenMVS\zlib-1.2.11" `
--link=static --runtime-link=static `
stage --stagedir="F:\BASE_ENV\forOpenMVS\boost_1_74_0\forOpenMVS"

```


__注意： 出现找不到cl和cstddef的问题多半是环境变量的问题，编辑如下环境变量：__
- 1 path中添加(运行cl.exe): C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\bin\Hostx64\x64
- 2 添加变量: INCLUDE, 值: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\include;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\shared;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\winrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\ucrt;
- 3 添加变量: LIB, 值: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\lib\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.18362.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.18362.0\ucrt\x64;


### 编译OpenMVS
按照其官方的buildingWilki[https://github.com/cdcseacave/openMVS/wiki/Building](https://github.com/cdcseacave/openMVS/wiki/Building)将相关的依赖库放到其同级目录，新建build然后进入，执行的cmake指令如下：
```sh
# 在 windows terminal中执行(cmake -G "Visual Studio 14 Win64" path\to\source\dir)

cmake . ..\openMVS\ `
-G "Visual Studio 16 2019" `
-DVCG_ROOT="..\VCG" `
-DBOOST_INCLUDEDIR="F:/BASE_ENV/forOpenMVS/boost_1_74_0" `
-DBOOST_LIBRARYDIR="F:/BASE_ENV/forOpenMVS/boost_1_74_0/forOpenMVS" `
-DOpenCV_DIR="F:/BASE_ENV/forOpenMVS/opencv/build" `
-DGMP_INCLUDE_DIR="F:\BASE_ENV\forOpenMVS\gmp_mpfr\include" `
-DGMP_LIBRARIES="F:\BASE_ENV\forOpenMVS\gmp_mpfr\lib\libgmp-10.lib" `
-DMPFR_INCLUDE_DIR="F:\BASE_ENV\forOpenMVS\gmp_mpfr\include" `
-DMPFR_LIBRARIES="F:\BASE_ENV\forOpenMVS\gmp_mpfr\lib\libmpfr-4.lib" `
-DEIGEN_ROOT="F:\BASE_ENV\forOpenMVS\eigen" `
-DCERES_ROOT="F:\BASE_ENV\forOpenMVS\ceres-solver\build" `
-DJPEG_INCLUDE_DIRS="F:\BASE_ENV\forOpenMVS\jpegsr9" `
-DJPEG_LIBRARIES="F:\BASE_ENV\forOpenMVS\jpegsr9\libjpeg.lib"

```

生成sln文件后，报错日志如下：
```log
...<作者省略>
boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
...<作者省略>
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const 
6>libgmp.a(dcpi1_bdiv_qr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
...<作者省略>
6>Done building project "ReconstructMesh.vcxproj" -- FAILED.
8>Done building project "TextureMesh.vcxproj" -- FAILED.
4>Finished generating code
4>InterfaceCOLMAP.vcxproj -> F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\InterfaceCOLMAP.exe
9>------ Skipped Build: Project: INSTALL, Configuration: Release x64 ------
9>Project not selected to build for this solution configuration 
========== Build: 1 succeeded, 5 failed, 6 up-to-date, 3 skipped ==========
```

以上日志说明两个问题：
- 1: gmp和mpfr库有问题 -> 使用cgal的setup安装，然后进入auxiliary/gmp中有gmp和mpfr的库。
- 2: 编译的boost的iostream库with zlib有问题，zlib说有些符号无法解析:
  - 注意： boost的编译，会将编译结果存在build_dir（bin.v2）中然后拷贝到指定的lib目录下，这就造成，如果需要新编库，必须手动删除build_dir中生成的库文件，否则新编译库是不会overwrite的，而是直接从build_dir中复制到指定的lib目录下，于是： 编译带有zlib的库的时候，需要先将iostream库删除（指定的lib目录和build_dir中的库），然后重新编译。


## 2020/10/29
### 0 OpenMVS读取图像依赖于jpeg, png等lib, 需要手动编好然后修改一下CMakeList.txt
```sh
# libjepg:文件下载： http://www.ijg.org/files/ (下载其中的jpeg9.zip或者jpeg9a.zip，具体看里面的readme)
解压进入根目录：
cp C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include ./
cp jconfig.vc jconfig.h
nmake -f makefile.vc
```

### 1 使用OpenMVS:
### 1.0 修改OpenMVS以使其支持jpeg
修改：OpenMVS/libs/IO/CMakeLists.txt
SET (JPEG_INCLUDE_DIR "F:/BASE_ENV/forOpenMVS/jpegsr9")
SET (JPEG_LIBRARY "F:/BASE_ENV/forOpenMVS/jpegsr9/libjpeg.lib")

### 1.1 命令行调用OpenMVS
```sh
# 依赖于opencv的几个dll库
-a----       2020/10/24     20:22        3063296 opencv_calib3d450.dll
-a----       2020/10/29      9:50       17019904 opencv_core450.dll
-a----       2020/10/24     20:22         952320 opencv_features2d450.dll
-a----       2020/10/24     20:20         528896 opencv_flann450.dll
-a----       2020/10/24     20:21        3386880 opencv_imgcodecs450.dll
-a----       2020/10/24     20:21       29256192 opencv_imgproc450.dll

# 将文件名转小写
for i in $(ls); do cp ${i} ../lowerCaseImages/${i,,}; done

# openMVS贴图使用方式:
F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\InterfaceVisualSFM.exe -i result.out -o result.mvs
F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\TextureMesh.exe --mesh-file result_dense_mesh_simple.ply -i result.mvs -o result.ply
```

## 2020/10/30
### 1 LRU实现：(参考colmap中的utils/cache.h)
least recently used(will be discard)
```log
务必将模板类的声明和实现放在一起！
```

## 2020/11/02 - 11/03

### 1 编译MVE
参考：[https://github.com/simonfuhrmann/mve/wiki/Build-Instructions-for-Windows](https://github.com/simonfuhrmann/mve/wiki/Build-Instructions-for-Windows)
参考该链接完成编译即可，主要问题在于若下载很慢，可能需要你手动将文件下载到对应目录，其次：
__F:\BASE_ENV\forMVE_TEXRecon\mve\3rdparty\qt5\src\qt5\qtbase\qmake\generators\win32\msvc_vcproj.cpp__ 里说明，19版本不支持，使用15以及以上工具编译：
```cpp
QT_BEGIN_NAMESPACE
// Filter GUIDs (Do NOT change these!) ------------------------------
const char _GUIDSourceFiles[]          = "{4FC737F1-C7A5-4376-A066-2A32D752A2FF}";
const char _GUIDHeaderFiles[]          = "{93995380-89BD-4b04-88EB-625FBE52EBFB}";
const char _GUIDGeneratedFiles[]       = "{71ED8ED8-ACB9-4CE9-BBE1-E00B30144E11}";
const char _GUIDResourceFiles[]        = "{D9D6E242-F8AF-46E4-B9FD-80ECBC20BA3E}";
const char _GUIDLexYaccFiles[]         = "{E12AE0D2-192F-4d59-BD23-7D3FA58D3183}";
const char _GUIDTranslationFiles[]     = "{639EADAA-A684-42e4-A9AD-28FC9BCB8F7C}";
const char _GUIDFormFiles[]            = "{99349809-55BA-4b9d-BF79-8FDBB0286EB3}";
const char _GUIDExtraCompilerFiles[]   = "{E0D8C965-CC5F-43d7-AD63-FAEF0BBC0F85}";
const char _GUIDDeploymentFiles[]      = "{D9D6E243-F8AF-46E4-B9FD-80ECBC20BA3E}";
const char _GUIDDistributionFiles[]    = "{B83CAF91-C7BF-462F-B76C-EA11631F866C}";
QT_END_NAMESPACE
```
两次使用的编译命令：
```sh
cmake -G "Visual Studio 14 Win64" .
```
### 2 编译MVS-Texturing (texrecon)
- 2.1 下载tbb库：[https://github.com/oneapi-src/oneTBB/releases/tag/v2020.3](https://github.com/oneapi-src/oneTBB/releases/tag/v2020.3)
将之前编译的mve中的库和tbb都放到texrecon/3rdparty下面：

- 2.2 下载MVS-Texturing(和mve同级目录)使用分支：*__cmake__* -> [https://github.com/andre-schulz/mvs-texturing/tree/cmake](https://github.com/andre-schulz/mvs-texturing/tree/cmake)
配置完成以后，jpeg，tiff，png，zlib，mve_util的库需要在生成texrecon项目的时候在其属性中配置连接器对于这些库输入(会出现link err2019)：
```sh
cmake . ..\ `
-G "Visual Studio 14 Win64" `
-DMVE_INCLUDE_DIRS="F:\BASE_ENV\forMVE_TEXRecon\mve\libs" `
-DTBB_ROOT_DIR="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\tbb" `
-DTBB_INCLUDE_DIRS="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\tbb\include" `
-DTBB_LIBRARIES="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\tbb\lib" `
-DPNG_PNG_INCLUDE_DIR="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\include" `
-DPNG_LIBRARY="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\lib" `
-DZLIB_INCLUDE_DIR="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\include" `
-DZLIB_LIBRARY="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\lib" `
-DJPEG_INCLUDE_DIR="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\include" `
-DJPEG_LIBRARY="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\lib" `
-DTIFF_INCLUDE_DIR="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\include" `
-DTIFF_LIBRARY="F:\BASE_ENV\forMVE_TEXRecon\mvs-texturing\3rdparty\png_tiff_zip_qt5\lib"
```

- 2.3 编译完成后，需要在texrecon.exe同级目录加入：
```log
jpeg62.dll*
libpng16.dll*
tiff.dll*
zlib.dll*
```
__注意： -1，为了适应我们的数据，需要对代码做一定的更改，给入的数据的flen必须是单位话以后的焦长。__
__注意： -2，输入数据的yz需要反转。__
__注意： -3，需要更改mve中的camera代码，在更改以后注意将对应的库替换掉，然后重新编译。或者修改输入的每张图片的相机参数，将其flen除以max(wight, height)__

- 2.4 texrecon使用方式：
```sh
# 使用如下脚本获得mvs-texture(texrecon)规定的数据格式。
sh turnYZ_in_camera_rot_and_translation.sh result.out camArgs 5433

# 运行该程序，注意：在windows下使用scene_dir，不要使用相对路径，使用绝对路径
F:/BASE_ENV/forMVE_TEXRecon/mvs-texturing/build/apps/texrecon/Release/texrecon.exe  --skip_geometric_visibility_test  F:/dataSets/1011OpenMVS/texrecon/scene201103/ result_dense_mesh_refined.ply textured
```

## 2020/11/10
### 1 MRF-based mosaicing(贴图)
主要参考:
[http://www.robots.ox.ac.uk/~vilem/SeamlessMosaicing.pdf](http://www.robots.ox.ac.uk/~vilem/SeamlessMosaicing.pdf)
主要思想：
mrf的每一个节点不是图片的像素，而是mesh中的三角片，然后定义好能量方程，LBP就是使这些能量方程迅速收敛的优化算法。
所谓的能量方程：
- 1 这个view对于这个face看的有多准，多清楚
- 2 face和它周围的faces（周围的faces可能来自于不同的views）之间，会有色差，这也是一方面的能量



## 2020/11/13 - 11/17
> ps: osg的PagedLod依赖于databasePager这个类(proxyNode也是依赖于该类)来实现动态调度，详情见：osg3 cookbook的321面。
> ps: osg中使用行主序矩阵实现矩阵变换，所以对于点的变换，按照前乘实现。

### 1 关于读取osgb文件的方法：
主要修改PrimitiveFunctor，然后更改nodevisitor即可。
A PrimitiveFunctor is used (in conjunction with osg::Drawable::accept (PrimitiveFunctor&)) to get access to the primitives that compose the things drawn by OSG.

标注工具使用labelimage

为什么运行cesium的官方的实例时，需要将使用到的属性和部件放到import中


## 2020/11/21
### 1 使用cesium加载gltf的模型报错：
问题描述：
在fregata项目中尝试添加贴地运动小车

参考实现代码：
https://sandcastle.cesium.com/?src=Clamp%20to%203D%20Tiles.html&label=3D%20Tiles

错误日志： //实际上就是没有按照webpack的配置来配路径而已
```log
RuntimeError {name: "RuntimeError", message: "Failed to load model: Cesium_Air.glb↵Unexpected token < in JSON at position 0", stack: "Error↵    at new RuntimeError (webpack-internal://…e_modules/cesium/Source/ThirdParty/when.js:646:4)"}message: "Failed to load model: Cesium_Air.glb↵Unexpected token < in JSON at position 0"name: "RuntimeError"stack: "Error↵    at new RuntimeError (webpack-internal:///./node_modules/cesium/Source/Core/RuntimeError.js:40:11)↵    at eval (webpack-internal:///./node_modules/cesium/Source/Scene/ModelUtility.js:185:32)↵    at Promise.eval [as then] (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:216:33)↵    at eval (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:296:13)↵    at processQueue (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:646:4)↵    at _resolve (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:332:4)↵    at promiseReject (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:365:11)↵    at Promise.eval [as then] (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:216:33)↵    at eval (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:297:7)↵    at processQueue (webpack-internal:///./node_modules/cesium/Source/ThirdParty/when.js:646:4)"__proto__: 
```

### 1 osg实现物体拖拽
调用osg::Dragger即可；
官方例子： [https://github.com/openscenegraph/OpenSceneGraph/blob/master/examples/osgmanipulator/osgmanipulator.cpp](https://github.com/openscenegraph/OpenSceneGraph/blob/master/examples/osgmanipulator/osgmanipulator.cpp)

osg实现相交代码：
```cpp
bool View::computeIntersections(float x,float y, osgUtil::LineSegmentIntersector::Intersections& intersections, osg::Node::NodeMask traversalMask)
{
    float local_x, local_y;
    const osg::Camera* camera = getCameraContainingPosition(x, y, local_x, local_y);

    OSG_INFO<<"computeIntersections("<<x<<", "<<y<<") local_x="<<local_x<<", local_y="<<local_y<<std::endl;

    if (camera) return computeIntersections(camera, (camera->getViewport()==0)?osgUtil::Intersector::PROJECTION : osgUtil::Intersector::WINDOW, local_x, local_y, intersections, traversalMask);
    else return false;
}

bool View::computeIntersections(const osg::Camera* camera, osgUtil::Intersector::CoordinateFrame cf, float x,float y, const osg::NodePath& nodePath, osgUtil::LineSegmentIntersector::Intersections& intersections,osg::Node::NodeMask traversalMask)
{
    if (!camera || nodePath.empty()) return false;

    osg::Matrixd matrix;
    if (nodePath.size()>1)
    {
        osg::NodePath prunedNodePath(nodePath.begin(),nodePath.end()-1);
        matrix = osg::computeLocalToWorld(prunedNodePath);
    }

    matrix.postMult(camera->getViewMatrix());
    matrix.postMult(camera->getProjectionMatrix());

    double zNear = -1.0;
    double zFar = 1.0;
    if (cf==osgUtil::Intersector::WINDOW && camera->getViewport())
    {
        matrix.postMult(camera->getViewport()->computeWindowMatrix());
        zNear = 0.0;
        zFar = 1.0;
    }

    osg::Matrixd inverse;
    inverse.invert(matrix);

    osg::Vec3d startVertex = osg::Vec3d(x,y,zNear) * inverse;
    osg::Vec3d endVertex = osg::Vec3d(x,y,zFar) * inverse;

    osg::ref_ptr< osgUtil::LineSegmentIntersector > picker = new osgUtil::LineSegmentIntersector(osgUtil::Intersector::MODEL, startVertex, endVertex);

    osgUtil::IntersectionVisitor iv(picker.get());
    iv.setTraversalMask(traversalMask);
    nodePath.back()->accept(iv);

    if (picker->containsIntersections())
    {
        intersections = picker->getIntersections();
        return true;
    }
    else
    {
        intersections.clear();
        return false;
    }
}
```
### 2 自己实现拖拽效果
```cpp
	//used for drag the bounding box
	//osg::ref_ptr<osgViewer::View> pViewer = NULL;
	//Scene *                       pScene = NULL;
	bool                          ifPicked = false;       // if a object is picked
	osg::MatrixTransform *        pPickedObject = NULL;   // the picked object 
	bool                          lButtonDown = false;
	bool                          rButtonDown = false;
	osg::Vec3                     firstIntersectionPoint; // when picking obj, the first intersect point
	float                         z;                      // firstIntersectionPoint's z value in frustrum space
	osg::Vec3                     stPoint;                // drag start point
	osg::Vec3                     endPoint;               // drag end point
	osg::Matrix                   stPos;                  // starting pos
	

	void pickByRay(float x, float y);
	osg::Vec3 screen2World(float x, float y);
	osg::Vec3 world2Screen(osg::Vec3& wV);


//事件处理函数
bool CPickHandler::handle(const osgGA::GUIEventAdapter& ea, osgGA::GUIActionAdapter& us)
{
	osgViewer::Viewer *viewer = dynamic_cast<osgViewer::Viewer*>(&us);
	if (!viewer)//如果转换失败则直接退出
	{
		return false;
	}
	CString s;
	switch (ea.getEventType())
	{
		case osgGA::GUIEventAdapter::PUSH: {
			if (viewer) {
				int button = ea.getButton();
				if (button == osgGA::GUIEventAdapter::LEFT_MOUSE_BUTTON) {
					lButtonDown = true;
					//MessageBox(NULL, "push doing!1", "hint", MB_OK);
					pickByRay(ea.getX(), ea.getY());
					//MessageBox(NULL, "push doing!2", "hint", MB_OK);
					if (pPickedObject) {
						stPoint = screen2World(ea.getX(), ea.getY());
						stPos = pPickedObject->getMatrix();
						//MessageBox(NULL, "push doing!2.1", "hint", MB_OK);
						// when push and drag, we want to shut down the camera manipulator
						// till the drag motion is done
						mViewer->setCameraManipulator(NULL);
						//mViewer->getCameraManipulator()-
						//mViewer->setUpdateOperations
					}
				}
				else {
					lButtonDown = false;
				}
			}
		return false;
	    }
	    case  osgGA::GUIEventAdapter::DRAG: {
	    	if (pPickedObject&&lButtonDown)	{
	    		endPoint = screen2World(ea.getX(), ea.getY());
				stPoint.z() = endPoint.z() = z;
	    		float dx = endPoint.x() - stPoint.x();
	    		float dy = endPoint.y() - stPoint.y();
	    		//float dz = endPoint.z() - stPoint.z() = 0;
	    		std::cout << dx << "  " << dy << std::endl;
	    		if (fabs(dx) + fabs(dy) < 0.06) {
	    		    std::cout << "small movement" << dx << "  " << dy << std::endl;
	    		}
	    		pPickedObject->setMatrix(stPos * osg::Matrix::translate(dx, 0, dy));
				//cosg->mViewer->setCameraManipulator(cosg->trackball, false);
				//MessageBox(NULL, "drag doing!", "hint", MB_OK);
	    	}
	    	return false;
	    }
	    case osgGA::GUIEventAdapter::RELEASE: {
			cosg->mViewer->setCameraManipulator(cosg->trackball, false);
	    	pPickedObject = false;
	    	return false;
	    }
	    case (osgGA::GUIEventAdapter::DOUBLECLICK): {
	    	MessageBox(NULL, "double click doing!", 0, MB_OK);
	    }
	default:
		return false;
	}
}

void CPickHandler::pickByRay(float x, float y) {
	osgUtil::LineSegmentIntersector::Intersections intersections;

	if (mViewer->computeIntersections(x, y, intersections)) {
		// get the first intersected object
		osgUtil::LineSegmentIntersector::Intersections::iterator hitr = intersections.begin();
		osg::NodePath getNodePath = hitr->nodePath;
		
		// find the object's matrix transform
		for (int i = getNodePath.size() - 1; i >= 0; --i) {
			osg::MatrixTransform* mt = dynamic_cast<osg::MatrixTransform*>(getNodePath[i]);
			if (mt == NULL) {
				continue;
			}
			else {
				pPickedObject = mt;
				ifPicked = true;
			    firstIntersectionPoint = hitr->getLocalIntersectPoint(); //in world 
				z = world2Screen(firstIntersectionPoint).z(); // in screen
				//MessageBox(NULL, tmp, "hint", MB_OK);
			}
		}
	}
	else {
		ifPicked = false;
	}
}
osg::Vec3 CPickHandler::screen2World(float x, float y)
{
	osg::Vec3 vec3;
	osg::ref_ptr<osg::Camera> camera = mViewer->getCamera();
	osg::Vec3 vScreen(x, y, 0);
	osg::Matrix mVPW = camera->getViewMatrix() * camera->getProjectionMatrix() * camera->getViewport()->computeWindowMatrix();
	osg::Matrix invertVPW;
	invertVPW.invert(mVPW);
	vec3 = vScreen * invertVPW;
	return vec3;
}

osg::Vec3 CPickHandler::world2Screen(osg::Vec3& wV) {
	osg::ref_ptr<osg::Camera> camera = mViewer->getCamera();
	osg::Matrix mVPW = camera->getViewMatrix() * camera->getProjectionMatrix() * camera->getViewport()->computeWindowMatrix();
	return wV * mVPW;
}
//osg::Vec3 CPickHandler::screen2World(float x, float y) {
//	osg::Vec3 point(0, 0, 0);
//	osgUtil::LineSegmentIntersector::Intersections intersections;
//	if (mViewer->computeIntersections(x, y, intersections)) {
//		osgUtil::LineSegmentIntersector::Intersections::iterator itr = intersections.begin();
//		point[0] = itr->getWorldIntersectPoint().x();
//		point[1] = itr->getWorldIntersectPoint().y();
//		point[2] = itr->getWorldIntersectPoint().z();
//	}
//	return point;
//}
```

```cpp
/*绘制并渲染几何体的主要步骤：
1.创建各种向量数据，如顶点、纹理坐标、颜色、法线。顶点数据按照逆时针顺序添加，以确保背面剔除的正确
2.实例化几何对象osg::Geometry，设置顶点坐标数组、纹理坐标数组、颜色数组、法线数组、绑定方式和数据解析
3.加入叶节点绘制并渲染
*/

osg::ref_ptr<osg::Node> createQuad(osg::ref_ptr<osg::Vec3Array>& v)
{
	//创建一个叶节点对象
	osg::ref_ptr<osg::Geode> geode = new osg::Geode();

	//创建一个几何体对象
	osg::ref_ptr<osg::Geometry> geom = new osg::Geometry();

	////创建顶点数组，注意顶点的添加顺序是逆时针的
	//osg::ref_ptr<osg::Vec3Array> v = new osg::Vec3Array();
	////添加数据
	//v->push_back(osg::Vec3(0.0f, 0.0f, 0.0f));
	//v->push_back(osg::Vec3(1.0f, 0.0f, 0.0f));
	//v->push_back(osg::Vec3(1.0f, 0.0f, 1.0f));
	//v->push_back(osg::Vec3(0.0f, 0.0f, 1.0f));
	//设置顶点数据setVertexArray(Array *array)
	geom->setVertexArray(v.get());

	//创建纹理数组
	osg::ref_ptr<osg::Vec2Array> vt = new osg::Vec2Array();
	//添加数据
	vt->push_back(osg::Vec2(0.0f, 0.0f));
	vt->push_back(osg::Vec2(1.0f, 0.0f));
	vt->push_back(osg::Vec2(1.0f, 1.0f));
	vt->push_back(osg::Vec2(0.0f, 1.0f));
	//设置纹理坐标数组setTexCoordArray(unsigned int unit, Array *)参数纹理单元/纹理坐标数组
	geom->setTexCoordArray(0, vt.get());

	//数据绑定：法线、颜色，绑定方式为：
	//BIND_OFF不启动用绑定/BIND_OVERALL绑定全部顶点/BIND_PER_PRIMITIVE_SET单个绘图基元绑定/BIND_PER_PRIMITIVE单个独立的绘图基元绑定/BIND_PER_VERTIE单个顶点绑定
	//采用BIND_PER_PRIMITIVE绑定方式，则OSG采用glBegin()/glEnd()函数进行渲染，因为该绑定方式为每个独立的几何图元设置一种绑定方式

	//创建颜色数组
	osg::ref_ptr<osg::Vec4Array> vc = new osg::Vec4Array();
	//添加数据
	vc->push_back(osg::Vec4(1.0f, 0.0f, 0.0f, 1.0f));
	vc->push_back(osg::Vec4(0.0f, 1.0f, 0.0f, 1.0f));
	vc->push_back(osg::Vec4(0.0f, 0.0f, 1.0f, 1.0f));
	vc->push_back(osg::Vec4(1.0f, 1.0f, 0.0f, 1.0f));
	//设置颜色数组setColorArray(Array *array)
	geom->setColorArray(vc.get());
	//设置颜色的绑定方式setColorBinding(AttributeBinding ab)为单个顶点
	geom->setColorBinding(osg::Geometry::BIND_PER_VERTEX);

	//创建法线数组
	osg::ref_ptr<osg::Vec3Array> nc = new osg::Vec3Array();
	//添加法线
	nc->push_back(osg::Vec3(0.0f, -1.0f, 0.0f));
	//设置法线数组setNormalArray(Array *array)
	geom->setNormalArray(nc.get());
	//设置法线的绑定方式setNormalBinding(AttributeBinding ab)为全部顶点
	geom->setNormalBinding(osg::Geometry::BIND_OVERALL);

	//添加图元，绘制基元为四边形
	//数据解析，即指定向量数据和绑定方式后，指定渲染几何体的方式，不同方式渲染出的图形不同，即时效果相同，可能面数或内部机制等也有区别，函数为：
	//bool addPrimitiveSet(PrimitiveSet *primitiveset)参数说明：osg::PrimitiveSet是无法初始化的虚基类，因此主要调用它的子类指定数据渲染，最常用为osg::DrawArrays
	//osg::DrawArrays(GLenum mode, GLint first, GLsizei count)参数为指定的绘图基元、绘制几何体的第一个顶点数在指定顶点的位置数、使用的顶点的总数
	//PrimitiveSet类继承自osg::Object虚基类，但不具备一般一般场景中的特性，PrimitiveSet类主要封装了OpenGL的绘图基元，常见绘图基元如下
	//POINTS点/LINES线/LINE_STRIP多线段/LINE_LOOP封闭线
	//TRIANGLES一系列三角形(不共顶点)/TRIANGLE_STRIP一系列三角形(共用后面两个顶点)/TRIANGLE_FAN一系列三角形，顶点顺序与上一条语句绘制的三角形不同
	//QUADS四边形/QUAD_STRIP一系列四边形/POLYGON多边形
	geom->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, 4));

	//添加到叶节点
	geode->addDrawable(geom.get());

	return geode.get();
}
```

## 2020/12/25 - 
### 0 模型精度渲染问题
meshlab读取模型时渲染的默认精度是float
osg渲染时也是float，这就导致了，转化成大地坐标系之后，比如3301412.7829会变成3301412.25之类的数据，
导致模型出现光栅效果。
如果需要读写自己的模型格式，考虑使用osg的注册器，注册自己模型读写方法。

### 1 从贴图中恢复出坐标
恢复坐标在重建软件中实现，然后转到体积测量的软件。

### 2 计算体积等(包括距离，面积，体积计算)功能
基于osgb的实现，然后解决0号问题

> fft2d: https://www.robots.ox.ac.uk/~az/lectures/ia/lect2.pd
>        https://www.zhihu.com/question/22611929/answer/341436331

### 3 实现步骤
- 1 东湖高新数据，哪些标记点，出现在哪些图片中，这些标记点的像素坐标:_config in yaml, transfer data in json_
```yaml
dataType: 文件表示的数据类型
metaData: 描述数据
imagesDir: 存储图片的目录
imageNameList: 图片名的列表
targetPointsAnnotation:
- point1:               # 第一个打标点的标注的像素坐标信息
    shownInImages:      # 第一个打标点出现在哪些图片中
    - 1.jpg
    - 2.jpg
    pixelPositions:     # 第一个打标点在出现的图片中对应的坐标
    - x: -2             # 对于该点对，则对应于point1在1.jpg中出现的坐标，图片的像素坐标以图像的中心点为原点
      y: -3
    - x: -2             # 对于该点对，则对应于point1在2.jpg中出现的坐标
      y: -3
- point2:               # 第二个打标点存储的像素坐标信息
    shownInImages:
    - 2.jpg
    - 3.jpg
    pixelPositions:
    - x: -2
      y: -3
    - x: -2
      y: -3

```
存储的信息格式如下列json所示，存储要求见上面的yaml注释
```json
{
  "dataType": "pixels annotations",
  "metaData": "this file try to store the information of the target points's pixel Positions on images where they may be on",
  "imagesDir": "D:\\images",
  "imagesNameList": [
    "1.jpg",
    "2.jpg",
    "3.jpg",
    "4.jpg"
  ],
  "targetPointsAnnotation": [
    {
      "point1": {
        "shownInImages": [
          "1.jpg",
          "2.jpg"
        ],
        "pixelPositions": [
          {"x": -2, "y": -3},
          {"x": -2, "y": -3}
        ]
      }
    },
    {
      "point2": {
        "shownInImages": [
          "2.jpg", 
          "3.jpg"
        ],
        "pixelPositions": [
          {"x": -2, "y": -3},
          {"x": -2, "y": -3}
        ]
      }
    }
  ]
```


## 2020/03/01 - 2020/03/26

### 1 osgb的内容组织以及读写


### 2 贴图的原理
### 2.0 使用phab比较两次提交的方法：
```
git log
git diff 808ed23da88251d7ff8df369c17441a574b00a98 1ebb2439e14bfe9f46e68c3b27c4807477a3db55 > a.txt
在phab中Diffrential中创建diff： 粘贴进入a.txt的代码即可
```

## 2020/04/13

### 1 修改模型：
```sh
# 本来apaqi.ive导弹模型头垂直屏幕向里面，之后头朝上，机身平行于屏幕
osgconv apaqi.ive apaqi.obj
# 绕着(0,1,1)旋转模型180度即可
osgconv -o 180-0,1,1 apaqi.ive apaqi_x-90_y180.obj

# 自动批量转换成gltf：
## step1: from ive to obj
for modelName in $(ls models | grep ive | sed s/.ive//g); do
    osgconv models/${modelName}.ive -o 180-0,1,1 models/${modelName}.obj
done;

## step2： from obj to gltf
for modelName in $(ls models | grep ive | sed s/.ive//g); do
    node bin/obj2gltf.js -i models/${modelName}.obj -o models/${modelName}.gltf
done;
```


