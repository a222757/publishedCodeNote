# 目的: 配置Clion, 打开QtCreator项目

惯于idea, ideavim的操作快捷键, 加上QtCreator上手不熟悉且不够优雅😁, 选择使用Clion尝试开发

在网上收集资料, 加上本地调试, 特此记录

## 1. 前提和本次配置软件版本

0. 系统Win10; Clion:2021.3.3 ; QtCreator 5.12.2
1. 需提前安装Clion 及 QtCreator,
    * 搜索资料时看到有人说QtCreator安装时需要选择32位, 测试好像可以只安装64位的, 但是为了打包兼容32位建议也勾择32位.

## 2. Clion 配置

### 2.1 本机Creator相关路径(依本地情况替换 目录|版本 即可)

**安装根目录**  D:\CodeTools\QtTool

**带qt5的Cmake路径**  D:\CodeTools\QtTool\5.12.2\mingw73_64\lib\cmake

**ToolSet路径**  D:\CodeTools\QtTool\Tools\mingw730_64

**GDB路径** D:\CodeTools\QtTool\Tools\mingw730_64\bin\gdb.exe`

### 2.2 Clion的ToolChains配置

1. 打开设置Settings -> Build, Execution, Deployment -> Toolchains
   或直接在设置中搜索ToolChains

![](https://pi-go.oss-cn-beijing.aliyuncs.com/img/202210261637506.png)

2. 然后在上图标号2处替换为前面的**Toolset路径**, 在上图标号3出替换为**GDB路径**

### 2.3 Clion的Debug配置

1. 接着配置ToolChains下面的CMake选项

![](https://pi-go.oss-cn-beijing.aliyuncs.com/img/202210261634591.png)

2. 增加CMake Options为 `-DCMAKE_PREFIX_PATH=D:\CodeTools\QtTool\Tools\mingw730_64\lib\cmake`,
   等号后面的路径即是前面对应的**带qt5的Cmake路径**,
   下面是本机的改路径截图供参考!

![](https://pi-go.oss-cn-beijing.aliyuncs.com/img/202210261637287.png)

## 3. Clion运行已有的QtCreator项目

### 3.1 配置CMakeLists.txt

1. 在QtCreator, 其自动识别头文件等文件类型, 分成header、sourcefile、 ui等文件,
   但实际不存在这些文件夹，Clion如果想如此分类可以手动添加，但不清楚会对QtCreator编译有无后续影响，故不改动文件路径。
2. 首先贴上本地可运行的demo的CMakeList.txt

```txt
cmake_minimum_required(VERSION 3.21)
project(Demon)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

set(CMAKE_PREFIX_PATH "D:/CodeTools/QtTool/5.12.2/mingw73_64/lib/cmake")

find_package(Qt5 COMPONENTS
        Core
        Gui
        Widgets
        REQUIRED)
include_directories(.)
add_executable(
		res.qrc
        linetask.cpp linetask.ui linetask.h
        )
        
target_link_libraries(Demon
        Qt5::Core
        Qt5::Gui
        Qt5::Widgets
        )

if (WIN32)
    set(DEBUG_SUFFIX)
    if (CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB Core Gui Widgets)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt5${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)
endif ()

```

3. 上面的txt中, 大多为固定的代码, 具体每个机器, 可已通过自己新建一个QtWidget Application查看其自动生成的CMakeLists.txt,
   这里说明几点遇到的坑:
    * `add_executable( ) ` 的括号内部, 添加所有的可执行文件, 如后缀为 .h, .cpp, .ui; 若有资源文件(如res.qrc)也要添加
    * 在编译时找不到一些已文件报错, 添加一行`include_directories(.)`解决.
    * 在Cmake中添加的options好像没有起效,
      故增加`set(CMAKE_PREFIX_PATH "D:/CodeTools/QtTool/5.12.2/mingw73_64/lib/cmake")`防止编译找不到**qt5**等报错提示

### 3.2 Clion打开 .Ui文件

1. 可查看[官方文档](https://www.jetbrains.com/help/clion/qt-tutorial.html#edit-ui)
2. 网上很多教程使用外部工具打开ui文件, 感觉不太清爽, 查看官方文档, 发现可以实现直接双击打开.

#### 双击默认使用QtCreator打开

1. 首先CLion打开设置settings -> Editor -> FileTypes ; 然后找到qt_ui文件点击,
   把对应的文件格式删除

![](https://pi-go.oss-cn-beijing.aliyuncs.com/img/202210261635595.png)

2. 选择任意ui文件, 右键选择使用系统默认应用打开(win10下, 在系统文件管理器设置ui文件默认应用即可)

   ![](https://pi-go.oss-cn-beijing.aliyuncs.com/img/202210261639003.png)

.

## 4. Clion插件(详细说明)

### IdeaVim, IdeaVimExtension, KJump

### Clang-format(CLion自带)

### FileWatcher (之前使用其调用clang-format, 现弃用)

### ExternalTools(之前用打开ui页面, 现弃用)

### SaveActions

### Tabnine

### GrepConsole

### Rainbow Brackets

### generate all setter


