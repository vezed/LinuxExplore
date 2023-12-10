# Linux可执行文件：ELF文件格式

## 1  ELF文件介绍

`ELF`，全称为`Executable and Linkable Format`，即可执行可链接文件格式。它是`UNIX`系统实验室（`USL`）开发和发布的一种用于二进制文件、可执行文件、目标代码、共享库以及核心转储格式文件的应用程序二进制接口（`ABI`）。`ELF`是`Linux`的主要可执行文件格式。

## 1.1 ELF 文件分类

ELF文件主要分为四类：**可执行文件**、**可重定位文件**、**共享目标文件**和**核心转储文件**。

- **可执行文件（Executable File）**：能被操作系统直接运行的程序，其代码和数据都有固定的内存地址，例如用户命令 `ls`；
- **可重定位文件（Relocatable File）**：用于静态链接。包含基础代码和数据的文件，用于链接生成可以执行文件或共享目标文件，它的代码和数据没有指定的绝对地址，适合于与其他目标文件链接来创建可执行文件或者共享目标文件，例如`*.o`或`lib*.a`文件；
- **共享目标文件（Shared Object File）**：用于动态链接。包含了在运行时被动态链接库加载的函数和变量，用于和可重定位文件或其他共享目标文件一起生成可执行文件，通常为`*.so`文件；
- **核心转储文件（Core Dump File）**：记录了进程被终止时的内存内容和有关进程状态的其他信息，这种信息对于故障排查和问题定位来说非常有价值。

## 1.2 ELF 文件头

ELF 文件头描述了 ELF 文件的总体信息，包括系统相关、类型相关、加载相关和链接相关的信息。

1. 系统相关，比如ELF 文件标识的魔数，以及硬件和平台等相关信息，增加了 ELF 文件的移植性，使交叉编译成为可能；
2. 类型相关，比如 ELF 文件类型，分别有目标文件、可执行文件、动态链接库与核心转储文件；
3. 加载相关，比如程序头，描述了 ELF 文件被加载时的段信息；
4. 链接相关，比如节头，描述了 ELF 文件的节信息。

## 2  ELF文件相关命令

以最简单的C程序为例：

```C
#include <stdio.h>

int main(int argc, char const *argv[]) {
        printf("hello world!\n");
        return 0;
}
```

使用`gcc`命令进行编译：

```C
gcc -o helloworld helloworld.c
```

编译产生的`helloworld`文件即为`ELF`格式的可执行文件。如下命令可查看其相关信息：

### 2.1  **`readelf`命令**

最常用的命令，用于显示一个或多个ELF格式对象文件的信息

```C
  -a --all               Equivalent to: -h -l -S -s -r -d -V -A -I
  -h --file-header       Display the ELF file header
  -l --program-headers   Display the program headers
     --segments          An alias for --program-headers
  -S --section-headers   Display the sections' header
     --sections          An alias for --section-headers
  -g --section-groups    Display the section groups
  -t --section-details   Display the section details
  -e --headers           Equivalent to: -h -l -S
  -s --syms              Display the symbol table
     --symbols           An alias for --syms
  --dyn-syms             Display the dynamic symbol table
  -n --notes             Display the core notes (if present)
  -r --relocs            Display the relocations (if present)
  -u --unwind            Display the unwind info (if present)
  -d --dynamic           Display the dynamic section (if present)
  -V --version-info      Display the version sections (if present)
  -A --arch-specific     Display architecture specific information (if any)
  -c --archive-index     Display the symbol/file index in an archive
  -D --use-dynamic       Use the dynamic section info when displaying symbols
  -x --hex-dump=<number|name>
                         Dump the contents of section <number|name> as bytes
  -p --string-dump=<number|name>
                         Dump the contents of section <number|name> as strings
  -R --relocated-dump=<number|name>
                         Dump the contents of section <number|name> as relocated bytes
  -z --decompress        Decompress section before dumping it
  -w[lLiaprmfFsoRtUuTgAckK] or
  --debug-dump[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,
               =frames-interp,=str,=loc,=Ranges,=pubtypes,
               =gdb_index,=trace_info,=trace_abbrev,=trace_aranges,
               =addr,=cu_index,=links,=follow-links]
                         Display the contents of DWARF debug sections
  --dwarf-depth=N        Do not display DIEs at depth N or greater
  --dwarf-start=N        Display DIEs starting with N, at the same depth
                         or deeper
  --ctf=<number|name>    Display CTF info from section <number|name>
  --ctf-parent=<number|name>
                         Use section <number|name> as the CTF parent

  --ctf-symbols=<number|name>
                         Use section <number|name> as the CTF external symtab

  --ctf-strings=<number|name>
                         Use section <number|name> as the CTF external strtab

  -I --histogram         Display histogram of bucket list lengths
  -W --wide              Allow output width to exceed 80 characters
  @<file>                Read options from <file>
  -H --help              Display this information
  -v --version           Display the version number of readelf
```

查看`ELF`文件的文件头：

```C
$ readelf -h helloworld
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x1060
  Start of program headers:          64 (bytes into file)
  Start of section headers:          14720 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

查看

### 2.2  **`ldd`（list dynamic dependencies）命令**

用于查看可执行文件或共享库所依赖的动态链接库及其绝对路径。

```C
$ ldd helloworld
	linux-vdso.so.1 (0x00007ffc527f1000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f0dcec44000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f0dcee4f000)
```

这个结果表示helloworld程序依赖于以下动态链接库：

- `linux-vdso.so.1`：全称为`Linux Virtual Dynamic Shared Object`，是Linux操作系统中的一个虚拟动态共享对象，它提供了一些基本的系统调用功能。它的地址是`0x00007ffc527f1000`。
- `libc.so.6`：全称为`GNU C Library`，也被称为`glibc`，是`Linux`系统中最基本的库之一，提供了许多通用的函数和数据结构，如`printf()`、`scanf()`、`malloc()`、`free()`等。它的地址是`0x00007f0dcec44000`。
- `/lib64/ld-linux-x86-64.so.2`：是针对64位x86-64架构的Linux系统上的动态链接器，用于加载其他共享库文件（以.so结尾的文件）。当一个程序需要使用某个共享库中的函数或变量时，动态链接器会在运行时将该库加载到内存中，并将程序中的符号引用与库中的实现进行绑定。它的地址是`0x00007f0dcee4f000`。

### 2.3  **`file`命令**

用于查看文件的基本信息。

```C
$ file helloworld
helloworld: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f784a16dbebee4db128cc09e54a2919218ae2354, for GNU/Linux 3.2.0, not stripped
```

这段信息是关于一个名为"helloworld"的可执行文件的属性。下面是对每个属性的解释：

- `ELF 64-bit LSB shared object`: 这是一个ELF（Executable and Linkable Format）格式的共享对象，表示它是一个可执行文件或共享库。
- `x86-64`: 这个文件是针对x86-64架构（即AMD64或Intel 64位处理器）编译的。
- `version 1 (SYSV)`: 这个文件的版本是1，使用的是System V接口。
- `dynamically linked`: 这个文件是动态链接的，意味着它依赖于其他共享库来运行。
- `interpreter /lib64/ld-linux-x86-64.so.2`: 这个文件使用了一个名为`/lib64/ld-linux-x86-64.so.2`的解释器来加载和链接其他共享库。
- `BuildID[sha1]=f784a16dbebee4db128cc09e54a2919218ae2354`: 这个文件有一个`Build ID`，用于唯一标识这个特定的构建版本。
- `for GNU/Linux 3.2.0`: 这个文件是为`GNU/Linux`操作系统版本`3.2.0`编译的。
- `not stripped`: 这个文件没有被剥离，也就是说，它的二进制代码中包含了一些调试信息和其他元数据。

### 2.4  **`nm`命令**

用于显示ELF格式文件中的符号表信息。

```C
$ nm helloworld
0000000000004010 B __bss_start
0000000000004010 b completed.8061
                 w __cxa_finalize@@GLIBC_2.2.5
0000000000004000 D __data_start
...
```

- `0000000000004010 B __bss_start`：表示一个全局变量或静态变量，其地址为`0x4010`，类型为字节（B）。
- `0000000000004010 b completed.8061`：表示一个字符串常量，值为`completed.8061`，类型为字节（b）。
- `0000000000004000 D __data_start`：表示一个数据段的起始地址，类型为双字（D）。
- ...

## 参考资料

标准文档：https://refspecs.linuxbase.org/elf/elf.pdf

