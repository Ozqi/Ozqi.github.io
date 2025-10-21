---
title: 性能分析与调试工具
date: 2025-10-21 10:00:00
categories:
  - 开发工具
tags:
  - Perf
  - GDB
  - 性能分析
  - 调试
  - Linux
---

# 性能分析、DEBUG

高质量的 log, error tracing, 和 profiling 工具。不仅仅是能搭件一个working的系统，而是有能力去**从头到尾追踪每一个延迟点**，把系统之间的关联和可能存在的bottleneck拆解成一系列可量化的问题，并在上线后持续做 cost/performance profiling。


## Perf
```bash
# 1. 记录程序执行数据（包含调用栈）
perf record -g --call-graph=dwarf ./your_program [args]

# 2. 查看报告，可以看到具体函数的性能占比
perf report

# 3. 只关注特定函数
perf report --symbol-filter=function_name

# 4. 生成详细的调用图
perf report --call-graph=graph,0.5,caller
```

### Perf top
**Perf Top**: 是 `perf` 工具的一部分，提供了实时查看系统或特定进程的性能热点（即消耗CPU最多的函数）。它对于快速定位哪个函数正在占用大量CPU时间特别有用。

### Perf 火焰图

https://github.com/brendangregg/FlameGraph

### perfetto(谷歌perf工具)
https://perfetto.dev/docs/getting-started/system-tracing
需要集成在C++项目中。


## Coredump

程序由于各种异常或者 bug 导致在运行过程中异常退出或者中止（并且在满足一定条件下）会产生一个叫做 core 的文件。

通常情况下，core 文件会包含了程序运行时的内存，寄存器状态，堆栈指针，内存管理信息还有各种函数调用堆栈信息等
默认的存储位置：

```shell
 cat  /proc/sys/kernel/core_pattern
```

使用 GDB 加载该 coredump

gdb /path/to/your/program /data/coredump/core.your_program_name.pid

### **如何主动生成 core dump**

```bash

gcore -o /desired/path/core_dump_filename PID

# 或者使用kill  -6， 但这会终止当前进程。

kill -SIGABRT PID

```

例如，要将核心转储（core dump）文件保存到工作空间（`${workspaceFolder}`）下的 `coredump` 目录，您可以按照以下步骤进行配置：


1. **创建 `coredump` 目录**：

   首先，确保在您的工作空间目录下存在一个名为 `coredump` 的目录。您可以使用以下命令创建该目录：

   ```bash
   mkdir -p ${workspaceFolder}/coredump
   ```

1. **设置核心转储文件的生成路径和命名格式**：

   核心转储文件的生成路径和命名由内核参数 `kernel.core_pattern` 控制。您可以通过以下命令将核心转储文件保存到 `coredump` 目录，并设置文件名格式：

   ```bash
   echo "${workspaceFolder}/coredump/core-%e-%p-%t" | sudo tee /proc/sys/kernel/core_pattern
   ```

例如，生成的核心转储文件可能命名为 `core-myapp-1234-1617181920`，表示应用程序名为 `myapp`，进程 ID 为 `1234`，时间戳为 `1617181920`。

**请注意，此设置仅对当前会话有效**



## GDB
### 一些命令

[^]: https://blog.csdn.net/zdy0_2004/article/details/80102076

```bash
gcc -g test.c -o test.exe	#-g=编译调试版 -o=输出
gdb	test.exe				#或者进(gdb)再file test.exe
```

### 查看代码

```bash
list n #显示以第n行
list main #查看某函数
set listsize 20 #修改默认显示行数
list <first> , <last>#显示first到last行

```

### 断点

```bash
#设置断点 break + 行号
break
#清除断点 delete [breakpoints] [rang...]
#breakpoints为断点号 range表示断点号的范围 不指定断点号则表明删除所有的断点

info break #查看断点信息(看断点号)

delete 1   #删除1号断点
```

| 命令     | 命令缩写 | 命令说明                                                                                 |
| -------- | -------- | ---------------------------------------------------------------------------------------- |
| file     | f        | 装入要调试的文件路径                                                                     |
|          |          |                                                                                          |
|          |          |                                                                                          |
| start    | st       | 开始执行程序,在 main 函数的第一条语句前面停下来                                          |
| run      | r        | 开始运行程序                                                                             |
| step     | s        | 执行下一条语句,如果该语句为函数调用,则进入函数执行其中的第一条语句                       |
| next     | n        | 执行下一条语句,如果该语句为函数调用,不会进入函数内部执行(即不会一步步地调试函数内部语句) |
| continue | c        | 继续程序的运行,直到遇到下一个断点                                                        |

### **观察变量**

```bash
 watch                 监视变量值的变化
 display         disp  跟踪查看某个变量,每次停下来都显示它的值
 print           p     打印内部变量值
 set var name=v        设置变量的值
```

### **观察内存**

```bash
#x/fmt addres
x/16x &x	#查看x地址往后的内存
```

![image-20220426193259786](./image/Ubuntu.assets/image-20220426193259786.png)

### 观察寄存器及 info

| info register | i reg     | 寄存器      |
| ------------- | --------- | -------- |
|               | i b       | 查看断点信息   |
|               | i display | 跟踪查看哪些变量 |
|               | i source  | 程序信息     |
|               |           |          |





## I/O 观察

实时观察：htop

### iostat\_观察 io 开销

https://zhuanlan.zhihu.com/p/649946956

```bash
iostat -d -k 1 10         #查看TPS和吞吐量信息(磁盘读写速度单位为KB)，每1s收集1次数据，共收集10次
iostat -d -m 2            #查看TPS和吞吐量信息(磁盘读写速度单位为MB)，每2s收集1次数据
iostat -d -x -k 1 10      #查看设备使用率（%util）、响应时间（await）等详细数据， 每1s收集1次数据，总共收集10次
iostat -c 1 10            #查看cpu状态，每1s收集1次数据，总共收集10次
```

iostat 输出内容分析

在 linux 命令行中输入 iostat，通常将会出现下面的输出：

```text
[root@localhost ~]# iostat
Linux 5.14.0-284.11.1.el9_2.x86_64 (localhost.localdomain)      08/07/2023      _x86_64_        (4 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.31    0.01    0.44    0.02    0.00   99.22

Device             tps    kB_read/s    kB_wrtn/s    kB_dscd/s    kB_read    kB_wrtn    kB_dscd
dm-0              3.19        72.63        35.90         0.00     202007      99835          0
dm-1              0.04         0.84         0.00         0.00       2348          0          0
nvme0n1           3.36        93.22        36.64         0.00     259264     101903          0
sr0               0.02         0.75         0.00         0.00       2096          0          0
```

首先第一行：

```text
Linux 5.14.0-284.11.1.el9_2.x86_64 (localhost.localdomain)      08/07/2023      _x86_64_        (4 CPU)
```

Linux 5.14.0-284.11.1.el9*2.x86_64 是内核的版本号，localhost.localdomain 则是主机的名字， `08/07/2023`当前的日期， \_x86_64*是 CPU 的架构， (4 CPU)显示了当前系统的 CPU 的数量。

接着看第二部分，这部分是 CPU 的相关信息，其实和**top 命令**的输出是类似的。

```text
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.31    0.01    0.44    0.02    0.00   99.22
```

cpu 属性值说明：

- %user：CPU 处在用户模式下的时间百分比。
- %nice：CPU 处在带 NICE 值的用户模式下的时间百分比。
- %system：CPU 处在系统模式下的时间百分比。
- %iowait：CPU 等待输入输出完成时间的百分比。
- %steal：管理程序维护另一个虚拟处理器时，虚拟 CPU 的无意识等待时间百分比。
- %idle：CPU 空闲时间百分比。

iowait 这个指标有点说法。