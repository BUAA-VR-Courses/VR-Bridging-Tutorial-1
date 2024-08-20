## 其他

在使用其他工具（如VSCode、CLion等）完成之后的课程时，请确保 CMake 已经配置完毕（见[CMake](../chapter2/cmake.md)）。

### VSCode和gcc

VSCode是一个轻量级的代码**编辑器**，可以通过插件扩展功能。gcc/g++是GNU的C/C++编译器，是Linux下的默认编译器，也可以在Windows下使用。

使用VSCode开发C++环境配置流程如下：
1. [下载VSCode](https://code.visualstudio.com/download)并安装。
2. [下载MinGW](https://sourceforge.net/projects/mingw/)，然后将MinGW下bin的完整路径粘贴到系统变量的Path中。在命令行中输入`gcc -v`和`g++ -v`检验是否安装成功。
3. 在VSCode中安装C/C++的插件和CMake Tools插件。
4. 在弹出窗口中选择1.中安装的GCC mingw编译器。
5. 在窗口下侧点击 build 进行项目构建，点击窗口下侧三角形符号运行（如果想要使用窗口左上的三角形符号运行需要自行配置）。

### CLion

在Windows下，较新版本的CLion已经自带了MinGW，可以直接使用。CLion是付费软件，但是学生可以申请免费使用，下载地址为[JetBrains官网](https://www.jetbrains.com/clion/)。

### Clang

Clang是一个C、C++、Objective-C和Objective-C++编程语言的**编译器**前端，它采用了LLVM作为其后端。在编译速度、错误提示等方面相比于gcc/g++有一定优势。

Clang在Windows上通常依赖于Microsoft的编译工具链，包括链接器和标准库，在安装完VS2022后，你可以在[LLVM官网](https://releases.llvm.org/download.html)下载Windows版本的Clang。