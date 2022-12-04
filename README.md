# bpf-develop-tutorial: learn CO-RE ebpf with example tools 

这是一个基于 `CO-RE`（一次编译，到处运行）的 `libbpf` 的 eBPF 的开发教程，提供了从入门到进阶的 eBPF 开发实践指南，包括基本概念、代码实例、实际应用等内容。我们主要提供了一些 eBPF 工具的案例，帮助开发者学习 eBPF 的开发方法和技巧。教程内容可以在目录中找到，每个目录都是一个独立的 eBPF 工具案例。

在学习 eBPF 的过程中，我们受到了 [tutorial_bcc_python_developer](https://github.com/iovisor/bcc/blob/master/docs/tutorial_bcc_python_developer.md) 的许多启发和帮助，但从 2022 年的角度出发，使用 libbpf 开发 eBPF 的应用是目前相对更好的选择。但目前似乎很少有基于 libbpf 和 BPF CO-RE 出发的、通过案例和工具介绍 eBPF 开发的教程，因此我们发起了这个项目。

## 目录

- [lesson 0-introduce](0-introduce/introduce.md) 介绍 eBPF 的基本概念和常见的开发工具
- [lesson 1-helloworld](1-helloworld/README.md) 演示如何使用 eBPF 开发最简单的「Hello World」程序，介绍 eBPF 的基本框架和开发流程
- [lesson 2-fentry-unlink](2-fentry-unlink/README.md) 基于 eBPF 的 fentry hook，演示了如何捕获并记录 unlink 系统调用
- [lesson 3-kprobe-unlink](3-kprobe-unlink/README.md) 基于 eBPF 的 kprobe hook，演示了如何捕获并记录 unlink 系统调用
- [lesson 4-opensnoop](4-opensnoop/README.md)
- [lesson 5-uprobe-bashreadline](5-uprobe-bashreadline/README.md)
- [lesson 6-sigsnoop](6-sigsnoop/README.md)
- [lesson 7-execsnoop](7-execsnoop/README.md)
- [lesson 8-runqslower](8-runqslower/README.md)
- [lesson 9-runqlat](9-runqlat/README.md)
- [lesson 10-hardirqs](20-hardirqs/README.md)
- [lesson 11-llcstat](21-llcstat/README.md)
- [lesson 12-bindsnoop](12-bindsnoop/README.md)
- [lesson 13-tcpconnlat](13-tcpconnlat/README.md)
- [lesson 14-tcpstates](14-tcpstates/README.md)
- [lesson 15-tcprtt](15-tcprtt/README.md)
- [lesson 16-profile](16-profile/README.md)
- [lesson 17-memleak](17-memleak/README.md)
- [lesson 18-biopattern](18-biopattern/README.md)
- [lesson 19-syscount](19-syscount/README.md)
- [lesson 20-lsm-connect](20-lsm-connect/README.md)
- [lesson 21-tc](21-tc/README.md)
  
## 为什么需要基于 libbpf 和 BPF CO-RE 的教程？

> 历史上，当需要开发一个BPF应用时可以选择BCC 框架，在实现各种用于Tracepoints的BPF程序时需要将BPF程序加载到内核中。BCC提供了内置的Clang编译器，可以在运行时编译BPF代码，并将其定制为符合特定主机内核的程序。这是在不断变化的内核内部下开发可维护的BPF应用程序的唯一方法。在BPF的可移植性和CO-RE一文中详细介绍了为什么会这样，以及为什么BCC是之前唯一的可行方式，此外还解释了为什么 libbpf 是目前比较好的选择。去年，Libbpf的功能和复杂性得到了重大提升，消除了与BCC之间的很多差异(特别是对Tracepoints应用来说)，并增加了很多BCC不支持的新的且强大的特性(如全局变量和BPF skeletons)。
> 
> 诚然，BCC会竭尽全力简化BPF开发人员的工作，但有时在获取便利性的同时也增加了问题定位和修复的困难度。用户必须记住其命名规范以及自动生成的用于Tracepoints的结构体，且必须依赖这些代码的重写来读取内核数据和获取kprobe参数。当使用BPF map时，需要编写一个半面向对象的C代码，这与内核中发生的情况并不完全匹配。除此之外，BCC使得用户在用户空间编写了大量样板代码，且需要手动配置最琐碎的部分。
> 
> 如上所述，BCC依赖运行时编译，且本身嵌入了庞大的LLVM/Clang库，由于这些原因，BCC与理想的使用有一定差距：
> - 编译时的高资源利用率(内存和CPU)，在繁忙的服务器上时有可能干扰主流程。
> - 依赖内核头文件包，不得不在每台目标主机上进行安装。即使这样，如果需要某些没有通过公共头文件暴露的内核内容时，需要将类型定义拷贝黏贴到BPF代码中，通过这种方式达成目的。
> - 即使是很小的编译时错误也只能在运行时被检测到，之后不得不重新编译并重启用户层的应用；这大大影响了开发的迭代时间(并增加了挫败感...)
>
> Libbpf + BPF CO-RE (Compile Once – Run Everywhere) 选择了一个不同的方式，其思想在于将BPF程序视为一个普通的用户空间的程序：仅需要将其编译成一些小的二进制，然后不用经过修改就可以部署到目的主机上。libbpf扮演了BPF程序的加载器，负责配置工作(重定位，加载和校验BPF程序，创建BPF maps，附加到BPF钩子上等)，开发者仅需要关注BPF程序的正确性和性能即可。这种方式使得开销降到了最低，消除了大量依赖，提升了整体开发者的开发体验。
>
> 在API和代码约定方面，libbpf坚持"最少意外"的哲学，即大部分内容都需要明确地阐述：不会隐含任何头文件，也不会重写代码。仅使用简单的C代码和适当的辅助宏即可消除大部分单调的环节。 此外，用户编写的是需要执行的内容，BPF应用程序的结构是一对一的，最终由内核验证并执行。
>
> 参考：[BCC 到libbpf 的转换指南【译】 - 深入浅出eBPF: https://www.ebpf.top/post/bcc-to-libbpf-guid/](https://www.ebpf.top/post/bcc-to-libbpf-guid/)