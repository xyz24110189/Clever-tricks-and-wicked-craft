## qca 编译

- qca是基于Qt代码框架进行编写的支持加解密的类库，编译时候需要依赖openssl，Qt开发库， 我这边是基于Qt4.8.7 mingw 和 openssl_1.0.2依赖版本进行开发的，记录下主要的编译过程：
  1. 下载mingw i686-5.3.0-release-posix-sjlj-rt_v4-rev0.7z 开发工具集，[mingw工具集合下载地址](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win32/Personal%20Builds/mingw-builds/5.3.0/threads-win32/sjlj/)
  2. 下载Win32OpenSSL-1_0_2u.exe安装包，[openssl安装包下载地址](https://slproweb.com/products/Win32OpenSSL.html) 
  3. 配置openssl环境变量 添加 D:\OpenSSL-Win32\bin; 配置Qt4.8.7环境变量 QTDIR=D:\Qt\4.8.7_mingw4.8.2, QMAKESPEC=D:\Qt\4.8.7_mingw4.8.2\mkspecs\win32-g++
  4. 编译 
    1. qca最近是基于cmake进行编译环境搭建， 创建编译目录：mkdir build & cd build;
    2. 打开终端，执行 cmake -G"MinGW Makefiles" -DOPENSSL_INCLUDE_DIR=D:\OpenSSL-Win32\include  -DCMAKE_INSTALL_PREFIX=D:\Qt\4.8.7_mingw4.8.2 -DCMAKE_BUILD_TYPE=Release -DQT4_BUILD=ON ..
    3. 编译及安装 mingw32-make.exe & mingw32-make.exe install
