1. 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b_bad_*.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

		[rustsbi] RustSBl version 0.2.2, adapting to RISC-V SBI V1.0.0
	- ch2b bad address.rs
		- [ERROR] [kernel] PageFault in application, core dumped.
		- 访问错误地址
	- ch2b bad instructions.rs
		- [ERROR] [kernel] lllegallnstruction in application, core dumped.
		- 执行非法指令
	- ch2b bad registerrs
		- [ERROR] [kernel] lllegallnstruction in application, core dumped.
		- 执行非法指令


2. 深入理解 trap.S 中两个函数 _alltraps和 _restore的作用，并回答如下问题:

 

	1. L40：刚进入_restore 时，a0 代表了什么值。请指出 _restore 的两种使用情景。

 


	 	a0在刚进入__restore时代表的是trap_handler的返回值，即要切换到的用户线程的TrapContext的地址。
	
		_restore有两种主要的使用情景：
	
		A.陷阱处理完成，返回到原来的线程：当一个线程在执行过程中发生陷阱，处理器会跳转到陷阱处	理程序alltraps，然后调用陷阱处理函数trap_handler。如果trap_handler决定陷阱已经被处理完毕，可以返回到原来的线程继续执行，那么它会调用__restore函数。在这种情况下，restore函数会恢复线程的上下文（包括寄存器的值和程序状态），然后返回到陷阱发生前的程序位置。
	
		B.陷阱处理过程中，切换到另一个线程：在处理陷阱的过程中，trap_handler可能决定需要切换到	另一个线程。例如，当前线程的时间片已经用完，或者当前线程因为等待I/O操作而被阻塞。在这	种情况下，trap_handler会设置a0寄存器为新线程的上下文地址，然后调用_restore函数。_restore	函数会加载新线程的上下文，然后跳转到新线程的程序位置。
	
		总的来说，_restore的作用是根据a0寄存器的值恢复程序的状态，然后返回到陷阱发生前的程序位	置，继续执行程序。这是实现线程切换和陷阱处理的关键步骤。

 


	2. L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。
	
		```
		ld t0, 32*8(sp)
	
		ld t1, 33*8(sp)
	
		ld t2, 2*8(sp)
	
		csrw sstatus, t0
	
		csrw sepc, t1
	
		csrw sscratch, t2
		```
	
		三个寄存器：sstatus，sepc和sscratch。意义：
	
		sstatus：保存了程序的状态信息。在这里，sstatus被从内核栈中加载到t0，然后写入sstatus寄存器。这样做的目的是恢复陷阱发生前的程序状态。在RISC-V中，sstatus寄存器包含了一些重要的状态位，如全局中断使能位、当前的特权级别等。恢复sstatus寄存器可以确保程序在用户态下正确执行。
	
		sepc：保存了陷阱发生时的程序计数器。在这里，sepc被从内核栈中加载到t1，然后写入sepc寄存器。这样做的目的是设置返回地址。在RISC-V中，当陷阱发生时，处理器会自动将程序计数器的值保存到sepc寄存器。恢复sepc寄存器可以确保程序在处理完陷阱后，返回到正确的地址继续执行。
	
		sscratch：一个特殊的寄存器，用于保存用户栈的地址。在这里，sscratch被从内核栈中加载到t2，然后写入sscratch寄存器。这样做的目的是恢复用户栈的地址。在RISC-V中，sscratch寄存器通常被用作系统调用或陷阱处理的临时存储。在这个场景中，sscratch寄存器被用来保存用户栈的地址，以便在返回用户态时，可以正确地恢复用户栈。

 


	3. L50-L56：为何跳过了 x2 和 x4？
	
		```
		ld x1, 1*8(sp)
	
		ld x3, 3*8(sp)
	
		.set n, 5
	
		.rept 27
	
		LOAD_GP %n
	
		.set n, n+1
	
		.endr
		```
	
		跳过x2和x4的加载是因为这两个寄存器在这个上下文中有特殊的用途，不能简单地通过加载内核栈中的值来恢复。具体来说：
	
		x2：在RISC-V架构中，x2寄存器是栈指针（sp），用于指向当前的栈顶。在这段代码中，x2被跳过是因为在陷阱处理过程中，我们使用了内核栈而不是用户栈。因此，我们不能简单地从内核栈中恢复x2的值，而是需要在所有其他寄存器恢复后，通过特殊的方式恢复x2的值。
	
		x4：在RISC-V架构中，x4寄存器是线程指针（tp），用于支持线程局部存储。在这段代码中，x4被跳过是因为应用程序不使用它。在某些系统中，x4寄存器可能被操作系统或者运行时系统用于存储线程特定的数据，但在这个上下文中，我们假设应用程序不使用x4寄存器，因此没有必要保存和恢复它的值。

 


	4. L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？
	
		```
		csrrw sp, sscratch, sp
		```
	
		sp（栈指针）：执行这条指令后，sp 被设置为 sscratch 寄存器中的值。在这个上下文中,sscratch 寄存器保存了之前（在异常或中断发生时，即在 _alltraps 函数开始时）用户模式下的栈指针。因此，执行这条指令后，sp 将指向用户栈，这意味着控制权即将从内核模式返回到用户模式。
	
		sscratch：与此同时，sp 寄存器的原始值（即在执行这条指令之前的值，也就是内核栈的指针）被写入 sscratch。此时，sscratch 寄存器保存的是内核栈的指针。
	
		这条指令实际上是在完成异常或中断处理后，从内核模式切换回用户模式的关键步骤。在这个过程中，它将栈指针从内核栈切换回用户栈，并且将内核栈的指针暂存到 sscratch 寄存器中。这是操作系统恢复用户程序执行前的必要步骤，确保了用户程序可以在正确的上下文中继续执行。

 


	5. _restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？
	
		①_restore函数中的状态切换发生在最后一条指令sret，它是RISC-V架构中的一条特殊指令，用于从特权级别更高的模式（如内核态）返回到特权级别更低的模式（如用户态）。
	
		②原因是，当执行sret指令时，处理器会做以下几件事情：
	
		将sepc寄存器的值加载到程序计数器（PC），这样程序就会从sepc指定地址继续执行。在_restore函数中，sepc寄存器的值已经被恢复为陷阱发生前的值，因此程序会返回到陷阱发生前的位置。
	
		将sstatus寄存器中的SIE（Supervisor Interrupt Enable）位加载到全局中断使能位（IE），这样就恢复了陷阱发生前的中断使能状态。
	
		将sstatus寄存器中的SPP（Supervisor Previous Privilege）位加载到当前特权级别，这样就恢复了陷阱发生前的特权级别。在_restore函数中，sstatus寄存器的值已经被恢复为陷阱发生前的值，因此特权级别会被设置为用户态。
	
		因此，执行sret指令后，程序会返回到用户态，并从陷阱发生前的位置继续执行。这就完成了从内核态到用户态的状态切换。

 


	6. L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？
	
		```
		csrrw sp, sscratch, sp
		```
	
		sp（栈指针）： sscratch 的原始值，即用户态的栈指针。这时，sp 被更新为指向内核栈。
	
		sscratch：原始的 sp 值，即内核模式被激活之前的用户栈指针。



	7. 从 U 态进入 S 态是哪一条指令发生的？
	
		在user/src/syscall.rs中，
	
		```
		pub fn syscall(id: usize, args: [usize; 3]) -> isize{
			let mut ret: isize;
			unsafe{
				core::arch::asm!(
					"ecall",
					inlateout("x10")args[o]=>ret,
					in("x11")args[1],
					in("x12")args[2],
					in("x17")id
			} ;
			ret
		}
		```
	
		之中的ecall进入S态。



