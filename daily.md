## 2020/10/13
margin probality

## 2020/10/19
- 3 ע�Ͳ����� for vim
- 2 BP�㷨��ϸ����
- 1 ����osgbȻ��鿴

bug:       ��ע��ȡͼƬ��m_image_data�ṹ���ֵ: �ڳ��뺯��ǰ�󣬱������޸ĵ�ͼƬ�Ŀ�������ڳ�������ȴ��0��
fix:          ��Ϊ��ʵ�Ĵ����߼�ȷʵ��ˣ���Ҫ����Ϊ�ڿ��������ʱ����һ�������ֽ�������ݸ��ĳ�0�ˡ�
suggest: debug��ʱ��һ��ÿһ�����̶�Ҫ���濴�������Ǿ����˺�������ؽ��룬Ȼ���������debugger��ֻ��λ���⣩��ֱ�ӿ�����Ч�ʸߡ�

## 2020/10/24
pac __SuiteSparse__. Needed for solving large sparse linear systems. Optional; strongly recomended for large scale bundle adjustment
pac __TBB__ is a C++11 template library for parallel programming that optionally can be used as an alternative to OpenMP. Optional

CGAL�������ܶ࣬QT��E:/360Downloads������

## 2020/10/25

### ��װGMP��MFPR
- 1 ��װCygwin
```log
Setup Environment
Install Cygwin, add the following packages to the default installation
gcc-core
gcc-g++
libgcc
m4
make #��֪Ϊ�ΰ�װ����Cygwin��Ŀ¼���Ѳ���make����
cmake
bash
Add the following Environment Variable to the User PATH: C:\Program Files (x86)\Microsoft Visual Studio\2017\Enterprise\VC\Tools\MSVC\14.15.26726\bin\Hostx64\x64
This is so you can use the lib command. Your lib.exe may be located elsewhere.
```
��װ��Cygwin�Լ�һЩ�����Ժ������Ŀ¼(����˵����Ϊ�� CygwinRoot="D:\CygwinRooto")�µ�bin/minnty.exe�����ն���ڣ�Ȼ��ÿ�δ򿪸��նˣ�������ǣ�$CygwinRoot/home/$(userName)�� ����"cd /"��Ϳ�������ˣ� 

- 2 ���ز���װmake
  - 2.1 ��������ַ����make��Դ�룬https://ftp.gnu.org/gnu/make/��Ȼ���ѹ
  - 2.2 ��Cygwin64 Terminal�����У�����Դ���Ŀ¼��Ȼ�����У�configure && ./build.sh
  - 2.3 ����õ���make.exe�����ƶ���Cygwin��binĿ¼��

- 3 ����gmp
���������� ./configure �� make install
```sh
./configure
  Version:           GNU MP 6.2.0
  Host type:         skylake-pc-cygwin
  ABI:               64
  Install prefix:    /usr/local #˵���Ὣ�ⰲװ����Ŀ¼�£����linux�Ǻ����Ƶ�
  Compiler:          gcc
  Static libraries:  yes
  Shared libraries:  no
```
��������Ĭ�����ɵ���static�����ݣ���
```log
@nash-5 ~/mylibs/gmp-6.2.0
$ ls /usr/local/include/
gmp.h

@nash-5 ~/mylibs/gmp-6.2.0
$ ls /usr/local/lib/
libgmp.a  libgmp.la  pkgconfig
```
���ɶ�̬���ӿ⣨ע�⣺ ��̬���ӿ�;�̬���ӿ��.h�ļ���ͬ������ע��ֳ�2���ļ��У����ٶ���gmp����ˣ���
```sh
./configure --prefix=/home/chenxy/mylibs/gmp-6.2.0/build/shared --enable-shared --disable-static
```
- 4 ����mfpr����Ҫgmp�������������Ƕ�̬���ӿ⣩
����mfpr�ĸ�Ŀ¼��
����./configure��
```log
checking for gmp.h... no
configure: error: gmp.h can't be found, or is unusable.
```
����./configure --help
```sh
������
  --with-gmp-include=DIR  GMP include directory
  --with-gmp-lib=DIR      GMP lib directory
������
```
���ԣ�
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

### cmake�鿴debug��Ϣ����cmake����
��boost����Ϊ����
�ڣ�cmake-gui�У������˱�����Boost_INCLUDE_DIR=Z:/BASE_ENV/forOpenMVS/boost_1_74_0����Ȼ����
```log
CMake Error at Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:165 (message):
  Could NOT find Boost (missing: Boost_INCLUDE_DIR iostreams program_options
  system serialization)
Call Stack (most recent call first):
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindPackageHandleStandardArgs.cmake:458 (_FPHSA_FAILURE_MESSAGE)
  Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:2177 (find_package_handle_standard_args)
  CMakeLists.txt:122 (FIND_PACKAGE)
```
���ṩ���bug��˼·��
- 1 ���ȣ���cmake�İ�װĿ¼��鿴�ļ��� _CMake/share/cmake-3.18/Modules/FindBoost.cmake_�� �䶨����Boost�ĸ��������ֱ����ʲô��˼�� ����ÿ��������Ӧ��ʵ��������������һ�¼��ɣ�����boost����Ҫ�������������� 
>  - Boost_INCLUDE_DIR: ����boostͷ�ļ���Ŀ¼
>  - Boost_LIBRARYDIR: ƫ�õĺ���boost��Ŀ�Ŀ¼

- 2 ����������Ȼ�����⣬���Ը��� _CMake/share/cmake-3.18/Modules/FindBoost.cmake_ �ļ���涨��debug������ Boost_DEBUG=ON(��cmake-gui���������һ��entery��Ȼ����boolֵΪtrue���ɣ�ֱ����CMakeLists.txt�������ӣ� set(Boost_DEBUG ON) �ͻ�򿪵�����Ϣ)�����ݵ�����Ϣ��ȥ�Ա����ɵĿ���ļ����ƺ�����Ŀ¼����Ϣ�����¸��������ĵ�����Ϣ��
```log
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1491 ] _boost_TEST_VERSIONS = "1.73.0;1.73;1.72.0;1.72;1.71.0;1.71;1.70.0;1.70;1.69.0;1.69;1.68.0;1.68;1.67.0;1.67;1.66.0;1.66;1.65.1;1.65.0;1.65;1.64.0;1.64;1.63.0;1.63;1.62.0;1.62;1.61.0;1.61;1.60.0;1.60;1.59.0;1.59;1.58.0;1.58;1.57.0;1.57;1.56.0;1.56;1.55.0;1.55;1.54.0;1.54;1.53.0;1.53;1.52.0;1.52;1.51.0;1.51;1.50.0;1.50;1.49.0;1.49;1.48.0;1.48;1.47.0;1.47;1.46.1;1.46.0;1.46;1.45.0;1.45;1.44.0;1.44;1.43.0;1.43;1.42.0;1.42;1.41.0;1.41;1.40.0;1.40;1.39.0;1.39;1.38.0;1.38;1.37.0;1.37;1.36.1;1.36.0;1.36;1.35.1;1.35.0;1.35;1.34.1;1.34.0;1.34;1.33.1;1.33.0;1.33"
<������ʡ��>
boost_1_74_0/../lib64-msvc-14.0;Z:/BASE_ENV/forOpenMVS/boost/lib64-msvc-14.0;PATHS;C:/boost/lib;C:/boost;/sw/local/lib"
[ Z:/BASE_ENV/CMake/share/cmake-3.18/Modules/FindBoost.cmake:1871 ] _boost_LIBRARY_SEARCH_DIRS_DEBUG = "Z:/BASE_ENV/forOpenMVS/boost_1_74_0/lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/../lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/stage/lib;Z:/BASE_ENV/forOpenMVS/boost_1_74_0/../lib64-msvc-14.1;Z:/BASE_ENV/forOpenMVS/local/lib"
<������ʡ��>
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



## 2020/10/26
### vs2019����boost
����boost��ѹ�ĸ�Ŀ¼��ִ�У�
```sh
# ����b2.exe
./bootstrap.bat

# ����ָ���Ĺ��߼��Ŀ�İ汾(���ܳ���û��cl��cstddef�����⣬�������ע��)
.\b2.exe  --toolset=msvc-14.2  `
--address-model=64 `
--architecture=x86 `
--threading=multi `
--with-iostreams --with-program_options --with-system --with-serialization `
stage --stagedir="F:\BASE_ENV\forOpenMVS\boost_1_74_0\forOpenMVS"

# ������Ҫʹ��boost׷�ӱ���zlib�Ŀ�(github��Դ��)�����£�
.\b2.exe  --toolset=msvc-14.2  `
--address-model=64 `
--architecture=x86 `
--threading=multi `
--with-iostreams -sZLIB_SOURCE="F:\BASE_ENV\forOpenMVS\zlib-1.2.11" -sZLIB_INCLUDE="F:\BASE_ENV\forOpenMVS\zlib-1.2.11" `
--link=static --runtime-link=static `
stage --stagedir="F:\BASE_ENV\forOpenMVS\boost_1_74_0\forOpenMVS"

```


__ע�⣺ �����Ҳ���cl��cstddef���������ǻ������������⣬�༭���»���������__
- 1 path�����(����cl.exe): C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\bin\Hostx64\x64
- 2 ��ӱ���: INCLUDE, ֵ: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\include;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\shared;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\winrt;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\um;C:\Program Files (x86)\Windows Kits\10\Include\10.0.18362.0\ucrt;
- 3 ��ӱ���: LIB, ֵ: C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Tools\MSVC\14.27.29110\lib\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.18362.0\um\x64;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.18362.0\ucrt\x64;

### ����OpenMVS
������ٷ���buildingWilki[https://github.com/cdcseacave/openMVS/wiki/Building](https://github.com/cdcseacave/openMVS/wiki/Building)��װ����ص������⣬ִ�е�cmakeָ�����£�
```sh
# �� windows terminal��ִ��

cmake . ..\openMVS\ `
-G "Visual Studio 16 2019" `
-DVCG_ROOT="..\VCG" `
-DBOOST_INCLUDEDIR="F:/BASE_ENV/forOpenMVS/boost_1_74_0" `
-DBOOST_LIBRARYDIR="F:/BASE_ENV/forOpenMVS/boost_1_74_0/forOpenMVS" `
-DOpenCV_DIR="F:/BASE_ENV/forOpenMVS/opencv/build" `
-DGMP_INCLUDE_DIR="F:\BASE_ENV\forOpenMVS\gmp-6.2.0\build\static\include" `
-DGMP_LIBRARIES="F:\BASE_ENV\forOpenMVS\gmp-6.2.0\build\static\lib\libgmp.a" `
-DMPFR_INCLUDE_DIR="F:\BASE_ENV\forOpenMVS\mpfr-4.1.0\build\include" `
-DMPFR_LIBRARIES="F:\BASE_ENV\forOpenMVS\mpfr-4.1.0\build\lib\libmpfr.a" `
-DEIGEN_ROOT="F:\BASE_ENV\forOpenMVS\eigen" `
-DCERES_ROOT="F:\BASE_ENV\forOpenMVS\ceres-solver\build" `

```

������־���£�
```log
1>------ Skipped Build: Project: clean_cotire, Configuration: Release x64 ------
1>Project not selected to build for this solution configuration 
2>------ Skipped Build: Project: uninstall, Configuration: Release x64 ------
2>Project not selected to build for this solution configuration 
3>------ Build started: Project: DensifyPointCloud, Configuration: Release x64 ------
4>------ Build started: Project: InterfaceCOLMAP, Configuration: Release x64 ------
5>------ Build started: Project: InterfaceVisualSFM, Configuration: Release x64 ------
6>------ Build started: Project: ReconstructMesh, Configuration: Release x64 ------
7>------ Build started: Project: RefineMesh, Configuration: Release x64 ------
8>------ Build started: Project: TextureMesh, Configuration: Release x64 ------
4>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/InterfaceCOLMAP.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/InterfaceCOLMAP.exp
4>Generating code
5>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/InterfaceVisualSFM.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/InterfaceVisualSFM.exp
3>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/DensifyPointCloud.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/DensifyPointCloud.exp
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_strategy" (?default_strategy@zlib@iostreams@boost@@3HB)
7>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/RefineMesh.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/RefineMesh.exp
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::deflated" (?deflated@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::best_speed" (?best_speed@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_compression" (?default_compression@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::reset(bool,bool)" (?reset@zlib_base@detail@iostreams@boost@@IEAAX_N0@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "private: void __cdecl boost::iostreams::detail::zlib_base::do_init(struct boost::iostreams::zlib_params const &,bool,void * (__cdecl*)(void *,unsigned int,unsigned int),void (__cdecl*)(void *,void *),void *)" (?do_init@zlib_base@detail@iostreams@boost@@AEAAXAEBUzlib_params@34@_NP6APEAXPEAXII@ZP6AX22@Z2@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::before(char const * &,char const *,char * &,char *)" (?before@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDPEBDAEAPEADPEAD@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xinflate(int)" (?xinflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::sync_flush" (?sync_flush@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::after(char const * &,char * &,bool)" (?after@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDAEAPEAD_N@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "public: static void __cdecl boost::iostreams::zlib_error::check(int)" (?check@zlib_error@iostreams@boost@@SAXH@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::stream_end" (?stream_end@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xdeflate(int)" (?xdeflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::finish" (?finish@zlib@iostreams@boost@@3HB)
5>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::no_flush" (?no_flush@zlib@iostreams@boost@@3HB)
5>F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\InterfaceVisualSFM.exe : fatal error LNK1120: 17 unresolved externals
6>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/ReconstructMesh.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/ReconstructMesh.exp
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_strategy" (?default_strategy@zlib@iostreams@boost@@3HB)
8>   Creating library F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/TextureMesh.lib and object F:/BASE_ENV/forOpenMVS/build/bin/x64/Release/TextureMesh.exp
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::deflated" (?deflated@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::best_speed" (?best_speed@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::reset(bool,bool)" (?reset@zlib_base@detail@iostreams@boost@@IEAAX_N0@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_compression" (?default_compression@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "private: void __cdecl boost::iostreams::detail::zlib_base::do_init(struct boost::iostreams::zlib_params const &,bool,void * (__cdecl*)(void *,unsigned int,unsigned int),void (__cdecl*)(void *,void *),void *)" (?do_init@zlib_base@detail@iostreams@boost@@AEAAXAEBUzlib_params@34@_NP6APEAXPEAXII@ZP6AX22@Z2@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::before(char const * &,char const *,char * &,char *)" (?before@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDPEBDAEAPEADPEAD@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xdeflate(int)" (?xdeflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::finish" (?finish@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::no_flush" (?no_flush@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::after(char const * &,char * &,bool)" (?after@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDAEAPEAD_N@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "public: static void __cdecl boost::iostreams::zlib_error::check(int)" (?check@zlib_error@iostreams@boost@@SAXH@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::stream_end" (?stream_end@zlib@iostreams@boost@@3HB)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xinflate(int)" (?xinflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
3>MVS.lib(DepthMap.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::sync_flush" (?sync_flush@zlib@iostreams@boost@@3HB)
3>F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\DensifyPointCloud.exe : fatal error LNK1120: 17 unresolved externals
5>Done building project "InterfaceVisualSFM.vcxproj" -- FAILED.
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_strategy" (?default_strategy@zlib@iostreams@boost@@3HB)
3>Done building project "DensifyPointCloud.vcxproj" -- FAILED.
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::deflated" (?deflated@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::best_speed" (?best_speed@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_compression" (?default_compression@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::reset(bool,bool)" (?reset@zlib_base@detail@iostreams@boost@@IEAAX_N0@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "private: void __cdecl boost::iostreams::detail::zlib_base::do_init(struct boost::iostreams::zlib_params const &,bool,void * (__cdecl*)(void *,unsigned int,unsigned int),void (__cdecl*)(void *,void *),void *)" (?do_init@zlib_base@detail@iostreams@boost@@AEAAXAEBUzlib_params@34@_NP6APEAXPEAXII@ZP6AX22@Z2@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::before(char const * &,char const *,char * &,char *)" (?before@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDPEBDAEAPEADPEAD@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xinflate(int)" (?xinflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::sync_flush" (?sync_flush@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::after(char const * &,char * &,bool)" (?after@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDAEAPEAD_N@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "public: static void __cdecl boost::iostreams::zlib_error::check(int)" (?check@zlib_error@iostreams@boost@@SAXH@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::stream_end" (?stream_end@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xdeflate(int)" (?xdeflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::finish" (?finish@zlib@iostreams@boost@@3HB)
7>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::no_flush" (?no_flush@zlib@iostreams@boost@@3HB)
7>F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\RefineMesh.exe : fatal error LNK1120: 17 unresolved externals
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_strategy" (?default_strategy@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_strategy" (?default_strategy@zlib@iostreams@boost@@3HB)
7>Done building project "RefineMesh.vcxproj" -- FAILED.
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::deflated" (?deflated@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::best_speed" (?best_speed@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_compression" (?default_compression@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::reset(bool,bool)" (?reset@zlib_base@detail@iostreams@boost@@IEAAX_N0@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "private: void __cdecl boost::iostreams::detail::zlib_base::do_init(struct boost::iostreams::zlib_params const &,bool,void * (__cdecl*)(void *,unsigned int,unsigned int),void (__cdecl*)(void *,void *),void *)" (?do_init@zlib_base@detail@iostreams@boost@@AEAAXAEBUzlib_params@34@_NP6APEAXPEAXII@ZP6AX22@Z2@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::before(char const * &,char const *,char * &,char *)" (?before@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDPEBDAEAPEADPEAD@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xinflate(int)" (?xinflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::sync_flush" (?sync_flush@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::after(char const * &,char * &,bool)" (?after@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDAEAPEAD_N@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "public: static void __cdecl boost::iostreams::zlib_error::check(int)" (?check@zlib_error@iostreams@boost@@SAXH@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::stream_end" (?stream_end@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xdeflate(int)" (?xdeflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::finish" (?finish@zlib@iostreams@boost@@3HB)
6>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::no_flush" (?no_flush@zlib@iostreams@boost@@3HB)
6>libgmp.a(dcpi1_bdiv_qr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(mullo_n.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(dcpi1_div_qr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(invertappr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(dcpi1_bdiv_q.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(dcpi1_divappr_q.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(divexact.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt99-tdiv_qr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt103-divexact.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(hgcd_reduce.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(toom42_mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(mul_n.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(toom53_mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt97-gcd.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(sqr.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(gcd.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt31-mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(nussbaumer_mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt55-cmp.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(aors.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt64-mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(lt82-mul.o) : error LNK2001: unresolved external symbol ___chkstk_ms
6>libgmp.a(memory.o) : error LNK2001: unresolved external symbol __getreent
6>libgmp.a(realloc.o) : error LNK2001: unresolved external symbol __getreent
6>libgmp.a(assert.o) : error LNK2001: unresolved external symbol __getreent
6>F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\ReconstructMesh.exe : fatal error LNK1120: 19 unresolved externals
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::deflated" (?deflated@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::best_speed" (?best_speed@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::zlib_base(void)" (??0zlib_base@detail@iostreams@boost@@IEAA@XZ)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: __cdecl boost::iostreams::detail::zlib_base::~zlib_base(void)" (??1zlib_base@detail@iostreams@boost@@IEAA@XZ)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::default_compression" (?default_compression@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::reset(bool,bool)" (?reset@zlib_base@detail@iostreams@boost@@IEAAX_N0@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "private: void __cdecl boost::iostreams::detail::zlib_base::do_init(struct boost::iostreams::zlib_params const &,bool,void * (__cdecl*)(void *,unsigned int,unsigned int),void (__cdecl*)(void *,void *),void *)" (?do_init@zlib_base@detail@iostreams@boost@@AEAAXAEBUzlib_params@34@_NP6APEAXPEAXII@ZP6AX22@Z2@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::before(char const * &,char const *,char * &,char *)" (?before@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDPEBDAEAPEADPEAD@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xinflate(int)" (?xinflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::sync_flush" (?sync_flush@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: void __cdecl boost::iostreams::detail::zlib_base::after(char const * &,char * &,bool)" (?after@zlib_base@detail@iostreams@boost@@IEAAXAEAPEBDAEAPEAD_N@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "public: static void __cdecl boost::iostreams::zlib_error::check(int)" (?check@zlib_error@iostreams@boost@@SAXH@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::stream_end" (?stream_end@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "protected: int __cdecl boost::iostreams::detail::zlib_base::xdeflate(int)" (?xdeflate@zlib_base@detail@iostreams@boost@@IEAAHH@Z)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::finish" (?finish@zlib@iostreams@boost@@3HB)
8>MVS.lib(Scene.obj) : error LNK2001: unresolved external symbol "int const boost::iostreams::zlib::no_flush" (?no_flush@zlib@iostreams@boost@@3HB)
8>F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\TextureMesh.exe : fatal error LNK1120: 17 unresolved externals
6>Done building project "ReconstructMesh.vcxproj" -- FAILED.
8>Done building project "TextureMesh.vcxproj" -- FAILED.
4>Finished generating code
4>InterfaceCOLMAP.vcxproj -> F:\BASE_ENV\forOpenMVS\build\bin\x64\Release\InterfaceCOLMAP.exe
9>------ Skipped Build: Project: INSTALL, Configuration: Release x64 ------
9>Project not selected to build for this solution configuration 
========== Build: 1 succeeded, 5 failed, 6 up-to-date, 3 skipped ==========
```

