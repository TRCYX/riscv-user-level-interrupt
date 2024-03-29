# riscv-user-level-interrupt

- [项目地址](https://github.com/TRCYX/rv-n-ext-impl/tree/uipi)
- [设计文档](https://github.com/TRCYX/rv-n-ext-impl/blob/uipi/docs/src/ch2_9_uipi.md)
- [AXU15EG 启动记录](AXU15EG%20启动记录.md)

初步计划
- 在 Qemu 中理解并复现已有工作
- 加入开发，验证用户态中断带来的性能提升
- 重新思考调度模型，尝试给出用户态中断的进一步设计

10-16
- 阅读已有工作文档和 RISC-V 标准中相应部分
- 尝试编译运行
- 初步思考用户态中断的用途
- 初步了解 Intel 用户态中断

10-23
- 阅读 Intel 用户态中断相关文档
- 运行 rCoreN
  - 排查在 Qemu 中因为内存用尽而无法正确 fork 出应用程序的问题
  - 阅读部分代码

10-30
- 阅读对 Qemu 的修改
- 阅读 ACLINT 标准
- 想

11-6
- Debug Qemu 6 上的 N 扩展实现，已能正常启动 rCore，但串口输出尚有问题
- 修改 Qemu 的 ACLINT 实现，使其支持用户态中断
- 开始编写在 rCore 上用 ACLINT 跨核发送用户态中断的演示程序
- 记录 FIFO 设计方案和存在的问题

11-13
- 放弃 ACLINT
- 完成邻接矩阵式跨核中断控制器的初版文档

11-20
- 完善文档，翻译接收方的 UIID 改由硬件完成
- 思考邻接表式中断控制器
- 开始在 Qemu 上编写模拟器实现，大致能初始化中断控制器

12-4
- 在 Qemu 6 上实现邻接矩阵版本
- Debug Qemu 6 N 扩展实现

12-11
- 在 rCore-N 中实现邻接矩阵版本的控制

1-15
- 了解 Zynq UltraScale+ MPSoC 和 AXU15EG 开发板，尝试在上面运行简单的 PL 和 PS 应用
- 了解 Labeled RISC-V，修改其硬件代码以尝试支持 AXU15EG

1-22
- 尝试编译 Labeled RISC-V 在 ZynqMP 上运行需要的 ARM 软件，编写 AXU15EG 设备树
- 整理文档
