# Android Build System

### Make 文件说明
整个 Build 系统的入口文件是源码树根目录下名称为“Makefile”的文件，当在源代码根目录上调用 make 命令时，make 命令首先将读取该文件。

Makefile 文件的内容只有一行：“include build/core/main.mk”。该行代码的作用很明显：包含 build/core/main.mk 文件。

在 main.mk 文件中又会包含其他的文件，其他文件中又会包含更多的文件，这样就引入了整个 Build 系统。
这些 Make 文件间的包含关系是相当复杂的，图 3 描述了这种关系，该图中黄色标记的文件（且除了 $开头的文件）都位于 build/core/ 目录下。


![](http://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image004.png)

|文件名	|说明|
|--|--|
|main.mk|	最主要的 Make 文件，该文件中首先将对编译环境进行检查，同时引入其他的 Make 文件。另外，该文件中还定义了几个最主要的 Make 目标，例如 droid，sdk，等（参见后文“Make 目标说明”）。|
|help.mk|	包含了名称为 help 的 Make 目标的定义，该目标将列出主要的 Make 目标及其说明。|
|pathmap.mk|	将许多头文件的路径通过名值对的方式定义为映射表，并提供 include-path-for函数来获取。例如，通过 $(call include-path-for, frameworks-native)便可以获取到 framework 本地代码需要的头文件路径。|
|envsetup.mk|配置 Build 系统需要的环境变量，例如：TARGET_PRODUCT，TARGET_BUILD_VARIANT，HOST_OS，HOST_ARCH 等。当前编译的主机平台信息（例如操作系统，CPU 类型等信息）就是在这个文件中确定的。另外，该文件中还指定了各种编译结果的输出路径。|
|combo/select.mk|根据当前编译器的平台选择平台相关的 Make 文件。|
|dumpvar.mk|在 Build 开始之前，显示此次 Build 的配置信息。|
|config.mk|整个 Build 系统的配置文件，最重要的 Make文件之一。该文件中主要包含以下内容：定义了许多的常量来负责不同类型模块的编译。定义编译器参数以及常见文件后缀，例如 .zip,.jar.apk。根据 BoardConfig.mk 文件，配置产品相关的参数。设置一些常用工具的路径，例如 flex，e2fsck，dx。|
|definitions.mk|最重要的 Make 文件之一，在其中定义了大量的函数。这些函数都是 Build 系统的其他文件将用到的。例如：my-dir，all-subdir-makefiles，find-subdir-files，sign-package 等，关于这些函数的说明请参见每个函数的代码注释。|
|distdir.mk|针对 dist 目标的定义。dist 目标用来拷贝文件到指定路径。|
|dex_preopt.mk|针对启动 jar 包的预先优化。|
|pdk_config.mk|顾名思义，针对 pdk（Platform Developement Kit）的配置文件。|
|${ONE_SHOT_MAKEFILE}|ONE_SHOT_MAKEFILE 是一个变量，当使用“mm”编译某个目录下的模块时，此变量的值即为当前指定路径下的 Make 文件的路径。|
|${subdir_makefiles}|各个模块的 Android.mk 文件的集合，这个集合是通过 Python 脚本扫描得到的。|
|post_clean.mk|在前一次 Build 的基础上检查当前 Build 的配置，并执行必要清理工作。|
|legacy_prebuilts.mk|该文件中只定义了 GRANDFATHERED_ALL_PREBUILT 变量。|
|Makefile|被 main.mk 包含，该文件中的内容是辅助 main.mk 的一些额外内容。|


Android 源码中包含了许多的模块，模块的类型有很多种，例如：Java 库，C/C++ 库，APK 应用，以及可执行文件等 。并且，Java 或者 C/C++ 库还可以分为静态的或者动态的，库或可执行文件既可能是针对设备（本文的“设备”指的是 Android 系统将被安装的设备，例如某个型号的手机或平板）的也可能是针对主机（本文的“主机”指的是开发 Android 系统的机器，例如装有 Ubuntu 操作系统的 PC 机或装有 MacOS 的 iMac 或 Macbook）的。不同类型的模块的编译步骤和方法是不一样，为了能够一致且方便的执行各种类型模块的编译，在 config.mk 中定义了许多的常量，这其中的每个常量描述了一种类型模块的编译方式，这些常量有：

* BUILD_HOST_STATIC_LIBRARY
* BUILD_HOST_SHARED_LIBRARY
* BUILD_STATIC_LIBRARY
* BUILD_SHARED_LIBRARY
* BUILD_EXECUTABLE
* BUILD_HOST_EXECUTABLE
* BUILD_PACKAGE
* BUILD_PREBUILT
* BUILD_MULTI_PREBUILT
* BUILD_HOST_PREBUILT
* BUILD_JAVA_LIBRARY
* BUILD_STATIC_JAVA_LIBRARY
* BUILD_HOST_JAVA_LIBRARY

通过名称大概就可以猜出每个变量所对应的模块类型。（在模块的 Android.mk 文件中，只要包含进这里对应的常量便可以执行相应类型模块的编译。对于 Android.mk 文件的编写请参见后文：“添加新的模块”。）

这些常量的值都是另外一个 Make 文件的路径，详细的编译方式都是在对应的 Make 文件中定义的。这些常量和 Make 文件的是一一对应的，对应规则也很简单：常量的名称是 Make 文件的文件名除去后缀全部改为大写然后加上“BUILD_”作为前缀。例如常量 BUILD_HOST_PREBUILT 的值对应的文件就是 host_prebuilt.mk。

这些 Make 文件的说明如表 3 所示：

|文件名|说明|
|--|--|
|host_static_library.mk|	定义了如何编译主机上的静态库。|
|host_shared_library.mk|	定义了如何编译主机上的共享库。|
|static_library.mk|	定义了如何编译设备上的静态库。|
|shared_library.mk|	定义了如何编译设备上的共享库。|
|executable.mk|定义了如何编译设备上的可执行文件。|
|host_executable.mk|	定义了如何编译主机上的可执行文件。|
|package.mk|	定义了如何编译 APK 文件。|
|prebuilt.mk|	定义了如何处理一个已经编译好的文件 ( 例如 Jar 包 )。|
|multi_prebuilt.mk|	定义了如何处理一个或多个已编译文件，该文件的实现依赖 prebuilt.mk。|
|host_prebuilt.mk|	处理一个或多个主机上使用的已编译文件，该文件的实现依赖 multi_prebuilt.mk。|
|java_library.mk|	定义了如何编译设备上的共享 Java 库。|
|static_java_library.mk|	定义了如何编译设备上的静态 Java 库。|
|host_java_library.mk|	定义了如何编译主机上的共享 Java 库|

不同类型的模块的编译过程会有一些相同的步骤，例如：编译一个 Java 库和编译一个 APK 文件都需要定义如何编译 Java 文件。因此，表 3 中的这些 Make 文件的定义中会包含一些共同的代码逻辑。为了减少代码冗余，需要将共同的代码复用起来，复用的方式是将共同代码放到专门的文件中，然后在其他文件中包含这些文件的方式来实现的。这些包含关系如图 5 所示。由于篇幅关系，这里就不再对其他文件做详细描述（其实这些文件从文件名称中就可以大致猜出其作用）。

![](http://www.ibm.com/developerworks/cn/opensource/os-cn-android-build/image005.png)