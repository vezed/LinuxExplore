# Linux进程管理：task_struct的分配与存放

## 一、进程描述符的分配与存放

`thread_info`（线程描述符）

`thread_info`的定义与架构有关，一般在`arch/xxx/include/asm/thread_info.h`。其中`xxx`表示架构名称，常见的有`arm`、`arm64`、`x86`、`riscv`等。下面展示`arch/mips/include/asm/thread_info.h`

有两种存储方式，一种是直接将`task_struct`存在寄存器，另一种是放在`thread_info`中

```C
struct thread_info {
	struct task_struct	*task;		/* main task structure */
	unsigned long		flags;		/* low level flags */
	unsigned long		tp_value;	/* thread pointer */
	__u32			cpu;		/* current CPU */
	int			preempt_count;	/* 0 => preemptible, <0 => BUG */
	struct pt_regs		*regs;
	unsigned long		syscall;	/* syscall number */
	unsigned long		syscall_work;	/* SYSCALL_WORK_ flags */
};
```

`arch/mips/includeasm/current.h`定义`current`

```C

```

`arch/arm64/include/asm/thread_info.h`

```C
struct thread_info {
	unsigned long		flags;		/* low level flags */
#ifdef CONFIG_ARM64_SW_TTBR0_PAN
	u64			ttbr0;		/* saved TTBR0_EL1 */
#endif
	union {
		u64		preempt_count;	/* 0 => preemptible, <0 => bug */
		struct {
#ifdef CONFIG_CPU_BIG_ENDIAN
			u32	need_resched;
			u32	count;
#else
			u32	count;
			u32	need_resched;
#endif
		} preempt;
	};
#ifdef CONFIG_SHADOW_CALL_STACK
	void			*scs_base;
	void			*scs_sp;
#endif
	u32			cpu;
};
```

`arch/arm64/include/asm/current.h`定义`current`

```C
static __always_inline struct task_struct *get_current(void)
{
	unsigned long sp_el0;

	asm ("mrs %0, sp_el0" : "=r" (sp_el0));

	return (struct task_struct *)sp_el0;
}

#define current get_current()
```

## 二、内核如何找到`task_struct`

`include/linux/thread_info.h`

```C
#include <asm/current.h>
#define current_thread_info() ((struct thread_info *)current)
```

`include/asm-generic/current.h`

```C
#include <linux/thread_info.h>

#define get_current() (current_thread_info()->task)
#define current get_current()
#endif
```



## 参考资料

[1] Bovet, D.P.，Cesati, M. 深入理解Linux内核[M]. 第三版. 陈莉君，张琼声，张宏伟译. 北京：中国电力出版社，2007.

[2] Love, R. Linux内核设计与实现[M]. 第三版. 陈莉君，康华译. 北京：机械工业出版社，2011. 
