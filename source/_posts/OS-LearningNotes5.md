---
title: 思考题
date: 2022-11-16 19:54:20
tags: 
  - 操作系统
  - Linux Kernel
categories: [国科大课程笔记,操作系统高级教程]
description: 国科大杨力祥《操作系统高级教程》思考题整理
cover: https://s3.bmp.ovh/imgs/2022/08/30/57e6003529c041b7.webp
---
# 思考题
## 为什么开始启动计算机的时候，执行的是BIOS代码而不是操作系统自身的代码？（P1，3）
加电的一瞬间，计算机内存中，准确的说是RAM中，还未初始化，没有任何程序。软盘里虽然有操作系统程序，但CPU的逻辑电路被设计为只能运行内存中的程序，没有能力直接从软盘运行操作系统。这就需要硬件主动加载0xffff0处的BIOS程序，由BIOS准备好中断向量表、中断服务程序，接着通过中断“int 0x19”将引导程序bootsect加载至内存，以及后续的一系列操作，最终操作系统自身代码才能位于内存中，被CPU执行。

在加电后，BIOS 需要完成一些检测工作，设置实模式下的中断向量表和服务程序，并将操作系统的引导扇区加载至 0x7C00 处，然后将跳转至 0x7C00运行操作系统自身的代码。

## 为什么BIOS只加载了一个扇区，后续扇区却是由bootsect代码加载？为什么BIOS没有直接把所有需要加载的扇区都加载？（P6，7）
BIOS和操作系统通常由不同的专业团队开发的，为了能协调工作，需要按固定的规则约定，进行灵活的各自设计相应的部分。
因此，现行的方法是定位识别：对BIOS而言，“约定”接到启动操作系统命令，“定位识别”只从启动扇区把代码加载至0x7c00(BOOTSEG)位置，至于该扇区内容是什么，一概不管。而后续扇区则由操作系统自己的bootsect代码加载，这些代码由编写系统的用户负责，与之前BIOS也无关。
这样构建的好处是站在整个体系的高度，统一设计安排，简单而有效。且BIOS和操作系统的开发都可遵循这一约定，按照自己的意愿进行各自的设计，包括内存的规划等都更为灵活。

如果BIOS一次性加载，会存在：
1）对于不同的操作系统，其代码长度不一样，可能导致操作系统加载不完全；
2）使用BIOS进行加载，且加载完成之后再执行，需要很长的时间，因此Linux采用的是边执行边加载的方法。

## 为什么BIOS把bootsect加载到0x07c00，而不是0x00000？加载后又马上挪到0x90000处，是何道理？为什么不一次加载到位？(P4、P5-17)
1）0x07c00是历史约定。
2）BIOS在从0x00000开始的1KB字节构建了中断向量表，接着的256KB字节内存空间构建了BIOS数据区，这些数据还有用处，所以不能把bootsect加载到0x00000。 0X07c00是BIOS设置的内存地址，不是bootsect能够决定的。
3）挪到0x90000处是操作系统内存规划行为，主要为了避免在内核system占据0x000000处时可能将0x07c00(bootsect)覆盖，造成在main中设置根设备时取不到正确数据。将该扇区挪到0x90000，在setup.s中，获取一些硬件数据保存在0x90000~0x901ff处，可以对一些后面内核将要利用的数据，集中保存和管理。

## bootsect、setup、head程序之间是怎么衔接的？给出代码证据。（P16、P24+P25+P26）
1）bootsect跳转至setup程序：jmpi 0, SETUPSEG;
解释：通过BIOS的“int 0x13”中断，找到bootsect自身的中断服务程序，将setup加载至SETUPSEG(0x90200)处。同样手法，将system加载至SYSSEG(0x10000)处。bootsect程序任务都已经完成。然后，通过“jmpi 0, SETUPSEG”跳转至setup程序的加载位置，此时CS:IP指向setup程序的第一条指令，意味着现在由setup程序接着bootsect程序继续执行。此时还是实模式。
2）setup跳转至head程序：jmpi 0, 8
解释：setup通过BIOS提供的中断服务程序提取了系统数据，存储在原来的bootsect位置只保留最后2字节未被覆盖（0x901fc，根设备号）。接着，将IF至0，完成关中断操作。然后，将system移动到0x00000位置，此时head已经占据了0x00000处，同时BIOS中断向量表彻底被覆盖。为此，setup开始为保护模式做准备，设置GDT、IDT并用CPU中专用寄存器IDTR、GDTR看住。接着，打开A20，也就是32位寻址模式，再对可编程中断控制器8259A进行重新编程，并置PE位为1，即设定处理器工作方式为保护模式，以后根据GDT决定执行哪里的程序。最后，通过“jmpi 0,8”跳转到head。“0”表示段内偏移，“8(1000)”是保护模式下的段选择符，最后两位“00”表示内核态，第二位“0”表示GDT，第一位“1”表示GDT表中GDT[1]项（内核代码段），从该项中得知段基址为0x00000000。结合上述偏移0，可知最终跳转至0x0000000处，执行head程序。

## setup程序的最后是jmpi 0,8 ，为什么这个8不能简单的当作阿拉伯数字8看待，究竟有什么内涵？(P25)
此时为32位保护模式，0表示段内偏移，8表示段选择符，转化为二进制：1000。
最后两位00表示内核特权级，第三位0表示GDT表，第四位1表示所选的表（在此就是GDT表）的1项来确定代码段的段基址和段限长等信息。
这样可以得到代码是从段基址0x00000000、偏移为0处开始执行的，即head的开始位置。

> 补充：
> - 最后两位表示特权，其中00是内核，11是用户。
> - 倒数第3位如果是0表示GDT，1则表示LDT。
> - 前两位表示这个表的第几项。GDT表的0表示空，1表示内核代码段，2表示内核数据段。LDT表的0表示空，1表示用户代码段，2表示用户数据段。

## 保护模式在“保护”什么？它的“保护”体现在哪里？特权级的目的和意义是什么？分页有“保护”作用吗？(P436-P439、P443)
1）打开了保护模式后，CPU的寻址模式发生了变化，需要依赖于GDT去获取代码或数据段的基址。从GDT可以看出，保护模式除了段基址外，还有段限长，这样相当于增加了一个段位寄存器。既有效地防止了对代码或数据段的覆盖，又防止了代码段自身的访问超限，明显增强了保护作用。
2）体现：①在GDT、LDT及IDT中，均有自己界限特、权级等属性，这是对描述符所描述的对象的保护；②在不同特权级间访问时，系统会对CPL、RPL、DPL、IOPL 等进行检验，对不同层级的程序进行保护，同还限制某些特殊指令的使用，如 lgdt, lidt,cli等。
3）特权级的目的和意义：①为了更好的管理资源并保护系统不受侵害，操作系统利用先机，以时间换取特权，先霸占所有特权；②依托CPU提供的保护模式，着眼于“段”，在所有的段选择符最后两位标示特权级，禁止用户执行cli、sti等对掌控局面至关重要的指令。③操作系统可以把内核设计成最高特权级，把用户进程设计成最低特权级。这样，操作系统可以访问 GDT、LDT、TR，而 GDT、LDT是逻辑地址形成线性地址的关键，因此操作系统可以掌控线性地址。物理地址是由内核将线性地址转换而成的，所以操作系统可以访问任何物理地址，而用户进程只能使用逻辑地址。
4）①分页机制中PDE和PTE中的R/W和U/S等，提供了页级保护；②分页机制将线性地址与物理地址加以映射，提供了对物理地址的保护。
补充：为什么特权级是基于段的?
在操作系统设计中，一般一个段实现的功能相对完整，可以把代码放在一个段，数据放在一个段，并通过段选择符（包括CS、SS、DS、ES、FS和GS）获取段的基址和特权级等信息。特权级基于段，这样当段选择子具有不匹配的特权级时，按照特权级规则判断是否可以访问。特权级基于段，是结合了程序的特点和硬件实现的一种考虑。


## 在setup程序里曾经设置过gdt，为什么在head程序中将其废弃，又重新设置了一个？为什么设置两次，而不是一次搞好？(P33)
原来GDT所在的位置是设计代码时在setup.s里面设置的数据，将来这个setup模块所在的内存位置会在设计缓冲区时被覆盖。如果不改变位置，将来GDT的内容肯定会被缓冲区覆盖掉，从而影响系统的运行。这样一来，将来整个内存空间中唯一安全的地方就是现在head.s所在的位置了。
不能在执行setup程序时直接把GDT的内容复制到head.s所在位置：如果先复制GDT内容，后移动system模块，它就会被后者覆盖；如果先移动system模块，后复制GDT内容，它又会把head.s对应的程序覆盖，而这时head.s还没有执行。所以无论如何都要重新建立GDT。

## 进程0的task_struct在哪？具体内容是什么？(P70)
进程0的task_struct、内核栈、用户栈都在内核数据段。
进程0的task_struct 的具体内容包括状态、信号、pid、alarm、ldt、tss等管理该进程所需的数据。如下代码所示：
``` c
// 进程0的task_struct
#define INIT_TASK \
/* state etc */	{ 0,15,15, \
/* signals */	0,{{},},0, \
/* ec,brk... */	0,0,0,0,0,0, \
/* pid etc.. */	0,-1,0,0,0, \
/* uid etc */	0,0,0,0,0,0, \
/* alarm */	0,0,0,0,0,0, \
/* math */	0, \
/* fs info */	-1,0022,NULL,NULL,NULL,0, \
/* filp */	{NULL,}, \
	{ \
		{0,0}, \
/* ldt */	{0x9f,0xc0fa00}, \
		{0x9f,0xc0f200}, \
	}, \
/*tss*/	{0,PAGE_SIZE+(long)&init_task,0x10,0,0,0,0,(long)&pg_dir,\
	 0,0,0,0,0,0,0,0, \
	 0,0,0x17,0x17,0x17,0x17,0x17,0x17, \
	 _LDT(0),0x80000000, \
		{} \
	}, \
}
```

以下代码说明user_stack在内核数据段。（0x10）（参考杨炯的内核代码完全注释）一个任务的数据结构与其内核态堆栈是放在同一内存页中。所以进程0的内核栈是跟着task struct后面的，所以进程0的内核栈不是user_stack，user_stack是进程0的用户栈。（P91的图）
``` c
long user_stack [ PAGE_SIZE>>2 ] ;//定义用户堆栈4K，指针指向最后一项
//该结构用于设置堆栈ss：esp（数据段选择符）
struct {
	long * a;
	short b;
	} stack_start = { & user_stack [PAGE_SIZE>>2] , 0x10 };//内核数据段

//每个进程在内核态运行时都有自己的内核态堆栈，这里定义了内核态堆栈的结构
union task_union {
	struct task_struct task;
	char stack[PAGE_SIZE];
};
static union task_union init_task = {INIT_TASK,};
```

## 内核的线性地址空间是如何分页的？（P37-38）画出从0x000000开始的7个页（包括页目录表、页表所在页）的挂接关系图，就是页目录表的前四个页目录项、第一个个页表的前7个页表项指向什么位置？（P39）给出代码证据。（P39）
1）head.s在setup_paging开始创建分页机制。将页目录表和4个页表放到物理内存的起始位置，从内存起始位置开始的5个页空间内容全部清零（每页4kb），然后设置页目录表的前4项，使之分别指向4个页表。然后开始从高地址向低地址方向填写4个页表，依次指向内存从高地址向低地址方向的各个页面。即将第4个页表的最后一项（pg3+4092指向的位置）指向寻址范围的最后一个页面。即从0xFFF000开始的4kb 大小的内存空间。将第4个页表的倒数第二个页表项（pg3-4+4092）指向倒数第二个页面，即0xFFF000-0x1000开始的4KB字节的内存空间，依此类推。
2）图见P39
{% asset_img p1.png p1 %}

3）Head.s中：（P39）\               
```
setup_paging: 
movl $1024*5,%ecx  /* 5 pages - pg_dir+4 page tables */ 
xorl %eax,%eax 
xorl %edi,%edi  /* pg_dir is at 0x000 */ 
cld;rep;stosl
 movl $pg0+7,pg_dir  /* set present bit/user r/w */  
movl $pg1+7,pg_dir+4  /*  --------- " " --------- */ 
movl $pg2+7,pg_dir+8  /*  --------- " " --------- */  
movl $pg3+7,pg_dir+12  /*  --------- " " --------- */ 
_pg_dir用于表示内核分页机制完成后的内核起始位置，也就是物理内存的起始位置0x000000，以上四句完成页目录表的前四项与页表1，2,3,4的挂接 
movl $pg3+4092,%edi 
movl $0xfff007,%eax  /*  16Mb - 4096 + 7 (r/w user,p) */ 
std 
1: 	stosl   /* fill pages backwards - more efficient :-) */ 
subl $0x1000,%eax
	jge 1b
	xorl %eax,%eax		/* pg_dir is at 0x0000 */
	movl %eax,%cr3		/* cr3 - page directory start */
	movl %cr0,%eax
	orl $0x80000000,%eax
	movl %eax,%cr0		/* set paging (PG) bit */
	ret			/* this also flushes prefetch-queue */
```
完成页表项与页面的挂接，是从高地址向低地址方向完成挂接的，16M内存全部完成挂接（注意页表从0开始，页表0-页表3）

## 在head程序执行结束的时候，在idt的前面有184个字节的head程序的剩余代码，剩余了什么？为什么要剩余？(P31、P36、P40)
1）剩余的内容：0x5400~0x54b7处，包含代码段：after_page_tables(栈中压入了些参数)、 ignore_int(初始化中断时的中断处理函数) 和 setup_paging(初始化分页)。 
2）原因：after_page_tables中压入了一些参数，为内核进入main函数的跳转做准备。为了谨慎起见，设计者在栈中压入了L6，以使得系统可能出错时，返回到L6处执行。ignore_int: 使用 ignore_int将 idt全部初始化，因此如果中断开启后，可能使用了未设置的中断向量，那么将默认跳转到 ignore_int处执行。这样做的好处是使得系统不会跳转到随机的地方执行错误的代码，所以ignore_int不能被覆盖。 setup_paging:为设置分页机制的代码，它在分页完成前不能被覆盖。

## 为什么不用call，而是用ret“调用”main函数？画出调用路线图，给出代码证据。(P42)
call指令会将EIP的值自动压栈，保护返回现场，然后执行被调函数的程序，等到执行被调函数的ret指令时，自动出栈给EIP并还原现场，继续执行call的下一条指令。然而对操作系统的main函数来说，如果用call调用main函数，那么ret时返回给谁呢？因为没有更底层的函数程序接收操作系统的返回。用ret实现的调用操作当然就不需要返回了，call做的压栈和跳转动作需要手工编写代码。
2）图在p42仿call示意图下面部分

3）代码：
``` 
after_page_tables:
	pushl $0		# These are the parameters to main :-)
	pushl $0
	pushl $0
	pushl $L6		# return address for main, if it decides to.
   	pushl $__main; //将main的地址压入栈，即EIP
jmp setup_paging
setup_paging:
   	ret; //弹出EIP，针对EIP指向的值继续执行，即main函数的入口地址。
```
## 用文字和图说明中断描述符表是如何初始化的，可以举例说明（比如：set_trap_gate(0,&divide_error)），并给出代码证据。（P52）
对中断描述符表的初始化，就是将中断、异常处理的服务程序与IDT进行挂接，逐步重建中断服务体系。 
（先画图见P54 图2-9然后解释）
以set_trap_gate(0,&divide_error)//除零错误 为例，进行宏展开后得到：

其中：n是0；gate_addr是&idt[0]，也就是IDT的第一项中断描述符的地址；type是15；dpl（描述符特权级）是0；addr是中断服务程序divide_error(void)的入口地址。




## 在IA-32中，有大约20多个指令是只能在0特权级下使用，其他的指令，比如cli，并没有这个约定。奇怪的是，在Linux0.11中，3特权级的进程代码并不能使用cli指令，这是为什么？请解释并给出代码证据。(P68、P79、P92)
根据IA-32手册，某些系统指令是禁止应用程序使用的。cli指令用于复位IF标志位，cli指令与CPL和EFLAGS[IOPL]有关。如果CPL的权限高于等于EFLAGS中的IOPL的权限，即数值上CPL<=IOPL，则可以执行该指令，IF位清除为0。如果CPL大于当前程序或过程的IOPL，则产生保护模式异常。
由于在内核中IOPL的值初始为0，且未经改变。INIT_TASK的TSS中设置了EFLAGS值，进程0又在move_to_user_mode中，继承了内核的EFLAGS。
``` c
\linux0.11\include\linux\sched.h
#define INIT_TASK \
//..
/*tss*/	{0,PAGE_SIZE+(long)&init_task,0x10,0,0,0,0,(long)&pg_dir,\
	 0,0,0,0,0,0,0,0, \     //eflags的值，决定了cli这类指令只能在0特权级使用
	 0,0,0x17,0x17,0x17,0x17,0x17,0x17, \
	 _LDT(0),0x80000000, \
		{} \
	}, \
}
\linux0.11\include\asm\system.h
#define move_to_user_mode() \
	......
	"pushfl\n\t" \   //eflags进栈
	......
	"iret\n" \
	//..
	:::"ax")
```
在进程0的TSS中，设置了eflags中的IOPL位为0，代码见P68，后续进程如果没有改动的话也是0，即IOPL=0。因此，通过设置IOPL，可以限制3特权级进程代码使用cli指令。
```c
\linux0.11\kernel\fork.c
int copy_process(int nr,long ebp,long edi,long esi,long gs,long none,
		long ebx,long ecx,long edx,
		long fs,long es,long ds,
		long eip,long cs,long eflags,long esp,long ss)
{
	//…
	p->tss.eip = eip;
	p->tss.eflags = eflags;
	p->tss.eax = 0;
	//…
	return last_pid;
}
```

## 在system.h里（见下），读懂代码。这里中断门、陷阱门、系统调用都是通过_set_gate设置的，用的是同一个嵌入汇编代码，比较明显的差别是dpl一个是3，另外两个是0，这是为什么？说明理由。(P55)
```
#define _set_gate(gate_addr,type,dpl,addr) \
__asm__ ("movw %%dx,%%ax\n\t" \
    "movw %0,%%dx\n\t" \
    "movl %%eax,%1\n\t" \
    "movl %%edx,%2" \
    : \
    : "i" ((short) (0x8000+(dpl<<13)+(type<<8))), \
    "o" (*((char *) (gate_addr))), \
    "o" (*(4+(char *) (gate_addr))), \
    "d" ((char *) (addr)),"a" (0x00080000))
#define set_intr_gate(n,addr) \
    _set_gate(&idt[n],14,0,addr)
#define set_trap_gate(n,addr) \
    _set_gate(&idt[n],15,0,addr)
#define set_system_gate(n,addr) \
    _set_gate(&idt[n],15,3,addr)
```

1）dpl机制：dpl表示的是特权级，0和3分别表示0特权级和3特权级。异常处理是由内核来完成，Linux处于对内核的保护，不允许用户进程直接访问内核。但是有些情况下，用户进程又需要内核代码的支持，因此就需要系统调用，它是用户进程与内核打交道的接口，是由用户进程直接调用的，因此其在3特权级下。
2）题目中：set_trap_gate 和set_intr_gate的dpl是0，set_system_gate的dpl是3。dpl为0表示只能在内核态下允许，dpl为3表示系统调用可以由3特权级调用。
当用户程序产生系统调用软中断后， 系统都通过system_call总入口找到具体的系统调用函数。set_system_gate设置系统调用，须将DPL设置为3，允许在用户特权级3的进程调用，否则会引发General Protection异常。set_trap_gate及set_intr_gate设置陷阱和中断为内核使用，需禁止用户进程调用，所以DPL为0。

## 进程0 fork进程1之前，为什么先调用move_to_user_mode()？用的是什么方法？解释其中的道理。（P78+80）
1）因为在Linux-011中，规定除了进程0以外的所有进程，都必须在特权级为3下创建。所以进程0 fork进程1之前，要调用move_to_user_mode将0特权级翻转到3特权级。
2）move_to_user_mode()用的方法是模仿中断硬件压栈，顺序是ss、esp、eflags、cs、eip。然后执行iret，出栈恢复现场，翻转0特权级到3特权级。
3）CPU响应中断的时候，根据DPL的设置，可以实现指定的特权级之间的翻转。所以模拟中断硬件压栈可以实现特权级的翻转。

## 在Linux操作系统中大量使用了中断、异常类的处理，究竟有什么好处？（P56）
在未引入中断、异常处理类处理的概念之前，CPU每隔一段时间就要对所有硬件进行轮询，以检测它的工作是否完成，如果没有完成就继续轮询，这样消耗了CPU处理用户程序的时间，降低了系统的综合效率。可见，CPU以“主动轮询”的方式来处理信号是非常不划算的。采用了这种中断方式，以“被动响应”模式代替“主动轮询”模式来处理主机与外设的I/O问题，只有在发生异常的时候CPU才停下正在进行的运算进行处理，其他时候都在做自己的事情。这大大提高了操作系统的综合效率。是计算机历史上的一大进步。

## copy_process函数的参数最后五项是：long eip,long cs,long eflags,long esp,long ss。查看栈结构确实有这五个参数，奇怪的是其他参数的压栈代码都能找得到，确找不到这五个参数的压栈代码，反汇编代码中也查不到，请解释原因。（P89）
copy_process执行时因为进程调用了fork函数，fork是一个系统调用，会导致中断，int 0x80中断导致CPU硬件自动将SS、ESP、EFLAGS、CS、EIP的值按照顺序压入进程0内核栈，又因为函数专递参数是使用栈的，所以刚好可以做为copy_process的最后五项参数。

## 分析get_free_page()函数的代码，叙述在主内存中获取一个空闲页的技术路线。（P90）
代码见P90  get_free_page函数

遍历mem_map[]，找到内存中（从高地址开始）第一个空闲（字节为0）页面，将其置为1。ecx左移12位加LOW_MEM得到该页的物理地址，并将页面清零。最后返回空闲页面物理内存的起始地址。
即：
(1)将EAX 设置为0，EDI设置指向mem_map的最后一项（mem_map+PAGING_PAGES-1），std设置扫描是从高地址向低地址。遍历mem_map[]，从mem_map的最后一项反向扫描，找出引用次数为0(AL)的页，如果没有则退出；如果找到，则将找到的页设引用数为1；
(2) ECX左移12位得到页的相对地址，加LOW_MEM得到物理地址，将此页最后一个字节的地址赋值给EDI（LOW_MEM+4092）；
(3) stosl将EAX的值设置到ES:EDI所指内存，即反向清零1024*32bit，将此页清空；
(4) 将页的地址（存放在EAX）返回。
注意!本函数只是指出在主内存区的一页空闲页面，但并没有映射到某个进程的线性地址去。后面的put_page()函数就是用来作映射的。

## 分析copy_page_tables（）函数的代码，叙述父进程如何为子进程复制页表。(P97)
先为新的页表申请一个空闲页面，并把进程0中第一个页表里面前160个页表项复制到这个页面中进程0和进程1页表暂时都指向了相同的页面，意味着进程1可以控制进程0的页面，之后对进程1的页目录表进行设置。最后重置CR3，刷新页变换高速缓存。设置完毕。

## 进程0创建进程1时，为进程1建立了task_struct及内核栈，第一个页表，分别位于物理内存16MB顶端倒数第一页、第二页。请问，这两个页究竟占用的是谁的线性地址空间，内核、进程0、进程1、还是没有占用任何线性地址空间？说明理由（可以图示）并给出代码证据。P99
这两个页占用的是内核的线性地址空间。
依据在setup_paging（文件head.s）中，
```
\linux0.11\boot\head.s
setup_paging:
	//…
	movl $pg3+4092,%edi
	movl $0xfff007,%eax		/*  16Mb - 4096 + 7 (r/w user,p) */
	std
1:	stosl			/* fill pages backwards - more efficient :-) */
	subl $0x1000,%eax
	//…
上面的代码，指明了内核的线性地址空间为0x000000 ~ 0xffffff（即前16M），且线性地址与物理地址呈现一一对应的关系。
进程0的局部描述符如下include/linux/sched.h/INIT_TASK：
/*ldt*/ 
{0x9f,0xc0fa00}.\
{0x9f,0xc0f200},\
```
为进程1分配的这两个页，在16MB的顶端倒数第一页、第二页，因此占用内核的线性地址空间。进程0的线性地址空间是内存前640KB，因为进程0的LDT中的limit属性限制了进程0能够访问的地址空间。进程1拷贝了进程0的页表（160项），而这160个页表项即为内核第一个页表的前160项，指向的是物理内存前640KB，因此无法访问到16MB的顶端倒数的两个页。

## 假设：经过一段时间的运行，操作系统中已经有5个进程在运行，且内核分别为进程4、进程5分别创建了第一个页表，这两个页表在谁的线性地址空间？用图表示这两个页表在线性地址空间和物理地址空间的映射关系。(P266、P270)
这两个页面均占用内核的线性地址空间。既然是内核线性地址空间，则与物理地址空间为一一对应关系。根据每个进程占用16个页目录表项，则进程4占用从第65～81项（即64-80项）的页目录表项。同理，进程5占用第81～96项（即80-95项）的页目录表项。由于目前只分配了一个页面（用做进程的第一个页表），则分别只需要使用第一个页目录表项即可。

## 下面代码中的"ljmp %0\n\t" 很奇怪，按理说jmp指令跳转到得位置应该是一条指令的地址，可是这行代码却跳到了"m" (*&__tmp.a)，这明明是一个数据的地址，更奇怪的，这行代码竟然能正确执行。请论述其中的道理。P107 
```
#define switch_to(n) {\
struct {long a,b;} __tmp; \
__asm__("cmpl %%ecx,_current\n\t" \
    "je 1f\n\t" \
    "movw %%dx,%1\n\t" \
    "xchgl %%ecx,_current\n\t" \
    "ljmp %0\n\t" \
    "cmpl %%ecx,_last_task_used_math\n\t" \
    "jne 1f\n\t" \
    "clts\n" \
    "1:" \
    ::"m" (*&__tmp.a),"m" (*&__tmp.b), \
    "d" (_TSS(n)),"c" ((long) task[n])); \
}
```

a对应EIP、b对应CS，ljmp通过CPU中的电路进行硬件切换，进程由当前进程切换到进程n。CPU将当前寄存器的值保存到当前进程的TSS中，将进程n的TSS数据和LDT的代码段和数据段描述符恢复给CPU的各个寄存器，实现任务切换。

进程0->1：ljmp %0\n\t通过任务门机制并未实际使用任务门，将CPU的各个寄存器值保存在进程0的TSS中，将进程1的TSS数据以LDT的代码段、数据段描述符数据恢复给CPU的各个寄存器，实现从0特权级的内核代码切换到3特权级的进程1代码执行。其中tss.eip也自然恢复给了CPU，此时EIP指向的就是fork中的if(__res >= 0)语句。


## 进程0开始创建进程1，调用fork（），跟踪代码时我们发现，fork代码执行了两次，第一次，执行fork代码后，跳过init（）直接执行了for(;;) pause()，第二次执行fork代码后，执行了init（）。奇怪的是，我们在代码中并没有看到向转向fork的goto语句，也没有看到循环语句，是什么原因导致fork反复执行？请说明理由（可以图示），并给出代码证据。P107
首先在copy_process()函数中，设置TSS“p->tss.eip = eip;”指向的是if (__res >= 0); 而“p->tss.eax = 0;”决定main()中if (!fork())后面的分支走向。
```c
\linux0.11\kernel\fork.c
int copy_process(int nr,long ebp,long edi,long esi,long gs,long none,
		long ebx,long ecx,long edx,
		long fs,long es,long ds,
		long eip,long cs,long eflags,long esp,long ss)
{
	//…
	*p = *current;	/* NOTE! this doesn't copy the supervisor stack */
	//…
	p->start_time = jiffies;
	p->tss.back_link = 0;
	p->tss.esp0 = PAGE_SIZE + (long) p;
	p->tss.ss0 = 0x10;
	p->tss.eip = eip;
	p->tss.eflags = eflags;
	p->tss.eax = 0;
	//…
	return last_pid;
}
```
接着，copy_process()函数返回后，通过“pushl %eax”将函数返回值，也就是进程1的进程号压栈。
```c
\linux0.11\kernel\system_call.s
_system_call:
	//…
	call _sys_call_table(,%eax,4)
	pushl %eax
```
“: “=a” (__res) \”将eax的值赋值给__res，所以“if (__res >= 0) \”实际上是看此时的eax时多少，由上可知，eax＝1。
```c
\linux0.11\init\main.c
#define _syscall0(type,name) \
type name(void) \
{ \
long __res; \
__asm__ volatile ("int $0x80" \
	: "=a" (__res) \
	: "0" (__NR_##name)); \
if (__res >= 0) \
	return (type) __res; \
errno = -__res; \
return -1; \
}
```
回到if (!fork())处执行，!1为“假”，不会执行init()，直接执行“for(;; ) pause();”。
```c
\linux0.11\kernel\sched.c
void main(void)		/* This really IS void, no error here. */
{			/* The startup routine assumes (well, ...) this */
	//…
	move_to_user_mode();
	if (!fork()) {		/* we count on this going ok */
		init();
	}
	for(;;) pause();
}
```
由pause()函数进入“schedule();”开始调度，然后通过“switch_to(next);”准备切换进程。
```c
\linux0.11\kernel\sched.c
int sys_pause(void)
{
	current->state = TASK_INTERRUPTIBLE;
	schedule();
	return 0;
}
```
```c
\linux0.11\kernel\sched.c
void schedule(void)
{
	//…
	switch_to(next);
}
```
执行switch_to()函数中，当程序执行到“ljmp %0\n\t”这行时，ljmp通过CPU任务门机制自动将进程1的TSS值恢复给CPU，自然也将其中的tss.eip恢复给CPU，这时EIP指向fork的if(__res >= 0)这行。而此时的__res值就是进程1中TSS的eax的值，这个值在前面被写死为0，即“p->tss.eax = 0;”所以执行到“return (type)__res;”这行时，返回值为0。返回后，执行到if(!fork())这一行，!0为“真”，调用init()函数！
```c
\linux0.11\fs\buffer.c
#define switch_to(n) {\
	//…
	"ljmp %0\n\t" \
	//…
}
```
总结：主要是利用两个系统调用sys_fork和sys_pause对进程状态的设置，以及利用了进程调度机制。

## 打开保护模式、分页后，线性地址到物理地址是如何转换的？P97
打开保护模式、分页后，线性地址需要通过MMU进行解析，以页目录表、页表、页面三级映射模式映射到物理地址。具体转换过程是这样的：“每个线性地址值是32位，MMU按照10-10-12的长度来识别地址值，分别解析为页目录项号、页表项号、页面内偏移。CR3中存放着页目录表的基址，通过CR3找到页目录表，再找到页目录项，进而找到对应页表，寻取页表项，然后找到页面物理地址，最后加上12位页内偏移形成的地址，才为最终物理地址”。
总结：CR3->页目录表->页目录项->页表->页表项->页面->物理地址


## getblk函数中，申请空闲缓冲块的标准就是b_count为0，而申请到之后，为什么在wait_on_buffer(bh)后又执行if（bh->b_count）来判断b_count是否为0？P115
字段b_count：用来标记“每个缓冲块有多少个进程在共享”。只有当b_count=0时，该缓冲块才能被再次分配。
举个引发异常例子：每个缓冲块有一个进程等待队列，假设此时B、C两进程在队列中，当该缓冲块被解锁时，进程C被唤醒（它开始使用缓冲区之前需先唤醒进程B，使进程B从挂起进入就绪状态），将缓冲区加锁，一段时间后，进程C又被挂起，但此时缓冲区进程C仍在使用。这时候，进程B被调度，“if (bh->b_count)”该缓冲区任是加锁状态，进程B重新选择缓冲区…如果不执行该判断，将造成进程B操作一个被加锁的缓冲区，引发异常。
总结：等待缓冲区解锁这段时间，缓冲块可能会被别的进程占用，因此需要再次判断一下b_count是否为0，如果不为0，则还得继续等。

## b_dirt已经被置为1的缓冲块，同步前能够被进程继续读、写？给出代码证据。(P331)
同步前可以被进程读写，但不能挪为它用（即关联其它物理块）。b_dirt是针对硬盘方向的，进程与缓冲块方向由b_uptodate标识。只要b_uptodate为1，缓冲块就能被进程读写。读操作不会改变缓冲块中数据的内容，写操作后，改变了缓冲区内容，需要将b_dirt置1。由于此前缓冲块中的数据已经用硬盘数据块更新了，所以后续同步过程中缓冲块没有写入新数据的部分和原来硬盘对应的部分相同，所有的数据都是进程希望同步到硬盘数据块上的，不会把垃圾数据同步到硬盘数据库上去，所以b_uptodate仍为1。
所以，b_dirt为1，进程仍能对缓冲区进行读写。

1）读写文件均与b_dirt无关：
```c
\linux0.11\fs\file_dev.c
int file_write(struct m_inode * inode, struct file * filp, char * buf, int count)
{
	//…
	if (filp->f_flags & O_APPEND)
		pos = inode->i_size;
	else
		pos = filp->f_pos;
	while (i<count) {
		if (!(block = create_block(inode,pos/BLOCK_SIZE)))
			break;
		if (!(bh=bread(inode->i_dev,block)))
			break;
	//…
}
int file_read(struct m_inode * inode, struct file * filp, char * buf, int count)
{
	//…
	if ((left=count)<=0)
		return 0;
	while (left) {
		if (nr = bmap(inode,(filp->f_pos)/BLOCK_SIZE)) {
			if (!(bh=bread(inode->i_dev,nr)))
				break;
		} 
	//…
}
```
2）在获取缓冲块时，亦与b_dirt无任何关系：
```c
\linux0.11\fs\buffer.c
struct buffer_head * bread(int dev,int block)
{
	struct buffer_head * bh;
	if (!(bh=getblk(dev,block)))
		panic("bread: getblk returned NULL\n");
	if (bh->b_uptodate)
		return bh;
	ll_rw_block(READ,bh);
	wait_on_buffer(bh);
	if (bh->b_uptodate)
		return bh;
	brelse(bh);
	return NULL;
}
```
```c
\linux0.11\fs\buffer.c
#define BADNESS(bh) (((bh)->b_dirt<<1)+(bh)->b_lock)
struct buffer_head * getblk(int dev,int block)
{
	struct buffer_head * tmp, * bh;

repeat:
	if (bh = get_hash_table(dev,block))
		return bh;
	//…
}
```



## 分析panic函数的源代码，根据你学过的操作系统知识，完整、准确的判断panic函数所起的作用。假如操作系统设计为支持内核进程（始终运行在0特权级的进程），你将如何改进panic函数？
1）panic函数是当系统发现无法继续运行下去的故障时调用它，会导致程序终止，由系统显示错误号。如果出现错误的函数不是进程0，则进行数据同步，把缓冲区的数据尽量同步到硬盘上去。
```c
\linux0.11\kernel\panic.c
volatile void panic(const char * s)
{
	printk("Kernel panic: %s\n\r",s);
	if (current == task[0])
		printk("In swapper task - not syncing\n\r");
	else
		sys_sync();
	for(;;);
}
```
panic()函数的执行分为两种情况，如果是0进程调用了panic（）说明在创建进程1过程中出现了严重错误不可继续执行（copy_page_table()中from没有按照4M对齐）且此时不涉及与硬盘中数据的交互，所以这时直接让系统执行死循环for（；；），且这个时候系统中也不存在可调度的进程，整个电脑死机。如果是其他进程调用了panic（）则说明是某个用户进程出现严重错误，这个过程中可能涉及缓冲区数据没有写回硬盘的情况，所以在死循环之前，需要将缓冲区中为写回的数据写回硬盘，保证程序执行后硬盘中数据的一致性，所以需要调用sys_sync()写回数据，而且这个时候系统也能够完成次工作。
2）改进：将死循环改成跳到的内核进程（始终运行在0特权级的进程），让内核继续执行。

## 详细分析进程调度的全过程。考虑所有可能（signal、alarm除外）P105
首先依据task[64]这个结构，从后往前遍历，寻找进程状态为“就绪态”且时间片最大的进程作为下一个要执行的进程。通过调用switch_to函数跳转到指定进程。在此过程中，如果发现存在状态为就绪态的进程，但没有时间片，则从后往前重新分配时间片。然后重新执行上述过程。寻找状态为就绪态，时间片最大的进程作为下一个要执行的进程。如果没有就绪态，就跳转到进程0。

答2（具体）：
schedule()函数的主要过程为，首先依据task[64]这个结构，第一次遍历所有进程，只要地址指针不为空，就要针对它的signal、alarm分析，这里先不考虑。第二次遍历所有进程，比较进程的状态和时间片，找出处在就绪态且counter最大的进程。
```c
\linux0.11\kernel\sched.cvoid schedule(void){
	int i,next,c;
	struct task_struct ** p;/* check alarm, wake up any interruptible tasks that have got a signal */
	for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
		if (*p) {
			if ((*p)->alarm && (*p)->alarm < jiffies) {
					(*p)->signal |= (1<<(SIGALRM-1));
					(*p)->alarm = 0;
				}
			if (((*p)->signal & ~(_BLOCKABLE & (*p)->blocked)) &&
			(*p)->state==TASK_INTERRUPTIBLE)
				(*p)->state=TASK_RUNNING;
		}/* this is the scheduler proper: */
	while (1) {
		c = -1;
		next = 0;
		i = NR_TASKS;
		p = &task[NR_TASKS];
		while (--i) {
			if (!*--p)
				continue;
			if ((*p)->state == TASK_RUNNING && (*p)->counter > c)
				c = (*p)->counter, next = i;
		}
		if (c) break;
		for(p = &LAST_TASK ; p > &FIRST_TASK ; --p)
			if (*p)
				(*p)->counter = ((*p)->counter >> 1) +
						(*p)->priority;
	}
	switch_to(next);}
```
执行switch_to()函数中，ljmp %0\n\t通过任务门机制并未实际使用任务门，将CPU的各个寄存器值保存在进程0的TSS中，将进程1的TSS数据以LDT的代码段、数据段描述符数据恢复给CPU的各个寄存器，实现从0特权级的内核代码切换到3特权级的进程1代码执行。其中tss.eip也自然恢复给了CPU，此时EIP指向的就是fork中的if(__res >= 0)语句。
```c
\linux0.11\fs\buffer.c#define switch_to(n) {\
	//…
	"ljmp %0\n\t" \
	//…}
```


## wait_on_buffer函数中为什么不用if（）而是用while（）？P125
有可能很多进程都在等待一个缓冲块。在缓冲块同步完毕后，唤醒等待进程到轮转到某一进程的过程中，很有可能之前等的缓冲块被别的进程占用并加锁。如果使用if，则该进程被唤醒以后回来不会再判断缓冲块是否被占用，而直接使用就会导致出错。使用while，就会再判断一下缓冲块是否被占用，确认未被占用后使用，就不会发生之前的错误。
```c
\linux0.11\fs\buffer.cstatic inline void wait_on_buffer(struct buffer_head * bh){
	cli();
	while (bh->b_lock)
		sleep_on(&bh->b_wait);
	sti();
}
```

## 操作系统如何利用b_uptodate保证缓冲块数据的正确性？new_block (int dev)函数新申请一个缓冲块后，并没有读盘，b_uptodate却被置1，是否会引起数据混乱？详细分析理由。P325+329
答1：只要缓冲块的b_uptodate字段被设置为1，缓冲块的数据已经是数据块最新的，就可以放心的支持进程共享缓冲块的数据。反之，如果b_uptodate为0，就提醒内核缓冲块并没有用绑定的数据块中的数据更新，不支持进程共享该缓冲块。值得注意的是b_uptodate被设置为1，是告诉内核，缓冲块中的数据已经用数据块中的数据更新过了，但并不等于两者的数据就完全一致。
如题中的，申请一个缓冲块后，并没有读盘，b_uptodate却被置1，这并不会引起数据混乱。这时因为只要为新建的数据块新申请了缓冲块，不管这个缓冲块将来用做什么，反正进程现在不需要里面的数据，干脆全部清零。这样不管与之绑定的数据块用来存储什么信息，都无所谓，将该缓冲块的b_uptodate置为1，更新问题“等效于”以解决。

答2：
b_uptodate：是缓冲块中针对进程方向的标志位，它的作用是告诉内核，缓冲块的数据是否已是数据块中最新的。当b_update置1时，就说明缓冲块中的数据是基于硬盘数据块的，内核可以放心地支持进程与缓冲块进行数据交互；如果b_uptodate为0，就提醒内核缓冲块并没有用绑定的数据块中的数据更新，不支持进程共享该缓冲块。
①当为文件创建新数据块，新建一个缓冲块时，b_uptodate被置1，但并不会引起数据混乱。此时，新建的数据块只可能有两个用途，一个是存储文件内容，一个是存储文件的i_zone的间接块管理信息。
②如果是存储文件内容，由于新建数据块和新建硬盘数据块，此时都是垃圾数据，都不是硬盘所需要的，无所谓数据是否更新，结果“等效于”更新问题已经解决。
③如果是存储文件的间接块管理信息，必须清零，表示没有索引间接数据块，否则垃圾数据会导致索引错误，破坏文件操作的正确性。虽然缓冲块与硬盘数据块的数据不一致，但同样将b_uptodate置1不会有问题。

## 分析add_reques（）函数中下列代码
```c
    if (!(tmp = dev->current_request)) {
        dev->current_request = req;
        sti();
        (dev->request_fn)();
        return;
    }
```
**其中的**
```c
    if (!(tmp = dev->current_request)) {
        dev->current_request = req;
```
**是什么意思？P121**
查看指定设备是否有当前请求项，即查看设备是否忙。
if判断首先将当前设备的请求队列的队首赋值给tmp，队列为空的情况下if条件成立，则说明当前没有需要处理的请求项（现在是第一次使用缓冲区，第一次申请空闲请求项与其挂接，当前请求项一定为空）。将当前请求项req作为请求队列的队首（dev->current_request = req;）。req是此前在make_request中申请的一个空闲请求项，与对应缓冲块挂接。（如果该设备的请求队列不为空，后面代码会将req直接挂在请求队列的队尾。）

## do_hd_request()函数中dev的含义始终一样吗？(P318)
do_hd_request()函数主要用于处理当前硬盘请求项。但其中的dev含义并不一致。
①含义1-逻辑设备号：“dev = MINOR(CURRENT->dev);”MINOR函数会取CURRENT->dev的低8位赋值给dev，低8位表示的含义是逻辑设备号。此时dev的值表示的是当前的逻辑设备号，目的是判断设备号是否合法。dev>=5*NR_HD 是判断设备号是否超出了0-4或5-9的范围。block+2 >hd[dev].nr_sects判断是否还有空闲盘块。
②含义2-硬盘号：“dev /= 5;”dev代表硬盘号（硬盘0还是硬盘1）：改变了dev的值，若原来在0-4的范围，此时则变成了0，若原来在5-9的范围此时则变成了1。目的是为了确定是第0块还是第一块物理盘。

## read_intr（）函数中，下列代码是什么意思？为什么这样做？(P323)
```c
    if (--CURRENT->nr_sectors) {
        do_hd = &read_intr;
        return;
    }
```
由于Linux0.1.1是PIO模式，PIO模式的做法是一次读一个扇区，读完以后发送中断告诉CPU当前已经读完了，此时CPU进行处理，将数据读到缓冲区。以引导块为例，需要进行两次PIO，相当于执行了两次中断。
该代码是为了判断请求项的缓冲块数据是否已经读完，“—CURRENT->nr_sectors”将递减请求项所需读取的扇区数值，如果没有读完则if条件成立。内核将再次把read_intr()绑定在硬盘中断服务程序上，以待下次使用，之后中断服务程序返回。
当所需要硬盘上的数据已经读完，硬盘产生中断，读盘中断服务程序再次响应这个中断，进入read_intr()函数后，仍然会判断请求项对应的缓存块数据是否已经读完，如果读完则不进入这个if 里。

## bread（）函数代码中为什么要做第二次if (bh->b_uptodate)判断？
    if (bh->b_uptodate)
        return bh;
    ll_rw_block(READ,bh);
    wait_on_buffer(bh);
    if (bh->b_uptodate)
        return bh;
答1：bread()函数主要是从块设备上读取数据。调用底层ll_rw_block()函数，产生读设备请求。然后等待指定数据块读入，并等待缓冲块解锁。在睡眠醒来之后，如果缓冲块已更新“if (bh->b_uptodate)”，则返回缓冲块指针。否则，表明读设备操作失败，于是释放该缓冲块返回NULL。

答2：第一次从缓冲区取出设备号块号一致的缓冲块，判断缓冲块是否有效，有效则使用；无效则发出读设备数据库请求。
第二次等待指定数据块读入，等缓冲块解锁以后，唤醒进程后，要重新判断缓冲块是否有效，如果缓冲区中数据有效，则返回缓冲区头指针退出，否则释放该缓冲区返回Null。
在等待过程中，数据可能发生了变化，所以要二次判断。


## getblk（）函数中，两次调用wait_on_buffer（）函数，两次的意思一样吗？(P372)
这里都是等待缓冲块解锁，但是此时缓冲区情况不一样：
第一次：已经找到一个比较合适的空闲缓冲块，但可能是加锁的，等待该缓冲块解锁；
第二次：如果该缓冲区已被修改，则将数据写盘，并再次等待缓冲块解锁。

执行到第一次wait_on_buffer时，意味着前面已经找到了空闲的缓冲块，但是这个缓冲块可能是是已经被加锁（其他进程在操作该空闲缓冲块），也可能是脏的，同样也可能是没有被别人占用的缓冲块。所以此时调用wait_on_buffer是检查是否该缓冲块被加锁，若加锁则阻塞当前进程。
执行到第二次wait_on_buffer时，会检查缓冲块的内容是否为脏（是否被修改过），若为脏，需要同步逻辑设备（写回设备）。写回设备是一个漫长的过程，不能保证在这个过程中没有其他进程对缓冲块进行加锁。当写回设备后发现该缓冲块被其他进程加锁，同样需要阻塞当前进程。

## getblk（）函数中，说明什么情况下执行continue、break。P372
```c
    do {
        if (tmp->b_count)
            continue;
        if (!bh || BADNESS(tmp)<BADNESS(bh)) {
            bh = tmp;
            if (!BADNESS(tmp))
                break;
        }
/* and repeat until we find something good */
    } while ((tmp = tmp->b_next_free) != free_list);
```
getblk()函数主要是获取高速缓冲中的指定缓冲块。下面的宏用于判断缓冲块的修改标志，并定义修改标志的权重比锁定标志大。
```c
//BADNESS定义
#define BADNESS(bh) (((bh)->b_dirt<<1)+(bh)->b_lock)
```
continue：tmp指向的是空闲链表的第一个空闲缓冲块头“tmp = free_list;”。如果该缓冲块正在被使用，引用计数“tmp->b_count”不等于0，则continue，继续遍历直到找到一个count为0的空闲块。
break：接下来，如果缓冲头指针bh为空，或者tmp所指的缓冲头标志（修改、锁定）权重小于bh头标志的权重，则让bh指向tmp缓冲块头。如果BADNESS为0，该tmp缓冲块头表明缓冲块既没有修改也没有锁定标志位，即这个块既没加锁，也没被写过，则说明已为指定设备上的块取得对应的高速缓冲块。然后break跳出循环。（也只有在系统运行初期才会存在这样的块）

## make_request（）函数中，其中的sleep_on(&wait_for_request)是谁在等？等什么？    
```c
if (req < request) {
    if (rw_ahead) {
        unlock_buffer(bh);
        return;
    }
    sleep_on(&wait_for_request);
    goto repeat;
```
此时wait_for_request指的是：当前进程等待request请求队列腾出空闲项。

make_request()函数主要功能为创建请求项并插入请求队列。根据具体读写操作，如果request[32]中没有一项是空闲的，则查看此次请求是不是提前读写，如果是则立即放弃此次请求操作。否则让本次请求先睡眠“sleep_on(&wait_for_request);”以等待request请求队列腾出空闲项，一段时间后再次搜索请求队列。
