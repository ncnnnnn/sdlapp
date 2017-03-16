    Android Studio 开发SDL2.0最佳实践　　本文2017-3-14　　传统的使用NDK按照命令行来编译SDL2以及自己的C文件，再导入到Android Studio工程编译成为APK，这样十分麻烦，没有代码提示，做个Hello World还可以，真正搞个工程就是恶梦了，让人无限留恋MS Visual Studo了。　　按照现在Android Studio 2.3编译NDK程序，简直是喜出望外了，有代码提示，也可以直接调试了。因此，将SDL2工程移植到Android Studio直接编译调试就是个好办法，这就是本文的内容了。本文代码共享地址：　　https://github.com/gongxp/sdlapp　　　　基本环境：　　1. 操作系统：Windows10  64位;　　2. Android Studio 2.3，64位。　　3. Android NDK开发包：直接使用Android Studio安装NDK。（需要梯子）　　本文主要参考：　　https://developer.android.google.cn/studio/projects/add-native-code.html?hl=zh-cn《向您的项目添加 C 和 C++ 代码》。　　　　第一步：准备SDL源代码包并使用命令行编译获得libsld.so，并且将工程导入Android Studio运行　　1. 去http://www.libsdl.org/官网下载最新版SDL2-2.0.5 　　2. 解压后，可以在根目录下找到android-project目录。　　3. 调整目录，使其成为一个可编译的工程:　　 (1) 将android-project目录剪切到与SDL2-2.0.5同级的目录,重新命令为sdlapp;　　 (2) 新建目录sdlapp\jni\sdl，在然后将SDL2-2.0.5目录下的SDL2-2.0.5\include目录和SDL2-2.0.5\src目录和文件SDL2-2.0.5\Android.mk拷贝到android-project\jni\sdl目录下;　　4. main.c文件，可以从http://www.dinomage.com/wp-content/uploads/2013/01/main.c下载,下载后将main.c放到目录sdlapp\jni\src\下，改目录下在Android.mk中默认有个YourSourceHere.c，将其替换为main.c。　　5、编辑sdlapp\jni\Application.mk，将其修改为：　　APP_ABI := armeabi-v7a　　6、命令行进入到sdl\app\jni执行ndk-build，这样就会编译出libsdl.so和libmain.so文件。　　7、选择File->Close Project。之后，选择”Import project(Eclipse ADT,Gradle,etc)”，然后选择android-project目录将工程文件导入到Android studio中。　　将工程jni目录下的文件，包括目录等全部删掉。　　下载一个bmp图片，提供一个官方文档中提到的bmp地址：http://www.dinomage.com/wp-content/uploads/2013/01/image.bmp。　　到Andriod Studio面板，选择app,按右键，New->Folder->Assets Folder，再将下载的image.bmp放到这个目录下，重新编译运行这个apk，这个就是SDL 2.05的hello world了。但是这样，不是我们通过andriod studio直接编译调试C代码的目的，下面就要进行进行如下步骤。　　　　第二步、清理导入工程的文件　　将导入到andriod studio的sdlapp工程sdlapp\app\src\main\jniLibs\libmain.so文件删掉，这个正需要将来添加c代码编译的。下面添加C代码部分，主要参考官方文档：《向您的项目添加 C 和 C++ 代码》。　　　　第三步，在这个工程中添加C文件，并做修改相关　　1、到andriod studio,按”Project”视图，在app\src\main下建立CPP目录，将main.c和sdlapp\jni\sdl\src\main\android\SDL_android_main.c俩个文件放到其中。另外将sdlapp\jni\sdl\include目录也放到这个目录中，将sdlapp\jni\sdl\src\dynapi这个目录拷贝过去，建立目录app\src\main\cpp\include\sdlmain，将sdlapp\jni\sdl\src目录下8个头文件和C文件拷贝到这个目录中。　　2、修改SDL_android_main.c，将前面包含头文件的修改为：　　#include "sdlmain/SDL_internal.h"　　3、修改app\src\mainCPP\include\ sdlmain\SDL_internal.h, 将前面包含头文件的修改为：　　#include "dynapi/SDL_dynapi.h"　　【坑】如果不修改这些，将来编译是就提示什么.H文件找不到的错误。　　　　第四步，在这个工程中添加CMakeLists.txt文件。　　到andriod studio,按”Project”视图，在app目录下，添加文件CMakeLists.txt，内容如下：　　cmake_minimum_required(VERSION 3.4.1)　　　　set(lib_src_DIR ${CMAKE_SOURCE_DIR}/src/main/jniLibs/${ANDROID_ABI})　　include_directories(　　     ${CMAKE_SOURCE_DIR}/src/main/cpp/include　　)　　add_library(sdl2_lib SHARED IMPORTED)　　set_target_properties(sdl2_lib PROPERTIES IMPORTED_LOCATION　　    ${lib_src_DIR}/libSDL2.so)　　add_library( main  SHARED　　             src/main/cpp/SDL_android_main.c　　             src/main/cpp/main.c　　             )　　　　find_library(log-lib    log )　　find_library(GLESv1_CM-lib    GLESv1_CM )　　find_library(GLESv2-lib    GLESv2 )　　　　target_link_libraries(  main　　                       ${log-lib}　　                       ${GLESv1_CM-lib}　　                       ${GLESv2-lib}　　                       sdl2_lib　　                       )　　　　第五步，将项目和CMakeLists.txt关联　　1、从 IDE 左侧打开 Project 窗格并选择 Android 视图。　　2、右键点击您想要关联到sdlapp原生库的模块，并从菜单中选择 Link C++ Project with Gradle，应看到的相应对话框。　　3、从下拉菜单中，选择 CMake，请使用 Project Path 旁的字段为您的外部 CMake 项目指定刚才建立的CMakeLists.txt 脚本文件。　　4、指定 ABI，在模块级 build.gradle 文件中使用 ndk.abiFilters 标志指定这些配置，如下所示：　　  defaultConfig {　　        applicationId "org.libsdl.app"　　        minSdkVersion 10　　        targetSdkVersion 12　　        ndk {　　            abiFilters  'armeabi-v7a'　　        }　　    }　　其中，abiFilters  'armeabi-v7a'是新增加的。　　【坑】如果不增加这个，在编译时，就会出现armeabi编译错误等相关提示。　　　　第六步，同步C工程后，同步，编译执行　　从菜单栏中选择 Build > Refresh Linked C++ Projects，将 Android Studio 与您的更改同步，之后，就可以编译、调试了！　　　　第七步、自定义sdl app　　自己定义APP名称，ICON图标，按照\SDL2-2.0.5\docs\ README-android.md即可。
