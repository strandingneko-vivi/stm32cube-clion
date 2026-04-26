# 1 安装软件

Reference: https://club.rt-thread.org/ask/article/59411cc998abc897.html

clion: [CLion: A Cross-Platform IDE for C and C++ by JetBrains](https://www.jetbrains.com/clion/), 免费使用30天, 教育可以免费使用, 或者淘宝

openocd: [Download OpenOCD for Windows (gnutoolchains.com)](https://gnutoolchains.com/arm-eabi/openocd/)

> 注意 openocd 不能使用 STM32CubeIDE  自带的，STM32CubeIDE  有做一定的魔改操作。
>
> 需要将 bin 文件夹添加到系统环境变量。

STM32CubeIDE : [STM32CubeIDE - STM32的集成开发环境 - 意法半导体STMicroelectronics](https://www.st.com/zh/development-tools/STM32CubeIDE .html)

STM32CubeMX : [STM32CubeMX - STM32Cube初始化代码生成器 - 意法半导体STMicroelectronics](https://www.st.com/zh/development-tools/STM32CubeMX .html)

> 建议使用 STM32CubeIDE  来配置项目，也建议安装 STM32CubeIDE  来获取捆绑的一些包，或自行安装也可以。
>
> 此处仅使用 STM32CubeMX  来初始化项目来进行演示。

MinGW: [Download File List - MinGW - Minimalist GNU for Windows - OSDN](https://osdn.net/projects/mingw/releases/)，[Download MinGW - Minimalist GNU for Windows (sourceforge.net)](https://sourceforge.net/projects/mingw/files/latest/download)

此处直接用 clion 捆绑的 MinGW， 后文介绍。

arm-none-eabi-gcc: 这个可以直接使用 stm32cube ide 自带的。

# 2 创建项目

## 2.1 使用 STM32CubeMX  创建项目

不建议使用 clion 直接新建 STM32CubeMX  项目，虽然 clion 提供了该功能。原因是 clion 会默认一个芯片来进行创建并打开 STM32CubeMX  来初始化项目，这个芯片往往不是我们想要的。

![image-20230806211300998](figures/image-20230806211300998.png)

我们使用 STM32CubeIDE  或者 STM32CubeMX  来创建一个项目，如果是 STM32CubeMX   需要注意以下内容， STM32CubeIDE  则默认配置好这些内容了。

![image-20230806212208530](figures/image-20230806212208530.png)

点击 GENERATE CODE 生成项目文件。

# 3 打开项目

使用 clion 打开项目，可以选择 ioc 文件并作为项目打开。

![image-20230806212710560](figures/image-20230806212710560.png)

![image-20230806212726324](figures/image-20230806212726324.png)

打开后，会自动加载内容，并弹出面板配置文件，自制板卡选择跳过即可。

![image-20230806212900461](figures/image-20230806212900461.png)

之后项目会自动更新 cmake 文件。

# 4 Clion 配置

## 4.1 工具链（程序编译）

添加一个 MinGW 环境。

MinGW 和 CMake 以及调试器，选择捆绑的版本。

构建工具， C 编译器，C++ 编译器可以选 STM32Cube IDE 内置的版本。

![image-20230806213303554](figures/image-20230806213303554.png)

CMake 栏可以确定一下工具链是否正确。

![image-20230806213502872](figures/image-20230806213502872.png)

此时，编译项目，可以发现项目可以正常编译。

![image-20230806221440656](figures/image-20230806221440656.png)

## 4.2 程序烧录

添加 openocd 和 STM32CubeMX  的路径。

![image-20230806224749145](figures/image-20230806224749145.png)

在此之前，建议新建一个 `config` 文件夹，来保存一些内容（后文介绍）。

![image-20230806221622439](figures/image-20230806221622439.png)

点击编辑配置。

![image-20230806221949142](figures/image-20230806221949142.png)

此时有一个默认的 CMake 配置，但这个配置不能用于烧录和调试。

![image-20230806222048406](figures/image-20230806222048406.png)

点击 `+` 号，添加一个 `openocd 下载并运行` 的配置。

![image-20230806222148397](figures/image-20230806222148397.png)

主要需要配置的内容为`面板配置文件`，此处翻译不准，原文为 `Board config file`，即 板级配置文件。

![image-20230806222231986](figures/image-20230806222231986.png)

这个文件需要自己生成，可以参考 `openocd` 目录下的这几个目录内的文件夹。

- **board**：板卡配置，各种官方板卡
- **interface**：仿真器类型配置，比如ST-Link、CMSIS-DAP等都在里面
- **target**：芯片类型配置，STM32F1xx、STM32L0XX等等都在里面

![image-20230806222531943](figures/image-20230806222531943.png)

此处，我使用的是 cmsis-dap（安富莱 H7-Tool），因此，可以新建一个 `cmsis-dap.cfg` ：

内容如下：

```
# choose st-link/j-link/dap-link etc.
adapter driver cmsis-dap
transport select swd

# 0x10000 = 64K Flash Size
set FLASH_SIZE 0x20000

source [find target/stm32h7x.cfg]

# download speed = 10MHz
adapter speed 10000
```

关于 `openocd` 的具体解释可以参考：[Top (OpenOCD User’s Guide)](https://openocd.org/doc/html/index.html#SEC_Contents)

如果使用的是 st-link，则可以这样修改：

```
source [find interface/stlink.cfg]
transport select hla_swd

source [find target/stm32f1x.cfg]

# download speed = 10MHz
adapter speed 10000
```

此时，就可以正常下载项目了。

![image-20230806224241914](figures/image-20230806224241914.png)

## 4.3 程序调试

点击编辑配置，新增 `嵌入式 GDB 服务器` 配置。

![image-20230806224945568](figures/image-20230806224945568.png)

修改内容如下：

1. `target remote` 对应 `openocd 下载并运行` 中的 `GDB 端口`；
2. `GDB 服务器` 选择 `openocd.exe` 的目录；
3. GDB 服务器实参根据对应的 **调试器** 和 **芯片** 选择。

![image-20230806225227352](figures/image-20230806225227352.png)

此时，点击调试，可以发现程序可以正常调试。

![image-20230807085637009](figures/image-20230807085637009.png)

我们可以在 **外设** 下添加 svd 文件，以实现寄存器监看。

![image-20230807085748806](figures/image-20230807085748806.png)

如果找不到 svd 文件，可以到以下目录中查找。

```
\STM32CubeIDE_1.13.1\STM32CubeIDE\plugins\com.st.stm32cube.ide.mcu.productdb.debug_2.1.0.202306151215\resources\cmsis\STMicroelectronics_CMSIS_SVD
```

此处包含了所有 stm 的 mcu 的 svd 文件，常用的 svd 文件也可以保存到 config 文件夹，用以拷贝使用。

![image-20230807085900660](figures/image-20230807085900660.png)

## 4.4 项目构建

> 以下配置最好在 STM32CubeIDE 里进行配置，否则 STM32CubeMX 重新配置初始化代码后，会删除以下配置。
>
> 似乎是 Clion 在检测到更改时，会自动从 .cproject 解析相关配置？

### 4.4.1 语言标准设置

![image-20230807091912285](figures/image-20230807091912285.png)

### 4.4.2 添加 .c/.h 文件

clion 项目基于 CMakeLists.txt 文件来构建项目，针对于我们自己添加的 .c/.h 文件，我们只需在以下两处进行添加即可。

![image-20230807090308848](figures/image-20230807090308848.png)

### 4.4.3 添加预处理器宏定义

在此处添加即可，前面加上 `-D`。

![image-20230807090459066](figures/image-20230807090459066.png)

- 无值宏：`add_definitions(-DMG_ENABLE_OPENSSL)`
- 有值宏：`add_definitions(-DLIBEVENT_VERSION_NUMBER=0x02010800)`

在高版本的 CMake 中，也可以使用 [`add_compile_definitions()`](https://cmake.org/cmake/help/latest/command/add_compile_definitions.html#command:add_compile_definitions) 来添加宏定义。

### 4.4.4 链接脚本设置

![image-20230807092617962](figures/image-20230807092617962.png)

### 4.4.5 优化等级

根据 `CMAKE_BUILD_TYPE` 来指定优化参数，对应参数为：

- Release
- Debug
- RelWithDebInfo
- MinSizeRel

`CMAKE_BUILD_TYPE`默认不是以上的参数，是空值，优化参数，此处默认 `Og -g` 等级。

![image-20230807093739227](figures/image-20230807093739227.png)

GCC 优化等级可以参考：[Optimize Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)

