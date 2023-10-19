# Linux进程管理：task_struct之进程标识

`Linux`中的`task_struct`是一个数据结构，定义在`include/linux/sched.h`文件中，用于描述一个运行在内核中的进程。它记录了进程的各种信息，包括但不限于进程的**状态**、**标识符（ID）**、**进程间的关系**、**调度信息**、**所接收的信号**、**内存信息**、**文件信息**等等。

如果学习过操作系统这门课程，会感到`task_struct`与操作系统中`PCB`的概念非常相似。实际上，我们可以说`task_struct`就是`PCB`在`Linux`上的具体实现。理解`task_struct`对于学习`Linux`的进程管理机制具有重要意义。

前面的文章介绍了`task_struct`中的**[进程状态](http://mp.weixin.qq.com/s?__biz=Mzk0NTU0MTE5NA==&mid=2247483661&idx=1&sn=76977f61d1784eb72b36a60be109c909&chksm=c3129828f465113e90a43d31da109ea958fb2eb9ddba0bc9a9074d02f7f69d23051b5a9e6f80&scene=21#wechat_redirect)。**

本篇文章将介绍`task_struct`中的**进程标识。**

## 一、task_struct中的进程标识

`Linux`中的每个进程都有一个进程描述符`task_struct`。`Linux`中的线程是轻量级进程，本质上也是进程。因此，使用`32`位的`task_struct`地址标识进程是可行的。实际上，内核对进程的大部分引用是通过进程描述符指针实现的。

此外，`Linux`还为每个进程提供唯一的进程标识符`PID`，定义在`task_struct`中：

```c
pid_t					pid;
```



## 二、`pid`的查看

### 2.1 在shell中查看



### 2.2 通过`get_pid()`查看



## 三、进程描述符处理

`thread_info`（线程描述符）

`thread_info`的定义与架构有关，一般在`arch/xxx/include/asm/thread_info.h`。其中`xxx`表示架构名称，常见的有`arm`、`arm64`、`x86`、`riscv`等。下面展示`arch/xxx/include/asm/thread_info.h`

```

```

`asm/current.h`定义`current`



`include/linux/thread_info.h`定义`current_thread_info()`



## 四、标识当前进程



## 参考资料

[1] Bovet, D.P.，Cesati, M. 深入理解Linux内核[M]. 第三版. 陈莉君，张琼声，张宏伟译. 北京：中国电力出版社，2007.

[2] Love, R. Linux内核设计与实现[M]. 第三版. 陈莉君，康华译. 北京：机械工业出版社，2011. 
