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
./configure
  Version:           GNU MP 6.2.0
  Host type:         skylake-pc-cygwin
  ABI:               64
  Install prefix:    /usr/local #说明会将库安装到该目录下，这和linux是很相似的
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
./configure --prefix=/home/chenxy/mylibs/mpfr-4.1.0/build \
--with-gmp-include=/home/chenxy/mylibs/gmp-6.2.0/build/shared/include \
--with-gmp-lib=/home/chenxy/mylibs/gmp-6.2.0/build/shared/lib

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


