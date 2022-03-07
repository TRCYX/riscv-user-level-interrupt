# AXU15EG 启动记录

### 基本信息

AXU15EG 是基于 Zynq UltraScale+ MPSoC 的开发板。

**Zynq** UltraScale+ **MP**SoC 包括 **P**rocessing **S**ystem 和 **P**rogrammable **L**ogic 两部分，前者包括 4 个 Cortex-A53 核（**A**pplication **P**rocessing **U**nit）和 2 个 Cortex-R5F 核（**R**eal-Time **P**rocessing **U**nit）以及 Mali-400 GPU 等，后者为 FPGA。两者都有输出引脚可以驱动外设。PS 和 PL 间可以通过提供的两个方向的若干 AXI 总线相互访问数据。

ZynqMP 的相关文档：

- [Zynq UltraScale+ MPSoC Data Sheet: Overview (UG1085)](https://www.xilinx.com/support/documentation/data_sheets/ds891-zynq-ultrascale-plus-overview.pdf)，包括一些整体信息。
- [Zynq UltraScale+ Device Technical Reference Manual](https://www.xilinx.com/content/dam/xilinx/support/documentation/user_guides/ug1085-zynq-ultrascale-trm.pdf)，千页长文，比较具体。

AXU15EG 将 ZynqMP 的引脚接到外设上，形成开发板。ZynqMP 本身处于“核心板”上。核心板位于电风扇下，从侧面可以看到。核心板提供了 DDR、Flash、时钟等，规格可能类似同公司的 ACU15EG 产品。AXU15EG 在此基础上提供了各外设的连接，包括但不限于 SD 卡、USB、以太网、UART、PCIe、少量 LED 和按钮等。

AXU15EG 的相关文档：

- [AXU15EG 用户手册](http://www.alinx.vip:81/ug_en/AXU15EG_User_Manual.pdf)，未找到中文版。AXU9EG 有中文版文档。
- [ACU15EG 原理图](http://www.alinx.com/en/data/attached/afbbf7a3525fbc54b8e2322f0bbd0ebc/file/2fa3e9a3e321ca645007d76b03bf4e9e.rar)
- [ALINX 在知乎上提供的以 AXU2CG 为基础的教程](https://zhuanlan.zhihu.com/p/339433531)，内容不少，可以参照做实验，但是缺乏原理讲解，需要自己寻找原理。

**注意：User Manual 里显然有错和从其他版本复制粘贴来的部分，需要明鉴。例如 PL 端串口的 rxd 是 D11 引脚。可以和 ACU15EG 原理图对照。**

### ZynqMP 的（手动）设计流程，以及 AXU15EG 的一些外设情况

#### 在 Vivado 中设计硬件

1. 创建以 ZynqMP 作为目标设备的工程（如果 Vivado 中有目标开发板，创建时选对应开发板，后续可能会提供相关支持）。对于 AXU15EG，Vivado 不提供板级支持，设备则为 XCZU15EG-2FFVB1156I（Vivado 中称为 xczu15eg-ffvb1156-2-i）。

2. 在 IP Integrator 中以 Block Design 的形式，添加 Zynq UltraScale+ MPSoC 的 IP 并配置。需要配置的部分包括外设、时钟、DDR 等。*可以参考知乎教程 18 PS Hello World（上），但是注意 AXU2CG 和 AXU15EG 不是所有外设配置都一样，用户手册通常相对更准确。*

    1. 外设：

        - 配置页面上方要配置各 bank 电压。对于 AXU15EG，用于 **M**ultiplexed **IO** 的 Bank 0-2 是 LVCMOS18，用于控制信号的 Bank 3 是 LVCMOS33（默认）。此信息可从知乎教程 18 获得，根据原理图的 Bank 500-503 确认。

        - MIO 应该是 PS 侧的引脚。不同外设可以以一定规则对应到特定 MIO 上。可行的分配参考 UG1085 808 页的 Table 28-1（注意同一类外设的不同编号可用的 MIO 不尽相同），具体分配方式依赖开发板连线。

            对于 AXU15EG，用户手册大致提供了各外设对应的 MIO 引脚，除了 SD 卡，这个可以参考知乎教程 18，配置为 SD 1，支持 SD 2.0，MIO 46..51，CD 信号打开，配置在 45，经测可以正常使用。

            *只需要配置要使用的外设，不用的可以不配。*

        - 对于高速传输，ZynqMP 有 4 个 **G**igabit **T**ransceiver Lane，编号为 0-3，从外部看来应也是一系列引脚，原理图中在 Bank 505。USB 3.0、DisplayPort 等各需要使用若干 Lane。可行的 Lane 分配可以参考 UG1085 821 页的 Table 29-1（同上，注意同意外设的不同编号可使用的 Lane 编号是不同）。（以太网应该既可以用 MIO 也可以用 GT Lane，细节我还不太清楚。）

            实际分配也应和开发板有关。AXU15EG 中，DP 应使用 Lane 2 和 3，USB 3.0 应使用 Lane 1，M.2 PCIe 使用 Lane 0。用户手册在引脚分配表和文字描述中有矛盾，文字描述声称 PCIe 使用 Lane 1，但参考知乎教程 18 以及 USB 部分，猜测是文字描述错误。未验证。

    2. 时钟：

        - 对于 AXU15EG，Input Clocks 中，PSS_REF_CLK 为默认的 33.333 MHz，各 GT Lane 的参考时钟，综合用户手册和知乎教程，PCIe 为 100 MHz，USB 3.0 为 26 MHz，DP 为 27 MHz。PCIe 应使用 Lane 0 时钟（Ref Clk 0），USB 应使用 Lane 1，而两个 Lane DP 应均使用 Lane 2。GT Lane 的参考时钟也未测试。
        - Output Clocks 不太清楚需要何种特定配置。Vivado 应该会将各时钟至少配置在可用范围内。

    3. DDR：

        - 此处配置 PS 侧的 DDR。对于 AXU15EG，PS 侧有 4 块 [MT40A512M16LY-062E](https://media-www.micron.com/-/media/client/global/documents/products/data-sheet/dram/ddr4/8gb_ddr4_sdram.pdf?rev=74679247a1e24e57b6726071152f1384)，PL 侧则是 2 块。根据链接中的 Data Sheet，此内存单块（die）数据线宽 16 b，容量 1 GB。注意此内存可达到 3200 MHz（-062E），但 Zynq MPSoC 最多以 2400 MHz 驱动。查找 Speed Bin Tables 一节（345 页）发现可以兼容 -083（不兼容 -083E），所以再往下可以看见 -083 的参数。在 Vivado 中 Zynq MPSoC IP 的 DDR 设置中，

            - Speed Bin 为 DDR4 2400T
            - Cas Latency（CL）、RAS to CAS Delay（RCD）、Precharge Time（RP）均是 17
            - Cas Write Latency（CWL）是 12
            - tRC 是 46.16
            - tRASmin 是 32
            - tFAW 是 30.0

            以下见 Data Sheet 第二页：

            - IC Bus Width per die 是 16 Bits
            - Device Capacity per die 是 8192 MBits
            - Bank Group Address Count 是 1
            - Bank Address Count 是 2
            - Row Address Count 是 16
            - Column Address Count 是 10

            经测试此参数可用。

    4. PS-PL 配置

        - 可以配置双方通信用 AXI 总线、中断等。未细究。猜测主要位于 ZynqMP 内，和开发板无关。

    5. 打开左上角 Switch to Advanced Mode，还有包括 PCIe 设置、Advanced Configuration 中 UART 波特率设置（默认 115200）等，未细究。

3. 配置完成 ZynqMP 的 IP 后可以连线。参照教程 18，将右侧输出的 pl_clk0 连到左侧 AXI 的时钟构成自环，加入一个顶层 verilog 模块调用此 Block Design，就可以开始尝试软件开发了。需要更复杂的 PL 功能，可以将此 IP 作为 Block Design 的一部分，加入更多模块，打开 PS-PL 间的 AXI 总线实现 PS-PL 互联。

#### 在 Vitis 中设计软件

在 Vitis 之前，软件设计用 Xilinx SDK 完成，但后者已经被前者取代了。带来的后果包括但不限于一些命令行上的改动，如 hsi 命令被 **X**ilinx **S**oftware **C**ommand-Line **T**ool 中的 `hsi::<命令>` 取代，命令参数格式似乎没有变化。[这个链接](https://www.xilinx.com/html_docs/xilinx2021_1/vitis_doc/jed1590410655455.html)包括 Vitis 自带的 xsct 文档和 Vitis 文档，可以查阅各命令。

1. Vivado 左上角 File > Export > Export Hardware，生成一个 .xsa 文件记录硬件信息。老版本中可能是 .hdf 文件。
2. Vitis 在创建项目时可以采用此 .xsa。可跟随知乎教程 19 Hello World（下）体验，其内容包含各种启动方式简介。AXU15EG 的启动模式拨码开关非常小，需要很细的工具拨动，如剪刀，注意安全。注意拨码开关顺序以及何方向为 0。可幸板上印刷了这些信息。
3. 具体来说，PS 部分需要 **F**irst **S**tage **B**oot **L**oader、**PMU** **F**irm**w**are、**A**RM **T**rusted **F**irmware 等组件，启动 Linux 需要 U-Boot 等。[Xilinx Wiki 的相关链接](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/460653138/Xilinx+Open+Source+Linux)提供了较详细的步骤介绍。不过由于 AXU15EG 没有板级支持，需要参照其他开发板加入设备树等新信息。在此我做了一些尝试，待完成后整理。

#### AXU15EG 的其他零碎细节

- LED 是低有效

### Labeled RISC-V 启动记录

#### LvNA 结构

LvNA 指 Labeled von Neumann Architecture，Labeled RISC-V 代码中常以此架构名作为简称。

LvNA 的 CPU 位于 PL 中，处于 pardcore **B**lock **D**esign 下，相关文件位于 fpga/pardcore，其中 rtl 下的 verilog 代码由 Chisel 生成。bd 下的 pardcore.tcl 在 Vivado 中运行（在 TCL Console 中 `source pardcore.tcl`）后会引入 pardcore BD。

- 关于 BD 对应的 tcl：可以在 Vivado 中 File > Export > Export Block Design 自动生成。其结构大致为大量检查、各个 hierarchy 和 BD 顶层的创建过程（`proc`）。后半部分各创建过程中，大致会先定义接口，然后配置各 IP，之后配置它们间的连线。可以手动改动此 tcl 快速修改配置。特别地，较开始的部分会检查 Vivado 版本，但大致来说，直接修改被检查的版本号，不会有太多影响。

PS 和连接部分则各开发板有所不同，处于 zynq_soc BD 下，相关文件位于 fpga/board。各个 board 的 bd 下 prm.tcl 生成 zynq_soc BD。

PS 必须的外设相对较少，主要为 UART 和用于启动的存储设备。

PL 和 PS 间用多个 AXI 总线沟通。其中 PL 的内存为 PS DDR 的高 2G，PS 必须保留这段内存不受影响。展开 BD 可以看到 pardcore 的 M_AXI_MEM 直接（经过若干 interconnect）连接到 PS-PL 间的 AXI。PS 和 PL 间另外接了相互连接的 DMA，具体用途尚未整理。PL 的 UART 和 PS 相互连接，PS 应代 PL 输出串口内容，没有用到 PL 串口。

#### PS 软件编译

参考 Xilinx Wiki 和 LvNA 的 README，多数代码可以在 Xilinx 的 GitHub 上找到。可以采用对应工具版本的 tag，如 xilinx-v2020.2。

> 如果 Linux 不是工作机器的主要操作系统，Vivado 很可能装在 Windows 下。这样运行 LvNA 提供的 Makefile 调用 Vivado 会有诸多麻烦，不妨手动执行命令。Vivado 和 Vitis 在安装目录下提供了 settings64.bat/sh 等，会将其二进制加入当前 shell 的 PATH，在 Windows 下提供了大量 posix 工具和 ARM 编译器，可以用来编译 ATF，其 Makefile 针对 Windows 做过特化。U-Boot 则必须在 WSL 下编译，Vivado/Vitis 提供的工具可能不足以执行其 Makefile。
>
> 生成 rootfs 则目前来看只能在 Linux 下完成，因为 Windows 难以操作 ext4 文件系统。

##### ATF

直接参照两份文档编译即可。

##### FSBL 和 PMUFW

用 fpga 文件夹下的 make bootgen 即可。注意 fpga/boot 下的 mk.tcl 需要更新对 Vitis/xsct 的支持，具体来说，需要在 `generate_app` `set_repo_path` `create_sw_design` `get_os` `generate_target` 前加上 `hsi::`，`set_property` 前加上 `common::`。

Vitis 2020 对 PS 中断控制器 gic 的设置有 bug，如果中断信号会不止连向 PS 的中断输入，还连向其他位置（如用来检查的 LED），某些步骤运行会失败。需要修改 Block Design，删除 DMA 中断和 LED 之间的连接。

##### U-Boot

U-Boot 需要设备树。我尝试给 AXU15EG 写了一份设备树，见 https://github.com/TRCYX/u-boot-xlnx/tree/rvn，虽然可能有不少错误，但至少可以正确从 SD 卡启动 Linux。如果也是使用 2020.2，可以直接 clone 这个分支。否则可以参考此分支相对于 Xilinx 的 xilinx-v2020.2 tag 的改动适配。

**注意：**>= 2020.1 版本编译 U-Boot 的方式和 < 2020.1 不同，具体见 [Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841973/Build+U-Boot)，**和 LvNA README 中提供的方式不一样。**大致可以执行如下命令编译：

```shell
export CROSS_COMPILE=aarch64-linux-gnu-
export ARCH=aarch64
make distclean
make xilinx_zynqmp_virt_defconfig
export DEVICE_TREE=axu15eg
make
```

##### Linux 内核

Linux 用的设备树生成由 Xilinx GitHub 提供的 `device-tree-xlnx` 帮助完成，可以参考 LvNA REAME。

**需要在 `sdhci1` 下加入一行 `no-1-8-v;` 来禁止 Ultra High Speed，Linux 内核才能驱动 SD 卡，具体原因尚不清楚。**可以加在自动生成的 pcw.dtsi 中。

##### Linux rootfs

参照 LvNA README 制作 debian rootfs 即可，大概也可以选用自己喜欢的发行版。

