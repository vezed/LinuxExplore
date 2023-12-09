# Linux进程管理-task_struct之进程状态

`Linux`中的`task_struct`是一个数据结构，定义在`include/linux/sched.h`文件中，用于描述一个运行在内核中的进程。它记录了进程的各种信息，包括但不限于进程的**状态**、**标识符（ID）**、**进程间的关系**、**调度信息**、**所接收的信号**、**内存信息**、**文件信息**等等。

如果学习过操作系统这门课程，会感到`task_struct`与操作系统中`PCB`的概念非常相似。实际上，我们可以说`task_struct`就是`PCB`在`Linux`上的具体实现。深入理解和掌握`task_struct`对于理解`Linux`的进程管理机制具有重要的意义。

本篇文章将介绍`task_struct`中的**进程状态**。

## 一、task_struct中的进程状态

`task_struct`中的`__state`和`exit_state`字段描述了进程当前所属的状态。

```c
unsigned int 	__state;
int				exit_state;
```

`__state`字段是关于进程的可运行性的，`exit_state`字段是关于进程退出的。这些状态是互斥的，只能设置一组 ，其余的标志会被清除。

## 二、常见进程状态

### 2.1 运行状态

进程的运行状态设置于`__state`字段，进程可能的运行状态有：

- **`TASK_RUNNING`（可运行状态）**：进程处于就绪状态。注意不要被这里的`RUNNING`误导，处于这一状态的进程未必在处理器上运行，也可能是在运行队列上等待被调度。
- **`TASK_INTERRUPTIBLE`（可中断的等待状态）**：进程的一种等待状态。进程阻塞地等待某个条件为真。
- **`TASK_UNINTERRUPTIBLE`（不可中断的等待状态）**：进程的另一种等待状态。与`TASK_INTERRUPTIBLE`类似，但特别是信号传递不能改变它的状态。值得注意的是，`kill`本质上也是`SIGKILL`信号，所以如果等待的条件无法满足，进程将一直处于等待状态无法被关闭。因此只有在有特殊需求且有把握的情况下才会采用这一状态。
- **`TASK_STOPPED`（暂停状态）**：进程在收到`SIGSTOP`、`SIGTSTP`、`SIGTTIN`、`SIGTTOU`信号后进入暂停状态。
- **`TASK_TRACED`（跟踪状态）**：用于调试和监控。

### 2.2 `exit_state`字段

进程的退出状态设置于`exit_state`字段，进程可能的退出状态有：

- **`EXIT_ZOMBIE`（僵尸状态）**：进程结束时，由于可能存在父进程所需要的信息，不会直接删除所有信息，而是会先进入`EXIT_ZOMBIE`状态，等待父进程发出`wait()`类系统调用。
- **`EXIT_DEAD`（死亡状态）**：进程结束的最终状态。由于统一进程的其他线程也可能发出`wait()`类系统调用，产生竞争条件，因此将状态改为`EXIT_DEAD`。

### 2.3 进程状态的转移

![](..\imgs\Linux进程管理：task_struct之进程状态\状态转移.png)

## 三、状态的定义

进程的状态以**比特位掩码**的形式在`include/linux/sched.h`中定义：

```c
/* Used in tsk->__state: */
#define TASK_RUNNING					0x00000000
#define TASK_INTERRUPTIBLE				0x00000001
#define TASK_UNINTERRUPTIBLE			0x00000002
#define __TASK_STOPPED					0x00000004
#define __TASK_TRACED					0x00000008
/* Used in tsk->exit_state: */
#define EXIT_DEAD						0x00000010
#define EXIT_ZOMBIE						0x00000020
#define EXIT_TRACE						(EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->__state again: */
#define TASK_PARKED						0x00000040
#define TASK_DEAD						0x00000080
#define TASK_WAKEKILL					0x00000100
#define TASK_WAKING						0x00000200
#define TASK_NOLOAD						0x00000400
#define TASK_NEW						0x00000800
#define TASK_RTLOCK_WAIT				0x00001000
#define TASK_FREEZABLE					0x00002000
#define __TASK_FREEZABLE_UNSAFE			(0x00004000 * IS_ENABLED(CONFIG_LOCKDEP))
#define TASK_FROZEN						0x00008000
#define TASK_STATE_MAX					0x00010000
```

此外，源码中还提供了组合的状态定义：

```C
#define TASK_FREEZABLE_UNSAFE		(TASK_FREEZABLE | __TASK_FREEZABLE_UNSAFE)

/* Convenience macros for the sake of set_current_state: */
#define TASK_KILLABLE				(TASK_WAKEKILL | TASK_UNINTERRUPTIBLE)
#define TASK_STOPPED				(TASK_WAKEKILL | __TASK_STOPPED)
#define TASK_TRACED					__TASK_TRACED

#define TASK_IDLE					(TASK_UNINTERRUPTIBLE | TASK_NOLOAD)

/* Convenience macros for the sake of wake_up(): */
#define TASK_NORMAL					(TASK_INTERRUPTIBLE | TASK_UNINTERRUPTIBLE)
```

可以看到，除`2.1.1`和`2.1.2`所示的常见状态外，内核还包含了其他状态。

例如，处于`TASK_WAKEKILL`状态的进程无法被常规信号唤醒，只能通过发送`SIGKILL`信号才能将其唤醒或终止。`TASK_KILLABLE`是`TASK_WAKEKILL`和`TASK_UNINTERRUPTIBLE`的组合。这就在保证进程不会被常规信号唤醒的同时，弥补了`2.1.1`中提到的`TASK_INTERRUPTIBLE`不能通过`SIGKILL`终止的问题。

此外，`TASK_WAKING`表示进程正在从其他状态（如`TASK_KILLABLE`或`TASK_INTERRUPTIBLE`）被唤醒并准备进入`TASK_RUNNING`状态的过程，是不独立存在的中间状态。

其余的状态将在后续文章中结合具体专题进行介绍。

## 四、进程状态的设置

内核中使用`set_current_state`宏设置当前进程的`__state`，其内部使用内存屏障强制其他处理器重新排序。如果不需要序列化，可以直接使用`__set_current_state`宏。定义如下：

```C
#define __set_current_state(state_value)				\
	do {								\
		debug_normal_state_change((state_value));		\
		WRITE_ONCE(current->__state, (state_value));		\
	} while (0)

#define set_current_state(state_value)					\
	do {								\
		debug_normal_state_change((state_value));		\
		smp_store_mb(current->__state, (state_value));		\
	} while (0)
```

## 参考资料

[1] Bovet, D.P.，Cesati, M. 深入理解Linux内核[M]. 第三版. 陈莉君，张琼声，张宏伟译. 北京：中国电力出版社，2007.

[2] Love, R. Linux内核设计与实现[M]. 第三版. 陈莉君，康华译. 北京：机械工业出版社，2011. 
