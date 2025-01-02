---
title: System Call
parent: Linux Kernel
---

# System Call
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

This is based on Linux v4.20

## x86_64

System Call Table
```c
/* arch/x86/entry/syscall_64.c */
...
#define __SYSCALL_64(nr, sym, qual) [nr] = sym,

asmlinkage const sys_call_ptr_t sys_call_table[__NR_syscall_max+1] = {
	/*
	 * Smells like a compiler bug -- it doesn't work
	 * when the & below is removed.
	 */
	[0 ... __NR_syscall_max] = &sys_ni_syscall,
#include <asm/syscalls_64.h>
};
...
```

System Call Entry. These entries will be converted by `arch/x86/entry/syscalltbl.sh`
```
/* arch/x86/entry/syscalls/syscall_64.tbl */
...
0	common	read			__x64_sys_read
1	common	write			__x64_sys_write
...
```

System Call Function
* In case of read system call, `SYSCALL_DEFINE3` is used because it contains 3 arguments.
* `SYSCALL_METADATA` is used for only trace purpose.
* `asmlinkage` means arguments are passed through not registers but stacks.
* `__PROTECT` means the stacks for arguments cannot be reused.

```c
/* include/linux/syscalls.h */
...
#define SYSCALL_DEFINE3(name, ...) SYSCALL_DEFINEx(3, _##name, __VA_ARGS__)
...
#define SYSCALL_DEFINEx(x, sname, ...)				\
	SYSCALL_METADATA(sname, x, __VA_ARGS__)			\
	__SYSCALL_DEFINEx(x, sname, __VA_ARGS__)
...
#define __SYSCALL_DEFINEx(x, name, ...)					\
	__diag_push();							\
	__diag_ignore(GCC, 8, "-Wattribute-alias",			\
		      "Type aliasing is used to sanitize syscall arguments");\
	asmlinkage long sys##name(__MAP(x,__SC_DECL,__VA_ARGS__))	\
		__attribute__((alias(__stringify(__se_sys##name))));	\
	ALLOW_ERROR_INJECTION(sys##name, ERRNO);			\
	static inline long __do_sys##name(__MAP(x,__SC_DECL,__VA_ARGS__));\
	asmlinkage long __se_sys##name(__MAP(x,__SC_LONG,__VA_ARGS__));	\
	asmlinkage long __se_sys##name(__MAP(x,__SC_LONG,__VA_ARGS__))	\
	{								\
		long ret = __do_sys##name(__MAP(x,__SC_CAST,__VA_ARGS__));\
		__MAP(x,__SC_TEST,__VA_ARGS__);				\
		__PROTECT(x, ret,__MAP(x,__SC_ARGS,__VA_ARGS__));	\
		return ret;						\
	}								\
	__diag_pop();							\
	static inline long __do_sys##name(__MAP(x,__SC_DECL,__VA_ARGS__))
...
/* fs/read_write.c */
...
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
	return ksys_read(fd, buf, count);
}
...
```

Caller of System Call Entry
* Initialize vectors table
  * swapgs: change current stack to kernel's one
  * MSR_STAR: change code segment when entering or exiting system call

```asm
/* arch/x86/entry/entry_64.S */
...
ENTRY(entry_SYSCALL_64)
...
	swapgs
...
	call	do_syscall_64		/* returns with IRQs disabled */
...
```

```c
/* arch/x86/kernel/cpu/common.c */
...
void syscall_init(void)
{
	wrmsr(MSR_STAR, 0, (__USER32_CS << 16) | __KERNEL_CS);
	wrmsrl(MSR_LSTAR, (unsigned long)entry_SYSCALL_64);
...
```

* Call Path

```c
/* arch/x86/entry/common.c */
...
__visible void do_syscall_64(unsigned long nr, struct pt_regs *regs)
...
	nr &= __SYSCALL_MASK;
	if (likely(nr < NR_syscalls)) {
		nr = array_index_nospec(nr, NR_syscalls);
		regs->ax = sys_call_table[nr](regs);
	}
...
```

* From Userspace (glibc)

```c
/* glibc/sysdeps/unix/sysv/linux/x86_64/syscall.S */
...
ENTRY (syscall)
	movq %rdi, %rax		/* Syscall number -> rax.  */
	movq %rsi, %rdi		/* shift arg1 - arg5.  */
	movq %rdx, %rsi
	movq %rcx, %rdx
	movq %r8, %r10
	movq %r9, %r8
	movq 8(%rsp),%r9	/* arg6 is on the stack.  */
	syscall			/* Do the system call.  */
	cmpq $-4095, %rax	/* Check %rax for error.  */
	jae SYSCALL_ERROR_LABEL	/* Jump to error handler if error.  */
	ret			/* Return to caller.  */
PSEUDO_END (syscall)
...
```

## arm64

System Call Table
```c
/* arch/arm64/kernel/sys.c */
...
#define __SYSCALL(nr, sym)    [nr] = sym,

void * const sys_call_table[__NR_syscalls] __aligned(4096) = {
	[0 ... __NR_syscalls - 1] = sys_ni_syscall,
#include <asm/unistd.h>
};
...
```

System Call Entry
```c
/* include/uapi/asm-generic/unistd.h */
...
#define __NR_read 63
__SYSCALL(__NR_read, sys_read)
...
```

System Call Function - same with x86_64

Caller of System Call Entry

* Initialize vectors table

```asm
/* arch/arm64/kernel/entry.S */
...
ENTRY(vectors)
...
	kernel_ventry	0, sync				// Synchronous 64-bit EL0
...
/* arch/arm64/kernel/head.S */
...
__primary_switched:
...
	adr_l	x8, vectors			// load VBAR_EL1 with virtual
	msr	vbar_el1, x8			// vector table address
...
```

* Call Path

```asm
/* arch/arm64/kernel/entry.S */
...
el0_sync:
	kernel_entry 0
	mrs	x25, esr_el1			// read the syndrome register
	lsr	x24, x25, #ESR_ELx_EC_SHIFT	// exception class
	cmp	x24, #ESR_ELx_EC_SVC64		// SVC in 64-bit state
	b.eq	el0_svc
...
el0_svc:
	mov	x0, sp
	bl	el0_svc_handler
	b	ret_to_user
ENDPROC(el0_svc)
...
```

```c
/* arch/arm64/kernel/syscall.c */
...
asmlinkage void el0_svc_handler(struct pt_regs *regs)
{
	sve_user_discard();
	el0_svc_common(regs, regs->regs[8], __NR_syscalls, sys_call_table);
}
...
```

* From Userspace (bionic)

```asm
/* libc/arch-arm64/bionic/syscall.S */
...
	ENTRY(syscall)
	/* Move syscall No. from x0 to x8 */
	mov	 x8, x0
	/* Move syscall parameters from x1 thru x6 to x0 thru x5 */
	mov	 x0, x1
	mov	 x1, x2
	mov	 x2, x3
	mov	 x3, x4
	mov	 x4, x5
	mov	 x5, x6
	svc	 #0
...
```

## References

* [Linux v4.20](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tag/?h=v4.20)
* [Anatomy of a system call, part 1 @ LWN](https://lwn.net/Articles/604287)
* [Anatomy of a system call, additional content @ LWN](https://lwn.net/Articles/604406)
* [Anatomy of a system call, part 2 @ LWN](https://lwn.net/Articles/604515)
* [Anatomy of Linux system call in ARM64](http://www.eastrivervillage.com/Anatomy-of-Linux-system-call-in-ARM64/)
* [Linux-insides](https://0xax.gitbooks.io/linux-insides/)
* [Glibc and System Calls](https://sys.readhtedocs.io/en/latest/doc/07_calling_system_calls.html)
* [bionic](https://android.googlesource.com/platform/bionic/)
