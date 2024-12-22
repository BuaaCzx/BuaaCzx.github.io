---
layout: post
title: BUAA-OS-lab4challenge
categories: [操作系统]
tags: [操作系统, OS实验]
slug: os-challenge-lab4
---

Challenge Sigaction 实现文档

## 代码改动与说明

1. `include/env.h`

   在这里添加了新结构体定义、宏定义，并给 Env 进程块结构体添加新的必要信息。注意为了实现队列，在 sigset_t 结构体中添加了 `TAILQ_ENTRY` 成员变量。在进程块中添加的信息有：注册新号的信息、信号队列、掩码栈、信号处理函数入口、当前正在处理的信号。

   ```c
   typedef struct sigset_t {
       uint32_t sig;
       TAILQ_ENTRY(sigset_t) sig_link;
   } sigset_t;
   
   struct sigaction {
       void     (*sa_handler)(int);
       sigset_t   sa_mask;
   };
   
   TAILQ_HEAD(Sig_list, sigset_t);
   
   #define SIGINT 2
   #define SIGILL 4
   #define SIGKILL 9
   #define SIGSEGV 11
   #define SIGCHLD 17
   #define SIGSYS 31
   
   #define SIG_BLOCK 1
   #define SIG_UNBLOCK 2
   #define SIG_SETMASK 3
   
   struct Env {
   	// challenge
   	struct sigaction env_sigactions[35];
   	struct Sig_list env_sig_list; // 接收到的信号队列
   	struct sigset_t env_mask_list[55]; // 掩码栈
   	int env_mask_cnt;
   	u_int env_sig_entry; // sig handler
   	int env_cur_sig;
   };
   ```

2. `kern/env.c`

   这里修改的是 env_alloc 函数，对 Env 结构体中新增的成员变量进行初始化。

   ```c
   int env_alloc(struct Env **new, u_int parent_id) {
   	// ...
   	// challenge
   	for(int i=1;i<=32;i++){
   		e->env_sigactions[i].sa_handler = NULL;
   		e->env_sigactions[i].sa_mask.sig = 1 << (i - 1);
   	}
   	TAILQ_INIT(&e->env_sig_list);
   	e->env_mask_cnt = 0;
   	e->env_sig_entry = 0;
   	e->env_cur_sig = 0;
   	e->env_mask_list[0].sig = 0;
   	// ...
   }
   ```

   

3. `kern/syscall_all.c`

   在这里添加了若干与信号机制有关的系统调用函数，并在销毁进程的时候向父进程发送信号，在创建子进程的时候也对进程块中新增变量进行继承。

   ```c
   int sys_env_destroy(u_int envid) {
   	// challenge
   	if(e->env_parent_id){
   		sys_kill(e->env_parent_id, SIGCHLD);
   	}
   	// end_challenge
   }
   
   int sys_exofork(void) {
       e->env_status = ENV_NOT_RUNNABLE;
   	e->env_pri = curenv->env_pri;
   
   	for(int i = 1;i <= 32; i++){
   		e->env_sigactions[i] = curenv->env_sigactions[i];
   	}
   	e->env_cur_sig = curenv->env_cur_sig;
   	e->env_mask_cnt = curenv->env_mask_cnt;
   	e->env_sig_entry = curenv->env_sig_entry;
   	for(int i = 0; i <= e->env_mask_cnt; i++) {
   		e->env_mask_list[i] = curenv->env_mask_list[i];
   	}
   	return e->env_id;
   }
   
   ```

   下面逐一对新增实现的系统调用进行介绍：

   - 信号注册 sys_sigaction，照着题意模拟一遍就好了，注意如果是 SIGKILL，应该遵照题目要求，不修改处理函数。

     ```c
     int sys_sigaction(u_int envid, int signum, const struct sigaction *newact, struct sigaction *oldact) { // 信号注册系统调用
     	struct Env *e;
     	try(envid2env(envid, &e, 0));
     	if (oldact) {
     		*oldact = e->env_sigactions[signum];
     	}
     	if (newact) {
     		if (signum == SIGKILL) { // 不修改处理函数
     			e->env_sigactions[signum].sa_mask = newact->sa_mask;
     		} else {
     			e->env_sigactions[signum] = *newact;
     		}
     	}
     	return 0;
     }
     ```

   - 信号发送 sys_kill，本质上是 new 一个信号对象，插入到进程控制块的信号列表里。new 的操作用一个 sigset_t 的数组来实现分配，实际上相当于一个空闲 sigseg_t 的列表。

     ```c
     static sigset_t sigs[4005];
     static int sigs_cnt = 0;
     
     int sys_kill(u_int envid, int sig) {
     	struct Env *e;
     	try(envid2env(envid, &e, 0));
     	sigs[sigs_cnt].sig = sig;
     	TAILQ_INSERT_TAIL(&e->env_sig_list, sigs + sigs_cnt, sig_link);
     	sigs_cnt = (sigs_cnt + 1) % 4000;
     	return 0;
     }
     ```

   - 掩码查询与设置 sys_proc_mask，对照题意模拟即可，注意 __set 和 __oset 两个参数为 NULL 时的处理。

     ```c
     int sys_proc_mask(int __how, const sigset_t * __set, sigset_t * __oset) {
     	struct Env *e;
     	try(envid2env(0, &e, 0));
     	sigset_t *s = e->env_mask_list + e->env_mask_cnt;
         if (__oset) {
     		*__oset = *s; // 取出栈顶元素
     	}
     
     	if (__set) {
     		if (__how == SIG_BLOCK) {
     			s->sig |= __set->sig;
     		} else if (__how == SIG_UNBLOCK) {
     			s->sig &= (~__set->sig);
     		} else if (__how == SIG_SETMASK) {
     			s->sig = __set->sig;
     		} else {
     			return -1;
     		}
     	}
     
     	return 0;
     }
     ```

   - 信号队列查询 sys_get_pending，遍历信号队列即可。

     ```c
     int sys_get_pending(sigset_t *__set) {
     	struct Env *e;
     	try(envid2env(0, &e, 0));
     	__set->sig = 0;
     	sigset_t *i;
     	TAILQ_FOREACH(i, &e->env_sig_list, sig_link) {
     		__set->sig |= 1 << (i->sig - 1);
     	}
     	return 0;
     }
     ```

   - 信号异常处理入口设置 sys_set_sig_entry。

     ```c
     int sys_set_sig_entry(u_int envid, u_int func) {
     	struct Env *e;
     	try(envid2env(envid, &e, 0));
     	e->env_sig_entry = func;
     	return 0;
     }
     ```

   - 信号处理恢复现场的函数 sys_set_sig_trapframe，仿照 sys_set_tf 函数实现即可，注意需要额外处理掩码栈，并切换进程当前处理的信号为 0。

     ```c
     int sys_set_sig_trapframe(u_int envid, struct Trapframe *tf){
     	if(is_illegal_va_range((u_long)tf, sizeof *tf)){
     		return -E_INVAL;
     	}
     	struct Env *e;
     	try(envid2env(envid, &e, 0));
     	e->env_mask_cnt--;
     	e->env_cur_sig = 0;
     	if(e == curenv){
     		*((struct Trapframe *)KSTACKTOP -1) = *tf;
     		return tf->regs[2];
     	} else {
     		e->env_tf = *tf;
     		return 0;
     	}
     }
     ```

4. `user/lib/fork.c`

   在这里主要是增加了信号的处理入口 sig_entry，首先判断是否设置了专门的处理函数，如果有，就直接调用，否则进行默认处理。注意处理后都需要调用 sys_set_sig_trapframe 来恢复现场。

   ```c
   void __attribute__((noreturn)) sig_entry(struct Trapframe *tf, void (*sa_handler)(int), int signum, int envid) {
       if (sa_handler != 0) {
           sa_handler(signum); // 直接调用定义好的处理函数
           int r = syscall_set_sig_trapframe(0, tf);
           user_panic("sig_entry syscall_set_trapframe returned %d", r);
       }
       switch (signum) {
           case SIGINT: case SIGILL: case SIGKILL: case SIGSEGV:
               syscall_env_destroy(envid); //默认处理
               user_panic("sig_entry syscall_env_destroy returned");
           default:;
               int r = syscall_set_sig_trapframe(0, tf);
               user_panic("sig_entry syscall_set_trapframe returned %d", r);
       }
   }
   ```

   此外，在 fork 函数里新增了子进程的 sig_entry 设置。

5. `user/lib/ipc.c`

   在这里实现了对信号集的一些处理函数，以及 sigaction 注册和 kill 发送，调用已经写好的系统调用即可。

   ```c
   int is_illegal_sig(int signum) {
   	return signum < 1 || signum > 32;
   }
   
   int sigaction(int signum, const struct sigaction *newact, struct sigaction *oldact){
   	if (is_illegal_sig(signum)) {
   		return -1;
   	}
   	try(syscall_set_sig_entry(0, (u_int) sig_entry));
   	return syscall_sigaction(0, signum, newact, oldact) < 0 ? -1 : 0;
   }
   
   int kill(u_int envid, int sig){
   	if (is_illegal_sig(sig)) {
   		return -1;
   	}
   	return syscall_kill(envid, sig) < 0 ? -1 : 0;
   }
   
   int sigemptyset(sigset_t *__set) {
   	if (__set) {
   		__set->sig = 0;
   	} else {
   		return -1;
   	}
   	return 0;
   }
   // 清空参数中的__set掩码，初始化信号集以排除所有信号。这意味着__set将不包含任何信号。(清0)
   
   int sigfillset(sigset_t *__set) {
   	if (__set) {
   		__set->sig = ~0;
   	} else {
   		return -1;
   	}
   	return 0;
   }
   // 将参数中的__set掩码填满，使其包含所有已定义的信号。这意味着__set将包括所有信号。(全为1)
   
   int sigaddset(sigset_t *__set, int __signo) {
   	if (__set && !is_illegal_sig(__signo)) {
   		__set->sig |= 1 << (__signo - 1);
   	} else {
   		return -1;
   	}
   	return 0;
   }
   // 向__set信号集中添加一个信号__signo。如果操作成功，__set将包含该信号。(置位为1)
   
   int sigdelset(sigset_t *__set, int __signo) {
   	if (__set && !is_illegal_sig(__signo)) {
   		__set->sig &= ~(1 << (__signo - 1));
   	} else {
   		return -1;
   	}
   	return 0;
   }
   // 从__set信号集中删除一个信号__signo。如果操作成功，__set将不再包含该信号。(置位为0)
   
   int sigismember(const sigset_t *__set, int __signo) {
   	if (!__set || is_illegal_sig(__signo)) {
   		return -1;
   	}
   	return (__set->sig >> (__signo - 1)) & 1;
   }
   // 检查信号__signo是否是__set信号集的成员。如果是，返回1；如果不是，返回0。
   
   int sigisemptyset(const sigset_t *__set) {
   	if (!__set) {
   		return -1;
   	}
   	return __set->sig == 0;
   }
   // 检查信号集__set是否为空。如果为空，返回1；如果不为空，返回0。
   
   int sigandset(sigset_t *__set, const sigset_t *__left, const sigset_t *__right) {
   	if (!__set || !__left || !__right) {
   		return -1;
   	}
   	__set->sig = __left->sig & __right->sig;
   	return 0;
   }
   // 计算两个信号集__left和__right的交集，并将结果存储在__set中。
   
   int sigorset(sigset_t *__set, const sigset_t *__left, const sigset_t *__right) {
   	if (!__set || !__left || !__right) {
   		return -1;
   	}
   	__set->sig = __left->sig | __right->sig;
   	return 0;
   }
   // 计算两个信号集__left和__right的并集，并将结果存储在__set中。
   
   int sigprocmask(int __how, const sigset_t * __set, sigset_t * __oset) {
   	return syscall_proc_mask(__how, __set, __oset) < 0 ? -1 : 0;
   }
   // 根据__how的值更改当前进程的信号屏蔽字。__set是要应用的新掩码，__oset（如果非NULL）则保存旧的信号屏蔽字。
   // __how可以是SIG_BLOCK（添加__set到当前掩码）、SIG_UNBLOCK（从当前掩码中移除__set）、或SIG_SETMASK（设置当前掩码为__set）。
   
   int sigpending(sigset_t *__set) {
   	if (!__set) {
   		return -1;
   	}
   	return syscall_get_pending(__set) < 0 ? -1 : 0;
   }
   // 获取当前被阻塞且未处理的信号集，并将其存储在__set中。
   ```

6. `user/lib/libos.c`

   在进入 main 函数之前，就设置好 sig_entry，防止在信号注册之前就进行了信号的处理。

   ```c
   void libmain(int argc, char **argv) {
   	// set env to point at our env structure in envs[].
   	env = &envs[ENVX(syscall_getenvid())];
   	syscall_set_sig_entry(syscall_getenvid(), sig_entry); // add for challenge
   	// call user main routine
   	main(argc, argv);
   	// exit gracefully
   	exit();
   }
   ```

7. 至此，信号系统的基础处理函数已经全部实现完毕，只剩下如何把整个信号的处理过程“打通”，也即实现信号处理的跳转。
   - `kern/tlbex.c`
      实现 do_signal 函数，先遍历信号列表找到将要处理的信号（考虑信号屏蔽、信号优先级、SIGKILL 最优先执行），然后设置掩码和 tf。

      ```c
      void do_signal(struct Trapframe *tf) {
      
   	if (curenv->env_cur_sig == SIGKILL) {
   		return;
   	}
      
   	struct sigset_t *sig_todo = NULL; // 将要处理的信号
      
   	struct sigset_t *ss;
   	TAILQ_FOREACH(ss, &curenv->env_sig_list, sig_link) {
   		if(ss->sig == SIGKILL) { // 如果有 SIGKILL，优先处理
   			sig_todo = ss;
   			break;
   		}
   		u_int cur_mask = curenv->env_mask_list[curenv->env_mask_cnt].sig;
   		if(!((cur_mask >> (ss->sig - 1)) & 1)) { // 如果信号未被屏蔽
   			if(sig_todo == NULL) { // 如果当前还没有需要处理的信号
   				sig_todo = ss;
   			} else { // 已经有了，对比优先级
   				if(ss->sig < sig_todo->sig){
   					sig_todo = ss;
   				}
   			}
   		}
   	}
      
   	if (!sig_todo) {
   		return;
   	} else {
   		// printk("catch signal %d\n", sig_todo->sig);
   	}
      
   	TAILQ_REMOVE(&curenv->env_sig_list, sig_todo, sig_link);
      
   	// 把新掩码 push 进掩码栈, 上一个掩码，该信号掩码及该信号本身
   	u_int mask = curenv->env_mask_list[curenv->env_mask_cnt].sig | curenv->env_sigactions[sig_todo->sig].sa_mask.sig | (1 << (sig_todo->sig - 1));
   	curenv->env_mask_cnt++;
   	curenv->env_mask_list[curenv->env_mask_cnt].sig = mask;
      
   	curenv->env_cur_sig = sig_todo->sig;
      
   	struct Trapframe tmp_tf = *tf;
      
   	if (tf->regs[29] < USTACKTOP || tf->regs[29] >= UXSTACKTOP) {
   		tf->regs[29] = UXSTACKTOP;
   	}
   	tf->regs[29] -= sizeof(struct Trapframe);
   	*(struct Trapframe *)tf->regs[29] = tmp_tf;
   	
   	tf->regs[4] = tf->regs[29];
   	tf->regs[5] = (unsigned int) (curenv->env_sigactions[sig_todo->sig].sa_handler);
   	tf->regs[6] = sig_todo->sig;
   	tf->regs[7] = curenv->env_id;
   	tf->regs[29] -= 4 * sizeof(tf->regs[4]);	
   	tf->cp0_epc = curenv->env_sig_entry; // 跳转到异常处理入口
      }
      ```

   - `kern/genex.S`

      异常处理跳转到 do_signal 函数。
   
   	```mips
   	FEXPORT(ret_from_exception)
   	move	a0, sp
   	addiu	sp, sp, -8
   	jal		do_signal
   	addiu	sp, sp, 8
      RESTORE_ALL
      eret
      ```
      
      