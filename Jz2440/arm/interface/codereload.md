# 代码重定向
[参考链接](https://blog.csdn.net/ty3219/article/details/53728169)

* [00 链接地址、加载地址、运行地址](#jump0)
* [01 跳转指令](#jump1)
* [02 位置无关码](#jump2)
* [03 代码案例](#jump3)

###  <span id="jump0">00 链接地址、加载地址、运行地址
* 1. 链接地址：通过反汇编可以查看的地址编号就是链接地址，链接地址在链接文件中定义；
* 2. 加载地址：程序加载到内存的物理地址
* 3. 运行地址：指令执行时所处的物理地址

*  链接脚本的一般形式
```
section_name  vma : AT(lma)
  {
    output-section-command
    output-section-command
    ...
  } [AT>lma_region]

  lma =  load memory address
  vma =  vitual memory address

  1. vma:就是链接地址
  2. lma：是内存加载地址，可以理解为：对于被加载进物理内存的代码首地址来说，此处加载地址相对于代码首地址的偏移量。
```


* 举例说明

```
  列如：
SECTIONS {

   .text   0  : { *(.text) }

   .rodata  : { *(.rodata) }

   .data 0x31000000 : AT(0x800) { *(.data) }

   .bss  : { *(.bss) *(.COMMON) }

}

1. 对于这个链接文件来说，当程序被2440从nand拷贝到sram中时，data段在sram的实际起始地址是0x800。
2. 程序运行时的地址查找是根据链接地址来查找的，而不是根据lma地址查找的，所以当lma与vma不一样时，我们需要代码重定向，将代码从lma地址拷贝到vma地址。
3. 如果没有写AT，则认为lma与vma地址一样。
4. 只有当程序的运行地址与vma一样时，程序才可以正常运行。
```  


###  <span id="jump1">01 跳转指令
```
能够改变PC值的指令叫做跳转指令。
跳转指令一般分为 b系列 指令和 ldr 指令。


1. 相对跳转指令（跳转地址没有写死在汇编命令中）
  b系列指令一定是相对跳转指令.

  ldr/mov 指令 可以当做相对跳转来使用（使用变量给PC赋值，变量的值在运行时求出），也可以做绝对跳转（直接赋值PC一个常量值）
    1. 相对赋值：
      ldr  r0, [pc,  #20] 这里表示是把将要跳转的地址放到r0中，没有把绝对地址固定在命令里。r0=pc+8+20=0x320068c8+8+20。这个8是由ARM的流水线机制决定的，PC里的值是当前地址+8.
      mov pc, r0 把r0的值赋给pc，实现跳转

    2. 绝对跳转（跳转地址写死在汇编命令中）
      mov pc,#0x00  跳转地址被写死在汇编指令中
```

###  <span id="jump2">02 位置无关码
```
  链接器要加上-pie的选项，这样由c代码转化成汇编码时，汇编码中跳转指令就会采用相对跳转指令，也就是位置无关的汇编代码。

  比如：
    1. 不加-pic，c转汇编可能是这样的：ldr pc ,funca
    2. 加-pic，c转汇编可能是这样的：ldr  r0, [pc,  #20];mov pc, r0;
```

###  <span id="jump3">03 代码案例

1. 链接脚本：

```
SECTIONS
{
	. = 0x30000000;
	__code_start = .;
	. = ALIGN(4);
	.text      :
	{
	  *(.text)
	}
	. = ALIGN(4);
	.rodata : { *(.rodata) }
	. = ALIGN(4);
	.data : { *(.data) }
	. = ALIGN(4);
	__bss_start = .;
	.bss : { *(.bss) *(.COMMON) }
	_end = .;
}
```


2. 汇编文件代码跳转

source/012_relocate_013/010_013_006/start.S
```
bl copy2sdram ;使用c代码进行代码重定向
```
source/012_relocate_013/010_013_006/init.c
```
void copy2sdram(void)
{
	extern int __code_start, __bss_start; //在链接脚本文件中定义
	volatile unsigned int *dest = (volatile unsigned int *)&__code_start;
	volatile unsigned int *end = (volatile unsigned int *)&__bss_start;
	volatile unsigned int *src = (volatile unsigned int *)0;
	while (dest < end)
	{
		*dest++ = *src++;
	}
}
```
